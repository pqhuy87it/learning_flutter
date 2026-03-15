# Dart Isolate — Giải thích chi tiết

## 1. Vấn đề cốt lõi: Dart là single-threaded

Dart chạy code trên một **single thread** gọi là **main isolate**. Mọi thứ — UI rendering, event handling, business logic, I/O callback — đều chạy trên thread này thông qua **Event Loop**:

```
┌──────────────────────────────────────────────────┐
│              MAIN ISOLATE                        │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │            EVENT LOOP                      │  │
│  │                                            │  │
│  │  ┌──────────────────┐                      │  │
│  │  │  Microtask Queue  │ ← Future.then,      │  │
│  │  │  (ưu tiên cao)    │   scheduleMicrotask │  │
│  │  └────────┬─────────┘                      │  │
│  │           │ Xử lý HẾT microtask            │  │
│  │           ▼                                │  │
│  │  ┌──────────────────┐                      │  │
│  │  │   Event Queue     │ ← Timer, I/O,       │  │
│  │  │  (ưu tiên thấp)   │   gesture, repaint  │  │
│  │  └────────┬─────────┘                      │  │
│  │           │ Xử lý 1 event                  │  │
│  │           ▼                                │  │
│  │     Quay lại đầu loop                      │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ⚠️ NẾU bất kỳ task nào chiếm > 16ms             │
│     → Frame bị miss → UI JANK                    │
└──────────────────────────────────────────────────┘
```

Event Loop xử lý **tuần tự**: lấy một event, chạy xong, lấy event tiếp theo. Nếu một task tốn 100ms (parse JSON lớn, xử lý ảnh, mã hóa...), Event Loop bị **block** 100ms — không thể xử lý gesture, không thể render frame → user thấy app **đơ**.

Đây là lý do Isolate tồn tại.

---

## 2. Isolate là gì

Isolate là **đơn vị thực thi độc lập** trong Dart — tương đương concept **process** hơn là **thread** ở các ngôn ngữ khác. Mỗi Isolate có:

```
┌─────────────────────────────────┐  ┌─────────────────────────────────┐
│         MAIN ISOLATE            │  │        WORKER ISOLATE           │
│                                 │  │                                 │
│  ┌───────────┐                  │  │  ┌───────────┐                  │
│  │  Heap     │  ← riêng         │  │  │  Heap     │  ← riêng         │
│  │  (memory) │                  │  │  │  (memory) │                  │
│  └───────────┘                  │  │  └───────────┘                  │
│                                 │  │                                 │
│  ┌───────────┐                  │  │  ┌───────────┐                  │
│  │  Stack    │  ← riêng         │  │  │  Stack    │  ← riêng         │
│  └───────────┘                  │  │  └───────────┘                  │
│                                 │  │                                 │
│  ┌───────────┐                  │  │  ┌───────────┐                  │
│  │Event Loop │  ← riêng         │  │  │Event Loop │  ← riêng         │
│  └───────────┘                  │  │  └───────────┘                  │
│                                 │  │                                 │
│  ┌───────────┐                  │  │  ┌───────────┐                  │
│  │ GC        │  ← riêng         │  │  │ GC        │  ← riêng         │
│  └───────────┘                  │  │  └───────────┘                  │
│                                 │  │                                 │
│  KHÔNG THỂ truy cập ──────────────────── KHÔNG THỂ truy cập          │
│  memory của isolate kia         │  │  memory của isolate kia         │
│                                 │  │                                 │
│  Giao tiếp qua MESSAGE PASSING  │  │  Giao tiếp qua MESSAGE PASSING  │
│  (SendPort / ReceivePort)       │  │  (SendPort / ReceivePort)       │
└─────────────────────────────────┘  └─────────────────────────────────┘
```

Đặc điểm quan trọng nhất: **No shared memory**. Hai isolate **không bao giờ** truy cập được heap của nhau. Đây là khác biệt cơ bản so với thread truyền thống (Java thread, C++ thread, pthread...) nơi các thread chia sẻ bộ nhớ chung.

---

## 3. Isolate vs Thread — Tại sao Dart chọn thiết kế này

### Thread truyền thống (shared memory)

```
Thread A ──┐                    ┌── Thread B
           │   Shared Memory    │
           ├────────────────────┤
           │   counter = 5      │
           │                    │
           │ Thread A: counter++│
           │ Thread B: counter++│  ← RACE CONDITION
           │                    │
           │ Kết quả: 6 hay 7?  │  ← KHÔNG XÁC ĐỊNH
           └────────────────────┘
           
Giải pháp: Mutex, Lock, Semaphore, Atomic operations
→ Complex, error-prone, deadlock risk
```

Shared memory tạo ra toàn bộ class vấn đề: race condition, deadlock, livelock, priority inversion, data corruption. Developer phải dùng mutex/lock/semaphore, code trở nên phức tạp và khó debug.

### Dart Isolate (message passing)

```
Isolate A ──┐              ┌── Isolate B
            │              │
            │  Heap A      │  Heap B
            │  counter = 5 │  counter = 5
            │              │
            │  A: counter++ │  B: counter++
            │  counter = 6 │  counter = 6
            │              │
            │  Không conflict│  Mỗi bên có bản riêng
            └──────────────┘

Giao tiếp: gửi MESSAGE (copy data hoặc transfer ownership)
→ No race condition by design
→ No locks needed
→ No deadlocks possible
```

Dart chọn model **Actor-based concurrency** (tương tự Erlang, Elixir). Mỗi isolate là một actor — nhận message, xử lý, gửi message. Không có shared state nên **không thể có race condition**. An toàn by design thay vì by discipline.

**Trade-off**: giao tiếp giữa isolate tốn hơn (phải serialize/copy data) so với shared memory (chỉ cần đọc pointer). Nhưng Dart optimize case này bằng cơ chế **transfer** (sẽ nói ở phần sau).

---

## 4. Cơ chế giao tiếp: Port System

Isolate giao tiếp qua **ReceivePort** và **SendPort**:

```
┌──────────────────┐          ┌───────────────────┐
│   MAIN ISOLATE   │          │  WORKER ISOLATE   │
│                  │          │                   │
│  ReceivePort ◄────── message ──── SendPort      │
│  (lắng nghe)     │          │  (gửi đi)         │
│                  │          │                   │
│  SendPort ────── message ────► ReceivePort      │
│  (gửi đi)        │          │  (lắng nghe)      │
└──────────────────┘          └───────────────────┘
```

`ReceivePort` là **điểm nhận** — tạo trên isolate nào thì listen trên isolate đó. Mỗi `ReceivePort` khi tạo ra sẽ kèm theo một `SendPort`. `SendPort` là **điểm gửi** — có thể gửi sang isolate khác. SendPort là lightweight, có thể copy/transfer giữa các isolate.

**Analogy**: `ReceivePort` giống hòm thư của bạn — bạn mở nó ra để đọc thư. `SendPort` giống địa chỉ gửi thư — bạn đưa địa chỉ cho người khác để họ gửi thư cho bạn.

### Data được gửi qua port

Khi gửi message qua `SendPort.send()`, data được **deep copy** sang isolate nhận. Isolate nhận có bản copy riêng hoàn toàn, thay đổi trên bản copy không ảnh hưởng bản gốc.

Các type có thể gửi qua port: primitive types (int, double, bool, String, null), List, Map, Set (kể cả nested), `SendPort` (đặc biệt — có thể gửi port để thiết lập two-way communication), `TransferableTypedData` (transfer ownership thay vì copy), typed data (Uint8List, Float64List...).

**Không thể gửi**: closure/function (trừ top-level hoặc static), object có native resource (Socket, File handle, UI object), RenderObject, BuildContext.

---

## 5. Cách sử dụng — Từ cơ bản đến nâng cao

### 5.1 — compute() — Cách đơn giản nhất

`compute()` là convenience function trong Flutter, wrap toàn bộ boilerplate tạo isolate:

```dart
// Signature
Future<R> compute<M, R>(
  FutureOr<R> Function(M message) callback,
  M message,
  {String? debugLabel}
);
```

```dart
// Ví dụ: Parse JSON nặng
class UserRepository {
  Future<List<User>> fetchUsers() async {
    final response = await http.get(Uri.parse('https://api.example.com/users'));
    
    // ❌ Parse trên main isolate → jank nếu JSON lớn
    // final users = _parseUsers(response.body);
    
    // ✅ Parse trên isolate riêng
    final users = await compute(_parseUsers, response.body);
    return users;
  }
}

// BẮT BUỘC là top-level function hoặc static method
// KHÔNG THỂ là instance method, closure, hoặc anonymous function
List<User> _parseUsers(String jsonString) {
  final List<dynamic> jsonList = jsonDecode(jsonString);
  return jsonList.map((json) => User.fromJson(json)).toList();
}
```

Bên trong, `compute()` thực hiện:

```
1. Tạo isolate mới
2. Gửi message (tham số) sang isolate mới
3. Chạy callback với message đó
4. Gửi kết quả về main isolate
5. Kill isolate mới
6. Return kết quả qua Future
```

**Hạn chế của compute()**: chỉ gửi được **một message**, nhận **một kết quả**, rồi isolate bị kill. Không có two-way communication, không reuse isolate. Phù hợp cho tác vụ **one-shot**: parse JSON, mã hóa, xử lý ảnh đơn lẻ.

**Tại sao callback phải là top-level hoặc static?** Vì closure capture scope bên ngoài, mà scope đó nằm trên heap của main isolate — worker isolate không thể truy cập. Top-level/static function không capture bất kỳ scope nào nên có thể gửi sang isolate khác an toàn.

### 5.2 — Isolate.spawn() — Dart 2.19+

API mới hơn, đơn giản hơn `Isolate.run`:

```dart
// Isolate.run — thay thế compute() với API gọn hơn
Future<List<User>> fetchUsers() async {
  final response = await http.get(Uri.parse('https://api.example.com/users'));
  
  final users = await Isolate.run(() {
    // Closure ở đây ĐƯỢC PHÉP vì Dart tự copy
    // các biến captured sang isolate mới
    final List<dynamic> jsonList = jsonDecode(response.body);
    return jsonList.map((json) => User.fromJson(json)).toList();
  });
  
  return users;
}
```

`Isolate.run` cho phép closure (khác `compute`) vì Dart 2.19+ hỗ trợ **implicit message passing** cho captured variables. Tuy nhiên, các biến captured vẫn phải là **sendable types**.

### 5.3 — Isolate.spawn() + Port — Two-way Communication

Khi cần giao tiếp qua lại, gửi nhiều message, hoặc giữ isolate sống lâu dài:

```dart
class ImageProcessor {
  late final Isolate _isolate;
  late final SendPort _sendPort;
  final ReceivePort _receivePort = ReceivePort();
  final Map<int, Completer<Uint8List>> _pendingRequests = {};
  int _requestId = 0;

  Future<void> initialize() async {
    // Bước 1: Spawn isolate, gửi main's SendPort qua
    _isolate = await Isolate.spawn(
      _workerEntryPoint,
      _receivePort.sendPort,  // Gửi port để worker gửi ngược lại
    );

    // Bước 2: Nhận worker's SendPort (message đầu tiên)
    final completer = Completer<SendPort>();
    _receivePort.listen((message) {
      if (message is SendPort) {
        completer.complete(message);
      } else if (message is Map) {
        // Bước 5: Nhận kết quả từ worker
        final id = message['id'] as int;
        final result = message['data'] as Uint8List;
        _pendingRequests.remove(id)?.complete(result);
      }
    });
    _sendPort = await completer.future;
  }

  Future<Uint8List> processImage(Uint8List imageData) {
    // Bước 4: Gửi request tới worker
    final id = _requestId++;
    final completer = Completer<Uint8List>();
    _pendingRequests[id] = completer;
    _sendPort.send({'id': id, 'data': imageData});
    return completer.future;
  }

  void dispose() {
    _isolate.kill(priority: Isolate.immediate);
    _receivePort.close();
  }
}

// Bước 3: Entry point của worker isolate
// BẮT BUỘC top-level function
void _workerEntryPoint(SendPort mainSendPort) {
  final workerReceivePort = ReceivePort();
  
  // Gửi worker's SendPort về main
  mainSendPort.send(workerReceivePort.sendPort);

  // Lắng nghe request từ main
  workerReceivePort.listen((message) {
    final id = message['id'] as int;
    final imageData = message['data'] as Uint8List;

    // Xử lý nặng ở đây — không ảnh hưởng UI
    final result = _heavyImageProcessing(imageData);

    // Gửi kết quả về main
    mainSendPort.send({'id': id, 'data': result});
  });
}
```

Sơ đồ giao tiếp:

```
MAIN ISOLATE                          WORKER ISOLATE
     │                                       │
     │  spawn(entryPoint, mainSendPort)      │
     │ ────────────────────────────────────► │
     │                                       │
     │  ◄── workerSendPort ──────────────────│  (message đầu tiên)
     │                                       │
     │  ── {id: 1, data: image} ──────────►  │
     │                                       │  (xử lý nặng...)
     │  ◄── {id: 1, data: result} ───────────│
     │                                       │
     │  ── {id: 2, data: image2} ─────────►  │
     │                                       │  (xử lý nặng...)
     │  ◄── {id: 2, data: result2} ──────────│
     │                                       │
     │  kill()                               │
     │ ────────────────────────────────────► │ ✝
```

### 5.4 — Isolate Pool — Tái sử dụng Isolate

Tạo/hủy isolate có chi phí (vài ms + memory allocation). Với workload liên tục, dùng pool:

```dart
class IsolatePool {
  final int size;
  final List<_IsolateWorker> _workers = [];
  int _nextWorker = 0;

  IsolatePool({this.size = 4});

  Future<void> initialize() async {
    for (int i = 0; i < size; i++) {
      final worker = _IsolateWorker();
      await worker.initialize();
      _workers.add(worker);
    }
  }

  /// Round-robin dispatch
  Future<R> execute<M, R>(FutureOr<R> Function(M) task, M message) {
    final worker = _workers[_nextWorker % _workers.length];
    _nextWorker++;
    return worker.execute(task, message);
  }

  void dispose() {
    for (final worker in _workers) {
      worker.dispose();
    }
  }
}
```

Trong thực tế, package `workmanager` hoặc các state management library đã implement pool pattern. Flutter engine bản thân cũng dùng 4 thread nội bộ (Platform, UI, Raster, IO) — một dạng specialized pool.

---

## 6. Data Transfer Optimization

### Copy semantics (mặc định)

Khi gửi data qua SendPort, Dart thực hiện **deep copy**. Với data nhỏ (vài KB) thì không đáng kể. Nhưng với data lớn (ảnh 10MB, JSON 5MB), deep copy tốn thời gian và memory:

```dart
// Gửi Uint8List 10MB → copy 10MB sang isolate mới
// Memory peak: 20MB (bản gốc + bản copy)
sendPort.send(largeImageBytes); // Deep copy xảy ra
```

### TransferableTypedData — Zero-copy transfer

Dart cung cấp `TransferableTypedData` để **transfer ownership** thay vì copy:

```dart
// Chuyển ownership — KHÔNG copy data
final transferable = TransferableTypedData.fromList([largeImageBytes]);
sendPort.send(transferable);

// Sau khi send:
// - Main isolate KHÔNG CÒN truy cập được data
// - Worker isolate SỞ HỮU data
// - Memory: vẫn chỉ 10MB (không có bản copy)
```

```dart
// Bên worker nhận:
workerReceivePort.listen((message) {
  if (message is TransferableTypedData) {
    final bytes = message.materialize().asUint8List();
    // Giờ worker sở hữu data
    // Main isolate không thể dùng data gốc nữa
  }
});
```

Cơ chế bên dưới: thay vì copy bytes, Dart chuyển **ownership của memory region** từ isolate này sang isolate kia. Đây là zero-copy operation — chỉ update pointer ownership, không move data.

### Dart 3.x — SendPort.send với typed data tự động optimize

Từ Dart 2.15+, khi gửi `TypedData` (Uint8List, Int32List...) qua SendPort, Dart VM tự động dùng **fast path** — gần như zero-copy cho typed data lớn:

```dart
// Dart 2.15+ tự optimize cho TypedData
sendPort.send(uint8List); // VM dùng fast transfer nội bộ
```

---

## 7. Isolate trong Flutter — Đặc thù

### Main Isolate = UI Isolate

Trong Flutter, main isolate chạy cả Dart code lẫn framework rendering pipeline. Đây là lý do task nặng trên main isolate gây jank — nó block cả event loop xử lý frame rendering.

```
Main Isolate Event Loop:

  ┌─ [VSync signal] ────────────────────────────┐
  │                                             │
  │  Animation callbacks (Ticker)               │
  │  Build phase (rebuild dirty widgets)        │  ← Frame pipeline
  │  Layout phase (RenderObject.performLayout)  │     ~16ms budget
  │  Paint phase (RenderObject.paint)           │
  │  Compositing                                │
  │                                             │
  ├─ [Microtask queue] ─────────────────────────┤
  │  Future.then callbacks                      │
  │  Stream listeners                           │
  │                                             │
  ├─ [Event queue] ─────────────────────────────┤
  │  Touch/gesture events                       │
  │  Timer callbacks                            │
  │  I/O completions                            │
  │  Platform channel messages                  │
  │                                             │
  │  ⚠️ Heavy computation here = JANK           │
  └─────────────────────────────────────────────┘
```

### Không thể dùng Flutter API trong worker isolate

Worker isolate **không có access** tới Flutter engine. Những thứ sau **không hoạt động** trong worker isolate:

```dart
void workerEntryPoint(SendPort port) {
  // ❌ Không có WidgetsBinding
  WidgetsFlutterBinding.ensureInitialized(); // CRASH

  // ❌ Không có RootBundle
  rootBundle.loadString('assets/data.json'); // CRASH
  
  // ❌ Không có Platform Channels
  MethodChannel('my_channel').invokeMethod('foo'); // CRASH
  
  // ❌ Không có SharedPreferences (dùng platform channel)
  SharedPreferences.getInstance(); // CRASH
  
  // ✅ Chỉ dùng pure Dart code
  jsonDecode(data);           // OK
  utf8.encode(string);        // OK
  File('path').readAsBytes(); // OK (dart:io, không phải Flutter)
  // Crypto, compression, parsing, computation... OK
}
```

Nếu cần đọc asset trong worker, phải đọc trên main isolate trước rồi gửi data sang:

```dart
// Main isolate
final jsonString = await rootBundle.loadString('assets/large_data.json');
sendPort.send(jsonString); // Gửi string sang worker

// Worker isolate
receivePort.listen((message) {
  final data = jsonDecode(message as String); // Parse ở đây
});
```

### Background Isolate Channels — Flutter 3.7+

Từ Flutter 3.7, `RootIsolateToken` cho phép worker isolate dùng **một số** platform channel:

```dart
// Main isolate: lấy token
final token = RootIsolateToken.instance!;

// Gửi token sang worker
sendPort.send(token);

// Worker isolate
void workerEntryPoint(List<dynamic> args) {
  final sendPort = args[0] as SendPort;
  final token = args[1] as RootIsolateToken;
  
  // Đăng ký background isolate channel
  BackgroundIsolateBinaryMessenger.ensureInitialized(token);
  
  // Giờ có thể dùng một số plugin
  // (plugin phải support background isolate)
  final prefs = await SharedPreferences.getInstance(); // OK nếu plugin hỗ trợ
}
```

Tuy nhiên, không phải mọi plugin đều hỗ trợ. Plugin phải được viết để handle multi-isolate. Các plugin phổ biến như `path_provider`, `shared_preferences`, `sqflite` đã hỗ trợ.

---

## 8. Khi nào NÊN và KHÔNG NÊN dùng Isolate

### NÊN dùng

```dart
// ✅ Parse JSON lớn (>100KB)
final users = await compute(parseUsers, jsonString);

// ✅ Image processing
final compressed = await compute(compressImage, imageBytes);

// ✅ Encryption / Hashing
final hash = await compute(hashPassword, password);

// ✅ Complex computation
final result = await compute(runPathfinding, graphData);

// ✅ File I/O nặng (đọc/ghi file lớn + process)
final processed = await compute(processCSV, csvBytes);

// ✅ Database query phức tạp (SQLite) 
// sqflite tự dùng isolate nội bộ cho write operations

// ✅ Audio/Video encoding
final encoded = await compute(encodeAudio, rawSamples);
```

Nguyên tắc: bất kỳ **synchronous computation** nào tốn hơn **vài milliseconds** đều nên xem xét offload sang isolate. Benchmark đơn giản: nếu dùng `Stopwatch` đo mà > 4ms → nên dùng isolate.

### KHÔNG NÊN dùng

```dart
// ❌ I/O đơn thuần — đã non-blocking sẵn
final response = await http.get(url);
// http request chạy trên IO thread của Dart VM
// main isolate chỉ chờ callback, KHÔNG bị block

// ❌ Computation quá nhẹ — overhead > benefit
final sum = await compute(_addTwoNumbers, [1, 2]);
// Tạo isolate ~2-5ms, computation ~0.001ms → lãng phí

// ❌ Cần access Flutter API
// UI rendering, widget tree, platform channel (trừ khi dùng BackgroundIsolateBinaryMessenger)

// ❌ Shared state liên tục — Isolate không chia sẻ memory
// Nếu cần đọc/ghi cùng một state từ nhiều nơi,
// isolate model không phù hợp
```

**Điểm hay bị hiểu nhầm**: `async/await` và `Future` **không chạy trên thread riêng**. Chúng chỉ là cơ chế **non-blocking I/O** trên cùng một thread. `await http.get()` không block main thread vì I/O được delegate cho OS kernel, Dart VM chỉ đăng ký callback. Nhưng `jsonDecode(bigString)` sau khi nhận response **là synchronous computation** trên main thread — đây mới là chỗ cần isolate.

```
// Minh họa sự khác biệt

await http.get(url);
// Timeline:
// Main thread: [send request]──free──free──free──[callback: data ready]
// OS kernel:   ──────────[handling I/O]──────────
// → Main thread FREE trong lúc chờ → KHÔNG CẦN isolate

jsonDecode(hugeJson);
// Timeline:
// Main thread: [parse──parse──parse──parse──parse──parse──done]
// → Main thread BLOCKED suốt quá trình parse → CẦN isolate
```

---

## 9. Error Handling trong Isolate

### Unhandled errors

Mặc định, error trong worker isolate **không tự động propagate** về main isolate. Worker crash thầm lặng:

```dart
// Worker crash → main KHÔNG biết
Isolate.spawn(_worker, data);

void _worker(SendPort port) {
  throw Exception('Something went wrong'); // Crash thầm lặng
}
```

### Bắt error qua error port

```dart
final receivePort = ReceivePort();
final errorPort = ReceivePort();

final isolate = await Isolate.spawn(
  _worker,
  receivePort.sendPort,
  onError: errorPort.sendPort,  // Nhận error ở đây
  onExit: exitPort.sendPort,    // Nhận notification khi isolate exit
);

errorPort.listen((errorData) {
  // errorData là List: [errorMessage, stackTrace]
  final error = errorData[0] as String;
  final stack = errorData[1] as String?;
  print('Worker error: $error');
  print('Stack: $stack');
});
```

### Try-catch trong worker

Pattern an toàn hơn — catch error trong worker và gửi về main dưới dạng message:

```dart
void _workerEntryPoint(SendPort mainPort) {
  final workerPort = ReceivePort();
  mainPort.send(workerPort.sendPort);

  workerPort.listen((message) {
    try {
      final result = heavyComputation(message);
      mainPort.send({'status': 'success', 'data': result});
    } catch (e, stackTrace) {
      mainPort.send({
        'status': 'error',
        'error': e.toString(),
        'stackTrace': stackTrace.toString(),
      });
    }
  });
}
```

---

## 10. Memory & Performance Characteristics

### Chi phí tạo Isolate

```
Isolate spawn:              ~2-5ms (mobile), ~1-2ms (desktop)
Memory overhead per isolate: ~2MB (heap + stack + event loop overhead)
Message copy (1KB data):     ~0.01ms
Message copy (1MB data):     ~1-5ms
Message copy (10MB data):    ~10-50ms (nên dùng TransferableTypedData)
```

Implication: đừng tạo isolate cho mỗi task nhỏ. Nếu có 100 tasks nhỏ, tốt hơn là gom thành batch và gửi vào 1 isolate, hoặc dùng isolate pool.

### GC Independence

Mỗi isolate có GC riêng. Đây là **ưu điểm lớn**: GC pause trên worker isolate không ảnh hưởng main isolate. Với task tạo nhiều object tạm (parse lớn), GC pressure dồn hết lên worker mà UI vẫn mượt.

```
Main Isolate:  [frame][frame][frame][frame][frame]  ← mượt, không GC pause
Worker Isolate:[compute──GC──compute──GC──compute]   ← GC ở đây, ai care
```

---

## 11. Ví dụ thực tế hoàn chỉnh — CSV Processing Pipeline

```dart
/// Service xử lý CSV lớn không ảnh hưởng UI
class CSVProcessingService {
  Isolate? _isolate;
  SendPort? _sendPort;
  final _receivePort = ReceivePort();
  final _resultController = StreamController<ProcessingResult>.broadcast();

  Stream<ProcessingResult> get results => _resultController.stream;

  Future<void> initialize() async {
    _isolate = await Isolate.spawn(
      _csvWorker,
      _receivePort.sendPort,
      onError: _receivePort.sendPort,
    );

    _receivePort.listen((message) {
      if (message is SendPort) {
        _sendPort = message;
      } else if (message is Map<String, dynamic>) {
        _resultController.add(ProcessingResult.fromMap(message));
      }
    });

    // Chờ worker gửi SendPort về
    await _resultController.stream
        .firstWhere((_) => _sendPort != null)
        .timeout(
          const Duration(seconds: 5),
          onTimeout: () => throw TimeoutException('Worker init timeout'),
        )
        .catchError((_) {}); // SendPort đến qua listen ở trên
  }

  void processCSV(Uint8List csvBytes, {String delimiter = ','}) {
    _sendPort?.send({
      'action': 'process',
      'data': TransferableTypedData.fromList([csvBytes]),
      'delimiter': delimiter,
    });
  }

  void dispose() {
    _isolate?.kill(priority: Isolate.immediate);
    _receivePort.close();
    _resultController.close();
  }
}

// Top-level entry point
void _csvWorker(SendPort mainPort) {
  final workerPort = ReceivePort();
  mainPort.send(workerPort.sendPort);

  workerPort.listen((message) {
    if (message is! Map) return;
    
    final action = message['action'] as String;
    
    if (action == 'process') {
      try {
        final transferable = message['data'] as TransferableTypedData;
        final bytes = transferable.materialize().asUint8List();
        final delimiter = message['delimiter'] as String;

        // Heavy work — main isolate hoàn toàn tự do
        final csvString = utf8.decode(bytes);
        final lines = csvString.split('\n');
        final headers = lines.first.split(delimiter);
        
        final rows = <Map<String, String>>[];
        for (int i = 1; i < lines.length; i++) {
          if (lines[i].trim().isEmpty) continue;
          final values = lines[i].split(delimiter);
          final row = <String, String>{};
          for (int j = 0; j < headers.length && j < values.length; j++) {
            row[headers[j].trim()] = values[j].trim();
          }
          rows.add(row);

          // Gửi progress mỗi 1000 rows
          if (i % 1000 == 0) {
            mainPort.send({
              'status': 'progress',
              'processed': i,
              'total': lines.length - 1,
            });
          }
        }

        mainPort.send({
          'status': 'complete',
          'rowCount': rows.length,
          'headers': headers,
          'data': rows,
        });
      } catch (e, stack) {
        mainPort.send({
          'status': 'error',
          'error': e.toString(),
          'stackTrace': stack.toString(),
        });
      }
    }
  });
}
```

Usage trong Flutter:

```dart
class _ImportScreenState extends State<ImportScreen> {
  final _csvService = CSVProcessingService();

  @override
  void initState() {
    super.initState();
    _csvService.initialize();
    _csvService.results.listen((result) {
      if (!mounted) return;
      setState(() {
        if (result.status == 'progress') {
          _progress = result.processed / result.total;
        } else if (result.status == 'complete') {
          _data = result.data;
          _isProcessing = false;
        }
      });
    });
  }

  void _onFilePicked(Uint8List bytes) {
    setState(() => _isProcessing = true);
    _csvService.processCSV(bytes);
    // UI vẫn mượt — progress bar animate bình thường
    // trong khi CSV đang parse trên worker isolate
  }

  @override
  void dispose() {
    _csvService.dispose();
    super.dispose();
  }
}
```

---

## 12. Tóm tắt quyết định

```
Task là I/O thuần (network, file read)?         → async/await là đủ, KHÔNG cần isolate
Task là computation < 2ms?                       → Chạy trên main, KHÔNG cần isolate
Task là computation 2-16ms?                      → Xem xét isolate nếu chạy mỗi frame
Task là computation > 16ms?                      → BẮT BUỘC dùng isolate
Task one-shot (parse, encode)?                   → compute() hoặc Isolate.run()
Task liên tục (stream processing)?               → Isolate.spawn() + port communication
Nhiều task nhỏ liên tục?                         → Isolate pool (reuse isolate)
Data gửi > 1MB?                                  → TransferableTypedData (zero-copy)
Cần Flutter API trong worker?                    → BackgroundIsolateBinaryMessenger (3.7+)
```

Isolate là công cụ mạnh nhất trong Dart để xử lý concurrency. Khác với thread truyền thống, nó **an toàn by design** nhờ no shared memory, nhưng đổi lại phải thiết kế giao tiếp qua message passing. Senior cần biết chính xác khi nào overhead của isolate (spawn time, memory, copy cost) được bù đắp bởi lợi ích (UI mượt, GC isolation), và khi nào `async/await` trên main isolate đã đủ tốt.

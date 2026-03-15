# RepaintBoundary — Giải thích chi tiết

## 1. Trước hết: Flutter paint như thế nào?

Để hiểu `RepaintBoundary`, cần hiểu cơ chế **Layer Tree** và **repaint** của Flutter.

### RenderObject và Paint

Sau khi layout xong, Flutter bước vào **paint phase**. Mỗi `RenderObject` có method `paint()` để vẽ chính nó lên một `Canvas`. Mặc định, nhiều RenderObject **chia sẻ chung một Layer** — tức là chúng vẽ lên cùng một bề mặt (backing store).

```
Không có RepaintBoundary:

┌─────────────────────────────────┐
│          Layer duy nhất          │
│                                  │
│  ┌──────┐ ┌──────┐ ┌──────┐    │
│  │  A   │ │  B   │ │  C   │    │
│  │      │ │(thay │ │      │    │
│  │      │ │ đổi) │ │      │    │
│  └──────┘ └──────┘ └──────┘    │
│                                  │
│  Khi B thay đổi → TOÀN BỘ      │
│  layer phải repaint             │
│  → A, B, C đều vẽ lại          │
└─────────────────────────────────┘
```

Khi **bất kỳ** RenderObject nào trên layer bị mark dirty (cần repaint), **toàn bộ layer** phải được vẽ lại từ đầu. Dù A và C không thay đổi gì, chúng vẫn phải chạy lại `paint()`. Với UI đơn giản thì không sao, nhưng với widget phức tạp (Custom Paint, Image, complex decoration...) thì chi phí repaint rất lớn.

### RepaintBoundary tạo Layer riêng

`RepaintBoundary` tạo ra một **layer mới** cho subtree bên dưới. Mỗi layer được cache **độc lập** như một bitmap trên GPU:

```
Có RepaintBoundary bọc B:

┌─────────────────────────────────┐
│        Layer cha                 │
│                                  │
│  ┌──────┐           ┌──────┐   │
│  │  A   │           │  C   │   │
│  └──────┘           └──────┘   │
│                                  │
│         ┌──────────────┐        │
│         │ Layer riêng   │        │
│         │ ┌──────┐     │        │
│         │ │  B   │     │        │
│         │ │(thay │     │        │
│         │ │ đổi) │     │        │
│         │ └──────┘     │        │
│         └──────────────┘        │
│                                  │
│  Khi B thay đổi → CHỈ layer    │
│  của B repaint                   │
│  → A, C KHÔNG bị ảnh hưởng     │
└─────────────────────────────────┘
```

Giờ khi B thay đổi, chỉ layer chứa B cần repaint. Layer cha (chứa A và C) giữ nguyên bitmap đã cache — GPU chỉ cần composite các layer lại với nhau, thay vì vẽ lại tất cả.

---

## 2. Cơ chế hoạt động bên trong

### isRepaintBoundary

Mỗi `RenderObject` có property `isRepaintBoundary`. Mặc định là `false`, nghĩa là RenderObject chia sẻ layer với parent. Khi `RepaintBoundary` widget được dùng, nó tạo ra `RenderRepaintBoundary` — một RenderObject override `isRepaintBoundary` thành `true`:

```dart
// Source code simplified
class RenderRepaintBoundary extends RenderProxyBox {
  @override
  bool get isRepaintBoundary => true;
}
```

Khi framework gặp RenderObject có `isRepaintBoundary == true`, nó tạo một **OffsetLayer** mới. Subtree bên dưới paint vào layer này thay vì layer cha.

### markNeedsPaint() propagation

Khi một RenderObject cần repaint (do `setState`, animation, v.v.), nó gọi `markNeedsPaint()`. Method này **đi ngược lên tree** cho đến khi gặp một repaint boundary:

```dart
// Simplified internal logic
void markNeedsPaint() {
  if (_needsPaint) return; // Đã dirty rồi
  _needsPaint = true;
  
  if (isRepaintBoundary) {
    // DỪNG ở đây — chỉ schedule repaint cho layer này
    owner!._nodesNeedingPaint.add(this);
    owner!.requestVisualUpdate();
  } else {
    // Không phải boundary → leo lên parent
    parent!.markNeedsPaint();
  }
}
```

```
Không có RepaintBoundary:

markNeedsPaint() trên C
    │ C.isRepaintBoundary? false → leo lên
    ▼
markNeedsPaint() trên B
    │ B.isRepaintBoundary? false → leo lên
    ▼
markNeedsPaint() trên A
    │ A.isRepaintBoundary? false → leo lên
    ▼
markNeedsPaint() trên Root (RenderView)
    │ Root.isRepaintBoundary? true → DỪNG
    ▼
Toàn bộ tree từ Root trở xuống repaint


Có RepaintBoundary bọc B:

markNeedsPaint() trên C (con của B)
    │ C.isRepaintBoundary? false → leo lên
    ▼
markNeedsPaint() trên RepaintBoundary
    │ isRepaintBoundary? true → DỪNG
    ▼
Chỉ subtree của RepaintBoundary repaint
A và phần còn lại KHÔNG bị ảnh hưởng
```

### Layer caching & Compositing

Sau khi paint, mỗi layer tồn tại như một **rasterized texture** trên GPU memory. Khi composite (ghép các layer thành frame cuối cùng), GPU chỉ cần overlay các texture — operation rất rẻ so với repaint. Đây là lý do RepaintBoundary cải thiện performance: **đánh đổi GPU memory (lưu thêm texture)** để **giảm CPU/GPU paint work**.

---

## 3. Cách sử dụng cơ bản

### Cú pháp

```dart
RepaintBoundary(
  child: MyExpensiveWidget(),
)
```

Chỉ cần wrap widget cần isolate bằng `RepaintBoundary`. Không có parameter nào khác ngoài `child` và `key`.

### Ví dụ 1 — Animation isolate

Một màn hình có header tĩnh phức tạp và một animation chạy liên tục:

```dart
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      // Header phức tạp với gradient, shadow, custom paint
      RepaintBoundary(
        child: ComplexHeader(
          gradient: _headerGradient,
          shadow: _headerShadow,
          // ... nhiều decoration
        ),
      ),
      
      // Animation chạy 60fps → trigger repaint mỗi frame
      Expanded(
        child: AnimatedBuilder(
          animation: _controller,
          builder: (ctx, child) {
            return Transform.rotate(
              angle: _controller.value * 2 * pi,
              child: child,
            );
          },
          child: const Icon(Icons.refresh, size: 48),
        ),
      ),
    ],
  );
}
```

Không có `RepaintBoundary`, mỗi frame animation sẽ repaint **cả ComplexHeader** — tốn kém vô ích vì header không thay đổi. Với `RepaintBoundary`, header được cache thành layer riêng, animation repaint chỉ ảnh hưởng layer phía dưới.

### Ví dụ 2 — ListView item isolation

`ListView.builder` mặc định đã wrap mỗi item trong `RepaintBoundary` (qua `addRepaintBoundaries: true`):

```dart
// Bên trong SliverChildBuilderDelegate
if (addRepaintBoundaries) {
  child = RepaintBoundary(child: child);
}
```

Nghĩa là khi một item trong list thay đổi (ví dụ: favorite button toggle), chỉ layer của item đó repaint, các item lân cận giữ nguyên cached layer. Đây là lý do `ListView.builder` có tham số `addRepaintBoundaries`.

### Ví dụ 3 — CustomPaint tốn kém

```dart
RepaintBoundary(
  child: CustomPaint(
    painter: HeavyChartPainter(data: _chartData),
    size: Size(400, 300),
  ),
)
```

Nếu `HeavyChartPainter` vẽ hàng ngàn data point, việc repaint nó rất tốn kém. Wrap trong `RepaintBoundary` đảm bảo chart chỉ repaint khi `_chartData` thực sự thay đổi, không bị repaint theo khi phần khác của screen thay đổi.

### Ví dụ 4 — Capture widget thành image

`RepaintBoundary` còn được dùng để **chụp screenshot** một widget:

```dart
final GlobalKey _boundaryKey = GlobalKey();

// Trong widget tree
RepaintBoundary(
  key: _boundaryKey,
  child: MyWidgetToCapture(),
)

// Capture thành image
Future<Uint8List?> captureWidget() async {
  final boundary = _boundaryKey.currentContext!
      .findRenderObject() as RenderRepaintBoundary;
  
  final image = await boundary.toImage(pixelRatio: 3.0);
  final byteData = await image.toByteData(format: ImageByteFormat.png);
  return byteData?.buffer.asUint8List();
}
```

Method `toImage()` chỉ có trên `RenderRepaintBoundary` — nó capture chính xác layer đó thành `dart:ui.Image`. Đây là cơ chế mà các package screenshot, share widget as image sử dụng.

---

## 4. Khi nào NÊN dùng

### 4.1 — Widget tĩnh nằm cạnh widget animate liên tục

Nguyên tắc: **bọc phần TĨNH** để isolate nó khỏi phần động. Hoặc bọc phần động nếu phần tĩnh chiếm đa số:

```dart
Stack(
  children: [
    // Nền phức tạp, tĩnh → bọc lại
    RepaintBoundary(
      child: ComplexBackground(),
    ),
    
    // Particle animation chạy 60fps
    ParticleSystem(),
  ],
)
```

### 4.2 — Tab/Page chứa nội dung phức tạp

Khi dùng `TabBarView` hoặc `PageView`, mỗi page nên được isolate:

```dart
TabBarView(
  children: [
    RepaintBoundary(child: ComplexPage1()),
    RepaintBoundary(child: ComplexPage2()),
    RepaintBoundary(child: ComplexPage3()),
  ],
)
```

Khi swipe giữa các tab, transition animation không cần repaint nội dung bên trong mỗi page — chỉ cần di chuyển cached layer.

### 4.3 — Phần UI ít thay đổi trong screen hay rebuild

Nếu screen rebuild thường xuyên (do state management) nhưng có phần UI luôn giữ nguyên:

```dart
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      // Logo, branding — KHÔNG BAO GIỜ thay đổi
      RepaintBoundary(
        child: const BrandingHeader(),
      ),
      
      // Phần này rebuild liên tục do data stream
      StreamBuilder<List<Item>>(
        stream: _itemStream,
        builder: (ctx, snapshot) {
          return ItemList(items: snapshot.data ?? []);
        },
      ),
    ],
  );
}
```

### 4.4 — Widget dùng ShaderMask, BackdropFilter, ColorFiltered

Các widget dùng shader/filter rất tốn kém khi repaint:

```dart
RepaintBoundary(
  child: BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
    child: Container(
      color: Colors.white.withOpacity(0.3),
      child: Text('Frosted Glass Effect'),
    ),
  ),
)
```

`BackdropFilter` mỗi lần paint phải đọc pixel từ layer bên dưới rồi apply blur — operation nặng. Isolate nó trong `RepaintBoundary` giảm đáng kể số lần phải chạy filter.

---

## 5. Khi nào KHÔNG nên dùng

### 5.1 — Widget đơn giản, paint rẻ

```dart
// ❌ THỪA — Text paint cực rẻ, overhead > benefit
RepaintBoundary(
  child: Text('Hello World'),
)

// ❌ THỪA — Container đơn giản
RepaintBoundary(
  child: Container(
    color: Colors.blue,
    width: 100,
    height: 100,
  ),
)
```

Mỗi `RepaintBoundary` tạo thêm một layer → chiếm thêm GPU memory. Với widget đơn giản, chi phí repaint rất nhỏ (microseconds), trong khi chi phí duy trì thêm layer (memory allocation, compositing overhead) có thể lớn hơn lợi ích tiết kiệm được.

### 5.2 — Widget thay đổi liên tục

```dart
// ❌ VÔ ÍCH — Widget này repaint MỖI FRAME rồi
RepaintBoundary(
  child: AnimatedBuilder(
    animation: _controller, // 60fps animation
    builder: (ctx, _) => CustomPaint(
      painter: WavePainter(_controller.value),
    ),
  ),
)
```

Nếu widget bên trong `RepaintBoundary` thay đổi mỗi frame, cached layer bị invalidate mỗi frame → **không bao giờ được dùng lại**. Kết quả: overhead tạo/hủy layer liên tục mà không có lợi ích nào. Trong trường hợp này chỉ nên dùng `RepaintBoundary` nếu mục đích là **bảo vệ phần bên ngoài** khỏi bị repaint theo.

### 5.3 — Lồng quá nhiều RepaintBoundary

```dart
// ❌ Mỗi item tạo 1 layer → 1000 layers
ListView.builder(
  itemCount: 1000,
  addRepaintBoundaries: true, // đã có sẵn!
  itemBuilder: (ctx, i) => RepaintBoundary( // ← thừa, đã có mặc định
    child: RepaintBoundary( // ← layer thứ 3!
      child: ListTile(title: Text('Item $i')),
    ),
  ),
)
```

Quá nhiều layer làm **compositing phase chậm đi** vì GPU phải merge nhiều texture. Đồng thời GPU memory tăng lên đáng kể. Mỗi layer chiếm memory tỷ lệ với kích thước pixel của nó.

### 5.4 — Widget bị clip bởi parent

Khi parent clip child (via `ClipRect`, `ClipRRect`, overflow hidden...), `RepaintBoundary` tạo layer có kích thước **trước clip** → lãng phí memory cho phần bị clip đi:

```dart
// Widget 2000px nhưng chỉ hiển thị 200px (do clip)
// RepaintBoundary vẫn cache 2000px layer → lãng phí
ClipRect(
  child: RepaintBoundary( // ← cache cả phần bị clip
    child: HugeWidget(height: 2000),
  ),
)
```

---

## 6. Đo lường hiệu quả — Performance Overlay & DevTools

### Debug flags

```dart
// Hiển thị border quanh mỗi RepaintBoundary
// Giúp thấy layer boundaries trên screen
import 'package:flutter/rendering.dart';

void main() {
  debugRepaintRainbowEnabled = true;  // Mỗi layer đổi màu border khi repaint
  debugRepaintTextRainbowEnabled = true;
  runApp(MyApp());
}
```

`debugRepaintRainbowEnabled` vẽ border **đổi màu mỗi lần repaint** quanh mỗi RenderObject. Nếu thấy một vùng đổi màu liên tục trong khi nội dung không thay đổi → đó là **repaint thừa**, cần RepaintBoundary.

### RenderRepaintBoundary debug metrics

```dart
final boundary = _key.currentContext!.findRenderObject() 
    as RenderRepaintBoundary;

print('Total paints: ${boundary.debugSymmetricPaintCount}');
print('Saved paints: ${boundary.debugAsymmetricPaintCount}');
```

Hai property này cho biết: `debugSymmetricPaintCount` là số lần **cả parent lẫn child đều repaint** (boundary không giúp gì). `debugAsymmetricPaintCount` là số lần **chỉ một bên repaint** (boundary phát huy tác dụng). Nếu symmetric >> asymmetric, boundary đang lãng phí.

### Flutter DevTools — Performance Tab

Trong DevTools, bật **"Show Repaint Rainbow"** để thấy trực tiếp vùng nào đang repaint trên app thực tế. Kết hợp với **Timeline view** để xem thời gian paint phase:

```
Nếu paint phase > 8ms (nửa frame budget) → cần tối ưu
Các bước:
1. Bật repaint rainbow → tìm vùng repaint thừa
2. Wrap vùng TĨNH trong RepaintBoundary
3. Đo lại paint phase
4. Kiểm tra layer count không tăng quá nhiều
```

---

## 7. RepaintBoundary trong hệ sinh thái Flutter

Nhiều widget **tự động tạo RepaintBoundary** mà developer không nhận ra:

```
Widget                          Tự tạo RepaintBoundary?
──────────────────────────────────────────────────────
ListView.builder items          ✅ (addRepaintBoundaries: true)
GridView.builder items          ✅ (addRepaintBoundaries: true)
Navigator route pages           ✅ (mỗi route là một layer)
Opacity                         ✅ (tạo OffsetLayer riêng)
ShaderMask                      ✅
BackdropFilter                  ✅
PhysicalModel                   ✅
AndroidView / UiKitView         ✅ (platform view cần layer riêng)
Texture (video, camera)         ✅
Flow                            ✅
EditableText (TextField)        ✅
```

Điều này có nghĩa là trong nhiều trường hợp, Flutter đã tối ưu sẵn. Senior cần biết để **không thêm RepaintBoundary thừa** lồng bên trong những widget đã có sẵn boundary.

---

## 8. RepaintBoundary vs các kỹ thuật tối ưu paint khác

### So với const widget

```dart
// const widget: tránh REBUILD (widget/element layer)
const Text('Hello')  // Framework skip rebuild vì widget instance không đổi

// RepaintBoundary: tránh REPAINT (render layer)
RepaintBoundary(child: text)  // Tạo layer riêng, skip repaint
```

Hai cơ chế hoạt động ở **hai layer khác nhau**. `const` ngăn widget tree rebuild → element không bị mark dirty → render object không cần layout/paint lại. `RepaintBoundary` không ảnh hưởng rebuild — widget vẫn có thể rebuild, nhưng nếu output paint giống hệt frame trước, cached layer được dùng lại.

Lý tưởng là dùng **cả hai**: `const` giảm work ở widget/element layer, `RepaintBoundary` giảm work ở render/paint layer.

### So với shouldRepaint trong CustomPainter

```dart
class MyPainter extends CustomPainter {
  final double value;
  MyPainter(this.value);

  @override
  void paint(Canvas canvas, Size size) { /* ... */ }

  @override
  bool shouldRepaint(MyPainter oldDelegate) {
    return value != oldDelegate.value; // Chỉ repaint khi value thay đổi
  }
}
```

`shouldRepaint` kiểm soát **chính CustomPaint đó** có cần paint lại không. Nhưng nếu **sibling hoặc parent** trigger repaint, CustomPaint vẫn bị repaint theo (vì cùng layer). `RepaintBoundary` bọc bên ngoài giải quyết vấn đề này — isolate CustomPaint vào layer riêng, kết hợp với `shouldRepaint` để có tối ưu kép.

### So với willChange trong Flow

```dart
Flow(
  delegate: MyFlowDelegate(),
  // Flow tự tạo RepaintBoundary cho mỗi child
  // Và dùng transformation matrix thay vì repaint
  children: [...],
)
```

`Flow` là widget đặc biệt — nó paint children bằng **transformation matrix** thay vì rebuild/repaint. Khi delegate thay đổi, Flow chỉ cập nhật matrix (translate, rotate, scale) mà không cần repaint child. Đây là cơ chế tối ưu paint ở mức cao hơn RepaintBoundary.

---

## 9. Ví dụ thực tế hoàn chỉnh — Chat Screen

```dart
class ChatScreen extends StatefulWidget {
  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen>
    with TickerProviderStateMixin {
  late final AnimationController _typingIndicator;

  @override
  void initState() {
    super.initState();
    _typingIndicator = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 600),
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _typingIndicator.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // ① AppBar tĩnh — isolate khỏi animation phía dưới
      appBar: PreferredSize(
        preferredSize: const Size.fromHeight(60),
        child: RepaintBoundary(
          child: AppBar(
            title: const Text('Chat'),
            actions: [IconButton(icon: const Icon(Icons.call), onPressed: () {})],
          ),
        ),
      ),

      body: Column(
        children: [
          // ② Message list — mỗi item đã có RepaintBoundary
          //    từ ListView.builder mặc định
          Expanded(
            child: ListView.builder(
              // addRepaintBoundaries: true (mặc định)
              itemCount: _messages.length,
              itemBuilder: (ctx, i) {
                final msg = _messages[i];
                return _MessageBubble(message: msg);
              },
            ),
          ),

          // ③ Typing indicator — animation 60fps
          //    Bọc trong RepaintBoundary để KHÔNG ảnh hưởng
          //    message list phía trên
          RepaintBoundary(
            child: AnimatedBuilder(
              animation: _typingIndicator,
              builder: (ctx, _) {
                return Opacity(
                  opacity: _typingIndicator.value,
                  child: const Padding(
                    padding: EdgeInsets.all(8.0),
                    child: Text('Đang nhập...'),
                  ),
                );
              },
            ),
          ),

          // ④ Input bar — có TextField (đã tự tạo boundary)
          //    nhưng bọc thêm vì custom decoration phức tạp
          RepaintBoundary(
            child: _ChatInputBar(onSend: _sendMessage),
          ),
        ],
      ),
    );
  }
}
```

Phân tích:

- **①** AppBar tĩnh được isolate — khi typing indicator animate, AppBar không repaint
- **②** ListView.builder tự thêm `RepaintBoundary` cho mỗi item — không cần thêm
- **③** Typing indicator animate liên tục — isolate để message list không bị repaint theo
- **④** Input bar có decoration phức tạp — isolate khỏi typing indicator

---

## 10. Tóm tắt quyết định

```
Widget tĩnh nằm cạnh animation?           → RepaintBoundary cho widget tĩnh  ✅
CustomPaint nặng thay đổi ít?             → RepaintBoundary bọc ngoài         ✅
BackdropFilter / ShaderMask?              → Đã tự tạo layer, KHÔNG cần thêm   ❌
ListView.builder item?                     → Đã có sẵn, KHÔNG cần thêm        ❌
Widget đơn giản (Text, Icon)?             → Overhead > benefit, KHÔNG dùng    ❌
Widget thay đổi mỗi frame?               → Vô ích cho chính nó               ❌
                                           (chỉ có ích để bảo vệ bên ngoài)
Cần capture widget thành image?           → RepaintBoundary + toImage()       ✅
Quá nhiều layers (>50)?                   → Giảm bớt RepaintBoundary          ⚠️
```

Nguyên tắc vàng: **đo trước, tối ưu sau**. Bật `debugRepaintRainbowEnabled`, quan sát vùng nào repaint thừa, thêm `RepaintBoundary` có chủ đích, rồi đo lại để xác nhận cải thiện. Không bao giờ thêm `RepaintBoundary` "phòng xa" mà không có dữ liệu chứng minh cần thiết.

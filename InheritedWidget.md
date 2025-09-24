Chào bạn, `InheritedWidget` là một trong những widget nền tảng và quan trọng nhất trong Flutter để quản lý trạng thái (state management). Mặc dù ngày nay chúng ta thường dùng các thư viện như Provider hay Riverpod, nhưng chúng đều được xây dựng dựa trên `InheritedWidget`. Hiểu rõ nó sẽ giúp bạn nắm được gốc rễ của việc truyền dữ liệu trong Flutter.

### Vấn đề `InheritedWidget` giải quyết là gì?

Hãy tưởng tượng bạn có một cây widget như sau:

```
MyApp
└── HomePage
    └── ProfileSection
        └── UserAvatar
```

Giả sử `MyApp` có thông tin về người dùng đang đăng nhập (ví dụ: `currentUser`). Bây giờ, widget `UserAvatar` ở tận sâu bên trong cần thông tin này để hiển thị ảnh đại diện.

**Cách làm thông thường (không hiệu quả):**
Bạn sẽ phải truyền đối tượng `currentUser` qua constructor của từng widget:
1.  `MyApp` truyền `currentUser` cho `HomePage`.
2.  `HomePage` nhận và lại truyền `currentUser` cho `ProfileSection`.
3.  `ProfileSection` nhận và lại truyền `currentUser` cho `UserAvatar`.

Cách này gọi là **"Prop Drilling"** (khoan luồng thuộc tính). Nó rất dài dòng, khó bảo trì, và các widget ở giữa (`HomePage`, `ProfileSection`) phải nhận một dữ liệu mà chúng không hề sử dụng.

### `InheritedWidget` xuất hiện như một vị cứu tinh!

`InheritedWidget` là một loại widget đặc biệt cho phép các widget con cháu (descendants) trong cây truy cập dữ liệu của nó một cách hiệu quả mà không cần phải truyền qua từng cấp.

**Cơ chế hoạt động:**

1.  Bạn đặt một `InheritedWidget` ở một vị trí cao trong cây widget.
2.  Bất kỳ widget nào ở bên dưới nó có thể "đăng ký" để nhận dữ liệu từ `InheritedWidget` đó.
3.  Khi dữ liệu trong `InheritedWidget` thay đổi, **chỉ những widget đã đăng ký** mới được build lại. Điều này cực kỳ hiệu quả về mặt hiệu suất.

Các ví dụ kinh điển mà bạn dùng hằng ngày chính là `Theme.of(context)` và `MediaQuery.of(context)`. Cả `Theme` và `MediaQuery` đều là các `InheritedWidget`.

---

### Cách tạo và sử dụng một `InheritedWidget`

Hãy cùng xem một ví dụ cụ thể: tạo một `InheritedWidget` để chia sẻ một bộ đếm (counter).

#### Bước 1: Tạo lớp `InheritedWidget` của riêng bạn

Bạn cần tạo một class kế thừa từ `InheritedWidget`. Class này thường có 3 phần chính:

1.  **Dữ liệu cần chia sẻ:** Ví dụ: `final int counterData;`
2.  **Hàm `updateShouldNotify`:** Quyết định xem có cần build lại các widget con đang "lắng nghe" hay không khi `InheritedWidget` được cập nhật.
3.  **Hàm static `of(context)`:** Một phương thức tiện ích (convention) để các widget con có thể dễ dàng truy cập dữ liệu.

```dart
import 'package:flutter/material.dart';

// 1. Tạo class kế thừa InheritedWidget
class CounterProvider extends InheritedWidget {
  // Dữ liệu sẽ được chia sẻ xuống cây widget
  final int counterData;

  const CounterProvider({
    Key? key,
    required this.counterData,
    required Widget child,
  }) : super(key: key, child: child);

  // 2. Quyết định xem có cần thông báo (build lại) cho các widget con hay không
  // Nó sẽ được gọi khi CounterProvider được build lại với dữ liệu mới.
  @override
  bool updateShouldNotify(CounterProvider oldWidget) {
    // Nếu dữ liệu cũ khác dữ liệu mới thì trả về true (cần build lại)
    return oldWidget.counterData != counterData;
  }

  // 3. Hàm static 'of' để các widget con dễ dàng truy cập
  static CounterProvider? of(BuildContext context) {
    // Dòng này tìm kiếm CounterProvider gần nhất ở phía trên trong cây
    // và đăng ký widget gọi hàm này để nhận cập nhật.
    return context.dependOnInheritedWidgetOfExactType<CounterProvider>();
  }
}
```

#### Bước 2: Đặt `InheritedWidget` lên trên cây widget

Bạn cần một `StatefulWidget` ở phía trên để giữ trạng thái và cập nhật `InheritedWidget`.

```dart
class CounterPage extends StatefulWidget {
  const CounterPage({Key? key}) : super(key: key);

  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    // Bọc cây widget con bằng CounterProvider
    // Mỗi khi _counter thay đổi và setState được gọi,
    // CounterProvider sẽ được tạo lại với giá trị counterData mới.
    return CounterProvider(
      counterData: _counter,
      child: Scaffold(
        appBar: AppBar(title: Text('InheritedWidget Demo')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text('You have pushed the button this many times:'),
              // Widget con sẽ lấy dữ liệu từ đây
              CounterDisplay(),
            ],
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _incrementCounter,
          tooltip: 'Increment',
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

#### Bước 3: Truy cập dữ liệu từ widget con

Bây giờ, widget `CounterDisplay` có thể lấy dữ liệu trực tiếp mà không cần truyền qua constructor.

```dart
class CounterDisplay extends StatelessWidget {
  const CounterDisplay({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // Sử dụng hàm of() để lấy InheritedWidget
    final counterProvider = CounterProvider.of(context);

    return Text(
      // Lấy dữ liệu từ đó
      '${counterProvider?.counterData}',
      style: Theme.of(context).textTheme.headlineMedium,
    );
  }
}
```

Khi bạn nhấn nút `+`, `_CounterPageState` sẽ gọi `setState`, `_counter` tăng lên. `CounterProvider` được build lại với `counterData` mới. Hàm `updateShouldNotify` trả về `true`, và Flutter sẽ tự động build lại `CounterDisplay` vì nó đã gọi `CounterProvider.of(context)`.

### Ưu và Nhược điểm

*   **Ưu điểm:**
    *   **Hiệu suất cao:** Chỉ build lại các widget thực sự phụ thuộc vào dữ liệu.
    *   **Nền tảng:** Là cốt lõi của Flutter, không cần thư viện bên ngoài.
    *   **Truyền dữ liệu hiệu quả:** Giải quyết triệt để vấn đề "prop drilling".

*   **Nhược điểm:**
    *   **Khá dài dòng (Boilerplate):** Phải tạo một class riêng, viết hàm `updateShouldNotify` và `of`.
    *   **Chỉ truyền dữ liệu xuống:** Nó không cung cấp cách dễ dàng để widget con "gọi ngược" lên để thay đổi trạng thái. Bạn vẫn phải dùng callback (như ví dụ `_incrementCounter` được truyền xuống).
    *   **Khó tách biệt logic:** Logic cập nhật trạng thái (`_incrementCounter`) và dữ liệu (`_counter`) thường nằm trong một `StatefulWidget` phía trên, có thể làm code khó quản lý hơn.

### Kết luận

`InheritedWidget` là một công cụ mạnh mẽ và là nền tảng cho việc quản lý trạng thái trong Flutter. Mặc dù trong các dự án hiện đại, bạn nên ưu tiên sử dụng các thư viện như **Provider** (vốn là một phiên bản cải tiến, dễ dùng hơn của `InheritedWidget`) hoặc **Riverpod**, việc hiểu rõ `InheritedWidget` sẽ cho bạn một cái nhìn sâu sắc về cách Flutter hoạt động bên trong.

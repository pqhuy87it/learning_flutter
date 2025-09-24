Chào bạn, tôi rất sẵn lòng giải thích chi tiết về `Navigator` trong Flutter. Đây là một trong những concept nền tảng và quan trọng nhất để xây dựng một ứng dụng có nhiều màn hình.

### `Navigator` là gì?

Hãy tưởng tượng các màn hình trong ứng dụng của bạn như một chồng thẻ bài. Khi bạn mở một màn hình mới, bạn đặt một thẻ mới lên trên cùng. Khi bạn quay lại, bạn lấy thẻ trên cùng ra. `Navigator` trong Flutter quản lý chính xác "chồng thẻ" này.

Về mặt kỹ thuật, `Navigator` là một widget quản lý một **stack (ngăn xếp)** các **routes (tuyến đường)**.
*   **Stack:** Hoạt động theo nguyên tắc LIFO (Last-In, First-Out - Vào sau, Ra trước). Màn hình bạn thấy là màn hình ở trên cùng của stack.
*   **Route:** Là một đối tượng đại diện cho một màn hình (hoặc một trang) trong ứng dụng. `MaterialPageRoute` là loại route phổ biến nhất, nó cung cấp hiệu ứng chuyển cảnh mặc định của Material Design.

---

### 1. Điều hướng cơ bản: `push` và `pop`

Đây là hai thao tác cốt lõi bạn sẽ sử dụng nhiều nhất.

#### a. `Navigator.push()` - Mở một màn hình mới

Hàm này "đẩy" (push) một route mới lên trên cùng của stack, làm cho màn hình mới xuất hiện.

*   **Cách dùng:** `Navigator.push(context, MaterialPageRoute(builder: (context) => NewScreen()));`
*   `context`: `BuildContext` của widget hiện tại, giúp Navigator tìm thấy stack gần nhất trong cây widget.
*   `MaterialPageRoute`: Tạo một route cho màn hình mới.
*   `builder`: Một hàm trả về instance của widget màn hình bạn muốn hiển thị (ví dụ: `NewScreen()`).

**Ví dụ: Từ Màn hình chính sang Màn hình chi tiết**

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: HomeScreen(),
    );
  }
}

// Màn hình 1: HomeScreen
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Màn hình chính')),
      body: Center(
        child: ElevatedButton(
          child: const Text('Đi tới màn hình chi tiết'),
          onPressed: () {
            // Đẩy màn hình DetailScreen lên stack
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const DetailScreen()),
            );
          },
        ),
      ),
    );
  }
}

// Màn hình 2: DetailScreen
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Màn hình chi tiết')), // AppBar tự động có nút back
      body: Center(
        child: ElevatedButton(
          child: const Text('Quay lại'),
          onPressed: () {
            // Lấy màn hình hiện tại ra khỏi stack
            Navigator.pop(context);
          },
        ),
      ),
    );
  }
}
```

#### b. `Navigator.pop()` - Quay lại màn hình trước

Hàm này "lấy ra" (pop) route trên cùng khỏi stack, quay trở về màn hình trước đó. `AppBar` của Flutter tự động thêm một nút "Back" và nó sẽ gọi `Navigator.pop(context)` khi được nhấn.

---

### 2. Truyền dữ liệu giữa các màn hình

#### a. Truyền dữ liệu tới màn hình mới (Ví dụ: HomeScreen → DetailScreen)

Cách phổ biến nhất là truyền dữ liệu qua constructor của widget màn hình mới.

1.  Trong widget `DetailScreen`, thêm một thuộc tính và cập nhật constructor để nhận dữ liệu.
2.  Khi gọi `Navigator.push`, hãy tạo instance của `DetailScreen` và truyền dữ liệu vào.

**Ví dụ cập nhật:**

```dart
// Màn hình 1: HomeScreen (Phần body)
ElevatedButton(
  child: const Text('Xem chi tiết của "Sản phẩm A"'),
  onPressed: () {
    Navigator.push(
      context,
      MaterialPageRoute(
        // Truyền dữ liệu qua constructor
        builder: (context) => const DetailScreen(productName: 'Sản phẩm A'),
      ),
    );
  },
)

// Màn hình 2: DetailScreen (Cập nhật)
class DetailScreen extends StatelessWidget {
  // 1. Thêm thuộc tính để lưu dữ liệu
  final String productName;

  // 2. Cập nhật constructor để nhận dữ liệu
  const DetailScreen({super.key, required this.productName});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(productName)), // Hiển thị dữ liệu
      body: Center(
        child: Text('Đây là trang chi tiết của: $productName'),
      ),
    );
  }
}
```

#### b. Gửi dữ liệu trở lại màn hình trước (Ví dụ: DetailScreen → HomeScreen)

`Navigator.push` trả về một `Future`, sẽ hoàn thành khi màn hình mới được `pop`. Bạn có thể gửi dữ liệu về thông qua tham số của `Navigator.pop`.

1.  Trong `DetailScreen`, khi gọi `Navigator.pop`, hãy truyền dữ liệu bạn muốn gửi về.
2.  Trong `HomeScreen`, khi gọi `Navigator.push`, hãy dùng `await` để chờ kết quả trả về.

**Ví dụ:** Màn hình lựa chọn trả về một giá trị.

```dart
// Màn hình 1: HomeScreen
class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  String _selection = 'Chưa có lựa chọn';

  // Hàm để điều hướng và nhận kết quả
  void _navigateAndGetSelection(BuildContext context) async {
    // Dùng await để chờ kết quả từ SelectionScreen
    final result = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => const SelectionScreen()),
    );

    // Cập nhật UI với kết quả nhận được
    if (result != null) {
      setState(() {
        _selection = result;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Màn hình chính')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Lựa chọn của bạn: $_selection'),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => _navigateAndGetSelection(context),
              child: const Text('Mở màn hình lựa chọn'),
            ),
          ],
        ),
      ),
    );
  }
}

// Màn hình 2: SelectionScreen
class SelectionScreen extends StatelessWidget {
  const SelectionScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chọn một tùy chọn')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            ElevatedButton(
              child: const Text('Tùy chọn 1'),
              onPressed: () {
                // Gửi "Tùy chọn 1" về màn hình trước
                Navigator.pop(context, 'Tùy chọn 1');
              },
            ),
            ElevatedButton(
              child: const Text('Tùy chọn 2'),
              onPressed: () {
                // Gửi "Tùy chọn 2" về màn hình trước
                Navigator.pop(context, 'Tùy chọn 2');
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 3. Điều hướng bằng tên (Named Routes)

Khi ứng dụng lớn hơn, việc gọi `MaterialPageRoute` ở khắp nơi sẽ trở nên lộn xộn. Named Routes giúp bạn quản lý các route một cách tập trung.

1.  **Định nghĩa các route:** Trong `MaterialApp`, khai báo thuộc tính `routes`. Đây là một `Map<String, WidgetBuilder>`.
2.  **Điều hướng:** Sử dụng `Navigator.pushNamed(context, '/routeName')`.

**Ví dụ:**

```dart
void main() {
  runApp(MaterialApp(
    title: 'Named Routes Demo',
    // 1. Khai báo route
    initialRoute: '/', // Route mặc định khi mở app
    routes: {
      '/': (context) => const HomeScreen(),
      '/details': (context) => const DetailScreen(),
    },
  ));
}

// Trong HomeScreen
ElevatedButton(
  child: const Text('Đi tới màn hình chi tiết (bằng tên)'),
  onPressed: () {
    // 2. Điều hướng bằng tên
    Navigator.pushNamed(context, '/details');
  },
)
```

**Truyền dữ liệu với Named Routes:**
Bạn có thể truyền dữ liệu qua thuộc tính `arguments` của `pushNamed`.

```dart
// Gửi dữ liệu
Navigator.pushNamed(
  context,
  '/details',
  arguments: 'Dữ liệu từ Named Route',
);

// Nhận dữ liệu trong DetailScreen
@override
Widget build(BuildContext context) {
  // Lấy arguments ra
  final args = ModalRoute.of(context)!.settings.arguments as String;

  return Scaffold(
    appBar: AppBar(title: const Text('Màn hình chi tiết')),
    body: Center(
      child: Text(args),
    ),
  );
}
```

### Tổng kết

| Phương thức | Chức năng | Khi nào dùng |
| :--- | :--- | :--- |
| **`Navigator.push()`** | Mở một màn hình mới. | Hầu hết các trường hợp điều hướng đơn giản. |
| **`Navigator.pop()`** | Đóng màn hình hiện tại, quay lại. | Khi muốn quay lại hoặc đóng một dialog/modal. |
| **`Navigator.pushNamed()`** | Mở một màn hình mới bằng tên đã đăng ký. | Trong các ứng dụng lớn để quản lý route tập trung, dễ bảo trì. |
| **`Navigator.pushReplacementNamed()`** | Thay thế màn hình hiện tại bằng một màn hình mới (không thể quay lại). | Dùng cho các luồng như Splash Screen → Home Screen, hoặc Login → Home. |
| **`Navigator.popAndPushNamed()`** | Pop màn hình hiện tại và push một màn hình mới trong một thao tác. | Tối ưu hơn so với việc gọi `pop()` rồi `pushNamed()` riêng lẻ. |

**Lưu ý về Navigator 2.0 (Router API):**
Những gì được giải thích ở trên thuộc về **Navigator 1.0**. Flutter còn có một API điều hướng mạnh mẽ hơn là **Navigator 2.0 (Router API)**, được thiết kế để xử lý các kịch bản phức tạp như deep linking, điều hướng lồng nhau, và đồng bộ hóa với thanh địa chỉ của trình duyệt web. Tuy nhiên, nó phức tạp hơn đáng kể. Đối với hầu hết các ứng dụng, Navigator 1.0 là đủ và dễ sử dụng hơn rất nhiều.

Hy vọng giải thích chi tiết này giúp bạn hiểu rõ và sử dụng thành thạo `Navigator` trong Flutter

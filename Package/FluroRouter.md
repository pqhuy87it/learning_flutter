Chào bạn,

`FluroRouter` không phải là một "từ khóa" (keyword) trong ngôn ngữ Dart, mà nó là **lớp (class) chính** của package `fluro` – một thư viện định tuyến (routing) rất phổ biến và mạnh mẽ cho Flutter.

Mục đích của Fluro là giúp bạn quản lý việc điều hướng giữa các màn hình trong ứng dụng một cách gọn gàng, có cấu trúc, và dựa trên URL, tương tự như cách hoạt động của các trang web.

Hãy cùng phân tích chi tiết cách sử dụng nó nhé.

### Fluro là gì và tại sao nên dùng?

Trong Flutter, cách điều hướng mặc định là dùng `Navigator.push(context, MaterialPageRoute(...))`. Cách này hoạt động tốt nhưng có một số nhược điểm:
*   **Code dài dòng**: Phải import file của màn hình đích ở mọi nơi bạn muốn điều hướng đến.
*   **Khó quản lý**: Khi ứng dụng lớn lên, việc quản lý tất cả các `MaterialPageRoute` trở nên phức tạp.
*   **Không thân thiện với URL**: Khó xử lý các tác vụ như deep linking (mở ứng dụng từ một link bên ngoài).

**Fluro giải quyết các vấn đề này bằng cách:**
1.  **Tập trung hóa việc định nghĩa route**: Tất cả các route của bạn được định nghĩa ở một nơi duy nhất.
2.  **Điều hướng bằng URL**: Bạn điều hướng bằng một chuỗi string (ví dụ: `/users/123`) thay vì một đối tượng `Widget`.
3.  **Hỗ trợ tham số (parameters)**: Dễ dàng truyền dữ liệu qua URL (ví dụ: lấy `id=123` từ `/users/123`).
4.  **Tùy chỉnh luồng chuyển trang (transitions)**: Dễ dàng thêm các hiệu ứng chuyển cảnh (fade, slide, v.v.).

---

### Hướng Dẫn Sử Dụng Chi Tiết

#### Bước 1: Thêm `fluro` vào `pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter
  fluro: ^2.0.5 # Kiểm tra phiên bản mới nhất trên pub.dev
```
Sau đó chạy `flutter pub get` trong terminal.

#### Bước 2: Tạo và Cấu hình Router

Cách tốt nhất là tạo một file riêng để quản lý router, ví dụ `lib/config/router.dart`. Trong file này, chúng ta sẽ tạo một instance duy nhất của `FluroRouter` và định nghĩa các route.

```dart
// lib/config/router.dart

import 'package:fluro/fluro.dart';
import 'package:flutter/material.dart';
import 'package:my_app/screens/home_screen.dart';
import 'package:my_app/screens/user_details_screen.dart';
import 'package:my_app/screens/not_found_screen.dart';

class AppRouter {
  static final FluroRouter router = FluroRouter();

  // Định nghĩa handler cho màn hình không tìm thấy
  static final Handler _notFoundHandler = Handler(
    handlerFunc: (BuildContext? context, Map<String, List<String>> params) {
      return const NotFoundScreen();
    },
  );

  // Định nghĩa handler cho màn hình chính
  static final Handler _homeHandler = Handler(
    handlerFunc: (BuildContext? context, Map<String, List<String>> params) {
      return const HomeScreen();
    },
  );

  // Định nghĩa handler cho màn hình chi tiết người dùng (có tham số)
  static final Handler _userDetailsHandler = Handler(
    handlerFunc: (BuildContext? context, Map<String, List<String>> params) {
      // Lấy 'id' từ URL. params['id'] là một List<String>, ta lấy phần tử đầu tiên.
      final String? userId = params['id']?.first;
      return UserDetailsScreen(userId: userId ?? '0');
    },
  );

  // Hàm setup các route
  static void setupRouter() {
    router.notFoundHandler = _notFoundHandler; // Gán handler khi không tìm thấy route

    // Định nghĩa các route
    router.define('/', handler: _homeHandler, transitionType: TransitionType.fadeIn);
    router.define(
      '/users/:id', // :id là một tham số động
      handler: _userDetailsHandler,
      transitionType: TransitionType.cupertino,
    );
  }
}
```

**Giải thích:**
*   **`FluroRouter router = FluroRouter()`**: Tạo một instance của router. Ta dùng `static` để có thể truy cập nó từ bất kỳ đâu trong ứng dụng mà không cần tạo đối tượng mới.
*   **`Handler`**: Là một đối tượng của Fluro, nó quyết định `Widget` nào sẽ được tạo ra khi một route được gọi. `handlerFunc` là hàm thực thi việc đó.
*   **`params`**: Là một `Map` chứa các tham số được truyền qua URL. Ví dụ, khi bạn điều hướng đến `/users/123`, `params` sẽ là `{'id': ['123']}`.
*   **`router.define()`**: Hàm dùng để "đăng ký" một route. Bạn cung cấp một chuỗi URL, một `Handler`, và có thể tùy chọn `transitionType` (hiệu ứng chuyển cảnh).

#### Bước 3: Tích hợp Router vào `MaterialApp`

Trong file `main.dart`, bạn cần gọi hàm `setupRouter()` và kết nối Fluro với `MaterialApp` thông qua thuộc tính `onGenerateRoute`.

```dart
// lib/main.dart

import 'package:flutter/material.dart';
import 'package:my_app/config/router.dart';

void main() {
  // Gọi hàm setup router trước khi chạy app
  AppRouter.setupRouter();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fluro Demo',
      // Dùng onGenerateRoute của Fluro để xử lý tất cả các yêu cầu điều hướng
      onGenerateRoute: AppRouter.router.generator,
    );
  }
}
```

#### Bước 4: Thực hiện Điều hướng (Navigation)

Bây giờ, từ bất kỳ đâu trong ứng dụng, bạn có thể điều hướng bằng cách sử dụng instance `router` đã tạo.

Ví dụ, trong `HomeScreen`, bạn có một nút để chuyển đến trang chi tiết của người dùng có id là "42".

```dart
// lib/screens/home_screen.dart

import 'package:flutter/material.dart';
import 'package:my_app/config/router.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Điều hướng đến trang chi tiết người dùng với id = 42
            AppRouter.router.navigateTo(context, '/users/42');
          },
          child: const Text('Go to User 42 Details'),
        ),
      ),
    );
  }
}
```

**Giải thích:**
*   `AppRouter.router.navigateTo(context, '/users/42')`:
    *   `context`: Cần thiết để Fluro tìm thấy `Navigator` trong cây widget.
    *   `'/users/42'`: Chuỗi URL bạn muốn điều hướng đến. Fluro sẽ tìm route đã được định nghĩa (`/users/:id`), khớp `id` với `42`, và gọi `_userDetailsHandler` tương ứng.

### Tóm Tắt Luồng Hoạt Động

1.  **Khởi tạo**: Trong `main()`, bạn gọi `AppRouter.setupRouter()` để định nghĩa tất cả các route của ứng dụng.
2.  **Tích hợp**: Bạn gán `AppRouter.router.generator` cho `onGenerateRoute` của `MaterialApp`. Từ giờ, mỗi khi `Navigator.pushNamed` hoặc một lệnh điều hướng được gọi, Flutter sẽ hỏi Fluro cách xử lý.
3.  **Điều hướng**: Khi bạn gọi `AppRouter.router.navigateTo(context, '/some/path')`.
4.  **Xử lý**: Fluro sẽ tìm trong danh sách các route đã `define` xem có route nào khớp với `/some/path` không.
5.  **Tạo Widget**: Nếu tìm thấy, Fluro sẽ thực thi `handlerFunc` của `Handler` tương ứng để tạo ra Widget cho màn hình mới.
6.  **Push**: Cuối cùng, Fluro sẽ dùng `Navigator` của Flutter để push màn hình mới lên stack với hiệu ứng chuyển cảnh đã được định nghĩa.

Sử dụng `FluroRouter` giúp code điều hướng của bạn trở nên sạch sẽ, dễ quản lý và mở rộng hơn rất nhiều, đặc biệt là trong các dự án lớn.

Chắc chắn rồi! Để thêm chức năng kiểm tra bảo trì (maintenance) vào `GoRouter` của bạn, chúng ta cần thực hiện 3 bước chính:

1.  **Tạo một Provider để quản lý trạng thái bảo trì:** Tương tự như `authProvider`, chúng ta cần một "công tắc" để bật/tắt chế độ bảo trì cho toàn ứng dụng.
2.  **Tạo một Route và một Màn hình cho trang bảo trì:** Đây là trang sẽ hiển thị thông báo "We'll be back" khi hệ thống đang bảo trì.
3.  **Cập nhật logic trong hàm `redirect`:** Thêm quy tắc kiểm tra trạng thái bảo trì *trước* tất cả các quy tắc khác.

Dưới đây là đoạn code hoàn chỉnh đã được cập nhật. Tôi sẽ giải thích chi tiết từng thay đổi ở ngay bên dưới.

### Code hoàn chỉnh

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

// --- Các màn hình giả lập ---
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(appBar: AppBar(title: Text('Home')));
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(appBar: AppBar(title: Text('Profile')));
}

class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(appBar: AppBar(title: Text('Login')));
}

class RegisterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(appBar: AppBar(title: Text('Register')));
}

// Màn hình mới cho chế độ bảo trì
class MaintenanceScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text(
          "We'll be back soon!",
          style: Theme.of(context).textTheme.headlineMedium,
        ),
      ),
    );
  }
}

// --- Các Provider ---

// 1. Provider quản lý trạng thái đăng nhập
final authProvider = StateProvider<bool>((ref) => false);

// 2. (MỚI) Provider quản lý trạng thái bảo trì
// Đặt là `true` để kiểm tra chế độ bảo trì
final maintenanceProvider = StateProvider<bool>((ref) => false);

// --- Cấu hình GoRouter ---

// (MỚI) Tạo một Notifier để lắng nghe nhiều provider và trigger GoRouter refresh
class GoRouterRefreshNotifier extends ChangeNotifier {
  GoRouterRefreshNotifier(Ref ref) {
    // Lắng nghe cả hai provider. Khi một trong hai thay đổi, notifyListeners() sẽ được gọi.
    ref.listen(authProvider, (_, __) => notifyListeners());
    ref.listen(maintenanceProvider, (_, __) => notifyListeners());
  }
}

// Provider cho GoRouter
final goRouterProvider = Provider<GoRouter>((ref) {
  // Tạo notifier để lắng nghe thay đổi
  final refreshNotifier = GoRouterRefreshNotifier(ref);

  return GoRouter(
    // Theo dõi notifier để kích hoạt redirect lại khi trạng thái thay đổi
    refreshListenable: refreshNotifier,

    redirect: (BuildContext context, GoRouterState state) {
      // Đọc trạng thái bảo trì và đăng nhập
      final bool isMaintenance = ref.read(maintenanceProvider);
      final bool isLoggedIn = ref.read(authProvider);
      final String location = state.location;

      // --- BƯỚC 1: KIỂM TRA BẢO TRÌ (ƯU TIÊN CAO NHẤT) ---
      // Nếu hệ thống đang bảo trì
      if (isMaintenance) {
        // Và người dùng chưa ở trang bảo trì -> chuyển hướng họ đến đó
        if (location != '/maintenance') {
          return '/maintenance';
        }
        // Nếu đã ở trang bảo trì rồi thì không làm gì cả (return null) để tránh lặp vô hạn
      } else {
        // Nếu người dùng đang ở trang bảo trì nhưng hệ thống đã hết bảo trì
        // -> chuyển họ về trang chủ
        if (location == '/maintenance') {
          return '/home';
        }
      }

      // --- BƯỚC 2: KIỂM TRA XÁC THỰC (Logic cũ của bạn) ---
      final bool isGoingToLogin = location == '/login';
      final bool isGoingToRegister = location == '/register';

      // Kịch bản 1: Người dùng chưa đăng nhập VÀ họ đang cố vào một trang cần bảo vệ
      if (!isLoggedIn && !isGoingToLogin && !isGoingToRegister) {
        return '/login';
      }

      // Kịch bản 2: Người dùng đã đăng nhập VÀ họ lại cố vào trang đăng nhập/đăng ký
      if (isLoggedIn && (isGoingToLogin || isGoingToRegister)) {
        return '/home';
      }

      // Kịch bản 3: Các trường hợp còn lại -> Cho phép đi tiếp
      return null;
    },

    routes: [
      GoRoute(path: '/', redirect: (_, __) => '/home'), // Chuyển từ / sang /home
      GoRoute(path: '/home', builder: (context, state) => HomeScreen()),
      GoRoute(path: '/profile', builder: (context, state) => ProfileScreen()),
      GoRoute(path: '/login', builder: (context, state) => LoginScreen()),
      GoRoute(path: '/register', builder: (context, state) => RegisterScreen()),
      // (MỚI) Thêm route cho màn hình bảo trì
      GoRoute(path: '/maintenance', builder: (context, state) => MaintenanceScreen()),
    ],
  );
});
```

### Phân tích các thay đổi

#### 1. Provider `maintenanceProvider`

```dart
final maintenanceProvider = StateProvider<bool>((ref) => false);
```

*   Chúng ta tạo một `StateProvider` mới tên là `maintenanceProvider`.
*   Nó chứa một giá trị boolean: `true` nghĩa là hệ thống đang bảo trì, `false` là đang hoạt động bình thường.
*   Bạn có thể thay đổi giá trị này từ bất kỳ đâu trong ứng dụng (ví dụ: `ref.read(maintenanceProvider.notifier).state = true;`) hoặc lấy giá trị này từ một API cấu hình từ xa.

#### 2. Route và Màn hình `/maintenance`

```dart
// Màn hình
class MaintenanceScreen extends StatelessWidget { ... }

// Route
GoRoute(
  path: '/maintenance',
  builder: (context, state) => MaintenanceScreen(),
),
```

*   Chúng ta định nghĩa một màn hình đơn giản `MaintenanceScreen` chỉ để hiển thị thông báo.
*   Quan trọng là phải đăng ký một `GoRoute` mới với `path: '/maintenance'` để `GoRouter` biết phải hiển thị màn hình nào khi chúng ta chuyển hướng đến đây.

#### 3. Cập nhật `redirect` và `refreshListenable`

Đây là phần quan trọng nhất.

*   **`GoRouterRefreshNotifier`**:
    `refreshListenable` chỉ có thể nhận một `Listenable`. Vì bây giờ chúng ta cần lắng nghe sự thay đổi của cả `authProvider` và `maintenanceProvider`, chúng ta tạo một lớp `ChangeNotifier` tùy chỉnh. Lớp này sẽ `listen` đến cả hai provider và gọi `notifyListeners()` khi một trong hai thay đổi, từ đó trigger `GoRouter` chạy lại hàm `redirect`. Đây là một pattern rất hiệu quả khi cần theo dõi nhiều trạng thái.

*   **Logic trong `redirect`**:
    ```dart
    redirect: (BuildContext context, GoRouterState state) {
      final bool isMaintenance = ref.read(maintenanceProvider);
      // ...

      // ƯU TIÊN KIỂM TRA BẢO TRÌ TRƯỚC TIÊN
      if (isMaintenance) {
        // Chỉ chuyển hướng nếu người dùng CHƯA ở trang bảo trì
        if (state.location != '/maintenance') {
          return '/maintenance';
        }
      }
      // ... logic xác thực cũ sẽ chạy sau nếu không bảo trì
    }
    ```
    *   **Ưu tiên hàng đầu**: Việc kiểm tra bảo trì được đặt lên **đầu tiên** trong hàm `redirect`. Điều này đảm bảo rằng nếu hệ thống đang bảo trì, không có logic nào khác (như kiểm tra đăng nhập) được thực thi. Mọi người dùng, dù đã đăng nhập hay chưa, đều sẽ bị chuyển hướng.
    *   **Tránh vòng lặp vô hạn**: Dòng `if (state.location != '/maintenance')` là cực kỳ quan trọng. Nó đảm bảo rằng nếu người dùng đã ở trang `/maintenance` rồi, chúng ta sẽ không chuyển hướng họ đến chính trang đó một lần nữa, tránh gây ra lỗi lặp chuyển hướng vô hạn.
    *   **Thoát khỏi chế độ bảo trì**: Tôi cũng đã thêm một đoạn logic nhỏ để xử lý trường hợp khi chế độ bảo trì vừa kết thúc. Nếu người dùng đang ở trang `/maintenance` và `isMaintenance` chuyển thành `false`, họ sẽ tự động được chuyển hướng về trang chủ (`/home`).

Bằng cách thêm các đoạn code trên, hệ thống điều hướng của bạn giờ đây đã có thêm một lớp kiểm tra bảo trì mạnh mẽ và linh hoạt.

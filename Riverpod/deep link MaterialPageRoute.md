Vấn đề cốt lõi khi bạn dùng `MaterialPageRoute` theo cách thông thường (Anonymous Route) là thuộc tính `name` của nó mặc định là `null`. Do đó, `NavigationService` (vốn là một `NavigatorObserver`) sẽ không biết bạn đang ở đâu.

Để giải quyết vấn đề này mà vẫn dùng `MaterialPageRoute`, bạn bắt buộc phải gán **`RouteSettings`** khi thực hiện `push`.

Dưới đây là giải pháp chi tiết:

### 1\. Cập nhật `NavigationService` (Observer)

Service này cần lắng nghe sự thay đổi của stack và cập nhật tên màn hình hiện tại.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final navigationServiceProvider = Provider((ref) => NavigationService());

class NavigationService extends NavigatorObserver {
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();
  
  String? _currentRoute;

  // Getter để kiểm tra xem có đang ở màn hình E hay không
  // Lưu ý: So sánh với cái tên mà bạn sẽ đặt ở bước 2
  bool get isAtScreenE => _currentRoute == '/screen_e';

  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPush(route, previousRoute);
    _updateRouteName(route);
  }

  @override
  void didPop(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPop(route, previousRoute);
    // Khi pop màn hình hiện tại ra, màn hình previous trở thành hiện tại
    if (previousRoute != null) {
      _updateRouteName(previousRoute);
    }
  }

  @override
  void didReplace({Route<dynamic>? newRoute, Route<dynamic>? oldRoute}) {
    super.didReplace(newRoute: newRoute, oldRoute: oldRoute);
    if (newRoute != null) {
      _updateRouteName(newRoute);
    }
  }

  void _updateRouteName(Route<dynamic> route) {
    // Chỉ cập nhật nếu route có tên (để tránh các dialog/popup làm nhiễu)
    if (route.settings.name != null) {
      _currentRoute = route.settings.name;
      print("📍 Current Screen: $_currentRoute");
    }
  }
  
  // ... Các hàm xử lý navigateToE ...
}
```

### 2\. Cách `push` màn hình (Quan trọng nhất)

Khi bạn di chuyển từ D sang E (hoặc bất kỳ màn hình nào), bạn **phải** thêm tham số `settings` vào `MaterialPageRoute`.

**Tại màn hình D (ScreenD):**

```dart
ElevatedButton(
  onPressed: () {
    Navigator.of(context).push(
      MaterialPageRoute(
        builder: (context) => const ScreenE(),
        // ⚠️ ĐÂY LÀ CHÌA KHÓA: Đặt tên cho Route
        settings: const RouteSettings(name: '/screen_e'), 
      ),
    );
  },
  child: const Text("Go to Screen E"),
);
```

Tương tự cho các màn hình khác nếu bạn muốn theo dõi chúng:

  * A -\> B: `settings: RouteSettings(name: '/screen_b')`
  * B -\> C: `settings: RouteSettings(name: '/screen_c')`
  * ...

### 3\. (Nâng cao) Cách quản lý tập trung để đảm bảo SOLID

Việc viết `settings: RouteSettings(name: '/screen_e')` rải rác ở khắp nơi (Màn hình A, B, C...) dễ gây lỗi chính tả (Typo) và khó bảo trì.

Để tuân thủ SOLID (Single Responsibility), bạn nên tạo một class `AppRouter` hoặc factory để tạo Route.

**`app_router.dart`**

```dart
class AppRouter {
  static const String screenA = '/';
  static const String screenB = '/screen_b';
  static const String screenC = '/screen_c';
  static const String screenD = '/screen_d';
  static const String screenE = '/screen_e';

  // Factory method để tạo Route chuẩn
  static Route<dynamic> createRoute(Widget screen, String routeName) {
    return MaterialPageRoute(
      builder: (_) => screen,
      settings: RouteSettings(name: routeName),
    );
  }
}
```

**Sử dụng trong UI:**

```dart
// Tại màn hình D
Navigator.of(context).push(
  AppRouter.createRoute(const ScreenE(), AppRouter.screenE),
);
```

### Tóm tắt

Để `NavigationService` nhận biết được màn hình E khi dùng `MaterialPageRoute`:

1.  **Observer:** `NavigationService` phải override `didPush` và `didPop` để đọc `route.settings.name`.
2.  **Implementation:** Khi gọi `Navigator.push`, bắt buộc phải truyền `RouteSettings(name: '/screen_e')` vào bên trong `MaterialPageRoute`.

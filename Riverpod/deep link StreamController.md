Việc chuyển từ `GlobalKey` sang dùng `BuildContext` trong kiến trúc tách biệt (như khi dùng Riverpod/Controller) đòi hỏi chúng ta phải thay đổi tư duy:

  * **Cách cũ (GlobalKey):** Controller tự tay điều khiển: *"Tôi ra lệnh Navigator chuyển trang ngay lập tức"*.
  * **Cách mới (Context):** Controller phát tín hiệu: *"Tôi muốn chuyển trang, ai có Context thì thực hiện giúp tôi"*.

Chúng ta sẽ sử dụng mô hình **Event-Driven (Hướng sự kiện)**. `NavigationService` sẽ phát ra các sự kiện (Events), và UI (nơi có `context`) sẽ lắng nghe và thực hiện lệnh `Navigator`.

Dưới đây là cách triển khai chi tiết tuân thủ SOLID:

### 1\. Định nghĩa Navigation Events

Đầu tiên, tạo các sự kiện để Service giao tiếp với UI.

```dart
// navigation_events.dart
abstract class NavigationEvent {}

class NavigateToEScreenWithStackEvent extends NavigationEvent {
  final String data;
  NavigateToEScreenWithStackEvent(this.data);
}

// Có thể thêm các event khác sau này
// class NavigateBackEvent extends NavigationEvent {}
```

### 2\. Cập nhật `NavigationService`

Class này không còn giữ `GlobalKey` nữa. Thay vào đó, nó chứa một `Stream` để bắn tín hiệu điều hướng. Nó vẫn giữ `RouteObserver` để biết mình đang ở đâu.

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final navigationServiceProvider = Provider((ref) => NavigationService());

class NavigationService extends NavigatorObserver {
  // Stream để bắn sự kiện ra ngoài UI
  final _eventController = StreamController<NavigationEvent>.broadcast();
  Stream<NavigationEvent> get eventStream => _eventController.stream;

  // Theo dõi màn hình hiện tại
  String? _currentRoute;
  bool get isAtScreenE => _currentRoute == '/screen_e';

  // --- Logic theo dõi Route (Giữ nguyên) ---
  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPush(route, previousRoute);
    _currentRoute = route.settings.name;
  }

  @override
  void didPop(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPop(route, previousRoute);
    _currentRoute = previousRoute?.settings.name;
  }

  // --- Thay vì gọi Navigator trực tiếp, ta bắn Event ---
  void requestNavigateToE(String data) {
    _eventController.add(NavigateToEScreenWithStackEvent(data));
  }
  
  void dispose() {
    _eventController.close();
  }
}
```

### 3\. Cập nhật `DeepLinkController`

Controller vẫn giữ nguyên logic nghiệp vụ, nhưng thay vì gọi hàm thực thi điều hướng, nó gọi hàm yêu cầu (`request...`).

```dart
final deepLinkControllerProvider = Provider<void>((ref) {
  final repo = ref.watch(deepLinkRepositoryProvider);
  final navService = ref.watch(navigationServiceProvider);
  final screenENotifier = ref.watch(screenEProvider.notifier);

  repo.uriStream.listen((uri) {
    if (uri.path.contains('/e')) {
      final data = uri.queryParameters['data'] ?? '';

      if (navService.isAtScreenE) {
        // Logic: Đang ở E -> Update State (Không cần context)
        print("Đang ở E, update state riverpod");
        screenENotifier.updateData(data);
      } else {
        // Logic: Không ở E -> Yêu cầu UI chuyển trang
        print("Không ở E, yêu cầu chuyển hướng");
        navService.requestNavigateToE(data);
      }
    }
  });
});
```

### 4\. Tạo Widget Lắng nghe Điều hướng (`NavigationListener`)

Đây là thành phần quan trọng nhất trong cách tiếp cận dùng `Context`. Widget này sẽ nằm ngay dưới `MaterialApp`, nó có `BuildContext` và sẽ lắng nghe `NavigationService`.

Đây là nơi thực hiện logic **Tái tạo Stack (Stack Recursion)**.

```dart
class NavigationListener extends ConsumerStatefulWidget {
  final Widget child;
  const NavigationListener({super.key, required this.child});

  @override
  ConsumerState<NavigationListener> createState() => _NavigationListenerState();
}

class _NavigationListenerState extends ConsumerState<NavigationListener> {
  @override
  void initState() {
    super.initState();
    
    // Lắng nghe stream sự kiện từ Service
    ref.read(navigationServiceProvider).eventStream.listen((event) {
      if (event is NavigateToEScreenWithStackEvent) {
        _handleNavigateToE(event.data);
      }
    });
  }

  void _handleNavigateToE(String data) {
    // Ở ĐÂY CHÚNG TA CÓ CONTEXT!
    final nav = Navigator.of(context);

    // Logic tái tạo Stack: A -> B -> C -> D -> E
    // 1. Xóa hết về màn hình gốc (A)
    nav.popUntil((route) => route.isFirst);

    // 2. Đẩy các màn hình trung gian (tắt animation để mượt hơn nếu muốn)
    nav.pushNamed('/screen_b');
    nav.pushNamed('/screen_c');
    nav.pushNamed('/screen_d');

    // 3. Đẩy màn hình đích E với dữ liệu
    nav.pushNamed('/screen_e', arguments: data);
  }

  @override
  Widget build(BuildContext context) {
    return widget.child;
  }
}
```

### 5\. Tích hợp vào `main.dart`

Bạn cần bọc ứng dụng (hoặc `MaterialApp` builder) bằng `NavigationListener`. Lưu ý là `NavigationListener` cần nằm **bên dưới** `Navigator` để có thể tìm thấy nó, nhưng vì chúng ta dùng `pushNamed` trên `MaterialApp` nên chúng ta thường đặt listener trong `builder` của `MaterialApp`.

```dart
void main() {
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends ConsumerStatefulWidget {
  const MyApp({super.key});
  @override
  ConsumerState<MyApp> createState() => _MyAppState();
}

class _MyAppState extends ConsumerState<MyApp> {
  @override
  void initState() {
    super.initState();
    // Khởi tạo Controller logic
    ref.read(deepLinkControllerProvider);
  }

  @override
  Widget build(BuildContext context) {
    final navService = ref.watch(navigationServiceProvider);

    return MaterialApp(
      // Gán observer để Service biết đang ở màn hình nào
      navigatorObservers: [navService], 
      initialRoute: '/',
      routes: {
        '/': (context) => const ScreenA(),
        '/screen_b': (context) => const ScreenB(),
        '/screen_c': (context) => const ScreenC(),
        '/screen_d': (context) => const ScreenD(),
        '/screen_e': (context) => const ScreenE(),
      },
      // QUAN TRỌNG: Sử dụng builder để chèn NavigationListener
      // Listener này sẽ có context chứa Navigator do MaterialApp tạo ra
      builder: (context, child) {
        return NavigationListener(
          child: child ?? const SizedBox(),
        );
      },
    );
  }
}
```

### Tóm tắt thay đổi

1.  **`NavigationService`**: Bỏ `GlobalKey`. Thêm `Stream<NavigationEvent>`. Chuyển hàm `MapsToE` thành `requestNavigateToE` (chỉ bắn event, không thực thi).
2.  **`DeepLinkController`**: Không thay đổi logic, chỉ thay đổi cách gọi service.
3.  **`NavigationListener` (Mới)**: Class này chịu trách nhiệm thực thi `Navigator.of(context)` khi nhận tín hiệu từ Service. Đây là nơi duy nhất thao tác trực tiếp với UI Context để chuyển trang.

Cách này đảm bảo **SOLID**:

  * **Controller/Service** không phụ thuộc vào UI (Context/Widget).
  * **UI** chịu trách nhiệm thực hiện các thay đổi về giao diện (Navigation).
  * Logic vẫn được giữ tập trung, không phân tán.

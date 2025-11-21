Chào bạn, yêu cầu này rất thú vị vì nó đòi hỏi sự kết hợp giữa kiến trúc phần mềm (SOLID), quản lý trạng thái hiện đại (Riverpod) và xử lý bất đồng bộ (Streams).

Để giải quyết bài toán này mà **không viết code lặp lại ở các màn hình** và tuân thủ **SOLID**, chúng ta sẽ tách biệt rõ ràng các trách nhiệm:

1.  **Data Layer:** Lắng nghe sự kiện từ `app_links` (Repository).
2.  **State Layer:** Quản lý dữ liệu của màn hình E (State).
3.  **Navigation Layer:** Theo dõi màn hình hiện tại và điều hướng (Service).
4.  **Logic Layer:** Điều phối giữa 3 lớp trên (Controller).

Dưới đây là giải pháp chi tiết.

### Kiến trúc tổng quan

  * `NavigatorService`: Chịu trách nhiệm điều hướng và biết mình đang ở đâu (GlobalKey + Observer).
  * `DeepLinkRepository`: Chịu trách nhiệm bọc thư viện `app_links` (Single Responsibility).
  * `ScreenEViewModel`: Quản lý state hiển thị của màn hình E.
  * `DeepLinkController`: "Bộ não" trung tâm, lắng nghe Repository và quyết định làm gì dựa trên NavigatorService.

-----

### 1\. Navigation Service (Theo dõi Route & Điều hướng)

Chúng ta cần biết người dùng đang ở đâu mà không cần truyền `context` lung tung.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Provider cho NavigationService
final navigationServiceProvider = Provider((ref) => NavigationService());

class NavigationService extends NavigatorObserver {
  // GlobalKey để điều hướng không cần Context
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();
  
  // Biến lưu tên màn hình hiện tại
  String? _currentRoute;

  bool get isAtScreenE => _currentRoute == '/screen_e';

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

  // Hàm dựng lại Stack: Reset về A -> B -> C -> D -> E
  void navigateToEWithStackRecursion(String data) {
    final nav = navigatorKey.currentState;
    if (nav == null) return;

    // 1. Xóa hết về màn hình A (Route đầu tiên)
    nav.popUntil((route) => route.isFirst);

    // 2. Đẩy lần lượt B, C, D vào
    nav.pushNamed('/screen_b');
    nav.pushNamed('/screen_c');
    nav.pushNamed('/screen_d');
    
    // 3. Đẩy E vào với data
    nav.pushNamed('/screen_e', arguments: data);
  }
}
```

### 2\. Deep Link Repository (Lớp dữ liệu)

Lớp này chỉ có nhiệm vụ duy nhất: cung cấp Stream của các link. (Tuân thủ SRP).

```dart
import 'package:app_links/app_links.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final deepLinkRepositoryProvider = Provider((ref) => DeepLinkRepository());

class DeepLinkRepository {
  final AppLinks _appLinks = AppLinks();

  Stream<Uri> get uriStream => _appLinks.uriLinkStream;
}
```

### 3\. Screen E State (Quản lý trạng thái màn E)

Chúng ta dùng `StateNotifier` để quản lý dữ liệu của màn hình E. Điều này giúp ta cập nhật UI E từ bất kỳ đâu mà không cần `setState` trong UI.

```dart
class ScreenEState {
  final String data;
  ScreenEState(this.data);
}

class ScreenENotifier extends StateNotifier<ScreenEState> {
  ScreenENotifier() : super(ScreenEState("Chưa có dữ liệu"));

  // Hàm này sẽ được gọi khi nhận DeepLink và đang ở màn E
  void updateData(String newData) {
    state = ScreenEState(newData);
  }
}

final screenEProvider = StateNotifierProvider<ScreenENotifier, ScreenEState>((ref) {
  return ScreenENotifier();
});
```

### 4\. Deep Link Controller (Lớp Logic trung tâm)

Đây là nơi phép màu xảy ra. Nó kết nối các thành phần trên. Đây là nơi duy nhất xử lý logic Deep Link (Centralized).

```dart
final deepLinkControllerProvider = Provider<void>((ref) {
  final repo = ref.watch(deepLinkRepositoryProvider);
  final navService = ref.watch(navigationServiceProvider);
  final screenENotifier = ref.watch(screenEProvider.notifier);

  // Lắng nghe Stream
  repo.uriStream.listen((uri) {
    // Giả sử link: myapp://example.com/e?data=HELLO
    if (uri.path.contains('/e')) {
      final data = uri.queryParameters['data'] ?? '';

      if (navService.isAtScreenE) {
        // CASE 1: Đang ở E -> Update State trực tiếp
        print("Đang ở E, update state riverpod");
        screenENotifier.updateData(data);
      } else {
        // CASE 2: Ở chỗ khác -> Dựng lại stack và chuyển trang
        print("Không ở E, dựng lại stack");
        navService.navigateToEWithStackRecursion(data);
      }
    }
  });
});
```

### 5\. Tích hợp vào UI (Main & Screens)

**`main.dart`**

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
    // Kích hoạt Controller lắng nghe ngay khi App khởi động
    // Chúng ta đọc nó 1 lần để khởi tạo listener
    ref.read(deepLinkControllerProvider);
  }

  @override
  Widget build(BuildContext context) {
    // Lấy NavigationService để gán Key và Observer
    final navService = ref.watch(navigationServiceProvider);

    return MaterialApp(
      navigatorKey: navService.navigatorKey,
      navigatorObservers: [navService], // Đăng ký observer để theo dõi route
      initialRoute: '/',
      routes: {
        '/': (context) => const ScreenA(),
        '/screen_b': (context) => const ScreenB(),
        '/screen_c': (context) => const ScreenC(),
        '/screen_d': (context) => const ScreenD(),
        '/screen_e': (context) => const ScreenE(),
      },
    );
  }
}
```

**`screen_e.dart`**
Màn hình E giờ đây cực kỳ đơn giản, nó chỉ việc lắng nghe Riverpod provider. Nó không cần biết DeepLink là gì.

```dart
class ScreenE extends ConsumerStatefulWidget {
  const ScreenE({super.key});

  @override
  ConsumerState<ScreenE> createState() => _ScreenEState();
}

class _ScreenEState extends ConsumerState<ScreenE> {
  @override
  void initState() {
    super.initState();
    // Nếu được push tới từ DeepLink (Case 2), lấy arguments để init state
    WidgetsBinding.instance.addPostFrameCallback((_) {
      final args = ModalRoute.of(context)?.settings.arguments;
      if (args is String) {
        ref.read(screenEProvider.notifier).updateData(args);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    // Lắng nghe state thay đổi (Case 1: Update tại chỗ)
    final state = ref.watch(screenEProvider);

    return Scaffold(
      appBar: AppBar(title: const Text("Màn hình E")),
      body: Center(
        child: Text(
          "Dữ liệu: ${state.data}",
          style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
        ),
      ),
    );
  }
}
```

-----

### Giải thích việc ứng dụng SOLID

1.  **S - Single Responsibility Principle (Đơn nhiệm):**

      * `DeepLinkRepository`: Chỉ lo việc lấy link từ thư viện.
      * `NavigationService`: Chỉ lo việc điều hướng và theo dõi vị trí.
      * `ScreenENotifier`: Chỉ lo quản lý dữ liệu hiển thị cho màn hình E.
      * `DeepLinkController`: Chỉ lo logic nghiệp vụ (Business Logic) của deep link.
      * UI (`ScreenE`): Chỉ lo hiển thị, không xử lý logic luồng.

2.  **O - Open/Closed Principle (Đóng/Mở):**

      * Nếu sau này bạn muốn thêm xử lý cho màn hình C, bạn chỉ cần sửa `DeepLinkController` hoặc thêm một Controller khác, không cần sửa code trong `NavigationService` hay `DeepLinkRepository` gốc.

3.  **L - Liskov Substitution Principle (Thay thế Liskov):**

      * `NavigationService` kế thừa từ `NavigatorObserver`. Nó có thể được dùng ở bất cứ đâu `NavigatorObserver` được yêu cầu (trong `MaterialApp`).

4.  **I - Interface Segregation Principle (Phân tách Interface):**

      * Màn hình UI chỉ quan tâm đến `screenEProvider`, nó không cần biết về `deepLinkControllerProvider` hay `navigationServiceProvider`. Các provider được tách biệt rõ ràng.

5.  **D - Dependency Inversion Principle (Đảo ngược phụ thuộc):**

      * `DeepLinkController` không phụ thuộc trực tiếp vào instance cụ thể của Widget hay Context. Nó phụ thuộc vào các abstractions (thông qua Riverpod Provider) như `NavigationService` và `ScreenENotifier`. Việc này giúp code dễ test và bảo trì.

### Kết quả

Với cách làm này, bạn **không cần viết bất kỳ dòng code xử lý Deep Link nào trong các file `ScreenA.dart`, `ScreenB.dart`...**. Tất cả được tập trung tại Controller và được khởi tạo một lần duy nhất tại `main.dart`.

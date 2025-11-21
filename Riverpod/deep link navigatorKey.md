Để xử lý Deep Link tập trung một nơi mà không phải viết code lặp lại ở màn hình A, B, C, D, E, giải pháp tối ưu nhất là sử dụng mô hình **Centralized Service (Service tập trung)** kết hợp với **GlobalKey** và **NavigatorObserver**.

Mô hình này hoạt động như sau:

1.  **DeepLinkService (Singleton):** Một class duy nhất chịu trách nhiệm lắng nghe `app_links` từ lúc app khởi động.
2.  **GlobalKey:** Giúp Service này điều khiển việc chuyển màn hình từ bất kỳ đâu mà không cần `BuildContext`.
3.  **RouteObserver:** Giúp Service biết được **"Hiện tại người dùng đang ở màn hình nào?"** để quyết định nên chuyển trang hay chỉ cập nhật dữ liệu.

Dưới đây là triển khai chi tiết:

### 1\. Tạo `DeepLinkService`

Class này sẽ chứa toàn bộ logic xử lý.

```dart
import 'dart:async';
import 'package:app_links/app_links.dart';
import 'package:flutter/material.dart';

class DeepLinkService {
  // 1. Singleton Pattern
  static final DeepLinkService _instance = DeepLinkService._internal();
  factory DeepLinkService() => _instance;
  DeepLinkService._internal();

  // 2. GlobalKey để điều hướng không cần context
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

  // 3. RouteObserver để theo dõi màn hình hiện tại
  final RouteObserver<PageRoute> routeObserver = RouteObserver<PageRoute>();
  String? _currentRouteName;

  // 4. Stream để bắn dữ liệu sang màn hình E (nếu đang ở E)
  final _dataController = StreamController<String>.broadcast();
  Stream<String> get dataStream => _dataController.stream;

  // Khởi tạo
  void init() {
    final appLinks = AppLinks();
    
    // Lắng nghe Deep Link
    appLinks.uriLinkStream.listen((uri) {
      _handleDeepLink(uri);
    });
  }

  // Cập nhật tên màn hình hiện tại (được gọi từ RouteObserver)
  void setCurrentRoute(String? routeName) {
    _currentRouteName = routeName;
    print("Màn hình hiện tại: $_currentRouteName");
  }

  void _handleDeepLink(Uri uri) {
    // Giả sử link: myapp://example.com/e?data=HELLO
    if (uri.path.contains('/e')) {
      final data = uri.queryParameters['data'] ?? '';

      if (_currentRouteName == '/screen_e') {
        // CASE 1: Đang ở màn hình E -> Chỉ bắn dữ liệu update, không chuyển trang
        print("Đang ở E, bắn data update: $data");
        _dataController.add(data);
      } else {
        // CASE 2: Đang ở A, B, C, D hoặc nơi khác -> Dựng lại stack và sang E
        print("Không ở E, thực hiện chuyển hướng sang E");
        _navigateToE(data);
      }
    }
  }

  void _navigateToE(String data) {
    final nav = navigatorKey.currentState;
    if (nav == null) return;

    // Reset về màn hình gốc (A)
    nav.popUntil((route) => route.isFirst);

    // Đẩy lần lượt B -> C -> D -> E vào stack
    // Dùng Future.microtask hoặc delay nhỏ nếu cần để đảm bảo hiệu ứng
    nav.pushNamed('/screen_b');
    nav.pushNamed('/screen_c');
    nav.pushNamed('/screen_d');
    
    // Đẩy E vào kèm theo tham số
    nav.pushNamed('/screen_e', arguments: data);
  }
  
  void dispose() {
    _dataController.close();
  }
}
```

### 2\. Tạo `CustomRouteObserver`

Chúng ta cần một class nhỏ để "méc" cho Service biết màn hình nào đang hiển thị.

```dart
class CustomRouteObserver extends RouteObserver<PageRoute<dynamic>> {
  final DeepLinkService _service = DeepLinkService();

  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPush(route, previousRoute);
    _service.setCurrentRoute(route.settings.name);
  }

  @override
  void didPop(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPop(route, previousRoute);
    _service.setCurrentRoute(previousRoute?.settings.name);
  }
  
  // Có thể override thêm didReplace nếu cần
}
```

### 3\. Cấu hình trong `main.dart`

Kết nối Service và Observer vào `MaterialApp`.

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  // Khởi tạo Service lắng nghe ngay lập tức
  DeepLinkService().init();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final service = DeepLinkService();

    return MaterialApp(
      title: 'Centralized Deep Link',
      // 1. Gán GlobalKey
      navigatorKey: service.navigatorKey, 
      // 2. Gán Observer
      navigatorObservers: [CustomRouteObserver()], 
      
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

### 4\. Xử lý tại màn hình E (`ScreenE`)

Màn hình E cần xử lý 2 nguồn dữ liệu:

1.  Dữ liệu từ `arguments` (khi được mở mới từ A, B, C, D).
2.  Dữ liệu từ `Stream` (khi đang đứng yên tại E mà nhận link mới).

<!-- end list -->

```dart
class ScreenE extends StatefulWidget {
  const ScreenE({super.key});

  @override
  State<ScreenE> createState() => _ScreenEState();
}

class _ScreenEState extends State<ScreenE> {
  String displayData = "Chưa có dữ liệu";
  late StreamSubscription _sub;

  @override
  void initState() {
    super.initState();
    
    // 1. Xử lý dữ liệu khi được MỞ MỚI (Navigate tới)
    // Cần delay nhẹ vì lúc initState, ModalRoute có thể chưa sẵn sàng arguments
    WidgetsBinding.instance.addPostFrameCallback((_) {
      final args = ModalRoute.of(context)?.settings.arguments;
      if (args is String) {
        setState(() {
          displayData = "Init: $args";
        });
      }
    });

    // 2. Lắng nghe dữ liệu khi ĐANG Ở ĐÂY (Update tại chỗ)
    _sub = DeepLinkService().dataStream.listen((data) {
      setState(() {
        displayData = "Updated: $data";
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("Nhận Deep Link mới: $data")),
      );
    });
  }

  @override
  void dispose() {
    _sub.cancel(); // Nhớ hủy lắng nghe
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Màn hình E")),
      body: Center(
        child: Text(
          displayData,
          style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
        ),
      ),
    );
  }
}
```

### Tổng kết ưu điểm của cách này:

1.  **Sạch sẽ (Clean):** Các màn hình A, B, C, D **không hề biết** gì về Deep Link. Code của chúng hoàn toàn "thuần khiết".
2.  **Tập trung (Centralized):** Mọi logic điều hướng (A-\>B-\>C-\>D-\>E) nằm gọn trong `DeepLinkService`. Nếu sau này luồng thay đổi, bạn chỉ cần sửa 1 file.
3.  **Thông minh:** Nó phân biệt được trạng thái "đang ở E" hay "không ở E" nhờ `RouteObserver` để đưa ra hành động phù hợp (Update UI hoặc Navigate).

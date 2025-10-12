Chắc chắn rồi! Dựa trên đoạn code bạn cung cấp, đây là một ví dụ hoàn chỉnh, có thể chạy được, để minh họa cách tất cả các thành phần—**Scaffold đáp ứng**, **GoRouter**, và **Riverpod** để quản lý trạng thái xác thực—hoạt động cùng nhau.

Ví dụ này sẽ tạo ra một ứng dụng có các màn hình được bảo vệ bằng đăng nhập.

### Bước 1: Cài đặt các package cần thiết

Thêm các package sau vào file `pubspec.yaml` của bạn:

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^14.1.0
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.9
  riverpod_generator: ^2.4.0
```

Sau đó, chạy các lệnh sau trong terminal:
1.  `flutter pub get`
2.  `flutter pub run build_runner build --delete-conflicting-outputs` (Bạn sẽ cần chạy lại lệnh này mỗi khi thay đổi các provider Riverpod có annotation `@riverpod`).

### Bước 2: Cấu trúc thư mục

Để dễ quản lý, hãy tạo cấu trúc thư mục như sau:

```
lib/
├── auth/
│   ├── auth_provider.dart
│   └── auth_state.dart
├── models/
│   └── navigation_item.dart
├── screens/
│   ├── login_screen.dart
│   ├── products_screen.dart
│   ├── profile_screen.dart
│   └── ... (các màn hình khác)
├── widgets/
│   └── scaffold_with_navigation.dart
├── main.dart
└── router.dart
```

### Bước 3: Viết mã cho các file

#### 1. `lib/auth/auth_state.dart`

File này định nghĩa các trạng thái xác thực và logic quyền truy cập.

```dart
// lib/auth/auth_state.dart
enum AuthState {
  unknown,
  authenticated,
  unauthenticated;

  /// Các đường dẫn được phép cho mỗi trạng thái.
  List<String> get allowedPaths {
    switch (this) {
      case AuthState.unknown:
        return []; // Không cho phép đi đâu khi đang kiểm tra
      case AuthState.unauthenticated:
        return ['/login']; // Chỉ được phép vào trang login
      case AuthState.authenticated:
        // Được phép vào tất cả các trang trừ trang login
        return ['/products', '/todos', '/posts', '/profile', '/settings'];
    }
  }

  /// Đường dẫn chuyển hướng nếu truy cập bị từ chối.
  String get redirectPath {
    switch (this) {
      case AuthState.unknown:
        return '/login'; // Tạm thời
      case AuthState.unauthenticated:
        return '/login';
      case AuthState.authenticated:
        // Nếu người dùng đã đăng nhập mà vào /login thì đá về trang chủ
        return '/products';
    }
  }
}
```

#### 2. `lib/auth/auth_provider.dart`

Provider Riverpod để quản lý trạng thái đăng nhập.

```dart
// lib/auth/auth_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'auth_state.dart';

part 'auth_provider.g.dart'; // File được tạo bởi build_runner

@Riverpod(keepAlive: true)
class CurrentAuthState extends _$CurrentAuthState {
  @override
  AuthState build() {
    // Giả sử ban đầu người dùng chưa đăng nhập
    return AuthState.unauthenticated;
  }

  void login() {
    state = AuthState.authenticated;
    print('User logged in');
  }

  void logout() {
    state = AuthState.unauthenticated;
    print('User logged out');
  }
}
```

#### 3. `lib/models/navigation_item.dart` và `lib/widgets/scaffold_with_navigation.dart`

Sao chép hai lớp này từ câu hỏi của bạn vào các file tương ứng. Đảm bảo thêm các import cần thiết.

**`lib/models/navigation_item.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class NavigationItem {
  // ... (giữ nguyên code từ câu hỏi)
}
```

**`lib/widgets/scaffold_with_navigation.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../models/navigation_item.dart';

class ScaffoldWithNavigation extends StatelessWidget {
  // ... (giữ nguyên code từ câu hỏi)
}
```

#### 4. `lib/screens/*.dart`

Tạo các màn hình placeholder.

**`lib/screens/login_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../auth/auth_provider.dart';

class LoginScreen extends ConsumerWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(title: const Text('Đăng nhập')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Gọi hàm login từ provider
            ref.read(currentAuthStateProvider.notifier).login();
          },
          child: const Text('Đăng nhập ngay'),
        ),
      ),
    );
  }
}
```

**`lib/screens/profile_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../auth/auth_provider.dart';

class ProfileScreen extends ConsumerWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Hồ sơ'),
        actions: [
          IconButton(
            icon: const Icon(Icons.settings),
            onPressed: () => context.go('/settings'),
          ),
        ],
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Gọi hàm logout từ provider
            ref.read(currentAuthStateProvider.notifier).logout();
          },
          child: const Text('Đăng xuất'),
        ),
      ),
    );
  }
}
```

**Tạo các màn hình còn lại (Products, Todos, Posts, Settings) dưới dạng placeholder đơn giản:**
```dart
// Ví dụ cho lib/screens/products_screen.dart
import 'package:flutter/material.dart';

class ProductsScreen extends StatelessWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Sản phẩm')),
      body: const Center(child: Text('Trang danh sách sản phẩm')),
    );
  }
}

// Tương tự cho các màn hình khác như TodosScreen, PostsScreen, SettingsScreen...
// Các màn hình chi tiết như ProductScreen(id) cũng có thể được tạo tương tự.
```

#### 5. `lib/router.dart` (Phần cốt lõi)

Đây là file chứa toàn bộ logic router bạn đã cung cấp.

```dart
// lib/router.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'auth/auth_provider.dart';
import 'auth/auth_state.dart';
import 'models/navigation_item.dart';
import 'screens/login_screen.dart';
import 'screens/products_screen.dart';
// Import tất cả các màn hình khác...
import 'screens/profile_screen.dart';
import 'screens/settings_screen.dart';
import 'screens/posts_screen.dart';
import 'screens/todos_screen.dart';
import 'widgets/scaffold_with_navigation.dart';

part 'router.g.dart';

// Placeholder screens for simplicity
class ProductScreen extends StatelessWidget {
  final int id;
  const ProductScreen(this.id, {super.key});
  @override
  Widget build(BuildContext context) => Scaffold(appBar: AppBar(title: Text('Product $id')));
}
// ... tạo các placeholder khác nếu cần

@riverpod
GoRouter router(RouterRef ref) {
  final authStateNotifier = ValueNotifier(AuthState.unknown);
  ref
    ..onDispose(authStateNotifier.dispose)
    ..listen(currentAuthStateProvider, (_, value) {
      authStateNotifier.value = value;
    });

  final navigationItems = [
    NavigationItem(
      path: '/products',
      body: (_) => const ProductsScreen(),
      icon: Icons.widgets_outlined,
      selectedIcon: Icons.widgets,
      label: 'Products',
      routes: [
        GoRoute(
          path: ':id',
          builder: (_, state) {
            final id = int.parse(state.pathParameters['id']!);
            return ProductScreen(id);
          },
        ),
      ],
    ),
    NavigationItem(
      path: '/todos',
      body: (_) => const TodosScreen(),
      icon: Icons.checklist_outlined,
      selectedIcon: Icons.checklist,
      label: 'Todos',
    ),
    NavigationItem(
      path: '/posts',
      body: (_) => const PostsScreen(),
      icon: Icons.article_outlined,
      selectedIcon: Icons.article,
      label: 'Posts',
    ),
    NavigationItem(
      path: '/profile',
      body: (_) => const ProfileScreen(),
      icon: Icons.person_outline,
      selectedIcon: Icons.person,
      label: 'Profile',
    ),
  ];

  final router = GoRouter(
    debugLogDiagnostics: true,
    initialLocation: '/products',
    routes: [
      GoRoute(
        path: '/login',
        builder: (_, __) => const LoginScreen(),
      ),
      GoRoute(
        path: '/settings',
        builder: (_, __) => const SettingsScreen(),
      ),
      ShellRoute(
        builder: (context, state, child) {
          // Tìm index của route hiện tại để highlight đúng mục
          final int selectedIndex = navigationItems.indexWhere((item) {
            final currentPath = state.uri.path;
            return currentPath.startsWith(item.path);
          });
          
          return ScaffoldWithNavigation(
            navigationItems: navigationItems,
            selectedIndex: selectedIndex < 0 ? 0 : selectedIndex,
            child: child,
          );
        },
        routes: [
          for (final item in navigationItems)
            GoRoute(
              path: item.path,
              builder: (context, state) => item.body(context),
              routes: item.routes,
            ),
        ],
      ),
    ],
    refreshListenable: authStateNotifier,
    redirect: (context, state) {
      final authState = ref.read(currentAuthStateProvider);
      final isLoggingIn = state.matchedLocation == '/login';

      // Nếu chưa đăng nhập và không ở trang login, chuyển hướng đến login
      if (authState == AuthState.unauthenticated && !isLoggingIn) {
        return '/login';
      }

      // Nếu đã đăng nhập và đang ở trang login, chuyển hướng vào trang chủ
      if (authState == AuthState.authenticated && isLoggingIn) {
        return '/products';
      }

      return null;
    },
  );
  ref.onDispose(router.dispose);

  return router;
}
```
*Lưu ý:* Tôi đã đơn giản hóa logic `redirect` và `ShellRoute` một chút để dễ hiểu và chạy ổn định hơn so với logic `NoTransitionPage` và `allowedPaths` phức tạp trong ví dụ gốc.

#### 6. `lib/main.dart`

File cuối cùng để kết nối mọi thứ.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'router.dart';

void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Lắng nghe provider router
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      title: 'Flutter Advanced Routing Demo',
      theme: ThemeData(
        primarySwatch: Colors.indigo,
        useMaterial3: true,
      ),
      routerConfig: router,
    );
  }
}
```

### Bước 4: Chạy và thử nghiệm

1.  **Chạy lại `build_runner`:**
    `flutter pub run build_runner build --delete-conflicting-outputs`

2.  **Chạy ứng dụng:**
    `flutter run`

**Luồng hoạt động bạn sẽ thấy:**

1.  **Khởi động:** Ứng dụng khởi động. Trạng thái ban đầu là `unauthenticated`. Logic `redirect` sẽ được kích hoạt và vì bạn không ở trang `/login`, nó sẽ tự động chuyển hướng bạn đến màn hình **Đăng nhập**.
2.  **Đăng nhập:** Nhấn vào nút "Đăng nhập ngay". Hàm `login()` trong provider được gọi, trạng thái đổi thành `authenticated`. `ValueNotifier` thông báo cho `GoRouter`.
3.  **Tự động chuyển hướng:** `GoRouter` chạy lại logic `redirect`. Lần này, bạn đã `authenticated` nhưng đang ở trang `/login`, vì vậy nó sẽ tự động chuyển hướng bạn vào trang `/products` (trang chủ).
4.  **Sử dụng ứng dụng:**
    *   Bây giờ bạn có thể điều hướng giữa các tab Products, Todos, Posts, Profile.
    *   Thay đổi kích thước cửa sổ (nếu bạn chạy trên web hoặc desktop) để xem giao diện chuyển đổi giữa `NavigationBar` (dưới cùng) và `NavigationRail` (bên trái).
5.  **Đăng xuất:** Vào trang Hồ sơ và nhấn "Đăng xuất". Trạng thái đổi thành `unauthenticated`. `GoRouter` lại chạy `redirect` và đưa bạn trở lại màn hình Đăng nhập.

Ví dụ này minh họa một cách hoàn hảo cách kết hợp các thư viện mạnh mẽ để xây dựng một ứng dụng Flutter có cấu trúc tốt, an toàn và linh hoạt.

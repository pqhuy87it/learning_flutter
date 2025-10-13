Chắc chắn rồi, tôi sẽ phân tích chi tiết về thuộc tính `redirect` trong `GoRouter`.

`redirect` là một trong những tính năng mạnh mẽ và quan trọng nhất của `GoRouter`. Nó hoạt động như một "người gác cổng" hay một "bộ lọc" cho hệ thống điều hướng của bạn. Mỗi khi một hành động điều hướng xảy ra, `GoRouter` sẽ gọi hàm `redirect` này *trước khi* thực sự hiển thị màn hình mới. Điều này cho phép bạn can thiệp và thay đổi đích đến dựa trên trạng thái của ứng dụng.

### Mục đích chính của `redirect`

Bạn sử dụng `redirect` để xử lý các logic nghiệp vụ phức tạp liên quan đến điều hướng, bao gồm:

1.  **Xác thực người dùng (Authentication):** Đây là trường hợp sử dụng phổ biến nhất. Kiểm tra xem người dùng đã đăng nhập hay chưa. Nếu chưa, chuyển hướng họ đến màn hình đăng nhập (`/login`) thay vì cho phép họ truy cập các trang được bảo vệ (ví dụ: `/profile`, `/settings`).
2.  **Phân quyền (Authorization):** Kiểm tra xem người dùng có quyền truy cập một trang cụ thể hay không. Ví dụ, chỉ người dùng có vai trò `admin` mới được vào trang `/admin`, những người khác sẽ bị chuyển hướng đi.
3.  **Onboarding (Hướng dẫn người dùng mới):** Buộc người dùng mới phải hoàn thành các bước thiết lập ban đầu trước khi có thể sử dụng các tính năng chính của ứng dụng.
4.  **Kiểm tra điều kiện dữ liệu:** Chuyển hướng người dùng nếu một trạng thái dữ liệu nào đó chưa sẵn sàng. Ví dụ, không cho vào trang `/cart/checkout` nếu giỏ hàng đang trống.
5.  **Feature Flags:** Chuyển hướng người dùng ra khỏi một tính năng nếu nó chưa được kích hoạt cho tài khoản của họ.

---

### Cách hoạt động

Hàm `redirect` có chữ ký như sau:

```dart
String? Function(BuildContext context, GoRouterState state)? redirect,
```

Nó là một hàm nhận vào 2 tham số và trả về một `String?` (có thể là `null` hoặc một chuỗi).

1.  **Khi nào nó được gọi?** Mỗi khi có một yêu cầu điều hướng đến một location (URL) mới.
2.  **Nó nhận vào cái gì?**
    *   `BuildContext context`: Context của ứng dụng, cho phép bạn truy cập vào các services hoặc providers (ví dụ: `ref.read(...)` trong Riverpod).
    *   `GoRouterState state`: Đây là đối tượng cực kỳ quan trọng, chứa mọi thông tin về yêu cầu điều hướng đang diễn ra.
3.  **Nó trả về cái gì?**
    *   **`null`**: **"Cho phép đi tiếp"**. Nếu hàm `redirect` trả về `null`, `GoRouter` sẽ hiểu rằng không có gì cần thay đổi và sẽ tiếp tục điều hướng đến location ban đầu mà người dùng yêu cầu.
    *   **`String` (ví dụ: `'/login'`)**: **"Dừng lại! Đi đến đây thay thế"**. Nếu hàm trả về một chuỗi đường dẫn mới, `GoRouter` sẽ hủy bỏ yêu cầu điều hướng ban đầu và thực hiện một yêu cầu điều hướng mới đến đường dẫn được trả về.

### Phân tích tham số `GoRouterState`

`GoRouterState` cung cấp cho bạn toàn bộ ngữ cảnh của việc điều hướng:

*   `state.location` (hoặc `state.uri` trong phiên bản mới): Chuỗi đầy đủ của đường dẫn mà người dùng đang cố gắng truy cập, bao gồm cả query parameters. Ví dụ: `/users/123?tab=photos`.
*   `state.matchedLocation`: Mẫu route đã khớp. Ví dụ, nếu route của bạn là `path: '/users/:id'` và người dùng đi đến `/users/123`, thì `matchedLocation` sẽ là `/users/:id`.
*   `state.pathParameters`: Một `Map` chứa các tham số động trên đường dẫn. Ví dụ, với `/users/123`, `state.pathParameters` sẽ là `{'id': '123'}`.
*   `state.queryParameters`: Một `Map` chứa các tham số truy vấn. Ví dụ, với `?tab=photos&sort=newest`, `state.queryParameters` sẽ là `{'tab': 'photos', 'sort': 'newest'}`.
*   `state.extra`: Dữ liệu bổ sung (bất kỳ đối tượng nào) được truyền qua phương thức `go()` hoặc `push()`.
*   `state.error`: Nếu có lỗi xảy ra trong quá trình tìm route, thuộc tính này sẽ chứa thông tin lỗi.

---

### Ví dụ thực tế: Xử lý xác thực người dùng

Đây là một ví dụ kinh điển sử dụng `redirect` kết hợp với `Riverpod` để quản lý trạng thái đăng nhập.

```dart
// 1. Provider quản lý trạng thái đăng nhập
final authProvider = StateProvider<bool>((ref) => false); // Mặc định là chưa đăng nhập

// 2. Cấu hình GoRouter với redirect
final goRouter = GoRouter(
  // Theo dõi sự thay đổi của authProvider để kích hoạt redirect lại khi trạng thái thay đổi
  refreshListenable: ValueNotifier<bool>(ref.watch(authProvider)),

  redirect: (BuildContext context, GoRouterState state) {
    final bool isLoggedIn = ref.read(authProvider); // Đọc trạng thái đăng nhập
    final String location = state.location;

    // Các trang không cần đăng nhập
    final bool isGoingToLogin = location == '/login';
    final bool isGoingToRegister = location == '/register';

    // Kịch bản 1: Người dùng chưa đăng nhập VÀ họ đang cố vào một trang cần bảo vệ
    if (!isLoggedIn && !isGoingToLogin && !isGoingToRegister) {
      // Chuyển hướng đến trang đăng nhập
      return '/login';
    }

    // Kịch bản 2: Người dùng đã đăng nhập VÀ họ lại cố vào trang đăng nhập/đăng ký
    if (isLoggedIn && (isGoingToLogin || isGoingToRegister)) {
      // Chuyển hướng họ vào trang chủ
      return '/home';
    }

    // Kịch bản 3: Các trường hợp còn lại -> Cho phép đi tiếp
    return null;
  },
  routes: [
    GoRoute(path: '/home', builder: (context, state) => HomeScreen()),
    GoRoute(path: '/profile', builder: (context, state) => ProfileScreen()),
    GoRoute(path: '/login', builder: (context, state) => LoginScreen()),
    GoRoute(path: '/register', builder: (context, state) => RegisterScreen()),
  ],
);
```

**Phân tích ví dụ trên:**

1.  **`refreshListenable`**: Đây là một thuộc tính rất quan trọng đi kèm với `redirect`. Nó nhận vào một `Listenable` (như `ChangeNotifier` hoặc `ValueNotifier`). Khi `Listenable` này thông báo có sự thay đổi (ví dụ, người dùng vừa đăng nhập hoặc đăng xuất), `GoRouter` sẽ tự động **thực thi lại hàm `redirect`** với location hiện tại. Điều này đảm bảo rằng giao diện sẽ được cập nhật đúng ngay cả khi trạng thái thay đổi mà không có hành động điều hướng mới.
2.  **Logic trong `redirect`**:
    *   Lấy trạng thái đăng nhập hiện tại.
    *   Kiểm tra xem người dùng có đang cố gắng truy cập các trang công khai (`/login`, `/register`) hay không.
    *   **Kịch bản 1**: Nếu chưa đăng nhập và không đi đến trang công khai -> chuyển hướng đến `/login`.
    *   **Kịch bản 2**: Nếu đã đăng nhập nhưng lại vào trang login/register -> chuyển hướng vào trang chủ, tránh trải nghiệm người dùng khó chịu.
    *   **Kịch bản 3 (quan trọng nhất)**: Nếu mọi thứ đều hợp lệ (ví dụ, đã đăng nhập và vào trang `/profile`), trả về `null` để cho phép điều hướng tiếp tục.

---

### Các lưu ý quan trọng và lỗi thường gặp

*   **Vòng lặp chuyển hướng vô hạn (Infinite Redirect Loop):** Đây là lỗi phổ biến nhất. Ví dụ: bạn chuyển hướng từ `/` đến `/login` khi chưa đăng nhập, nhưng logic của bạn lại chuyển hướng từ `/login` về `/` vì một lý do nào đó.
    *   **Cách tránh:** Luôn đảm bảo có một điều kiện thoát. Trong ví dụ trên, điều kiện `!isGoingToLogin` đảm bảo rằng nếu chúng ta đã được chuyển hướng đến `/login`, lần `redirect` tiếp theo sẽ không chuyển hướng đi đâu nữa (nó sẽ trả về `null`).
*   **Redirect bất đồng bộ (Async Redirect):** Hàm `redirect` cũng có thể là một hàm `async` và trả về `Future<String?>`. Điều này cực kỳ hữu ích khi bạn cần kiểm tra một trạng thái từ một nguồn bất đồng bộ (ví dụ: kiểm tra token từ `SharedPreferences` hoặc `SecureStorage`).
    ```dart
    redirect: (BuildContext context, GoRouterState state) async {
      final bool hasOnboarded = await checkOnboardingStatus(); // Hàm async
      if (!hasOnboarded) {
        return '/onboarding';
      }
      return null;
    },
    ```
*   **Vị trí của `redirect`**: Bạn có thể đặt `redirect` ở cấp cao nhất của `GoRouter` (áp dụng cho toàn bộ ứng dụng) hoặc đặt nó bên trong một `GoRoute` hoặc `ShellRoute` cụ thể (chỉ áp dụng cho route đó và các route con của nó).

Tóm lại, `redirect` là một công cụ linh hoạt và thiết yếu trong `GoRouter`, cho phép bạn thực thi các quy tắc điều hướng phức tạp một cách tập trung và rõ ràng, giúp mã nguồn của bạn sạch sẽ và dễ bảo trì hơn.

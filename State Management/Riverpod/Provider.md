Chào bạn,

Tôi sẽ giải thích chi tiết về cách sử dụng `Provider` trong Riverpod, đây là loại provider cơ bản và quan trọng nhất mà bạn cần nắm vững.

### `Provider` là gì?

`Provider` là loại provider cơ bản nhất trong Riverpod. Chức năng chính của nó là **cung cấp một giá trị không thay đổi (immutable) hoặc một đối tượng service** cho toàn bộ ứng dụng của bạn.

Hãy tưởng tượng nó như một "hộp chứa" mà bạn đặt một đối tượng vào đó, và sau đó bất kỳ widget hay provider nào khác trong ứng dụng cũng có thể lấy đối tượng đó ra để sử dụng. Đây chính là cốt lõi của **Dependency Injection (DI)** trong Riverpod.

### Khi nào nên sử dụng `Provider`?

Bạn nên sử dụng `Provider` trong các trường hợp sau:

1.  **Dependency Injection cho các Service/Repository:** Đây là trường hợp sử dụng phổ biến nhất. Bạn tạo ra các đối tượng như `ApiService`, `DatabaseService`, `UserRepository` và cung cấp chúng thông qua `Provider`. Các phần khác của ứng dụng sẽ lấy các đối tượng này để thực hiện công việc.
2.  **Cung cấp các giá trị cấu hình (Configuration):** Cung cấp các giá trị không đổi trong suốt vòng đời ứng dụng, ví dụ như API Key, các đối tượng cấu hình, hoặc một `Dio` instance đã được thiết lập sẵn.
3.  **Tính toán các giá trị dựa trên các provider khác:** Bạn có thể kết hợp nhiều provider để tạo ra một giá trị mới.

**Quan trọng:** `Provider` không được thiết kế để chứa trạng thái có thể thay đổi (mutable state) và tự động rebuild UI khi trạng thái đó thay đổi. Nếu bạn cần một trạng thái có thể thay đổi và cập nhật UI, hãy sử dụng `StateProvider`, `StateNotifierProvider`, hoặc `ChangeNotifierProvider`.

---

### Cách sử dụng `Provider` chi tiết

Chúng ta sẽ đi qua 3 bước chính: **Tạo Provider**, **Cung cấp Provider**, và **Sử dụng/Đọc Provider**.

#### Bước 1: Tạo một Provider

`Provider` được khai báo dưới dạng một biến `final` toàn cục.

**Cú pháp:**

```dart
import 'package:riverpod/riverpod.dart';

// 1. Định nghĩa một class mà bạn muốn cung cấp (ví dụ: một service)
class ApiService {
  Future<String> fetchData() async {
    // Giả lập một cuộc gọi mạng
    await Future.delayed(const Duration(seconds: 1));
    return "Dữ liệu từ API";
  }
}

// 2. Tạo một Provider để cung cấp instance của ApiService
// Provider<ApiService>: Chỉ định rằng provider này sẽ cung cấp một đối tượng loại ApiService.
// (ref) => ApiService(): Hàm này được gọi một lần để tạo ra đối tượng.
// `ref` là một đối tượng cho phép provider này tương tác với các provider khác.
final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService();
});
```

Trong ví dụ trên:
*   `apiServiceProvider` là một `Provider` sẽ tạo và trả về một instance duy nhất của `ApiService`.
*   Instance này sẽ được "cached" (lưu trữ). Các lần gọi sau để đọc provider này sẽ luôn trả về cùng một instance đó, giúp tiết kiệm tài nguyên.

#### Bước 2: Cung cấp Provider cho ứng dụng

Để các widget có thể truy cập vào provider, bạn cần bọc widget gốc của ứng dụng (thường là `MaterialApp`) bằng `ProviderScope`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    // ProviderScope là nơi lưu trữ trạng thái của tất cả các provider.
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: HomePage(),
    );
  }
}
```

#### Bước 3: Đọc/Sử dụng giá trị từ Provider

Đây là phần quan trọng nhất. Có nhiều cách để đọc giá trị từ một provider, tùy thuộc vào ngữ cảnh. Để làm điều này, bạn cần một đối tượng `ref`. Bạn có thể lấy `ref` trong các widget bằng cách kế thừa từ các lớp đặc biệt của Riverpod.

##### A. Đọc Provider trong một Widget

Để đọc provider trong widget, bạn cần chuyển widget của mình thành một `ConsumerWidget` hoặc `ConsumerStatefulWidget`.

**1. `ref.watch` - Để lắng nghe và rebuild UI**

Sử dụng `ref.watch` bên trong phương thức `build` của widget. Khi giá trị của provider thay đổi (mặc dù `Provider` thường không thay đổi, nhưng nó có thể phụ thuộc vào provider khác có thay đổi), widget sẽ tự động được build lại.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Kế thừa từ ConsumerWidget thay vì StatelessWidget
class HomePage extends ConsumerWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Sử dụng ref.watch để lấy instance của ApiService.
    // Widget này sẽ "lắng nghe" provider này.
    final apiService = ref.watch(apiServiceProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Riverpod Provider')),
      body: Center(
        // Sử dụng service để gọi hàm
        child: FutureBuilder<String>(
          future: apiService.fetchData(),
          builder: (context, snapshot) {
            if (snapshot.connectionState == ConnectionState.waiting) {
              return const CircularProgressIndicator();
            }
            if (snapshot.hasError) {
              return Text('Lỗi: ${snapshot.error}');
            }
            return Text('Dữ liệu nhận được: ${snapshot.data}');
          },
        ),
      ),
    );
  }
}
```

**2. `ref.read` - Để đọc một lần, không rebuild UI**

Sử dụng `ref.read` khi bạn chỉ cần lấy giá trị một lần và không muốn widget rebuild khi provider thay đổi. Thường được dùng bên trong các hàm callback như `onPressed`, `initState`, `didChangeDependencies`.

```dart
class ActionButton extends ConsumerWidget {
  const ActionButton({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // Sử dụng ref.read bên trong một callback.
        // Nó sẽ lấy instance của ApiService mà không làm widget này rebuild.
        final apiService = ref.read(apiServiceProvider);
        
        // Thực hiện một hành động, ví dụ: hiển thị SnackBar
        apiService.fetchData().then((data) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Đã nhận được: $data')),
          );
        });
      },
      child: const Text('Thực hiện hành động'),
    );
  }
}
```

**Quy tắc vàng:**
*   Dùng `ref.watch()` trong phương thức `build` để hiển thị dữ liệu lên UI.
*   Dùng `ref.read()` trong các hàm callback (như `onPressed`) để thực thi một hành động.

##### B. Đọc Provider trong một Provider khác

Đây là sức mạnh lớn nhất của Riverpod: các provider có thể "nói chuyện" với nhau. Một provider có thể đọc giá trị từ một provider khác bằng cách sử dụng `ref`.

Ví dụ: Chúng ta tạo một `userRepositoryProvider` phụ thuộc vào `apiServiceProvider`.

```dart
class UserRepository {
  final ApiService _apiService;

  // Constructor yêu cầu một ApiService
  UserRepository(this._apiService);

  Future<String> getUserName() {
    // Sử dụng apiService để lấy dữ liệu người dùng
    return _apiService.fetchData(); 
  }
}

// Provider này phụ thuộc vào apiServiceProvider
final userRepositoryProvider = Provider<UserRepository>((ref) {
  // Dùng ref.watch để lấy instance của ApiService
  final apiService = ref.watch(apiServiceProvider);
  // Truyền instance đó vào constructor của UserRepository
  return UserRepository(apiService);
});
```

Bây giờ, trong UI, bạn có thể sử dụng `userRepositoryProvider` mà không cần quan tâm nó được tạo ra như thế nào. Riverpod sẽ tự động quản lý việc tạo `apiServiceProvider` trước, sau đó mới tạo `userRepositoryProvider`.

```dart
class UserInfo extends ConsumerWidget {
  const UserInfo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Chỉ cần watch userRepositoryProvider
    final userRepository = ref.watch(userRepositoryProvider);
    // ... sử dụng userRepository
    return Text('Đang lấy thông tin người dùng...');
  }
}
```

---

### Các modifier hữu ích cho `Provider`

#### 1. `.autoDispose`

Mặc định, trạng thái của một provider sẽ được giữ lại ngay cả khi không còn widget nào lắng nghe nó. Để tự động hủy trạng thái của provider khi nó không còn được sử dụng, hãy dùng `.autoDispose`. Điều này rất tốt cho việc giải phóng bộ nhớ.

```dart
// Provider này sẽ bị hủy khi không còn ai "watch" nó.
final myAutoDisposeProvider = Provider.autoDispose<ApiService>((ref) {
  // Khi provider bị hủy, bạn có thể thực hiện dọn dẹp tại đây
  ref.onDispose(() {
    print('ApiService provider đã bị hủy!');
    // ví dụ: đóng kết nối, hủy stream...
  });
  return ApiService();
});
```

#### 2. `.family`

Sử dụng `.family` khi bạn cần tạo một provider mà giá trị của nó phụ thuộc vào một tham số từ bên ngoài.

Ví dụ: Lấy thông tin người dùng dựa trên `userId`.

```dart
// Định nghĩa một provider family
// Tham số thứ nhất: Kiểu dữ liệu trả về (String)
// Tham số thứ hai: Kiểu dữ liệu của tham số đầu vào (String - userId)
final userProvider = Provider.family<String, String>((ref, userId) {
  // Giả lập lấy tên người dùng từ ID
  return 'Tên người dùng cho ID: $userId';
});

// Cách sử dụng trong widget
class UserDetails extends ConsumerWidget {
  final String userId;
  const UserDetails({required this.userId, Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Truyền userId vào khi watch provider
    final userName = ref.watch(userProvider(userId));
    
    return Text(userName);
  }
}
```

### Tổng kết

*   `Provider` dùng để cung cấp các đối tượng **không thay đổi** như services, repositories, hoặc các giá trị cấu hình.
*   **Tạo provider** bằng cách khai báo một biến `final` toàn cục.
*   **Bọc ứng dụng** trong `ProviderScope` ở file `main.dart`.
*   **Sử dụng provider** trong widget bằng `ref.watch` (để rebuild UI) hoặc `ref.read` (cho các hành động).
*   Các provider có thể **phụ thuộc lẫn nhau**, tạo thành một biểu đồ phụ thuộc (dependency graph) mà Riverpod tự động quản lý.
*   Sử dụng `.autoDispose` để quản lý bộ nhớ và `.family` để tạo provider có tham số.

Hi vọng giải thích chi tiết này sẽ giúp bạn hiểu rõ và sử dụng `Provider` trong Riverpod một cách hiệu quả

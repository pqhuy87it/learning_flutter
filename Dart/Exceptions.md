Chào bạn,

Rất vui được giải thích chi tiết về **Exceptions (Ngoại lệ)** và cách xử lý chúng trong Flutter. Đây là một kỹ năng cực kỳ quan trọng để xây dựng các ứng dụng ổn định, thân thiện với người dùng và dễ gỡ lỗi.

### 1. Exception là gì và tại sao nó quan trọng?

Hãy tưởng tượng bạn đang viết một chương trình. Mọi thứ hoạt động hoàn hảo trong điều kiện lý tưởng. Nhưng trong thực tế, rất nhiều điều không mong muốn có thể xảy ra:
*   Người dùng nhập chữ vào ô yêu cầu nhập số.
*   Ứng dụng cố gắng đọc một file không tồn tại.
*   Điện thoại mất kết nối Internet khi đang tải dữ liệu từ server.
*   Server trả về lỗi 500.

Khi những sự kiện bất thường này xảy ra, chúng được gọi là **Ngoại lệ (Exception)**.

Nếu bạn không "xử lý" (handle) những ngoại lệ này, ứng dụng của bạn sẽ **sụp đổ (crash)**, hiển thị một màn hình đỏ đáng sợ và để lại trải nghiệm tồi tệ cho người dùng.

**Xử lý ngoại lệ** là kỹ thuật bạn viết code để "bắt" những lỗi này một cách duyên dáng, ngăn chặn ứng dụng bị crash và thực hiện các hành động phù hợp như:
*   Hiển thị một thông báo lỗi thân thiện cho người dùng ("Không có kết nối mạng, vui lòng thử lại.").
*   Thử lại hành động không thành công.
*   Ghi lại lỗi vào một hệ thống giám sát (logging) để lập trình viên có thể sửa chữa.

### 2. Sự khác biệt giữa `Error` và `Exception` trong Dart

Dart phân biệt rõ ràng hai loại vấn đề này:

*   **`Exception`**: Là các sự cố có thể lường trước và có thể phục hồi được trong quá trình chạy ứng dụng. Ví dụ: `FormatException` (khi parse chuỗi thành số thất bại), `IOException` (lỗi đọc/ghi file). **Bạn nên bắt và xử lý những cái này.**
*   **`Error`**: Là các lỗi nghiêm trọng hơn, thường là do lỗi lập trình và không nên cố gắng bắt để xử lý. Ví dụ: `NoSuchMethodError` (gọi một phương thức không tồn tại), `ArgumentError` (truyền sai đối số cho hàm), `OutOfMemoryError`. **Bạn không nên bắt `Error`, thay vào đó hãy sửa code của mình.**

### 3. Các công cụ để xử lý ngoại lệ: `try`, `on`, `catch`, `finally`

Đây là khối lệnh cốt lõi để xử lý ngoại lệ.

```dart
try {
  // Đặt code có khả năng gây ra ngoại lệ ở đây.
  // Ví dụ: gọi API, đọc file, parse dữ liệu.
} on SomeSpecificException {
  // Khối này sẽ chạy NẾU một ngoại lệ CỤ THỂ `SomeSpecificException` xảy ra.
  // Đây là cách xử lý được khuyến khích.
} catch (e, s) {
  // Khối này sẽ chạy NẾU có bất kỳ ngoại lệ nào khác xảy ra.
  // `e` là đối tượng ngoại lệ (chứa thông tin về lỗi).
  // `s` là StackTrace (dấu vết cuộc gọi hàm, cực kỳ hữu ích để debug).
} finally {
  // Khối này LUÔN LUÔN chạy, dù có ngoại lệ hay không.
  // Thường dùng để dọn dẹp tài nguyên (ví dụ: đóng file, ẩn loading indicator).
}
```

#### Ví dụ thực tế:

```dart
void testExceptionHandling() {
  print('Bắt đầu...');
  try {
    // Giả sử đây là một chuỗi không hợp lệ để parse thành số
    String invalidNumberString = 'abc';
    int number = int.parse(invalidNumberString);
    print('Parse thành công: $number'); // Dòng này sẽ không bao giờ được chạy
  } on FormatException catch (e) {
    // Bắt đúng loại ngoại lệ mà chúng ta mong đợi
    print('Đã xảy ra lỗi định dạng! Chi tiết: $e');
  } catch (e) {
    // Bắt các lỗi không lường trước khác
    print('Đã xảy ra một lỗi không xác định: $e');
  } finally {
    // Luôn luôn chạy
    print('Kết thúc khối xử lý.');
  }
}

// Gọi hàm:
// Output:
// Bắt đầu...
// Đã xảy ra lỗi định dạng! Chi tiết: FormatException: Invalid number (at character 1)
// abc
// ^
// Kết thúc khối xử lý.
```

### 4. Tự tạo và ném ngoại lệ: `throw` và `rethrow`

Đôi khi, bạn cần tự tạo ra ngoại lệ trong logic của mình. Ví dụ, kiểm tra dữ liệu đầu vào.

#### a. `throw`
Dùng để "ném" ra một đối tượng ngoại lệ.

```dart
void checkAge(int age) {
  if (age < 18) {
    // Ném ra một ngoại lệ với một thông báo tùy chỉnh.
    throw Exception('Người dùng phải đủ 18 tuổi.');
  }
  print('Tuổi hợp lệ.');
}

void main() {
  try {
    checkAge(15);
  } catch (e) {
    print('Lỗi: $e'); // Output: Lỗi: Exception: Người dùng phải đủ 18 tuổi.
  }
}
```

#### b. `rethrow`
Dùng trong một khối `catch` để "ném lại" ngoại lệ vừa bắt được lên cấp cao hơn để xử lý. Điều này hữu ích khi bạn muốn ghi log lỗi ở một cấp, nhưng để cho UI ở cấp cao hơn hiển thị thông báo.

```dart
void outerFunction() {
  try {
    innerFunction();
  } catch (e) {
    print('Lớp ngoài cùng đã bắt được lỗi và hiển thị cho người dùng: $e');
  }
}

void innerFunction() {
  try {
    // Giả lập một lỗi
    throw Exception('Lỗi kết nối cơ sở dữ liệu');
  } catch (e) {
    print('Lớp bên trong ghi log lỗi: $e');
    rethrow; // Ném lại ngoại lệ này cho outerFunction xử lý
  }
}
```

### 5. Tạo ngoại lệ tùy chỉnh (Custom Exceptions)

Để code dễ đọc và dễ quản lý hơn, bạn nên tạo các lớp ngoại lệ của riêng mình. Điều này giúp bạn bắt các loại lỗi cụ thể một cách rõ ràng.

```dart
// Định nghĩa các lớp ngoại lệ tùy chỉnh
class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);

  @override
  String toString() => 'NetworkException: $message';
}

class AuthenticationException implements Exception {
  final String message;
  AuthenticationException(this.message);

  @override
  String toString() => 'AuthenticationException: $message';
}

// Sử dụng chúng
Future<void> login(String username, String password) async {
  if (username.isEmpty || password.isEmpty) {
    throw AuthenticationException('Tên đăng nhập và mật khẩu không được để trống.');
  }
  // Giả lập gọi API
  // Giả sử server trả về lỗi 401 Unauthorized
  // throw AuthenticationException('Sai tên đăng nhập hoặc mật khẩu.');
  
  // Giả sử không có mạng
  // throw NetworkException('Không thể kết nối tới máy chủ.');
}

// Bắt các ngoại lệ tùy chỉnh một cách rõ ràng
void handleLogin() async {
  try {
    await login('', '');
  } on AuthenticationException catch (e) {
    // Xử lý lỗi xác thực
    print('Lỗi đăng nhập: $e');
  } on NetworkException catch (e) {
    // Xử lý lỗi mạng
    print('Lỗi kết nối: $e');
  } catch (e) {
    // Xử lý các lỗi chung khác
    print('Đã có lỗi xảy ra: $e');
  }
}
```

### 6. Ví dụ hoàn chỉnh trong Flutter: Gọi API

Đây là kịch bản phổ biến nhất.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'dart:io'; // Để dùng SocketException

// Định nghĩa lớp ngoại lệ tùy chỉnh
class HttpException implements Exception {
  final String message;
  HttpException(this.message);
}

class ApiService {
  Future<String> fetchData() async {
    try {
      final response = await http.get(Uri.parse('https://api.example.com/data'));

      if (response.statusCode == 200) {
        return json.decode(response.body)['message'];
      } else {
        // Ném lỗi HTTP nếu server trả về mã lỗi
        throw HttpException('Lỗi server: ${response.statusCode}');
      }
    } on SocketException {
      // Bắt lỗi khi không có kết nối mạng
      throw HttpException('Không có kết nối Internet.');
    } catch (e) {
      // Ném lại các lỗi không xác định khác
      throw HttpException('Đã có lỗi không mong muốn xảy ra.');
    }
  }
}

class MyApiWidget extends StatefulWidget {
  const MyApiWidget({super.key});

  @override
  State<MyApiWidget> createState() => _MyApiWidgetState();
}

class _MyApiWidgetState extends State<MyApiWidget> {
  final ApiService _apiService = ApiService();
  String _message = 'Nhấn nút để tải dữ liệu';
  bool _isLoading = false;

  Future<void> _fetchData() async {
    setState(() {
      _isLoading = true;
      _message = 'Đang tải...';
    });

    try {
      final data = await _apiService.fetchData();
      setState(() {
        _message = data;
      });
    } on HttpException catch (e) {
      // Bắt lỗi tùy chỉnh và hiển thị thông báo thân thiện
      setState(() {
        _message = 'Lỗi: ${e.message}';
      });
    } finally {
      // Luôn đảm bảo rằng loading indicator được tắt
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text(_message),
        const SizedBox(height: 20),
        if (_isLoading)
          const CircularProgressIndicator()
        else
          ElevatedButton(
            onPressed: _fetchData,
            child: const Text('Tải lại'),
          ),
      ],
    );
  }
}
```

### 7. Các phương pháp hay nhất (Best Practices)

1.  **Cụ thể hóa lỗi (`on SpecificException`):** Luôn cố gắng bắt loại ngoại lệ cụ thể nhất có thể. Tránh bắt `Exception` hoặc `Object` một cách chung chung.
2.  **Không "nuốt" ngoại lệ:** Đừng bao giờ tạo một khối `catch` trống. Ít nhất, hãy ghi log lỗi đó ra console (`print(e); print(s);`) để bạn biết nó đã xảy ra.
3.  **Tách biệt logic:** Logic nghiệp vụ (như gọi API) nên xử lý và ném ra các ngoại lệ có ý nghĩa. Lớp UI chỉ nên bắt những ngoại lệ đó và hiển thị thông báo cho người dùng.
4.  **Sử dụng `finally` để dọn dẹp:** Đảm bảo các tài nguyên được giải phóng hoặc trạng thái UI được cập nhật (như tắt loading) bất kể thành công hay thất bại.
5.  **Tạo ngoại lệ tùy chỉnh:** Giúp code của bạn rõ ràng, dễ đọc và dễ bảo trì hơn rất nhiều.

Bằng cách áp dụng các kỹ thuật xử lý ngoại lệ này, bạn sẽ xây dựng được các ứng dụng Flutter mạnh mẽ và chuyên nghiệp hơn.

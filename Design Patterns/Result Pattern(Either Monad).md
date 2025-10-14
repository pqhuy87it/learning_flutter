```dart
sealed class Result<T> {
  const Result();

  /// Creates a successful [Result], completed with the specified [value].
  const factory Result.ok(T value) = Ok._;

  /// Creates an error [Result], completed with the specified [error].
  const factory Result.error(Exception error) = Error._;
}

/// Subclass of Result for values
final class Ok<T> extends Result<T> {
  const Ok._(this.value);

  /// Returned value in result
  final T value;

  @override
  String toString() => 'Result<$T>.ok($value)';
}

/// Subclass of Result for errors
final class Error<T> extends Result<T> {
  const Error._(this.error);

  /// Returned error in result
  final Exception error;

  @override
  String toString() => 'Result<$T>.error($error)';
}
```
Tất nhiên rồi! Đây là một đoạn code rất hay và phổ biến trong Flutter/Dart, thể hiện một design pattern tuyệt vời để xử lý lỗi. Hãy cùng mình phân tích chi tiết nhé.

### Mục đích chính của đoạn code này là gì?

Đoạn code này định nghĩa một "wrapper" class tên là `Result`. Mục đích của nó là để gói kết quả của một tác vụ (ví dụ: gọi API, truy vấn database) có thể **thành công** hoặc **thất bại**. Thay vì ném ra một `Exception` (phải dùng `try-catch`) hoặc trả về `null` (dễ gây lỗi `NullPointerException`), hàm sẽ trả về một đối tượng `Result`. Đối tượng này sẽ cho bạn biết rõ ràng:

*   Hoặc là nó **thành công** và chứa giá trị bạn cần (`Ok`).
*   Hoặc là nó đã **thất bại** và chứa thông tin lỗi (`Error`).

Điều này làm cho việc xử lý lỗi trở nên **tường minh**, **an toàn** và **dễ đoán** hơn.

---

### Phân tích chi tiết từng phần

#### 1. `sealed class Result<T>`

*   **`class Result<T>`**: Đây là một lớp trừu tượng (abstract class) và là một generic class.
    *   `abstract`: Bạn không thể tạo một đối tượng trực tiếp từ `Result` như `Result()`. Bạn phải dùng một trong các lớp con của nó.
    *   `<T>` (Generic): Nó có thể chứa bất kỳ kiểu dữ liệu nào. Ví dụ, `Result<User>` sẽ chứa đối tượng `User` nếu thành công, `Result<String>` sẽ chứa một chuỗi `String` nếu thành công.
*   **`sealed`**: Đây là từ khóa "siêu năng lực" ở đây.
    *   Nó có nghĩa là tất cả các lớp con trực tiếp của `Result` **phải được định nghĩa trong cùng một file này**. Trong trường hợp này, đó là `Ok` và `Error`.
    *   **Lợi ích lớn nhất**: Khi bạn dùng `switch` statement để kiểm tra một đối tượng `Result`, trình biên dịch Dart sẽ biết tất cả các trường hợp có thể xảy ra (`Ok` và `Error`). Nếu bạn quên xử lý một trường hợp nào đó, nó sẽ báo lỗi ngay lúc biên dịch. Điều này giúp tránh lỗi runtime.

#### 2. `const factory Result.ok(T value) = Ok._;` và `const factory Result.error(Exception error) = Error._;`

*   **`factory` constructor**: Đây không phải là constructor thông thường. Thay vì tạo một instance của `Result`, nó ủy quyền việc tạo đối tượng cho một lớp khác (ở đây là các lớp con `Ok` và `Error`).
*   Khi bạn gọi `Result.ok(someValue)`, nó sẽ thực sự tạo ra một đối tượng `Ok._(someValue)`.
*   Khi bạn gọi `Result.error(someException)`, nó sẽ thực sự tạo ra một đối tượng `Error._(someException)`.
*   Việc này tạo ra một API rất sạch sẽ và dễ sử dụng. Bạn chỉ cần tương tác với `Result`, không cần quan tâm đến `Ok` hay `Error` lúc tạo đối tượng.

#### 3. `final class Ok<T> extends Result<T>`

*   Đây là lớp con đại diện cho trường hợp **thành công**.
*   `final class`: `final` có nghĩa là lớp này không thể được kế thừa hay implement bởi bất kỳ lớp nào khác, giúp đảm bảo tính đóng của `sealed class`.
*   `extends Result<T>`: Nó là một dạng của `Result`.
*   `const Ok._(this.value);`: Đây là một constructor riêng tư (private) vì có dấu `_`. Bạn không thể gọi `Ok(value)` từ bên ngoài file này. Bạn **bắt buộc** phải dùng `Result.ok(value)`.
*   `final T value;`: Thuộc tính này lưu trữ giá trị kết quả khi tác vụ thành công.

#### 4. `final class Error<T> extends Result<T>`

*   Đây là lớp con đại diện cho trường hợp **thất bại**.
*   Tương tự như `Ok`, nó có một constructor riêng tư `Error._`.
*   `final Exception error;`: Thuộc tính này lưu trữ đối tượng `Exception` chứa thông tin chi tiết về lỗi đã xảy ra.

---

### Cách sử dụng trong thực tế

Hãy tưởng tượng bạn có một hàm lấy dữ liệu người dùng từ API.

```dart
// Định nghĩa một model User đơn giản
class User {
  final String name;
  User(this.name);
}

// Hàm fetch dữ liệu, trả về Result<User>
Future<Result<User>> fetchUserData(int userId) async {
  try {
    // Giả lập việc gọi API có thể thành công hoặc thất bại
    if (userId == 1) {
      await Future.delayed(Duration(seconds: 1)); // Giả lập độ trễ mạng
      final user = User('Taro Yamada');
      return Result.ok(user); // Trả về kết quả thành công
    } else {
      throw Exception('User not found'); // Giả lập lỗi
    }
  } on Exception catch (e) {
    return Result.error(e); // Trả về kết quả lỗi
  }
}

// Cách sử dụng hàm trên trong UI hoặc logic
void main() async {
  print('Fetching user data...');
  final result = await fetchUserData(2); // Thử với userId = 2 để gây lỗi

  // Dùng switch để xử lý kết quả một cách an toàn
  switch (result) {
    // Nếu kết quả là Ok, giải nén giá trị `user` ra
    case Ok(:final value):
      print('Success! User name: ${value.name}');

    // Nếu kết quả là Error, giải nén giá trị `error` ra
    case Error(:final error):
      print('Failed! Reason: $error');
  }
}
```

**Kết quả chạy `main()`:**

```
Fetching user data...
Failed! Reason: Exception: User not found
```

### Tổng kết

Đoạn code này là một cách triển khai **"Result Pattern"** (còn gọi là "Either Monad" trong lập trình hàm). Nó vượt trội hơn các cách xử lý lỗi truyền thống vì:

1.  **An toàn về kiểu (Type-Safe)**: `sealed class` và `switch` bắt buộc bạn phải xử lý cả hai trường hợp thành công và thất bại.
2.  **Tường minh (Explicit)**: Nhìn vào chữ ký hàm `Future<Result<User>>`, bạn biết ngay lập tức rằng hàm này có thể thất bại và bạn cần phải chuẩn bị để xử lý nó.
3.  **Mang nhiều thông tin hơn `null`**: Thay vì `null` mơ hồ, `Error` chứa một đối tượng `Exception` cụ thể, giúp bạn gỡ lỗi dễ dàng hơn.
4.  **Code sạch hơn**: Tránh được việc lồng nhiều khối `try-catch` hoặc kiểm tra `if (result != null)` ở khắp nơi.

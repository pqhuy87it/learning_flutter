Chào bạn,

Rất vui được giải thích chi tiết về bộ tứ `throw`, `catch`, `rethrow`, và `finally` trong Dart/Flutter. Đây là những khối lệnh nền tảng để xây dựng hệ thống **xử lý ngoại lệ (Exception Handling)**, một kỹ năng không thể thiếu để viết các ứng dụng ổn định, mạnh mẽ và thân thiện với người dùng.

Hãy coi việc xử lý ngoại lệ như việc trang bị một "lưới an toàn" cho người đi trên dây. Bạn hy vọng họ sẽ không bao giờ ngã, nhưng bạn luôn chuẩn bị sẵn sàng nếu điều đó xảy ra.

---

### Sơ đồ tổng quan

```dart
try {
  // 1. TRY (Thử): Đặt code có khả năng gây lỗi vào đây.
  // Đây là "đoạn dây" mà người biểu diễn đi qua.
  // Có thể có một lệnh `throw` ở đây.
} on SpecificException catch (e, s) {
  // 2. CATCH (Bắt): Nếu có lỗi cụ thể xảy ra, khối này sẽ được thực thi.
  // Đây là "lưới an toàn" được thiết kế cho một loại tai nạn cụ thể.
  // Có thể có một lệnh `rethrow` ở đây.
} catch (e, s) {
  // 3. CATCH (Bắt): Bắt tất cả các loại lỗi khác không được xử lý ở trên.
  // Đây là "lưới an toàn chung".
} finally {
  // 4. FINALLY (Cuối cùng): Khối này LUÔN LUÔN được thực thi.
  // Dù người biểu diễn có qua dây thành công hay ngã vào lưới,
  // họ cũng phải cúi chào khán giả.
}
```

Bây giờ, hãy đi vào chi tiết từng phần.

---

### 1. `throw`: Ném ra một ngoại lệ

`throw` là hành động bạn chủ động **tạo ra và báo hiệu một lỗi**. Bạn không đợi lỗi tự xảy ra, mà bạn tạo ra nó khi phát hiện một điều kiện không hợp lệ trong logic của mình.

*   **Mục đích:** Báo hiệu rằng một tình huống bất thường đã xảy ra và không thể tiếp tục thực thi theo luồng thông thường.
*   **Nên `throw` cái gì?** Bạn có thể `throw` bất cứ thứ gì (một `String`, một `int`), nhưng **thực hành tốt nhất là `throw` một đối tượng kế thừa từ lớp `Exception`**. Điều này giúp việc bắt lỗi trở nên có cấu trúc và rõ ràng hơn.

**Ví dụ: Tạo ngoại lệ tùy chỉnh**

```dart
// 1. Định nghĩa một lớp Exception tùy chỉnh
class InsufficientFundsException implements Exception {
  final double shortage;
  InsufficientFundsException(this.shortage);

  @override
  String toString() {
    return 'Không đủ tiền trong tài khoản! Bạn thiếu: $shortage VNĐ';
  }
}

// 2. Sử dụng `throw` trong logic
void withdraw(double amount, double balance) {
  if (amount > balance) {
    // Ném ra ngoại lệ khi logic không hợp lệ
    throw InsufficientFundsException(amount - balance);
  }
  print('Rút tiền thành công!');
}
```

---

### 2. `catch`: Bắt và xử lý một ngoại lệ

`catch` là "lưới an toàn". Nó được đặt bên trong một khối `try` để "bắt" các ngoại lệ được `throw` ra từ bên trong khối `try` đó.

*   **Mục đích:** Ngăn chặn ứng dụng bị crash, thực hiện các hành động khắc phục hoặc thông báo lỗi cho người dùng một cách thân thiện.
*   **Các cách `catch`:**
    *   **`on SpecificException`:** Đây là cách **tốt nhất**. Nó chỉ bắt một loại ngoại lệ cụ thể, giúp bạn xử lý từng loại lỗi một cách riêng biệt.
    *   **`catch (e)`:** Bắt mọi loại ngoại lệ. `e` là đối tượng ngoại lệ (ví dụ: đối tượng `InsufficientFundsException` ở trên).
    *   **`catch (e, s)`:** Mạnh mẽ nhất. `e` là ngoại lệ, `s` là **StackTrace** (dấu vết ngăn xếp), cực kỳ hữu ích để gỡ lỗi vì nó cho bạn biết chính xác lỗi đã xảy ra ở hàm nào, dòng nào.

**Ví dụ: Sử dụng `try-catch`**

```dart
void main() {
  double myBalance = 500000;
  
  try {
    print('Đang thử rút 1,000,000 VNĐ...');
    withdraw(1000000, myBalance); // Code có khả năng `throw`
  } on InsufficientFundsException catch (e) {
    // Bắt đúng loại lỗi mà chúng ta mong đợi
    print('Lỗi đã xử lý: $e');
  } catch (e, s) {
    // Bắt các lỗi không lường trước khác
    print('Đã xảy ra lỗi không xác định: $e');
    print('StackTrace: $s'); // In ra dấu vết để debug
  }
  
  print('Giao dịch kết thúc.');
}

// Output:
// Đang thử rút 1,000,000 VNĐ...
// Lỗi đã xử lý: Không đủ tiền trong tài khoản! Bạn thiếu: 500000.0 VNĐ
// Giao dịch kết thúc.
```
Nếu không có `try-catch`, chương trình sẽ crash ngay khi `throw` được gọi.

---

### 3. `rethrow`: Ném lại ngoại lệ

`rethrow` được sử dụng bên trong một khối `catch`. Nó cho phép bạn thực hiện một hành động nào đó với ngoại lệ (ví dụ: ghi log) rồi sau đó **ném lại chính ngoại lệ đó** để một khối `try-catch` ở cấp cao hơn có thể xử lý tiếp.

*   **Mục đích:** Cho phép xử lý lỗi theo nhiều lớp. Lớp cấp thấp có thể ghi log kỹ thuật, trong khi lớp cấp cao (UI) có thể hiển thị thông báo cho người dùng.

**Ví dụ:**

```dart
void main() {
  try {
    // Lớp ngoài cùng (ví dụ: tầng UI)
    processPayment();
  } catch (e) {
    // Chỉ quan tâm đến việc hiển thị thông báo lỗi cho người dùng
    print('Thông báo cho người dùng: Giao dịch thất bại. Vui lòng thử lại.');
  }
}

void processPayment() {
  try {
    // Lớp ở giữa (ví dụ: tầng logic nghiệp vụ)
    withdraw(1000000, 500000);
  } catch (e, s) {
    // Tầng này muốn ghi lại lỗi chi tiết để lập trình viên phân tích
    print('--- GHI LOG LỖI ---');
    print('Lỗi: $e');
    print('Tại: $s');
    print('--- KẾT THÚC LOG ---');
    
    rethrow; // Ném lại ngoại lệ cho lớp gọi nó (main) xử lý tiếp
  }
}

// Output:
// --- GHI LOG LỖI ---
// Lỗi: Không đủ tiền trong tài khoản! Bạn thiếu: 500000.0 VNĐ
// Tại: #0      withdraw (file:///...)
//      #1      processPayment (file:///...)
//      #2      main (file:///...)
// --- KẾT THÚC LOG ---
// Thông báo cho người dùng: Giao dịch thất bại. Vui lòng thử lại.
```

---

### 4. `finally`: Luôn luôn được thực thi

Khối `finally` là phần dọn dẹp. Code bên trong nó **luôn được chạy** sau khi khối `try-catch` kết thúc, bất kể:
*   Khối `try` chạy thành công.
*   Một ngoại lệ được `throw` và được `catch`.
*   Một ngoại lệ được `throw` và không được `catch`.

*   **Mục đích:** Đảm bảo các tác vụ dọn dẹp quan trọng luôn được thực hiện, ví dụ: đóng file, hủy kết nối cơ sở dữ liệu, và đặc biệt trong Flutter là **ẩn đi chỉ báo tải (loading indicator)**.

**Ví dụ trong Flutter:**

```dart
// Trong một State của StatefulWidget

bool _isLoading = false;

Future<void> fetchData() async {
  setState(() {
    _isLoading = true; // 1. Hiển thị loading indicator
  });

  try {
    // Giả lập gọi API, có thể thành công hoặc thất bại
    final response = await http.get(Uri.parse('https://invalid-url'));
    if (response.statusCode != 200) {
      throw Exception('Lỗi server');
    }
    // Xử lý dữ liệu thành công...
  } catch (e) {
    // Xử lý lỗi, ví dụ hiển thị SnackBar
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Không thể tải dữ liệu: $e')),
    );
  } finally {
    // 3. Dù thành công hay thất bại, khối này LUÔN chạy
    setState(() {
      _isLoading = false; // 2. Đảm bảo loading indicator luôn được ẩn đi
    });
  }
}
```
Nếu không có `finally`, trong trường hợp có lỗi, `_isLoading` sẽ mãi mãi là `true` và người dùng sẽ bị kẹt ở màn hình loading.

### Kết luận

*   **`throw`**: Chủ động tạo và báo hiệu một lỗi.
*   **`catch`**: Bắt và xử lý lỗi để ứng dụng không bị crash. Hãy dùng `on SpecificException` để xử lý cụ thể.
*   **`rethrow`**: Xử lý lỗi ở một mức (ghi log) và "đẩy" nó lên mức cao hơn để xử lý tiếp (hiển thị UI).
*   **`finally`**: Đảm bảo code dọn dẹp (như ẩn loading) luôn được thực thi, bất kể kết quả của khối `try`.

Nắm vững bộ tứ này là chìa khóa để xây dựng các ứng dụng Flutter có khả năng phục hồi tốt, dễ gỡ lỗi và mang lại trải nghiệm người dùng chuyên nghiệp.

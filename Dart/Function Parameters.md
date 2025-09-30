Chào bạn,

Rất vui được giải thích chi tiết về **Function Parameters (Tham số hàm)** trong Flutter/Dart. Đây là một trong những khái niệm cơ bản nhưng cực kỳ quan trọng, và Dart cung cấp nhiều cách linh hoạt để định nghĩa chúng, giúp code của bạn trở nên dễ đọc và mạnh mẽ hơn.

### 1. Tham số là gì?

Hãy tưởng tượng một hàm là một cỗ máy, ví dụ như máy pha cà phê. Để máy hoạt động, bạn cần cung cấp cho nó "nguyên liệu đầu vào" như loại cà phê, lượng đường, có sữa hay không. Những "nguyên liệu đầu vào" này chính là **tham số (parameters)**.

Trong lập trình, **tham số** là các biến mà bạn định nghĩa trong chữ ký của một hàm, cho phép bạn truyền dữ liệu vào bên trong hàm để xử lý.

```dart
// 'a' và 'b' là các tham số
int add(int a, int b) {
  // Hàm này nhận vào 2 số nguyên và trả về tổng của chúng
  return a + b;
}

void main() {
  // 5 và 3 là các đối số (arguments) được truyền vào
  int result = add(5, 3);
  print(result); // Output: 8
}
```

Dart hỗ trợ nhiều loại tham số khác nhau, mỗi loại có mục đích và cú pháp riêng.

---

### 2. Các loại tham số trong Dart

#### a. Tham số vị trí bắt buộc (Required Positional Parameters)

Đây là loại tham số cơ bản nhất.
*   Chúng **bắt buộc** phải được cung cấp khi gọi hàm.
*   Chúng phải được truyền vào **đúng thứ tự** đã định nghĩa.

```dart
void printUserInfo(String name, int age, String country) {
  print('Tên: $name, Tuổi: $age, Quốc gia: $country');
}

void main() {
  printUserInfo('Sơn', 28, 'Việt Nam'); // Đúng
  // printUserInfo('Sơn', 28); // Lỗi! Thiếu tham số 'country'.
  // printUserInfo(28, 'Sơn', 'Việt Nam'); // Lỗi! Sai thứ tự và kiểu dữ liệu.
}
```
**Khi nào dùng:** Khi hàm có rất ít tham số (1-3) và ý nghĩa của chúng rõ ràng dựa trên vị trí.

---

#### b. Tham số được đặt tên (Named Parameters)

Đây là loại tham số cực kỳ phổ biến và hữu ích trong Flutter.
*   Chúng được đặt trong cặp dấu ngoặc nhọn `{}`.
*   Mặc định, chúng là **tùy chọn (optional)**.
*   Khi gọi hàm, bạn phải chỉ định tên của tham số, do đó **thứ tự không quan trọng**.
*   Vì là tùy chọn, chúng phải có khả năng nhận giá trị `null` hoặc phải có một giá trị mặc định.

```dart
void login({String? username, String? password}) {
  print('Đăng nhập với username: $username');
}

void main() {
  login(username: 'admin', password: '123');
  login(password: '123', username: 'admin'); // Thứ tự không quan trọng
  login(username: 'admin'); // OK, password sẽ là null
  login(); // OK, cả hai đều là null
}
```

**`required` Named Parameters:**
Đây là sự kết hợp tốt nhất: dễ đọc như tham số được đặt tên và bắt buộc như tham số vị trí. Bạn chỉ cần thêm từ khóa `required`.

```dart
// Đây là cách bạn thấy trong hầu hết các constructor của Widget trong Flutter
void createPost({required String title, String? content, bool isPublished = false}) {
  print('Tạo bài viết: $title');
}

void main() {
  createPost(title: 'Flutter thật tuyệt!');
  // createPost(content: 'Nội dung...'); // Lỗi! Thiếu tham số bắt buộc 'title'.
}
```
**Khi nào dùng:** Hầu hết mọi lúc trong Flutter! Đặc biệt là với các hàm hoặc constructor có nhiều tham số. Nó giúp code tự diễn giải và rất dễ đọc.

---

#### c. Tham số vị trí tùy chọn (Optional Positional Parameters)

Chúng tương tự như tham số vị trí, nhưng không bắt buộc.
*   Chúng được đặt trong cặp dấu ngoặc vuông `[]`.
*   Chúng phải được đặt sau tất cả các tham số vị trí bắt buộc.
*   Giống như tham số được đặt tên, chúng phải có khả năng nhận giá trị `null` hoặc có giá trị mặc định.

```dart
void sendMessage(String to, String message, [String? attachment]) {
  print('Gửi tin nhắn "$message" tới $to');
  if (attachment != null) {
    print('Kèm theo file: $attachment');
  }
}

void main() {
  sendMessage('An', 'Chào bạn');
  sendMessage('Bình', 'Đây là báo cáo', 'report.pdf');
}
```
**Khi nào dùng:** Khi bạn có một vài tham số tùy chọn và ý nghĩa của chúng vẫn rõ ràng theo vị trí. Loại này ít phổ biến hơn tham số được đặt tên trong Flutter.

---

#### d. Giá trị mặc định (Default Values)

Bạn có thể gán một giá trị mặc định cho các tham số tùy chọn (cả đặt tên và vị trí) bằng cách sử dụng dấu `=`. Khi đó, nếu người dùng không cung cấp giá trị, giá trị mặc định sẽ được sử dụng.

```dart
void setupConnection({
  String host = 'localhost',
  int port = 8080,
  required String user
}) {
  print('Kết nối tới $host:$port với người dùng $user');
}

void main() {
  setupConnection(user: 'admin'); // Output: Kết nối tới localhost:8080 với người dùng admin
  setupConnection(user: 'guest', host: '192.168.1.1'); // Output: Kết nối tới 192.168.1.1:8080 với người dùng guest
}
```

---

### 3. Hàm là một tham số (Function as a Parameter - Callbacks)

Đây là một khái niệm cực kỳ quan trọng trong Flutter để xử lý sự kiện. Bạn có thể truyền cả một hàm vào một hàm khác như một tham số. Hàm được truyền vào này thường được gọi là **callback**.

Ví dụ kinh điển nhất là thuộc tính `onPressed` của một `ElevatedButton`.

```dart
// Định nghĩa một widget tùy chỉnh nhận một callback
class CustomButton extends StatelessWidget {
  final String text;
  // Tham số này là một hàm không có đối số và không trả về gì.
  // VoidCallback là một kiểu dữ liệu được định nghĩa sẵn, tương đương với void Function().
  final VoidCallback? onPressed;

  const CustomButton({
    super.key,
    required this.text,
    this.onPressed,
  });

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed, // Truyền callback vào widget con
      child: Text(text),
    );
  }
}

// Cách sử dụng
class MyScreen extends StatelessWidget {
  void _handleLogin() {
    print('Nút đăng nhập được nhấn!');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: CustomButton(
          text: 'Đăng nhập',
          onPressed: _handleLogin, // Truyền hàm _handleLogin như một tham số
        ),
      ),
    );
  }
}
```

---

### 4. Ví dụ tổng hợp trong một Widget Flutter

Hãy tạo một widget `UserProfileTile` để thể hiện tất cả các loại tham số.

```dart
import 'package:flutter/material.dart';

class UserProfileTile extends StatelessWidget {
  // Tham số vị trí bắt buộc
  final String avatarUrl;

  // Tham số được đặt tên bắt buộc
  final String name;

  // Tham số được đặt tên tùy chọn có giá trị mặc định
  final bool isActive;

  // Tham số được đặt tên tùy chọn có thể null
  final int? age;

  // Tham số hàm (callback) tùy chọn
  final VoidCallback? onTap;

  const UserProfileTile(
    this.avatarUrl, // Vị trí
    {
      super.key,
      required this.name, // Đặt tên, bắt buộc
      this.isActive = false, // Đặt tên, tùy chọn, có mặc định
      this.age, // Đặt tên, tùy chọn, có thể null
      this.onTap, // Đặt tên, tùy chọn, có thể null
    }
  );

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: CircleAvatar(
        backgroundImage: NetworkImage(avatarUrl),
      ),
      title: Text(name),
      subtitle: Text(age != null ? '$age tuổi' : 'Chưa cập nhật tuổi'),
      trailing: Icon(
        Icons.circle,
        color: isActive ? Colors.green : Colors.grey,
        size: 12,
      ),
      onTap: onTap,
    );
  }
}

// Cách sử dụng Widget này trong ứng dụng
class UserList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView(
      children: [
        // Cung cấp các tham số bắt buộc
        UserProfileTile(
          'https://i.pravatar.cc/150?img=1',
          name: 'Nguyễn Văn A',
          age: 25,
          isActive: true,
          onTap: () => print('Nhấn vào người dùng A'),
        ),
        // Bỏ qua các tham số tùy chọn
        UserProfileTile(
          'https://i.pravatar.cc/150?img=2',
          name: 'Trần Thị B',
        ),
      ],
    );
  }
}
```

### Bảng tóm tắt

| Loại tham số | Cú pháp | Đặc điểm | Ví dụ sử dụng |
| :--- | :--- | :--- | :--- |
| **Vị trí bắt buộc** | `(int a, String b)` | Bắt buộc, thứ tự quan trọng. | `add(5, 3)` |
| **Đặt tên** | `({int? a, String? b})` | Tùy chọn, thứ tự không quan trọng, tự diễn giải. | `login(user: 'a', pass: 'b')` |
| **Đặt tên bắt buộc** | `({required int a})` | Bắt buộc, thứ tự không quan trọng, rất dễ đọc. | Hầu hết các constructor widget. |
| **Vị trí tùy chọn** | `([int? a, String? b])` | Tùy chọn, thứ tự quan trọng. | `sendMessage('msg', 'attachment.zip')` |
| **Hàm (Callback)** | `(VoidCallback? onTap)` | Truyền một hành động vào hàm/widget. | `ElevatedButton(onPressed: ...)` |

Việc hiểu và sử dụng đúng loại tham số sẽ giúp bạn viết code Flutter không chỉ hoạt động đúng mà còn rất sạch sẽ, dễ đọc và dễ bảo trì.

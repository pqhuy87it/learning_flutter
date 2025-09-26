Chào bạn! `InputDecoration` là một trong những class quan trọng và linh hoạt nhất khi làm việc với các form trong Flutter. Nó chính là "chuyên gia trang điểm" cho các ô nhập liệu của bạn.

Hãy cùng nhau tìm hiểu chi tiết về cách sử dụng nó nhé!

### `InputDecoration` là gì?

`InputDecoration` **không phải là một widget**. Nó là một **đối tượng trang trí** bất biến (immutable object) được sử dụng bên trong thuộc tính `decoration` của các widget nhập liệu như `TextField` và `TextFormField`.

Nó cho phép bạn tùy chỉnh gần như mọi khía cạnh hình ảnh của một ô nhập liệu, từ đường viền, nhãn, icon, cho đến màu sắc và văn bản trợ giúp.

**Cấu trúc cơ bản:**

```dart
TextField(
  decoration: InputDecoration(
    // ... tất cả các thuộc tính trang trí nằm ở đây
  ),
)
```

---

### Các thuộc tính "siêu năng lực" của `InputDecoration`

Hãy xem `InputDecoration` như một hộp đồ trang điểm, và đây là các món đồ trong đó:

#### 1. Các loại văn bản (Labels & Texts)

*   **`labelText`**:
    *   Một đoạn text nhỏ hiển thị bên trong ô nhập liệu khi nó trống và chưa được focus. Khi người dùng nhấn vào, nó sẽ **trượt lên trên** và thu nhỏ lại một cách mượt mà để trở thành tiêu đề. Đây là thuộc tính được dùng nhiều nhất.
    *   `labelText: 'Tên đăng nhập'`

*   **`hintText`**:
    *   Một văn bản gợi ý, hiển thị mờ bên trong ô nhập liệu khi nó trống. Nó sẽ **biến mất** khi người dùng bắt đầu gõ.
    *   `hintText: 'Nhập email của bạn'`

*   **`helperText`**:
    *   Một đoạn text nhỏ hiển thị **bên dưới** ô nhập liệu, thường dùng để cung cấp hướng dẫn bổ sung.
    *   `helperText: 'Mật khẩu phải có ít nhất 8 ký tự.'`

*   **`errorText`**:
    *   Một đoạn text (thường là màu đỏ) hiển thị **bên dưới** ô nhập liệu khi có lỗi validation. Khi `errorText` không phải là `null`, toàn bộ ô nhập liệu (đường viền, nhãn) sẽ chuyển sang màu lỗi.
    *   `errorText: 'Email không hợp lệ'`

*   **`counterText`**:
    *   Hiển thị một bộ đếm ký tự ở góc dưới bên phải. Nếu bạn muốn ẩn bộ đếm mặc định của `TextField`, hãy gán `counterText: ''` (một chuỗi rỗng).

#### 2. Icons

*   **`icon`**:
    *   Một icon hiển thị **bên ngoài và bên trái** của ô nhập liệu.

*   **`prefixIcon`**:
    *   Một icon hiển thị **bên trong và bên trái** của ô nhập liệu, ngay trước vị trí con trỏ. Thường dùng cho icon username, email.
    *   `prefixIcon: Icon(Icons.person)`

*   **`suffixIcon`**:
    *   Một icon hiển thị **bên trong và bên phải** của ô nhập liệu. Thường dùng cho nút hiển thị/ẩn mật khẩu, hoặc nút xóa nội dung.
    *   `suffixIcon: Icon(Icons.visibility_off)`

#### 3. Đường viền (Borders)

Đây là phần quan trọng nhất để tạo ra các giao diện hiện đại. `InputDecoration` cho phép bạn định nghĩa các đường viền khác nhau cho các trạng thái khác nhau:

*   **`border`**: Đường viền mặc định cho tất cả các trạng thái nếu các trạng thái cụ thể không được định nghĩa.
*   **`enabledBorder`**: Đường viền khi ô nhập liệu đang hoạt động nhưng **không được focus**.
*   **`focusedBorder`**: Đường viền khi người dùng đang **nhấn vào và gõ chữ**.
*   **`errorBorder`**: Đường viền khi có lỗi (`errorText` không `null`).
*   **`focusedErrorBorder`**: Đường viền khi có lỗi và ô nhập liệu đang được focus.

Các loại đường viền phổ biến để gán cho các thuộc tính trên:
*   `OutlineInputBorder()`: Tạo ra một đường viền hình chữ nhật bao quanh ô nhập liệu (phổ biến nhất).
*   `UnderlineInputBorder()`: Tạo ra một đường gạch chân bên dưới ô nhập liệu (kiểu Material cũ).

```dart
// Ví dụ về OutlineInputBorder
enabledBorder: OutlineInputBorder(
  borderRadius: BorderRadius.circular(10.0),
  borderSide: BorderSide(color: Colors.grey, width: 1.0),
),
focusedBorder: OutlineInputBorder(
  borderRadius: BorderRadius.circular(10.0),
  borderSide: BorderSide(color: Colors.blue, width: 2.0),
),
```

#### 4. Màu sắc và Style

*   **`filled`**: Một giá trị `bool`. Nếu là `true`, ô nhập liệu sẽ có màu nền.
*   **`fillColor`**: Màu nền để sử dụng khi `filled: true`.
*   **`labelStyle`**, **`hintStyle`**, **`helperStyle`**, **`errorStyle`**: Cho phép bạn tùy chỉnh `TextStyle` (font, cỡ chữ, màu sắc) cho các loại văn bản tương ứng.

#### 5. Đệm (Padding)

*   **`contentPadding`**: Cho phép bạn kiểm soát khoảng cách từ đường viền vào bên trong đến nội dung (văn bản, hint text).
    `contentPadding: EdgeInsets.symmetric(vertical: 15.0, horizontal: 10.0)`

---

### Ví dụ hoàn chỉnh: Tạo một ô nhập mật khẩu đẹp mắt

Ví dụ này sẽ kết hợp rất nhiều thuộc tính ở trên để tạo ra một `TextField` mật khẩu hiện đại và đầy đủ chức năng.

```dart
import 'package:flutter/material.dart';

class PasswordTextFieldDemo extends StatefulWidget {
  const PasswordTextFieldDemo({super.key});

  @override
  State<PasswordTextFieldDemo> createState() => _PasswordTextFieldDemoState();
}

class _PasswordTextFieldDemoState extends State<PasswordTextFieldDemo> {
  // Biến state để kiểm soát việc ẩn/hiện mật khẩu
  bool _isObscured = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('InputDecoration Demo')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: TextField(
          // Ẩn văn bản khi _isObscured là true
          obscureText: _isObscured,
          decoration: InputDecoration(
            // 1. LabelText
            labelText: 'Mật khẩu',
            labelStyle: const TextStyle(color: Colors.grey),

            // 2. HintText
            hintText: 'Nhập mật khẩu của bạn',

            // 3. HelperText
            helperText: 'Phải chứa ít nhất 1 chữ hoa và 1 số',

            // 4. PrefixIcon
            prefixIcon: const Icon(Icons.lock_outline),

            // 5. SuffixIcon với chức năng ẩn/hiện
            suffixIcon: IconButton(
              icon: Icon(
                _isObscured ? Icons.visibility_off : Icons.visibility,
              ),
              onPressed: () {
                // Cập nhật state để vẽ lại UI
                setState(() {
                  _isObscured = !_isObscured;
                });
              },
            ),

            // 6. Borders
            enabledBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12.0),
              borderSide: const BorderSide(color: Colors.grey, width: 1.5),
            ),
            focusedBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12.0),
              borderSide: const BorderSide(color: Colors.deepPurple, width: 2.0),
            ),
            
            // 7. Màu nền
            filled: true,
            fillColor: Colors.grey.shade100,

            // 8. Padding
            contentPadding: const EdgeInsets.symmetric(vertical: 18.0, horizontal: 15.0),
          ),
        ),
      ),
    );
  }
}
```

### Mẹo chuyên nghiệp: Dùng `InputDecorationTheme`

Nếu ứng dụng của bạn có nhiều `TextField` và bạn muốn chúng có chung một style, đừng lặp lại code `InputDecoration` ở mọi nơi. Thay vào đó, hãy định nghĩa một `InputDecorationTheme` trong `ThemeData` của bạn.

```dart
// trong main.dart
MaterialApp(
  theme: ThemeData(
    // ... các theme khác
    inputDecorationTheme: InputDecorationTheme(
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8.0),
        borderSide: const BorderSide(color: Colors.grey),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(8.0),
        borderSide: const BorderSide(color: Colors.blue),
      ),
      filled: true,
      fillColor: Colors.grey[200],
    ),
  ),
  // ...
)
```
Bây giờ, tất cả các `TextField` trong ứng dụng của bạn sẽ tự động áp dụng style này mà không cần bạn phải khai báo lại `decoration`.

Hy vọng phần giải thích chi tiết này giúp bạn hoàn toàn làm chủ `InputDecoration`

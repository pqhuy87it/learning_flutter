Chào bạn,

`TextButton` là một trong những widget nút bấm cơ bản và được sử dụng rộng rãi nhất trong Flutter. Nó tuân theo hướng dẫn của Material Design, đại diện cho một hành động dưới dạng văn bản đơn giản, không có nền nổi bật.

Dưới đây là giải thích chi tiết về cách sử dụng `TextButton`, từ cơ bản đến nâng cao.

---

### 1. `TextButton` là gì?

`TextButton` là một widget nút bấm có giao diện tối giản, thường chỉ là một đoạn văn bản. Nó có độ "nhấn mạnh" thấp nhất so với `ElevatedButton` (nền nổi) và `OutlinedButton` (có viền). Nó thường được sử dụng cho các hành động phụ, ít quan trọng hơn trên màn hình.

*   **Khi không tương tác**: Chỉ là một đoạn text.
*   **Khi di chuột qua (hover) hoặc nhấn (press)**: Một lớp nền mờ sẽ xuất hiện để cung cấp phản hồi trực quan cho người dùng.

![So sánh các loại Button](https://flutter.github.io/assets-for-api-docs/assets/material/button_types.png)
*(Từ trái qua: ElevatedButton, TextButton, OutlinedButton)*

---

### 2. Cấu trúc cơ bản và các thuộc tính thiết yếu

Một `TextButton` tối thiểu cần có hai thuộc tính:

1.  **`child`**: Nội dung bên trong nút bấm. Thường là một widget `Text`, nhưng cũng có thể là `Icon` hoặc bất kỳ widget nào khác.
2.  **`onPressed`**: Một hàm (`VoidCallback`) sẽ được thực thi khi người dùng nhấn vào nút. **Đây là thuộc tính quan trọng nhất.**
    *   Nếu `onPressed` được cung cấp một hàm, nút sẽ ở trạng thái **enabled** (có thể nhấn).
    *   Nếu `onPressed` có giá trị là `null`, nút sẽ tự động chuyển sang trạng thái **disabled** (bị vô hiệu hóa, mờ đi và không thể nhấn).

#### Ví dụ cơ bản:

```dart
import 'package:flutter/material.dart';

class MyTextButtonExample extends StatelessWidget {
  const MyTextButtonExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("TextButton Demo")),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Nút bấm ở trạng thái có thể nhấn
            TextButton(
              child: const Text('NHẤN VÀO ĐÂY'),
              onPressed: () {
                // Hành động sẽ xảy ra khi nút được nhấn
                print('Nút đã được nhấn!');
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Hành động đã được thực hiện')),
                );
              },
            ),
            const SizedBox(height: 20),
            // Nút bấm ở trạng thái bị vô hiệu hóa
            TextButton(
              child: const Text('NÚT BỊ VÔ HIỆU HÓA'),
              onPressed: null, // Đặt là null để vô hiệu hóa
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 3. Tùy chỉnh giao diện (Styling)

Để tùy chỉnh giao diện của `TextButton`, bạn sử dụng thuộc tính `style`. Cách tốt nhất và hiện đại nhất để tạo style là dùng phương thức `TextButton.styleFrom()`.

Phương thức này cung cấp các tham số dễ hiểu để thay đổi giao diện của nút.

#### Các thuộc tính styling phổ biến trong `TextButton.styleFrom()`:

*   **`foregroundColor`**: Màu của `child` (thường là màu chữ và icon).
*   **`backgroundColor`**: Màu nền của nút, thường chỉ hiện rõ khi người dùng tương tác (hover, focus, press).
*   **`disabledForegroundColor`**: Màu của `child` khi nút bị vô hiệu hóa.
*   **`disabledBackgroundColor`**: Màu nền khi nút bị vô hiệu hóa.
*   **`padding`**: Khoảng đệm giữa viền của nút và `child`.
*   **`textStyle`**: Tùy chỉnh font chữ (kích thước, độ đậm, ...).
*   **`shape`**: Hình dạng của nút. Thường dùng `RoundedRectangleBorder` để bo tròn các góc.

#### Ví dụ về Styling:

```dart
TextButton(
  style: TextButton.styleFrom(
    foregroundColor: Colors.white, // Màu chữ
    backgroundColor: Colors.teal, // Màu nền
    disabledForegroundColor: Colors.grey.withOpacity(0.38),
    disabledBackgroundColor: Colors.grey.withOpacity(0.12),
    padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
    textStyle: const TextStyle(
      fontSize: 18,
      fontWeight: FontWeight.bold,
    ),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(12), // Bo tròn góc
    ),
  ),
  child: const Text('NÚT ĐÃ ĐƯỢC STYLE'),
  onPressed: () {
    print('Nút được style đã được nhấn!');
  },
)
```

---

### 4. `TextButton` với Icon

Để tạo một nút bấm vừa có icon vừa có văn bản, Flutter cung cấp một constructor riêng rất tiện lợi: `TextButton.icon`.

Constructor này yêu cầu 3 thuộc tính chính:

*   **`onPressed`**: Tương tự như trên.
*   **`icon`**: Widget `Icon` bạn muốn hiển thị.
*   **`label`**: Widget `Text` (hoặc widget khác) đi kèm với icon.

#### Ví dụ về `TextButton.icon`:

```dart
TextButton.icon(
  icon: const Icon(Icons.favorite), // Icon trái tim
  label: const Text('Yêu thích'),   // Nhãn văn bản
  onPressed: () {
    print('Nút Yêu thích đã được nhấn!');
  },
  style: TextButton.styleFrom(
    foregroundColor: Colors.red, // Áp dụng màu cho cả icon và text
  ),
)
```

---

### 5. Quản lý trạng thái Enabled/Disabled một cách linh hoạt

Trong thực tế, bạn sẽ thường xuyên bật/tắt một nút bấm dựa trên một điều kiện nào đó (ví dụ: người dùng đã điền đủ thông tin vào form hay chưa). Điều này được thực hiện dễ dàng trong một `StatefulWidget`.

#### Ví dụ về quản lý trạng thái:

```dart
class InteractiveButtonScreen extends StatefulWidget {
  const InteractiveButtonScreen({super.key});

  @override
  State<InteractiveButtonScreen> createState() => _InteractiveButtonScreenState();
}

class _InteractiveButtonScreenState extends State<InteractiveButtonScreen> {
  bool _canSubmit = false; // Biến trạng thái để kiểm soát nút

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Enable/Disable Button")),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Một công tắc để thay đổi trạng thái
            SwitchListTile(
              title: const Text('Cho phép gửi?'),
              value: _canSubmit,
              onChanged: (bool newValue) {
                setState(() {
                  _canSubmit = newValue;
                });
              },
            ),
            const SizedBox(height: 30),
            // Nút bấm sẽ bật/tắt dựa trên biến _canSubmit
            TextButton(
              child: const Text('GỬI'),
              // Sử dụng toán tử 3 ngôi:
              // Nếu _canSubmit là true, gán hàm cho onPressed.
              // Nếu là false, gán null.
              onPressed: _canSubmit ? () {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Đang gửi...')),
                );
              } : null,
              style: TextButton.styleFrom(
                foregroundColor: Colors.blue,
                disabledForegroundColor: Colors.blue.withAlpha(100),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Tóm tắt

| Tính năng | Cách thực hiện |
| :--- | :--- |
| **Hành động khi nhấn** | Cung cấp một hàm cho thuộc tính `onPressed`. |
| **Vô hiệu hóa nút** | Đặt `onPressed: null;`. |
| **Thay đổi giao diện** | Sử dụng thuộc tính `style` với `TextButton.styleFrom(...)`. |
| **Thêm icon** | Sử dụng constructor `TextButton.icon(...)`. |
| **Thay đổi màu chữ** | Dùng `foregroundColor` trong `styleFrom`. |
| **Bo tròn góc** | Dùng `shape: RoundedRectangleBorder(...)` trong `styleFrom`. |

`TextButton` là một công cụ đơn giản nhưng rất mạnh mẽ, và việc nắm vững cách sử dụng và tùy chỉnh nó sẽ giúp bạn xây dựng giao diện người dùng sạch sẽ và hiệu quả.

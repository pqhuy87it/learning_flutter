Chào bạn, rất vui được giải thích về "Styled Widget" trong Flutter.

Trong Flutter, **"Styled Widget"** không phải là một widget cụ thể có sẵn như `Text` hay `Container`. Thay vào đó, nó là một **khái niệm** hoặc một **mẫu thiết kế (design pattern)** mà các lập trình viên sử dụng để tạo ra các widget có thể tái sử dụng với một phong cách (style) đã được định sẵn.

Nói một cách đơn giản, bạn tạo ra một widget tùy chỉnh (custom widget) của riêng mình, gói một widget cơ bản của Flutter (như `ElevatedButton` hay `Text`) và áp dụng tất cả các thuộc tính style mà bạn muốn.

### Tại sao nên sử dụng Styled Widget?

Việc này mang lại rất nhiều lợi ích, đặc biệt là trong các dự án lớn:

1.  **Tính tái sử dụng (Reusability):** Bạn chỉ cần định nghĩa style một lần và sử dụng widget đó ở mọi nơi trong ứng dụng. Không cần phải lặp lại code styling ở nhiều chỗ.
2.  **Tính nhất quán (Consistency):** Đảm bảo tất cả các thành phần giao diện (UI) trong ứng dụng của bạn có một giao diện đồng nhất (ví dụ: tất cả các nút chính đều có cùng màu nền, kích thước chữ, bo góc).
3.  **Dễ bảo trì (Maintainability):** Khi bạn muốn thay đổi thiết kế (ví dụ: đổi màu chủ đạo của ứng dụng), bạn chỉ cần sửa ở một nơi duy nhất – chính là file định nghĩa Styled Widget đó. Mọi nơi sử dụng widget này sẽ tự động được cập nhật.
4.  **Code sạch hơn (Cleaner Code):** Code giao diện của bạn sẽ trở nên gọn gàng và dễ đọc hơn rất nhiều. Thay vì một `Container` với 10 dòng code styling, bạn chỉ cần gọi một widget tùy chỉnh như `PrimaryCard`.

---

### Cách tạo một Styled Widget

Cách phổ biến nhất là tạo một class mới kế thừa từ `StatelessWidget`.

**Hãy xem một ví dụ kinh điển: Tạo một nút chính (`PrimaryButton`) cho ứng dụng.**

**Tình huống "chưa" sử dụng Styled Widget:**
Mỗi khi cần một nút chính, bạn có thể phải viết như thế này:

```dart
ElevatedButton(
  onPressed: () { /* Xử lý logic */ },
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blueAccent, // Màu nền
    foregroundColor: Colors.white, // Màu chữ
    padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(30.0),
    ),
    textStyle: TextStyle(
      fontSize: 16,
      fontWeight: FontWeight.bold,
    ),
  ),
  child: Text('Đăng nhập'),
)
```

Nếu bạn có 10 cái nút như vậy, bạn sẽ phải lặp lại đoạn code `style` này 10 lần. Rất dài dòng và khó bảo trì!

**Tình huống "đã" sử dụng Styled Widget:**

**Bước 1: Tạo Widget `PrimaryButton`**

Tạo một file mới, ví dụ `lib/widgets/primary_button.dart`, và định nghĩa widget của bạn:

```dart
import 'package:flutter/material.dart';

class PrimaryButton extends StatelessWidget {
  // Định nghĩa các thuộc tính cần thiết
  final String text;
  final VoidCallback? onPressed; // VoidCallback là kiểu của hàm không có tham số và không trả về gì

  // Constructor
  const PrimaryButton({
    Key? key,
    required this.text,
    this.onPressed,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // Trả về widget gốc với style đã được định nghĩa sẵn
    return ElevatedButton(
      onPressed: onPressed,
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.blueAccent,
        foregroundColor: Colors.white,
        padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(30.0),
        ),
        textStyle: const TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.bold,
        ),
      ),
      child: Text(text),
    );
  }
}
```

**Bước 2: Sử dụng `PrimaryButton` trong ứng dụng**

Bây giờ, ở bất cứ đâu bạn cần nút đó, bạn chỉ cần gọi nó một cách cực kỳ gọn gàng:

```dart
// Import widget bạn vừa tạo
import 'package:your_app/widgets/primary_button.dart';

// ... trong code giao diện của bạn
Column(
  children: [
    PrimaryButton(
      text: 'Đăng nhập',
      onPressed: () {
        print('Nút Đăng nhập được nhấn!');
      },
    ),
    SizedBox(height: 10),
    PrimaryButton(
      text: 'Đăng ký',
      onPressed: () {
        print('Nút Đăng ký được nhấn!');
      },
    ),
    SizedBox(height: 10),
    // Một nút bị vô hiệu hóa
    PrimaryButton(
      text: 'Gửi',
      onPressed: null, // Truyền null vào onPressed sẽ tự động vô hiệu hóa nút
    ),
  ],
)
```

Như bạn thấy, code giao diện giờ đây sạch sẽ và tập trung vào logic (`text` là gì, `onPressed` làm gì) thay vì các chi tiết styling.

### Mẹo và các phương pháp hay nhất

1.  **Sử dụng `ThemeData`:** Để chuyên nghiệp hơn, thay vì hard-code màu sắc (`Colors.blueAccent`), bạn nên lấy màu từ `Theme.of(context)`. Điều này giúp ứng dụng của bạn dễ dàng hỗ trợ Chế độ Sáng/Tối (Light/Dark Mode).
    ```dart
    // Trong PrimaryButton
    style: ElevatedButton.styleFrom(
      backgroundColor: Theme.of(context).colorScheme.primary, // Lấy màu từ theme
      foregroundColor: Theme.of(context).colorScheme.onPrimary,
      // ...
    )
    ```

2.  **Cho phép tùy chỉnh:** Đôi khi bạn cần một phiên bản hơi khác của widget. Bạn có thể thêm các tham số tùy chọn vào constructor để cho phép ghi đè style mặc định.
    ```dart
    // Thêm thuộc tính color vào PrimaryButton
    final Color? backgroundColor;

    // Trong build method
    backgroundColor: this.backgroundColor ?? Theme.of(context).colorScheme.primary, // Nếu không cung cấp thì dùng màu mặc định
    ```

3.  **Tổ chức thư mục:** Tạo một thư mục riêng như `lib/widgets/` hoặc `lib/common/` để chứa tất cả các Styled Widget của bạn.

### Kết luận

Tóm lại, **Styled Widget** là một kỹ thuật nền tảng trong Flutter giúp bạn xây dựng các ứng dụng có quy mô lớn, dễ bảo trì, và có thiết kế nhất quán. Bằng cách đóng gói style vào các widget tùy chỉnh, bạn đang tuân theo nguyên tắc Lập trình Không lặp lại (DRY - Don't Repeat Yourself) và làm cho code của mình sạch sẽ, chuyên nghiệp hơn rất nhiều.

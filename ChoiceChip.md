Chào bạn, rất vui được giải thích chi tiết về `ChoiceChip`, một widget cực kỳ hữu ích trong Flutter để cho phép người dùng đưa ra một lựa chọn duy nhất từ một nhóm các tùy chọn.

### 1. `ChoiceChip` là gì?

Hãy tưởng tượng `ChoiceChip` như một **nút Radio (RadioButton) trong hình hài của một chiếc Chip**. Mục đích chính của nó là để người dùng **chọn một và chỉ một** tùy chọn từ một danh sách các lựa chọn có sẵn.

Khi một `ChoiceChip` được chọn, các `ChoiceChip` khác trong cùng một nhóm sẽ tự động bị bỏ chọn (tất nhiên, bạn phải tự lập trình logic này, và chúng ta sẽ tìm hiểu ngay sau đây).

**Các trường hợp sử dụng phổ biến:**
*   Chọn kích cỡ cho sản phẩm (S, M, L, XL).
*   Chọn một phương thức vận chuyển (Nhanh, Tiết kiệm).
*   Chọn một mức độ ưu tiên (Thấp, Trung bình, Cao).
*   Chọn một câu trả lời trong một câu hỏi trắc nghiệm.

### 2. So sánh `ChoiceChip` và `FilterChip`

Đây là điểm dễ gây nhầm lẫn nhất. Cả hai đều có trạng thái "được chọn", nhưng mục đích của chúng khác nhau hoàn toàn:

| Tính năng | `ChoiceChip` (Chọn một) | `FilterChip` (Lọc nhiều) |
| :--- | :--- | :--- |
| **Mục đích** | Chọn **một** lựa chọn duy nhất từ một tập hợp. Giống như `RadioButton`. | Chọn **nhiều** bộ lọc từ một danh sách. Giống như `Checkbox`. |
| **Logic** | Khi chọn một cái, những cái khác phải bị bỏ chọn. | Mỗi chip có trạng thái chọn/bỏ chọn độc lập với nhau. |
| **Ví dụ** | Chọn size áo: S, M, L. | Lọc sản phẩm theo: "Áo", "Quần", "Giảm giá". |

### 3. Nguyên tắc hoạt động cốt lõi: Quản lý trạng thái

Giống như `Checkbox` và `RadioButton`, `ChoiceChip` là một "controlled component". Nó không tự biết mình có đang được chọn hay không. Bạn, với tư cách là lập trình viên, phải quản lý trạng thái đó.

**Để một nhóm `ChoiceChip` hoạt động đúng, bạn cần:**
1.  **Một biến trạng thái:** Tạo một biến trong `State` của `StatefulWidget` để lưu trữ giá trị của lựa chọn *hiện tại*. Ví dụ: `String? _selectedSize;`.
2.  **Logic `selected`:** Đối với mỗi `ChoiceChip` bạn tạo ra, thuộc tính `selected` của nó phải được tính toán bằng cách so sánh giá trị của chip đó với biến trạng thái. Ví dụ: `selected: _selectedSize == 'M'`.
3.  **Logic `onSelected`:** Cung cấp một hàm callback cho `onSelected`. Bên trong hàm này, bạn phải gọi `setState` để cập nhật biến trạng thái với giá trị mới khi người dùng nhấn vào một chip.

Do đó, bạn **bắt buộc phải sử dụng `StatefulWidget`**.

### 4. Các thuộc tính quan trọng

```dart
ChoiceChip({
  Key? key,
  required Widget label, // Nội dung chính, thường là Text
  Widget? avatar, // Icon hoặc hình ảnh ở đầu
  required bool selected, // Trạng thái chọn/bỏ chọn
  required void Function(bool)? onSelected, // Callback khi người dùng nhấn
  Color? selectedColor, // Màu nền khi được chọn
  Color? backgroundColor, // Màu nền khi không được chọn
  TextStyle? labelStyle, // Style cho text
  // ... các thuộc tính khác như elevation, shape, padding
})
```

*   `label`: (Bắt buộc) Nội dung của chip.
*   `selected`: (Bắt buộc) Một `bool` cho biết chip này có đang được chọn hay không.
*   `onSelected`: (Bắt buộc) Hàm được gọi khi người dùng nhấn vào chip. Nó nhận một tham số `bool` cho biết trạng thái mới (luôn là `true` khi nhấn vào một `ChoiceChip` chưa được chọn).
*   `avatar`: Hiển thị một widget ở đầu chip.
*   `selectedColor`: Màu nền của chip khi `selected` là `true`.

### 5. Ví dụ hoàn chỉnh: Bộ chọn kích cỡ áo

Đây là một ví dụ đầy đủ và thực tế nhất, cho thấy cách quản lý một nhóm `ChoiceChip`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('ChoiceChip Demo')),
        body: const Center(
          child: SizeSelector(),
        ),
      ),
    );
  }
}

class SizeSelector extends StatefulWidget {
  const SizeSelector({Key? key}) : super(key: key);

  @override
  State<SizeSelector> createState() => _SizeSelectorState();
}

class _SizeSelectorState extends State<SizeSelector> {
  // 1. Danh sách các lựa chọn có sẵn
  final List<String> _sizes = ['S', 'M', 'L', 'XL', 'XXL'];

  // 2. Biến trạng thái để lưu lựa chọn hiện tại.
  // Dùng String? (nullable) vì ban đầu có thể chưa có gì được chọn.
  String? _selectedSize;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text(
          'Vui lòng chọn kích cỡ:',
          style: Theme.of(context).textTheme.titleLarge,
        ),
        const SizedBox(height: 16),
        // Dùng Wrap để các chip tự động xuống dòng
        Wrap(
          spacing: 8.0, // Khoảng cách ngang
          runSpacing: 4.0, // Khoảng cách dọc
          children: _sizes.map((size) {
            return ChoiceChip(
              label: Text(size),
              avatar: const Icon(Icons.check_circle_outline),
              // 3. Logic xác định trạng thái 'selected'
              // Chip này được chọn NẾU giá trị của nó khớp với lựa chọn đã lưu
              selected: _selectedSize == size,
              selectedColor: Colors.deepPurple,
              labelStyle: TextStyle(
                color: _selectedSize == size ? Colors.white : Colors.black,
              ),
              // 4. Logic cập nhật trạng thái khi người dùng nhấn
              onSelected: (bool isSelected) {
                setState(() {
                  // Khi một chip được nhấn, isSelected luôn là true.
                  // Chúng ta cập nhật biến trạng thái với giá trị của chip đó.
                  if (isSelected) {
                    _selectedSize = size;
                  }
                });
              },
            );
          }).toList(),
        ),
        const SizedBox(height: 24),
        Text(
          'Kích cỡ đã chọn: ${_selectedSize ?? 'Chưa chọn'}',
          style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
        ),
      ],
    );
  }
}
```

**Phân tích ví dụ trên:**

1.  **`_sizes`**: Một danh sách đơn giản chứa các tùy chọn.
2.  **`_selectedSize`**: Biến trạng thái quan trọng nhất. Nó lưu `String` của size đang được chọn.
3.  **`selected: _selectedSize == size`**: Đây là "phép màu". Khi `build` chạy, mỗi `ChoiceChip` sẽ tự hỏi: "Giá trị `size` của tôi ('S', 'M', 'L',...) có bằng với giá trị đang được lưu trong `_selectedSize` không?". Chỉ có chip nào trả lời "có" mới được hiển thị là đã chọn.
4.  **`onSelected` và `setState`**: Khi người dùng nhấn vào chip 'L', `onSelected` được gọi. Chúng ta gọi `setState` và cập nhật `_selectedSize = 'L'`. Lệnh `setState` kích hoạt một lần `build` mới. Ở lần `build` này, chip 'L' sẽ thấy `_selectedSize == 'L'` là `true` và tự vẽ lại mình ở trạng thái được chọn, trong khi tất cả các chip khác thấy điều kiện này là `false` và vẫn ở trạng thái bình thường.

### Kết luận

`ChoiceChip` là một công cụ tuyệt vời để tạo ra các giao diện lựa chọn đơn lẻ, trực quan và hiện đại. Chìa khóa để sử dụng nó thành công là nắm vững mô hình quản lý trạng thái:
*   Sử dụng `StatefulWidget`.
*   Tạo một biến duy nhất để lưu trữ lựa chọn hiện tại.
*   Sử dụng biến này để xác định thuộc tính `selected` cho mỗi chip.
*   Cập nhật biến này trong callback `onSelected` bằng `setState`.

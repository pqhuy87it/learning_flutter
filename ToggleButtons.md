Chào bạn! Chắc chắn rồi, `ToggleButtons` là một widget kinh điển trong Flutter. Mặc dù đã có "người kế nhiệm" hiện đại hơn, nhưng nó vẫn rất hữu ích và quan trọng để hiểu. Hãy cùng nhau "mổ xẻ" chi tiết về "người lính già" này nhé!

Hãy tưởng tượng `ToggleButtons` như một dãy các công tắc đèn được gắn liền với nhau. Bạn có thể bật một hoặc nhiều công tắc cùng một lúc.

### 1. `ToggleButtons` là gì?

`ToggleButtons` là một widget cho phép bạn hiển thị một nhóm các nút bấm liền kề nhau. Nó được thiết kế để người dùng có thể chọn một hoặc nhiều tùy chọn từ một nhóm có liên quan.

Nó hoạt động dựa trên một nguyên tắc rất đơn giản: **sự tương ứng giữa hai danh sách (List)**.

### 2. Các thành phần cốt lõi (Bắt buộc phải có)

Để `ToggleButtons` hoạt động, bạn cần cung cấp 3 thuộc tính chính và quản lý chúng trong một `StatefulWidget`:

1.  **`children`**: `List<Widget>` - Đây là danh sách các widget sẽ được hiển thị bên trong mỗi nút. Thường là các `Icon` hoặc `Text`.
2.  **`isSelected`**: `List<bool>` - Đây là **trái tim** của widget. Nó là một danh sách các giá trị boolean (`true`/`false`) để lưu trạng thái "được chọn" hay "không được chọn" của mỗi nút. **Danh sách này phải có độ dài bằng với danh sách `children`**.
3.  **`onPressed`**: `Function(int index)` - Một hàm callback được gọi khi người dùng nhấn vào một trong các nút. Nó trả về `index` (vị trí) của nút vừa được nhấn. Bên trong hàm này, bạn phải gọi `setState` để cập nhật lại danh sách `isSelected`.

### 3. Cách sử dụng: Chọn một (Single Selection)

Đây là trường hợp phổ biến, hoạt động giống như một nhóm `RadioButton`.

**Ví dụ:** Tạo bộ lọc căn chỉnh văn bản (Trái, Giữa, Phải).

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const Scaffold(
        appBar: AppBar(title: Text('ToggleButtons Demo')),
        body: Center(child: TextAlignmentToggle()),
      ),
    );
  }
}

class TextAlignmentToggle extends StatefulWidget {
  const TextAlignmentToggle({super.key});

  @override
  State<TextAlignmentToggle> createState() => _TextAlignmentToggleState();
}

class _TextAlignmentToggleState extends State<TextAlignmentToggle> {
  // 1. Tạo danh sách bool để lưu trạng thái.
  // Ban đầu, nút đầu tiên (index 0) được chọn.
  final List<bool> _isSelected = [true, false, false];

  @override
  Widget build(BuildContext context) {
    return ToggleButtons(
      // 2. Cung cấp danh sách trạng thái
      isSelected: _isSelected,

      // 3. Callback khi một nút được nhấn
      onPressed: (int index) {
        // 4. Logic để đảm bảo chỉ có một nút được chọn
        setState(() {
          // Lặp qua tất cả các nút
          for (int i = 0; i < _isSelected.length; i++) {
            // Nếu vị trí lặp bằng với vị trí nút vừa nhấn, đặt là true
            // Ngược lại, đặt là false
            _isSelected[i] = (i == index);
          }
        });
      },

      // 5. Cung cấp danh sách các widget con (phải cùng độ dài với isSelected)
      children: const <Widget>[
        Icon(Icons.format_align_left),
        Icon(Icons.format_align_center),
        Icon(Icons.format_align_right),
      ],
    );
  }
}
```

**Phân tích ví dụ:**
1.  `_isSelected` là một `List<bool>` có 3 phần tử, tương ứng với 3 `Icon` trong `children`.
2.  Khi người dùng nhấn vào một nút (ví dụ, nút ở giữa có `index = 1`), hàm `onPressed` được gọi.
3.  Bên trong `setState`, vòng lặp `for` sẽ đặt tất cả các giá trị trong `_isSelected` thành `false`, ngoại trừ phần tử tại `index` vừa nhấn, nó sẽ được đặt thành `true`. Kết quả mới sẽ là `[false, true, false]`.
4.  Giao diện được build lại và `ToggleButtons` sẽ hiển thị đúng trạng thái mới.

---

### 4. Cách sử dụng: Chọn nhiều (Multiple Selection)

Hoạt động giống như một nhóm `Checkbox`.

**Ví dụ:** Chọn định dạng văn bản (Đậm, Nghiêng, Gạch chân).

```dart
// ... (Widget StatefulWidget)
class TextFormattingToggle extends StatefulWidget {
  const TextFormattingToggle({super.key});

  @override
  State<TextFormattingToggle> createState() => _TextFormattingToggleState();
}

class _TextFormattingToggleState extends State<TextFormattingToggle> {
  // Trạng thái ban đầu: không có nút nào được chọn
  final List<bool> _isSelected = [false, false, false];

  @override
  Widget build(BuildContext context) {
    return ToggleButtons(
      isSelected: _isSelected,
      onPressed: (int index) {
        // Logic cho chọn nhiều đơn giản hơn rất nhiều
        setState(() {
          // Chỉ cần đảo ngược giá trị boolean tại vị trí đã nhấn
          _isSelected[index] = !_isSelected[index];
        });
      },
      children: const <Widget>[
        Icon(Icons.format_bold),
        Icon(Icons.format_italic),
        Icon(Icons.format_underline),
      ],
    );
  }
}
```

**Phân tích ví dụ:**
Logic trong `onPressed` giờ đây cực kỳ đơn giản. Nếu nút `format_bold` (index 0) đang là `false` và người dùng nhấn vào, `_isSelected[0]` sẽ trở thành `true`. Nếu nhấn lại, nó sẽ trở về `false`. Các nút khác không bị ảnh hưởng.

### 5. Tùy chỉnh giao diện (Styling)

`ToggleButtons` cung cấp nhiều thuộc tính để bạn tùy chỉnh giao diện:

*   `color`: Màu của icon/text khi **không** được chọn.
*   `selectedColor`: Màu của icon/text khi **được** chọn.
*   `fillColor`: Màu nền của nút khi được chọn.
*   `splashColor`: Màu hiệu ứng "giọt nước" khi nhấn.
*   `borderColor`: Màu đường viền.
*   `selectedBorderColor`: Màu đường viền khi được chọn.
*   `borderRadius`: Bo tròn các góc.

```dart
ToggleButtons(
  // ... isSelected, onPressed, children
  color: Colors.grey,
  selectedColor: Colors.white,
  fillColor: Colors.blue,
  splashColor: Colors.blue.withOpacity(0.2),
  borderColor: Colors.grey.shade300,
  selectedBorderColor: Colors.blue,
  borderRadius: const BorderRadius.all(Radius.circular(10)),
)
```

### 6. Hạn chế và "Người kế nhiệm" hiện đại

Mặc dù hữu ích, `ToggleButtons` là một widget thuộc **Material 2** và có một vài hạn chế:
*   Giao diện trông hơi "cũ".
*   Việc quản lý trạng thái bằng một `List<bool>` và `index` đôi khi hơi rườm rà và dễ gây lỗi nếu hai danh sách không khớp nhau.
*   Không linh hoạt bằng: Khó để kết hợp cả icon và text trong một nút một cách đẹp mắt.

**Lời khuyên:**
Đối với các dự án mới sử dụng **Material 3**, bạn nên ưu tiên sử dụng **`SegmentedButton`**. Nó là "người kế nhiệm" hiện đại với các ưu điểm:
*   Thiết kế Material 3 đẹp mắt.
*   An toàn kiểu dữ liệu (type-safe) hơn vì dùng `enum` và `Set` thay vì `index` và `List<bool>`.
*   Dễ dàng kết hợp `icon` và `label` trong mỗi nút.

Tuy nhiên, hiểu cách `ToggleButtons` hoạt động vẫn rất quan trọng, đặc biệt khi bạn làm việc với các dự án cũ hoặc cần một giải pháp nhanh gọn.

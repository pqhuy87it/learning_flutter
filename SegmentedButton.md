Chắc chắn rồi! `SegmentedButton` là một trong những widget mới toanh và cực kỳ "xịn sò" của Material 3 trong Flutter. Nó là sự thay thế hiện đại và linh hoạt hơn rất nhiều so với widget `ToggleButtons` cũ.

Hãy cùng nhau khám phá chi tiết cách sử dụng nó nhé!

### 1. `SegmentedButton` là gì? - Nâng cấp của `ToggleButtons`

Hãy tưởng tượng bạn cần một nhóm các nút bấm liền kề nhau, hoạt động như một bộ điều khiển duy nhất, ví dụ:
*   Chọn chế độ xem: Ngày / Tuần / Tháng
*   Chọn bộ lọc: Tất cả / Đang hoạt động / Đã hoàn thành
*   Chọn các món thêm cho pizza: Phô mai / Nấm / Hành tây

Trước đây, bạn sẽ dùng `ToggleButtons`. Nhưng `SegmentedButton` mang lại:
*   **Thiết kế Material 3:** Giao diện đẹp, hiện đại và tuân thủ nguyên tắc thiết kế mới nhất.
*   **Linh hoạt hơn:** Dễ dàng tùy chỉnh icon, label, và hỗ trợ cả chọn một (single-choice) và chọn nhiều (multi-choice) một cách rõ ràng.
*   **Type-safe (An toàn kiểu dữ liệu):** Sử dụng generics (`<T>`) để làm việc với bất kỳ kiểu dữ liệu nào, giúp code của bạn an toàn và dễ đọc hơn.

### 2. Các thành phần cốt lõi

`SegmentedButton` hoạt động dựa trên 3 yếu tố chính:

1.  **`SegmentedButton<T>`**: Widget cha. Chữ `T` này đại diện cho kiểu dữ liệu định danh (identifier) cho mỗi nút. Nó có thể là `String`, `int`, hoặc tốt nhất là một `enum`.
2.  **`segments`**: Một danh sách (`List`) các `ButtonSegment<T>`. Mỗi `ButtonSegment` đại diện cho một nút trong nhóm.
3.  **`selected`**: Một `Set<T>` chứa giá trị (`value`) của (các) nút đang được chọn. **Lưu ý:** Kể cả khi chỉ chọn một, nó vẫn là một `Set`.
4.  **`onSelectionChanged`**: Một callback được gọi mỗi khi người dùng thay đổi lựa chọn. Nó trả về một `Set<T>` mới, và bạn cần dùng nó để cập nhật state của mình.

### 3. Cách sử dụng: Chọn một (Single Selection)

Đây là trường hợp phổ biến nhất, hoạt động giống như một nhóm `RadioButton`.

**Ví dụ:** Tạo bộ lọc chế độ xem Lịch (Ngày, Tuần, Tháng).

#### Bước 1: Tạo một `enum` để định danh

Sử dụng `enum` là cách làm tốt nhất vì nó an toàn và dễ đọc.

```dart
enum CalendarView { day, week, month }
```

#### Bước 2: Xây dựng trong `StatefulWidget`

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
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.green),
      home: const Scaffold(
        body: Center(child: SingleChoiceSegmentedButton()),
      ),
    );
  }
}

class SingleChoiceSegmentedButton extends StatefulWidget {
  const SingleChoiceSegmentedButton({super.key});

  @override
  State<SingleChoiceSegmentedButton> createState() =>
      _SingleChoiceSegmentedButtonState();
}

class _SingleChoiceSegmentedButtonState
    extends State<SingleChoiceSegmentedButton> {
  // 1. Biến trạng thái để lưu lựa chọn hiện tại.
  // Mặc dù chỉ chọn một, nó vẫn phải là một Set.
  Set<CalendarView> _selection = {CalendarView.day};

  @override
  Widget build(BuildContext context) {
    return SegmentedButton<CalendarView>(
      // 2. Cung cấp trạng thái lựa chọn hiện tại
      selected: _selection,

      // 3. Callback để cập nhật trạng thái
      onSelectionChanged: (Set<CalendarView> newSelection) {
        setState(() {
          // Khi multiSelectionEnabled = false, newSelection luôn chỉ có 1 phần tử.
          _selection = newSelection;
        });
      },

      // 4. Tắt chế độ chọn nhiều
      multiSelectionEnabled: false,

      // 5. Quan trọng: Hiển thị icon check khi được chọn (tùy chọn)
      showSelectedIcon: true, 

      // 6. Danh sách các segment (các nút)
      segments: const <ButtonSegment<CalendarView>>[
        ButtonSegment<CalendarView>(
          value: CalendarView.day, // Giá trị định danh
          label: Text('Ngày'),
          icon: Icon(Icons.calendar_view_day),
        ),
        ButtonSegment<CalendarView>(
          value: CalendarView.week,
          label: Text('Tuần'),
          icon: Icon(Icons.calendar_view_week),
        ),
        ButtonSegment<CalendarView>(
          value: CalendarView.month,
          label: Text('Tháng'),
          icon: Icon(Icons.calendar_view_month),
        ),
      ],
    );
  }
}
```

**Phân tích ví dụ:**
1.  `_selection` là một `Set<CalendarView>` lưu trạng thái.
2.  `onSelectionChanged` nhận về một `Set` mới chứa lựa chọn của người dùng. Chúng ta chỉ cần gọi `setState` để cập nhật `_selection`.
3.  `multiSelectionEnabled: false` đảm bảo người dùng chỉ có thể chọn một mục tại một thời điểm.
4.  Mỗi `ButtonSegment` có một `value` duy nhất (từ `enum` của chúng ta), một `label` (Text), và một `icon`.

---

### 4. Cách sử dụng: Chọn nhiều (Multiple Selection)

Hoạt động giống như một nhóm `Checkbox`.

**Ví dụ:** Chọn các món thêm cho Pizza.

```dart
// (Phần main và MyApp có thể giữ nguyên)

enum Toppings { cheese, mushrooms, onions, bacon }

class MultiChoiceSegmentedButton extends StatefulWidget {
  const MultiChoiceSegmentedButton({super.key});

  @override
  State<MultiChoiceSegmentedButton> createState() =>
      _MultiChoiceSegmentedButtonState();
}

class _MultiChoiceSegmentedButtonState
    extends State<MultiChoiceSegmentedButton> {
  // Trạng thái ban đầu có thể có nhiều lựa chọn
  Set<Toppings> _selection = {Toppings.cheese, Toppings.bacon};

  @override
  Widget build(BuildContext context) {
    return SegmentedButton<Toppings>(
      selected: _selection,
      onSelectionChanged: (Set<Toppings> newSelection) {
        setState(() {
          // Widget tự động xử lý việc thêm/bớt phần tử khỏi Set
          _selection = newSelection;
        });
      },

      // BẬT chế độ chọn nhiều
      multiSelectionEnabled: true,

      // Không hiển thị icon check cho gọn gàng hơn (tùy chọn)
      showSelectedIcon: false,

      segments: const <ButtonSegment<Toppings>>[
        ButtonSegment<Toppings>(
          value: Toppings.cheese,
          label: Text('Phô mai'),
        ),
        ButtonSegment<Toppings>(
          value: Toppings.mushrooms,
          label: Text('Nấm'),
        ),
        ButtonSegment<Toppings>(
          value: Toppings.onions,
          label: Text('Hành tây'),
        ),
        ButtonSegment<Toppings>(
          value: Toppings.bacon,
          label: Text('Thịt xông khói'),
        ),
      ],
    );
  }
}
```

**Phân tích ví dụ:**
Sự khác biệt duy nhất so với chọn một là `multiSelectionEnabled: true`. Logic trong `onSelectionChanged` vẫn y hệt. Flutter sẽ tự động quản lý việc thêm hoặc bớt các `value` ra khỏi `Set` cho bạn mỗi khi người dùng nhấn vào một nút.

### 5. Tùy chỉnh giao diện (Styling)

Bạn có thể tùy chỉnh giao diện của `SegmentedButton` thông qua thuộc tính `style`.

```dart
SegmentedButton<CalendarView>(
  // ... các thuộc tính khác
  style: ButtonStyle(
    // Màu nền khi nút ở trạng thái thường
    backgroundColor: MaterialStateProperty.all(Colors.grey[200]),
    // Màu nền khi nút được chọn
    foregroundColor: MaterialStateProperty.resolveWith<Color?>((states) {
      if (states.contains(MaterialState.selected)) {
        return Colors.white; // Màu icon/text khi được chọn
      }
      return Colors.blue; // Màu icon/text khi ở trạng thái thường
    }),
    side: MaterialStateProperty.all(
      BorderSide(color: Colors.blue.shade200, width: 2),
    ),
  ),
  segments: //...
)
```

### Tóm tắt

*   `SegmentedButton` là widget Material 3 hiện đại để tạo nhóm các nút lựa chọn.
*   Sử dụng `enum` và generic `<T>` để đảm bảo an toàn kiểu dữ liệu.
*   Quản lý trạng thái bằng một `Set<T>` và cập nhật nó trong `onSelectionChanged`.
*   Sử dụng `multiSelectionEnabled` để chuyển đổi giữa chế độ **chọn một** (`false`) và **chọn nhiều** (`true`).
*   Nó rất linh hoạt và dễ dàng tùy chỉnh, là một sự thay thế tuyệt vời cho `ToggleButtons` và các layout tự tạo khác.

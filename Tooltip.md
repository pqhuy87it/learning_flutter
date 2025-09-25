Chào bạn! Chắc chắn rồi! `Tooltip` là một trong những "người hùng thầm lặng" trong Flutter, giúp cải thiện trải nghiệm người dùng (UX) một cách tinh tế mà không tốn nhiều công sức.

Hãy cùng nhau "bóc tách" chi tiết về widget này nhé!

### 1. `Tooltip` là gì? - Lời thì thầm hữu ích

Hãy tưởng tượng bạn đang thiết kế một ứng dụng có nhiều nút bấm chỉ chứa icon (biểu tượng) để tiết kiệm không gian, ví dụ: nút `Lưu` (hình đĩa mềm), nút `Xóa` (hình thùng rác), nút `Chia sẻ`.

Người dùng mới có thể sẽ không hiểu ngay ý nghĩa của tất cả các icon đó. `Tooltip` chính là **lời thì thầm hữu ích** xuất hiện để giải thích cho họ.

*   **Trên Desktop/Web:** Khi người dùng **di chuột (hover)** lên widget.
*   **Trên Mobile:** Khi người dùng **nhấn và giữ (long-press)** widget.

`Tooltip` sẽ hiển thị một hộp thoại nhỏ chứa một đoạn văn bản ngắn (`message`) để giải thích chức năng của widget đó.

### 2. Cách sử dụng cơ bản (Cực kỳ đơn giản!)

Cấu trúc của `Tooltip` vô cùng đơn giản. Bạn chỉ cần **bọc (wrap)** widget mà bạn muốn thêm lời giải thích bằng `Tooltip`.

Nó có 2 thuộc tính chính và bắt buộc:

1.  **`message`**: `String` - Đây chính là nội dung lời giải thích sẽ được hiển thị.
2.  **`child`**: `Widget` - Widget sẽ kích hoạt `Tooltip` (ví dụ: `IconButton`, `Icon`, `Text`...).

**Ví dụ kinh điển: `IconButton` trong `AppBar`**

```dart
import 'package.flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Tooltip Demo'),
          actions: [
            // 1. Bọc IconButton bằng Tooltip
            Tooltip(
              // 2. Cung cấp một message rõ ràng
              message: 'Lưu vào danh sách yêu thích',
              
              // 3. Đặt IconButton làm child
              child: IconButton(
                icon: const Icon(Icons.favorite),
                onPressed: () {
                  // Logic khi nhấn nút
                },
              ),
            ),
            Tooltip(
              message: 'Chia sẻ',
              child: IconButton(
                icon: const Icon(Icons.share),
                onPressed: () {},
              ),
            ),
          ],
        ),
        body: const Center(
          child: Tooltip(
            message: 'Đây là một Flutter Logo!',
            child: FlutterLogo(size: 150),
          ),
        ),
      ),
    );
  }
}
```
**Kết quả:**
*   **Trên máy tính:** Di chuột qua icon trái tim, một hộp thoại nhỏ với dòng chữ "Lưu vào danh sách yêu thích" sẽ hiện ra.
*   **Trên điện thoại:** Nhấn và giữ icon trái tim, hộp thoại tương tự sẽ xuất hiện cùng với một rung nhẹ (haptic feedback).

### 3. Tùy chỉnh nâng cao - Khi bạn muốn "Tooltip" trông "xịn" hơn

`Tooltip` cung cấp rất nhiều thuộc tính để bạn tùy chỉnh giao diện và hành vi.

#### a. Tùy chỉnh Giao diện (Styling)

*   **`decoration`**: `BoxDecoration` - Đây là thuộc tính mạnh nhất để thay đổi giao diện. Bạn có thể đổi màu nền, thêm viền, bo góc, đổ bóng.
    ```dart
    Tooltip(
      message: 'Custom Tooltip',
      decoration: BoxDecoration(
        color: Colors.blue.withOpacity(0.9),
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.blueAccent),
      ),
      child: //...
    )
    ```
*   **`textStyle`**: `TextStyle` - Thay đổi style của chữ bên trong tooltip (cỡ chữ, màu chữ, font chữ...).
    ```dart
    Tooltip(
      message: 'Styled Text',
      textStyle: TextStyle(
        fontSize: 16,
        color: Colors.white,
        fontWeight: FontWeight.bold,
      ),
      child: //...
    )
    ```
*   **`height`**: `double` - Chiều cao của hộp tooltip.
*   **`padding`**: `EdgeInsets` - Khoảng đệm bên trong hộp tooltip, giữa viền và chữ.
*   **`margin`**: `EdgeInsets` - Khoảng cách từ widget `child` đến hộp tooltip.

#### b. Tùy chỉnh Vị trí và Hành vi

*   **`verticalOffset`**: `double` - Tinh chỉnh khoảng cách dọc giữa widget và tooltip. Số dương đẩy tooltip ra xa, số âm kéo lại gần.
*   **`preferBelow`**: `bool` (mặc định là `true`) - Quyết định xem tooltip nên ưu tiên xuất hiện ở **bên dưới** hay **bên trên** widget.
    *   `true`: Mặc định, tooltip sẽ cố gắng hiện ở dưới. Nếu không đủ không gian, nó sẽ tự động nhảy lên trên.
    *   `false`: Ưu tiên hiện ở trên. Nếu không đủ không gian, nó sẽ nhảy xuống dưới.
*   **`waitDuration`**: `Duration` - Khoảng thời gian chờ (delay) từ lúc người dùng hover/long-press cho đến khi tooltip xuất hiện. Hữu ích để tránh tooltip hiện lên quá nhanh khi người dùng chỉ vô tình lướt chuột qua.
    ```dart
    Tooltip(
      message: 'Delayed Tooltip',
      waitDuration: const Duration(seconds: 1), // Chờ 1 giây
      child: //...
    )
    ```
*   **`showDuration`**: `Duration` - Khoảng thời gian tooltip sẽ hiển thị trước khi tự động biến mất.
*   **`enableFeedback`**: `bool` - Bật/tắt phản hồi rung (haptic feedback) khi long-press trên mobile.

### 4. Khi nào nên và không nên dùng `Tooltip`?

**Nên dùng:**
*   **`IconButton` và các nút chỉ có icon:** Đây là trường hợp sử dụng số một.
*   **Văn bản bị cắt ngắn (`...`):** Bạn có thể bọc một `Text` bị `overflow` bằng `Tooltip` để hiển thị toàn bộ nội dung khi người dùng hover/long-press.
*   **Các yếu tố UI phức tạp:** Khi một biểu đồ hoặc một thành phần giao diện có ý nghĩa không rõ ràng ngay lập tức.

**Không nên dùng:**
*   **Cho thông tin quan trọng, bắt buộc:** `Tooltip` mang tính chất "khám phá". Nếu thông tin đó là cốt lõi, hãy hiển thị nó bằng một `Text` widget thông thường.
*   **Trên một widget đã có hành động long-press khác:** Ví dụ: một item trong danh sách có hành động long-press để xóa. Thêm `Tooltip` sẽ gây xung đột hành vi.
*   **Với nội dung quá dài:** `Tooltip` chỉ dành cho những lời giải thích ngắn gọn, súc tích.

### Tóm tắt

`Tooltip` là một công cụ đơn giản nhưng cực kỳ hiệu quả để cải thiện khả năng sử dụng (usability) của ứng dụng. Bằng cách bọc widget của bạn và cung cấp một `message` ngắn gọn, bạn đã giúp người dùng hiểu rõ hơn về giao diện mà không làm nó trở nên lộn xộn.

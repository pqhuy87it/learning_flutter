Chào bạn, rất vui được giải thích chi tiết về widget `Card` trong Flutter. Đây là một widget cực kỳ phổ biến và hữu ích để tạo ra các giao diện người dùng sạch sẽ, có cấu trúc theo chuẩn Material Design.

### 1. `Card` là gì?

Hãy tưởng tượng `Card` trong Flutter giống như một tấm thẻ bài hoặc một danh thiếp ngoài đời thực. Nó là một tấm panel của Material Design với các đặc điểm nổi bật:

*   **Các góc bo tròn nhẹ:** Tạo cảm giác mềm mại.
*   **Hiệu ứng đổ bóng (Elevation):** Tạo ra một lớp chiều sâu, làm cho tấm thẻ trông như đang "nổi" lên trên nền phía sau.

**Mục đích chính của `Card` là để chứa và nhóm các thông tin liên quan lại với nhau thành một khối trực quan duy nhất.** Nó giúp tách biệt nội dung một cách rõ ràng, cải thiện cấu trúc và khả năng đọc của giao diện.

### 2. So sánh `Card` và `Container`

Đây là câu hỏi rất phổ biến. Khi nào nên dùng `Card`, khi nào nên dùng `Container`?

| Tính năng | `Container` | `Card` |
| :--- | :--- | :--- |
| **Mục đích chính** | Một widget đa năng để trang trí, định vị, và định kích thước. Giống như thẻ `<div>` trong web. | Một widget chuyên dụng để hiển thị một khối nội dung theo chuẩn Material Design. |
| **Hiệu ứng đổ bóng** | ❌ Không có sẵn. Phải tự làm phức tạp bằng `BoxDecoration`. | ✅ **Có sẵn** thông qua thuộc tính `elevation`. Đây là lý do chính để chọn `Card`. |
| **Góc bo tròn** | Có, thông qua `decoration: BoxDecoration(borderRadius: ...)` | ✅ **Có sẵn** và là mặc định. Dễ dàng tùy chỉnh qua thuộc tính `shape`. |
| **Tuân thủ Material**| Không bắt buộc. | ✅ Tuân thủ chặt chẽ. Nó được xây dựng trên widget `Material`. |
| **Nền tảng** | Là một widget cơ bản. | Được xây dựng dựa trên `Material`, cung cấp "canvas" cho các hiệu ứng như `InkWell`. |

**Tóm lại:**
*   Dùng `Container` khi bạn cần một khối hộp đơn giản để trang trí (màu sắc, viền, gradient) hoặc để căn chỉnh (`padding`, `margin`).
*   Dùng `Card` khi bạn muốn nhóm một cụm thông tin và làm nó "nổi bật" lên khỏi nền với hiệu ứng đổ bóng và góc bo tròn một cách nhanh chóng và đúng chuẩn Material.

### 3. Các thuộc tính quan trọng

```dart
Card({
  Key? key,
  Color? color, // Màu nền của Card
  Color? shadowColor, // Màu của bóng đổ
  double? elevation, // Độ "nổi" của Card, quyết định độ đậm của bóng
  ShapeBorder? shape, // Hình dạng của Card, thường dùng để bo góc
  EdgeInsetsGeometry? margin, // Khoảng cách bên ngoài Card
  Clip clipBehavior = Clip.none, // Cách cắt nội dung con (child)
  Widget? child, // Nội dung bên trong Card
  bool borderOnForeground = true,
})
```

*   `child`: (Quan trọng nhất) Widget con chứa nội dung của thẻ. Thường là một `Column`, `Row`, hoặc `ListTile`.
*   `elevation`: Một giá trị `double` quyết định độ cao của thẻ so với nền. Giá trị càng cao, bóng đổ càng lớn và đậm. Giá trị `0` sẽ không có bóng. Giá trị phổ biến là `2.0`, `4.0`, `8.0`.
*   `shape`: Cho phép bạn tùy chỉnh hình dạng của thẻ. Phổ biến nhất là dùng `RoundedRectangleBorder` để thay đổi độ bo của các góc.
    ```dart
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(15.0), // Bo góc 15px
    ),
    ```
*   `margin`: Khoảng cách từ các cạnh của `Card` đến các widget xung quanh nó. Mặc định `Card` có một `margin` là `4.0` ở tất cả các cạnh.
*   `color`: Màu nền của `Card`.
*   `clipBehavior`: **Rất quan trọng khi `child` là một `Image`**. Mặc định, `Card` không cắt nội dung con của nó. Nếu bạn đặt một `Image` với các góc vuông vào một `Card` có góc bo tròn, các góc của ảnh sẽ tràn ra ngoài.
    *   Hãy đặt `clipBehavior: Clip.antiAlias` để cắt `child` theo đúng hình dạng bo tròn của `Card`, tạo ra một kết quả mượt mà.

### 4. Ví dụ thực tế

#### Ví dụ 1: Card thông tin người dùng đơn giản

Đây là cách sử dụng `Card` với `ListTile`, một sự kết hợp rất phổ biến.

```dart
import 'package:flutter/material.dart';

class UserProfileCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[200], // Nền xám để Card nổi bật
      appBar: AppBar(title: Text('Card Demo')),
      body: Center(
        child: Card(
          margin: EdgeInsets.all(20.0),
          elevation: 5.0,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(10.0),
          ),
          child: ListTile(
            leading: CircleAvatar(
              backgroundImage: NetworkImage('https://i.pravatar.cc/150?img=3'),
              radius: 30,
            ),
            title: Text(
              'Jane Doe',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            subtitle: Text('Software Engineer at FAI.ABC'),
            trailing: Icon(Icons.arrow_forward_ios),
            onTap: () {
              print('Card tapped!');
            },
          ),
        ),
      ),
    );
  }
}
```

#### Ví dụ 2: Card sản phẩm phức tạp hơn

Ví dụ này cho thấy cách kết hợp `Card` với `Column`, `Image` và sử dụng `clipBehavior`.

```dart
import 'package:flutter/material.dart';

class ProductCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.blueGrey[50],
      appBar: AppBar(title: Text('Product Card')),
      body: Center(
        child: Card(
          // Cắt nội dung con theo hình dạng bo tròn của Card
          clipBehavior: Clip.antiAlias,
          elevation: 8.0,
          shadowColor: Colors.blueGrey.withOpacity(0.5),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16.0),
          ),
          child: Container(
            width: 300,
            child: Column(
              mainAxisSize: MainAxisSize.min, // Cột chỉ chiếm không gian cần thiết
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Hình ảnh sản phẩm
                Image.network(
                  'https://images.unsplash.com/photo-1523275335684-37898b6baf30',
                  height: 180,
                  width: double.infinity,
                  fit: BoxFit.cover,
                ),
                // Phần nội dung text
                Padding(
                  padding: const EdgeInsets.all(16.0),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Đồng hồ sang trọng',
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      SizedBox(height: 8),
                      Text(
                        'Một chiếc đồng hồ thanh lịch, phù hợp cho mọi dịp. Chống nước và có độ bền cao.',
                        style: TextStyle(color: Colors.grey[700]),
                      ),
                    ],
                  ),
                ),
                // Phần nút hành động
                Padding(
                  padding: const EdgeInsets.fromLTRB(16, 0, 16, 16),
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.end,
                    children: [
                      TextButton(
                        child: Text('CHI TIẾT'),
                        onPressed: () {},
                      ),
                      SizedBox(width: 8),
                      ElevatedButton(
                        child: Text('THÊM VÀO GIỎ'),
                        onPressed: () {},
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```
**Phân tích ví dụ 2:**
1.  `clipBehavior: Clip.antiAlias` là rất quan trọng ở đây. Nếu không có nó, các góc vuông của `Image.network` sẽ tràn ra ngoài các góc bo tròn 16px của `Card`.
2.  `Column` được sử dụng để sắp xếp các phần (ảnh, text, nút) theo chiều dọc.
3.  `Padding` được sử dụng bên trong `Column` để tạo khoảng cách cho phần text và nút, thay vì đặt `padding` cho cả `Card`.

### Kết luận

`Card` là một widget đơn giản nhưng cực kỳ hiệu quả để tổ chức giao diện của bạn. Bằng cách sử dụng các thuộc tính như `elevation`, `shape`, và `margin`, bạn có thể dễ dàng tạo ra các thành phần UI có cấu trúc, phân cấp và đẹp mắt, tuân thủ theo ngôn ngữ thiết kế Material và mang lại trải nghiệm người dùng tốt hơn.

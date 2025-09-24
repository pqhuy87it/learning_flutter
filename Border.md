Chào bạn! Rất vui được giúp bạn tìm hiểu chi tiết về cách sử dụng `Border` trong Flutter. Đây là một trong những thuộc tính trang trí (decoration) quan trọng nhất để tạo ra các giao diện đẹp mắt và chuyên nghiệp.

`Border` thường được sử dụng bên trong thuộc tính `decoration` của một `Container`, cụ thể là trong `BoxDecoration`. Hãy cùng đi qua từng cách sử dụng từ cơ bản đến nâng cao nhé!

### 1. Cách sử dụng cơ bản nhất: `Border.all()`

Đây là cách phổ biến và đơn giản nhất để tạo một đường viền đồng nhất cho cả 4 cạnh của một widget.

**Mục đích:** Khi bạn muốn tất cả các cạnh (trên, dưới, trái, phải) có cùng màu sắc, độ dày và kiểu dáng.

**Các thuộc tính chính:**
*   `color`: Màu của đường viền (mặc định là `Colors.black`).
*   `width`: Độ dày của đường viền (mặc định là `1.0`).
*   `style`: Kiểu đường viền, thường là `BorderStyle.solid` (nét liền) hoặc `BorderStyle.none` (không có viền).
*   `strokeAlign`: Vị trí của đường viền so với cạnh của box. Có thể là `BorderSide.strokeAlignInside` (mặc định), `strokeAlignCenter`, hoặc `strokeAlignOutside`.

**Ví dụ:**
```dart
Container(
  height: 150,
  width: 150,
  decoration: BoxDecoration(
    color: Colors.lightBlue[100], // Màu nền của box
    border: Border.all(
      color: Colors.black,      // Màu của đường viền
      width: 3.0,               // Độ dày của đường viền
      style: BorderStyle.solid, // Kiểu đường viền
    ),
  ),
  child: Center(child: Text('Border.all()')),
)
```
**Kết quả:** Bạn sẽ có một hình vuông màu xanh nhạt với một đường viền đen, nét liền, dày 3.0 pixel bao quanh.

---

### 2. Tùy chỉnh từng cạnh riêng biệt: `Border()`

Khi bạn muốn mỗi cạnh của box có một kiểu đường viền khác nhau (ví dụ: chỉ có viền trên và dưới, hoặc mỗi viền một màu).

**Mục đích:** Toàn quyền kiểm soát từng cạnh một.

**Cách hoạt động:** Constructor `Border()` nhận các tham số `top`, `right`, `bottom`, `left`. Mỗi tham số này là một đối tượng `BorderSide`.

`BorderSide` là đối tượng định nghĩa kiểu dáng cho **một cạnh duy nhất**, với các thuộc tính tương tự như `Border.all()`: `color`, `width`, `style`.

**Ví dụ:** Tạo một box chỉ có viền trên màu đỏ và viền dưới màu xanh.
```dart
Container(
  height: 150,
  width: 150,
  decoration: BoxDecoration(
    color: Colors.yellow[100],
    border: Border(
      top: BorderSide(
        color: Colors.red,
        width: 5.0,
      ),
      bottom: BorderSide(
        color: Colors.blue,
        width: 2.0,
      ),
    ),
  ),
  child: Center(child: Text('Border()')),
)
```
**Kết quả:** Một hình vuông màu vàng nhạt với viền trên màu đỏ dày 5.0 và viền dưới màu xanh dương dày 2.0. Hai cạnh trái và phải không có viền.

---

### 3. Tạo viền đối xứng: `Border.symmetric()`

Đây là cách viết tắt tiện lợi khi bạn muốn hai cạnh đối xứng (ngang hoặc dọc) có cùng kiểu đường viền.

**Mục đích:** Tạo viền cho cặp cạnh trên/dưới hoặc trái/phải.

**Các thuộc tính:**
*   `vertical`: Áp dụng `BorderSide` cho cạnh trái và phải.
*   `horizontal`: Áp dụng `BorderSide` cho cạnh trên và dưới.

**Ví dụ:** Tạo viền cho hai cạnh dọc (trái và phải).
```dart
Container(
  height: 150,
  width: 150,
  decoration: BoxDecoration(
    color: Colors.green[100],
    border: Border.symmetric(
      vertical: BorderSide(
        color: Colors.green,
        width: 4.0,
      ),
    ),
  ),
  child: Center(child: Text('Border.symmetric()')),
)
```
**Kết quả:** Một hình vuông màu xanh lá nhạt với hai đường viền dọc ở bên trái và phải màu xanh lá cây đậm, dày 4.0.

---

### 4. Kết hợp `Border` với Bo góc (`BorderRadius`)

Đây là một kỹ thuật cực kỳ phổ biến. `Border` sẽ tự động đi theo các góc được bo tròn nếu bạn định nghĩa `borderRadius` trong cùng `BoxDecoration`.

**Lưu ý:** `borderRadius` là một thuộc tính **ngang hàng** với `border` bên trong `BoxDecoration`.

**Ví dụ:** Tạo một box có viền và các góc được bo tròn.
```dart
Container(
  height: 150,
  width: 150,
  decoration: BoxDecoration(
    color: Colors.purple[100],
    border: Border.all(
      color: Colors.purple,
      width: 2.0,
    ),
    // Thêm borderRadius ở đây
    borderRadius: BorderRadius.circular(20), // Bo tròn tất cả các góc
  ),
  child: Center(child: Text('Border + Radius')),
)
```
Bạn cũng có thể bo tròn từng góc riêng lẻ bằng `BorderRadius.only()`:
```dart
borderRadius: BorderRadius.only(
  topLeft: Radius.circular(25),
  bottomRight: Radius.circular(25),
),
```

---

### 5. Các loại Border nâng cao: `ShapeBorder`

Ngoài `Border` dùng trong `BoxDecoration`, Flutter còn có một khái niệm trừu tượng hơn là `ShapeBorder`. Nó không chỉ vẽ đường viền mà còn định hình cả widget. `ShapeBorder` thường được dùng với các widget như `Card`, `Material`, hoặc `ShapeDecoration`.

Các lớp `ShapeBorder` phổ biến:
*   `RoundedRectangleBorder`: Tạo hình chữ nhật với các góc bo tròn (rất giống `BoxDecoration` với `borderRadius`).
*   `CircleBorder`: Tạo viền hình tròn.
*   `StadiumBorder`: Tạo viền hình "viên thuốc" (hình chữ nhật có hai đầu bán nguyệt).
*   `BeveledRectangleBorder`: Tạo viền với các góc được vát chéo.

**Ví dụ với `ShapeDecoration`:**
```dart
Container(
  height: 150,
  width: 150,
  decoration: ShapeDecoration(
    color: Colors.orange[100],
    shape: CircleBorder(
      side: BorderSide(
        color: Colors.orange,
        width: 4.0,
      ),
    ),
  ),
  child: Center(child: Text('CircleBorder')),
)
```
**Kết quả:** Một hình tròn màu cam nhạt với đường viền màu cam đậm bao quanh.

### Tóm tắt khi nào nên dùng loại nào

| Trường hợp sử dụng | Nên dùng |
| :--- | :--- |
| Cần viền đơn giản, đồng nhất 4 cạnh | `Border.all()` |
| Mỗi cạnh có một kiểu viền khác nhau | `Border()` với các `BorderSide` riêng |
| Chỉ cần viền cho 2 cạnh đối xứng | `Border.symmetric()` |
| Cần viền kết hợp bo góc | `Border` + `borderRadius` trong `BoxDecoration` |
| Cần tạo viền cho các hình dạng phức tạp (tròn, viên thuốc) | `ShapeDecoration` với các loại `ShapeBorder` |

Hy vọng giải thích chi tiết này sẽ giúp bạn nắm vững cách sử dụng `Border` trong các dự án Flutter của mình. Chúc bạn code vui vẻ

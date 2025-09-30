Chào bạn,

Rất vui được giải thích chi tiết về **Operators (Toán tử)** trong Flutter. Thực chất, các toán tử này thuộc về ngôn ngữ Dart, là nền tảng của Flutter. Việc hiểu rõ và sử dụng thành thạo các toán tử sẽ giúp bạn viết code logic một cách ngắn gọn, hiệu quả và dễ đọc hơn rất nhiều.

### Toán tử là gì?

Toán tử là những **ký hiệu đặc biệt** mà trình biên dịch hiểu được, dùng để thực hiện các phép toán trên các giá trị và biến. Ví dụ, trong phép tính `5 + 3`, dấu `+` chính là một toán tử.

Chúng ta sẽ chia các toán tử thành các nhóm chính để dễ theo dõi.

---

### 1. Arithmetic Operators (Toán tử số học)

Dùng để thực hiện các phép toán quen thuộc.

| Toán tử | Tên gọi | Mô tả | Ví dụ |
| :--- | :--- | :--- | :--- |
| `+` | Cộng | Cộng hai giá trị. | `5 + 3` → `8` |
| `-` | Trừ | Trừ hai giá trị. | `5 - 3` → `2` |
| `*` | Nhân | Nhân hai giá trị. | `5 * 3` → `15` |
| `/` | Chia | Chia hai giá trị, kết quả là `double`. | `5 / 2` → `2.5` |
| `~/` | Chia lấy nguyên | Chia hai giá trị, kết quả là `int`. | `5 ~/ 2` → `2` |
| `%` | Chia lấy dư (Modulo) | Lấy phần dư của phép chia. | `5 % 2` → `1` |
| `++` | Tăng | Tăng giá trị của biến lên 1. | `var a = 5; a++;` → `a` là `6` |
| `--` | Giảm | Giảm giá trị của biến xuống 1. | `var a = 5; a--;` → `a` là `4` |

**Ví dụ trong Flutter:**
```dart
// Tính toán kích thước của một widget
double screenWidth = 360.0;
double padding = 16.0;
int columnCount = 3;

double itemWidth = (screenWidth - padding * 2) / columnCount;
print(itemWidth); // Tính toán chiều rộng cho mỗi item trong một GridView

// Dùng `~/` để tính số dòng cần thiết
int totalItems = 10;
int itemsPerRow = 3;
int rowCount = (totalItems + itemsPerRow - 1) ~/ itemsPerRow; // Mẹo tính số dòng
print('Số dòng cần: $rowCount'); // Output: 4
```

---

### 2. Equality and Relational Operators (Toán tử so sánh và quan hệ)

Dùng để so sánh các giá trị, kết quả luôn là một giá trị `bool` (`true` hoặc `false`).

| Toán tử | Tên gọi | Ví dụ |
| :--- | :--- | :--- |
| `==` | Bằng | `5 == 5` → `true` |
| `!=` | Không bằng | `5 != 3` → `true` |
| `>` | Lớn hơn | `5 > 3` → `true` |
| `<` | Nhỏ hơn | `5 < 3` → `false` |
| `>=` | Lớn hơn hoặc bằng | `5 >= 5` → `true` |
| `<=` | Nhỏ hơn hoặc bằng | `5 <= 3` → `false` |

**Ví dụ trong Flutter:**
```dart
int unreadMessages = 5;

// Hiển thị một badge thông báo nếu có tin nhắn chưa đọc
if (unreadMessages > 0) {
  // showBadge();
  print('Hiển thị badge thông báo');
}

bool isUserLoggedIn = true;
if (isUserLoggedIn == true) { // Có thể viết gọn là: if (isUserLoggedIn)
  // showUserProfile();
  print('Hiển thị profile người dùng');
}
```

---

### 3. Logical Operators (Toán tử logic)

Dùng để kết hợp các biểu thức `bool`.

| Toán tử | Tên gọi | Mô tả |
| :--- | :--- | :--- |
| `!` | Phủ định (NOT) | Đảo ngược giá trị bool. `!true` → `false`. |
| `&&` | Và (AND) | Trả về `true` nếu **cả hai** vế đều `true`. |
| `\|\|` | Hoặc (OR) | Trả về `true` nếu **ít nhất một** vế là `true`. |

**Ví dụ trong Flutter:**
```dart
bool isLoggedIn = true;
bool hasSubscription = false;

// Hiển thị nội dung premium chỉ khi người dùng đã đăng nhập VÀ có gói thuê bao
if (isLoggedIn && hasSubscription) {
  print('Chào mừng thành viên premium!');
} else {
  print('Vui lòng đăng ký để xem nội dung này.');
}

bool isScreenLoading = false;
if (!isScreenLoading) {
  print('Hiển thị nội dung chính.');
}
```

---

### 4. Assignment Operators (Toán tử gán)

Dùng để gán giá trị cho biến.

| Toán tử | Ví dụ | Tương đương với |
| :--- | :--- | :--- |
| `=` | `a = 5;` | |
| `+=` | `a += 2;` | `a = a + 2;` |
| `-=` | `a -= 2;` | `a = a - 2;` |
| `*=` | `a *= 2;` | `a = a * 2;` |
| `/=` | `a /= 2;` | `a = a / 2;` |
| `??=` | `a ??= 5;` | `if (a == null) { a = 5; }` (Sẽ nói rõ hơn ở phần Null-aware) |

---

### 5. Type Test Operators (Toán tử kiểm tra kiểu)

Dùng để kiểm tra kiểu dữ liệu của một đối tượng tại thời điểm chạy.

| Toán tử | Tên gọi | Mô tả |
| :--- | :--- | :--- |
| `as` | Ép kiểu (Typecast) | Ép một đối tượng sang một kiểu khác. Sẽ ném ra lỗi nếu ép kiểu thất bại. |
| `is` | Là kiểu | Trả về `true` nếu đối tượng là một thể hiện của kiểu đó. |
| `is!` | Không là kiểu | Trả về `true` nếu đối tượng không phải là một thể hiện của kiểu đó. |

**Ví dụ:**
```dart
Object myValue = 'Hello Flutter';

if (myValue is String) {
  // Dart thông minh, tự động "thăng cấp" myValue thành String trong khối này
  print(myValue.toUpperCase());
}

// Tránh dùng 'as' nếu không chắc chắn, vì nó có thể gây crash
try {
  (myValue as int).isEven;
} catch (e) {
  print('Ép kiểu thất bại: $e');
}
```

---

### 6. Conditional Operators (Toán tử điều kiện)

Cung cấp một cách ngắn gọn để viết các biểu thức `if-else`.

#### a. Toán tử ba ngôi (Ternary Operator): `condition ? expr1 : expr2`
Nếu `condition` là `true`, trả về `expr1`. Ngược lại, trả về `expr2`.

**Ví dụ trong Flutter (cực kỳ phổ biến):**
```dart
bool isLoggedIn = false;

// Thay vì viết:
// Widget button;
// if (isLoggedIn) {
//   button = Text('Đăng xuất');
// } else {
//   button = Text('Đăng nhập');
// }

// Bạn có thể viết gọn trong cây widget:
Center(
  child: isLoggedIn ? Text('Chào mừng trở lại!') : ElevatedButton(onPressed: () {}, child: Text('Đăng nhập')),
)
```

#### b. Toán tử If-null: `expr1 ?? expr2`
Nếu `expr1` khác `null`, trả về giá trị của `expr1`. Ngược lại, trả về giá trị của `expr2`.

**Ví dụ trong Flutter:**
```dart
String? username; // username có thể null
// String displayName = username; // Lỗi! Không thể gán String? cho String

// Cung cấp một giá trị mặc định nếu username là null
String displayName = username ?? 'Khách';
print(displayName); // Output: Khách
```

---

### 7. Cascade Notation (Toán tử thác nước): `..`

Đây là một toán tử rất đặc biệt và mạnh mẽ của Dart. Nó cho phép bạn thực hiện một chuỗi các thao tác trên cùng một đối tượng.

**Ví dụ (Không dùng cascade):**
```dart
var paint = Paint();
paint.color = Colors.blue;
paint.strokeWidth = 5.0;
paint.style = PaintingStyle.stroke;
```

**Ví dụ (Dùng cascade `..`):**
```dart
var paint = Paint()
  ..color = Colors.blue
  ..strokeWidth = 5.0
  ..style = PaintingStyle.stroke;

// Rất hữu ích khi cấu hình các đối tượng phức tạp như Path, Canvas, List...
var path = Path()
  ..moveTo(10, 10)
  ..lineTo(100, 100)
  ..lineTo(10, 100)
  ..close();
```

---

### 8. Null-aware Operators (Toán tử xử lý Null)

Cực kỳ quan trọng trong Dart với tính năng Null Safety.

| Toán tử | Tên gọi | Mô tả |
| :--- | :--- | :--- |
| `?.` | Null-aware access | Nếu đối tượng khác `null`, thực hiện thao tác. Ngược lại, trả về `null` và không làm gì cả (tránh crash). |
| `??` | If-null | Đã giải thích ở trên. Cung cấp giá trị mặc định. |
| `??=` | Null-aware assignment | Gán giá trị cho biến chỉ khi biến đó đang là `null`. |
| `!` | Null assertion | Khẳng định với trình biên dịch rằng bạn **chắc chắn 100%** biến đó không `null` tại thời điểm đó. **Hãy cẩn thận khi dùng!** |

**Ví dụ trong Flutter:**
```dart
class User {
  String name;
  Address? address; // Địa chỉ có thể null
  User(this.name, {this.address});
}
class Address {
  String street;
  Address(this.street);
}

User? currentUser; // Người dùng hiện tại có thể chưa đăng nhập (null)

// 1. Sử dụng `?.` để truy cập an toàn
// Nếu currentUser là null, toàn bộ biểu thức trả về null, không bị crash.
String? streetName = currentUser?.address?.street;
print(streetName); // Output: null

// 2. Sử dụng `??` để cung cấp giá trị mặc định
String userCity = currentUser?.address?.street ?? 'Không rõ';
print(userCity); // Output: Không rõ

// 3. Sử dụng `??=` để khởi tạo lười
Map<String, String> cache = {};
void addToCache(String key, String value) {
  cache[key] ??= value; // Chỉ gán nếu key chưa tồn tại trong cache
}
```

Việc hiểu và vận dụng các toán tử này một cách chính xác sẽ giúp bạn viết code logic cho ứng dụng Flutter của mình một cách hiệu quả và an toàn.

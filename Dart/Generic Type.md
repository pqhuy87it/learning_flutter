Chào bạn,

Rất vui được giải thích chi tiết về **Generic Type (Kiểu dữ liệu chung)** trong Flutter/Dart. Đây là một trong những tính năng mạnh mẽ nhất của ngôn ngữ, giúp bạn viết code an toàn hơn, linh hoạt hơn và có khả năng tái sử dụng cao.

### 1. Generic là gì? (Một cách hiểu đơn giản)

Hãy tưởng tượng bạn có một chiếc hộp.
*   Một chiếc hộp thông thường có thể chứa **bất cứ thứ gì**: một quả táo, một cuốn sách, một chiếc điện thoại. Khi bạn lấy đồ ra, bạn phải kiểm tra xem nó là thứ gì trước khi sử dụng. Đây là cách làm việc với `Object` hoặc `dynamic`.
*   Bây giờ, hãy tưởng tượng bạn có một chiếc **"hộp chuyên dụng"**: một "hộp đựng táo". Bạn biết chắc chắn rằng bên trong nó chỉ có táo. Bạn không thể vô tình bỏ một cuốn sách vào đó, và khi lấy ra, bạn không cần kiểm tra, bạn biết ngay đó là một quả táo.

**Generic Type** chính là cách để bạn tạo ra những "chiếc hộp chuyên dụng" đó trong code. Nó cho phép bạn viết các lớp (class) hoặc hàm (function) hoạt động với nhiều kiểu dữ liệu khác nhau, nhưng vẫn đảm bảo **an toàn kiểu (type safety)** tại thời điểm biên dịch (compile-time).

Ký hiệu phổ biến nhất cho generic là `<T>`, trong đó `T` là một **"trình giữ chỗ" (placeholder)** cho một kiểu dữ liệu cụ thể sẽ được cung cấp sau. Bạn có thể dùng bất kỳ chữ cái nào, nhưng `T` (Type), `E` (Element), `K` (Key), `V` (Value) là các quy ước phổ biến.

### 2. Vấn đề mà Generic giải quyết

Hãy xem một ví dụ *không* dùng generic. Giả sử chúng ta muốn tạo một lớp để lưu trữ một giá trị bất kỳ.

```dart
class DataHolder {
  Object? data; // Sử dụng Object để có thể chứa mọi thứ

  DataHolder(this.data);
}

void main() {
  var stringHolder = DataHolder('Xin chào');
  var numberHolder = DataHolder(123);

  // Vấn đề bắt đầu ở đây
  // Trình biên dịch không biết kiểu dữ liệu thực sự là gì
  String myString = stringHolder.data as String; // Phải ép kiểu (cast) một cách tường minh
  print(myString.toUpperCase());

  // Rủi ro tiềm ẩn!
  // Giả sử ta nhầm lẫn
  int myNumber = stringHolder.data as int; // Lỗi! Sẽ crash ứng dụng khi chạy (runtime error)
                                           // Vì không thể ép kiểu 'String' thành 'int'
}
```

**Vấn đề:**
1.  **Mất an toàn kiểu:** Trình biên dịch không thể phát hiện lỗi. Lỗi chỉ xảy ra khi ứng dụng đang chạy, gây ra crash.
2.  **Code rườm rà:** Phải liên tục ép kiểu (`as Type`) ở khắp nơi.

### 3. Giải pháp với Generic

Bây giờ, hãy viết lại lớp `DataHolder` bằng cách sử dụng generic.

```dart
// Sử dụng <T> để biến nó thành một lớp generic
class DataHolder<T> {
  T data; // Kiểu dữ liệu của 'data' bây giờ là T

  DataHolder(this.data);
}

void main() {
  // Khi tạo đối tượng, ta chỉ định kiểu dữ liệu cụ thể
  // Dart thường có thể tự suy ra kiểu, nhưng viết rõ ràng sẽ tốt hơn
  var stringHolder = DataHolder<String>('Xin chào');
  var numberHolder = DataHolder<int>(123);

  // An toàn tuyệt đối!
  String myString = stringHolder.data; // Không cần ép kiểu! Dart biết chắc chắn đây là String.
  print(myString.toUpperCase());

  // Lỗi được phát hiện ngay lập tức tại thời điểm biên dịch!
  // int myNumber = stringHolder.data; // Lỗi! A value of type 'String' can't be assigned to a variable of type 'int'.
                                      // Ứng dụng của bạn thậm chí sẽ không chạy được, giúp bạn sửa lỗi sớm.
}
```

**Lợi ích:**
1.  **An toàn kiểu (Type Safety):** Lỗi được phát hiện ngay khi bạn đang code, không phải đợi đến lúc chạy.
2.  **Tái sử dụng code (Code Reusability):** Bạn chỉ cần viết lớp `DataHolder<T>` một lần và có thể sử dụng nó cho `String`, `int`, `User`, `Product`, hay bất kỳ kiểu dữ liệu nào khác.
3.  **Code sạch hơn:** Không cần ép kiểu (`as`) lung tung.

---

### 4. Các trường hợp sử dụng Generic trong Flutter/Dart

#### a. Collections (Phổ biến nhất)
Bạn đã và đang sử dụng generic mỗi ngày mà có thể không nhận ra!
*   `List<T>`: Một danh sách chỉ chứa các phần tử kiểu `T`.
    ```dart
    List<String> names = ['An', 'Bình', 'Chi'];
    // names.add(123); // Lỗi! Không thể thêm int vào List<String>.
    ```
*   `Map<K, V>`: Một map có các khóa (key) kiểu `K` và các giá trị (value) kiểu `V`.
    ```dart
    Map<String, int> studentScores = {
      'An': 9,
      'Bình': 8,
    };
    ```
*   `Set<E>`: Một tập hợp chỉ chứa các phần tử duy nhất kiểu `E`.

#### b. Hàm Generic (Generic Functions)
Bạn có thể tạo một hàm hoạt động trên nhiều kiểu dữ liệu khác nhau.

```dart
// Hàm này có thể lấy phần tử đầu tiên của bất kỳ danh sách nào
T? getFirstElement<T>(List<T> list) {
  if (list.isEmpty) {
    return null;
  }
  return list.first;
}

void main() {
  List<String> names = ['An', 'Bình'];
  String? firstName = getFirstElement(names); // Dart tự suy ra T là String
  print(firstName); // Output: An

  List<int> numbers = [10, 20, 30];
  int? firstNumber = getFirstElement(numbers); // Dart tự suy ra T là int
  print(firstNumber); // Output: 10
}
```

#### c. Ví dụ thực tế trong Flutter: Hàm gọi API

Đây là một mẫu cực kỳ mạnh mẽ. Bạn có thể viết một hàm gọi API và parse JSON chung cho nhiều loại model khác nhau.

```dart
import 'dart.convert';

// Giả sử bạn có các lớp model
class User {
  final String name;
  User.fromJson(Map<String, dynamic> json) : name = json['name'];
}
class Product {
  final String title;
  Product.fromJson(Map<String, dynamic> json) : title = json['title'];
}

// Hàm generic để fetch và parse dữ liệu
// T là kiểu model (User, Product,...)
// fromJson là một hàm nhận Map và trả về một đối tượng kiểu T
Future<T> fetchAndParse<T>(String url, T Function(Map<String, dynamic>) fromJson) async {
  // Giả lập việc gọi API
  // final response = await http.get(Uri.parse(url));
  // Giả lập response JSON
  String jsonString;
  if (url.contains('users')) {
    jsonString = '{"name": "Sơn Tùng M-TP"}';
  } else {
    jsonString = '{"title": "Điện thoại XYZ"}';
  }

  final Map<String, dynamic> jsonMap = json.decode(jsonString);
  return fromJson(jsonMap);
}

void main() async {
  // Lấy User
  User user = await fetchAndParse('api/users/1', (json) => User.fromJson(json));
  print(user.name); // Output: Sơn Tùng M-TP

  // Lấy Product
  Product product = await fetchAndParse('api/products/123', (json) => Product.fromJson(json));
  print(product.title); // Output: Điện thoại XYZ
}
```

### 5. Ràng buộc kiểu Generic (`extends`)

Đôi khi, bạn muốn giới hạn các kiểu dữ liệu có thể được sử dụng với generic của bạn. Ví dụ, bạn có một hàm tính tổng giá của các mặt hàng, bạn cần đảm bảo rằng mọi đối tượng được truyền vào đều có thuộc tính `price`.

Sử dụng từ khóa `extends` để ràng buộc `T`.

```dart
// Tạo một lớp cơ sở hoặc abstract class
abstract class Item {
  double get price;
}

class Product extends Item {
  final String name;
  @override
  final double price;

  Product(this.name, this.price);
}

class Service extends Item {
  final String serviceName;
  @override
  final double price;

  Service(this.serviceName, this.price);
}

// Ràng buộc T phải là một kiểu kế thừa từ Item
double calculateTotalPrice<T extends Item>(List<T> items) {
  double total = 0.0;
  for (var item in items) {
    total += item.price; // An toàn! Vì T chắc chắn có thuộc tính 'price'.
  }
  return total;
}

void main() {
  var cart = [
    Product('Laptop', 1200.0),
    Service('Cài đặt phần mềm', 50.0),
    Product('Chuột', 25.0),
  ];
  
  // List<String> names = ['a', 'b'];
  // calculateTotalPrice(names); // Lỗi! String không kế thừa từ Item.

  print('Tổng giá trị giỏ hàng: ${calculateTotalPrice(cart)}'); // Output: 1275.0
}
```

### 6. Generics trong các Widget Flutter

*   **`FutureBuilder<T>` / `StreamBuilder<T>`**:
    Khi bạn dùng `FutureBuilder<String>`, bạn báo cho Flutter biết rằng future sẽ trả về một `String`. Do đó, `snapshot.data` sẽ có kiểu là `String?`, giúp bạn truy cập mà không cần ép kiểu.
    ```dart
    FutureBuilder<String>(
      future: fetchUserData(), // Giả sử hàm này trả về Future<String>
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          // snapshot.data ở đây là kiểu String?, không phải dynamic
          return Text(snapshot.data!.toUpperCase());
        }
        return CircularProgressIndicator();
      },
    )
    ```
*   **State Management (Provider, BLoC, Riverpod...):**
    Generics là trái tim của các thư viện quản lý trạng thái.
    `context.watch<UserModel>()` (Provider): Bạn yêu cầu Provider tìm và cung cấp một đối tượng có kiểu chính xác là `UserModel`.

### Kết luận

Generic Type là một công cụ thiết yếu để viết code Dart/Flutter chất lượng cao.
*   **Bắt đầu với:** Hiểu cách sử dụng chúng trong Collections (`List<T>`, `Map<K, V>`).
*   **Tiến xa hơn:** Bắt đầu tạo các lớp và hàm generic của riêng bạn để giảm sự lặp lại code (DRY - Don't Repeat Yourself).
*   **Nâng cao:** Sử dụng ràng buộc `extends` để tạo ra các API generic mạnh mẽ và an toàn hơn.

Việc nắm vững generic sẽ giúp code của bạn không chỉ chạy đúng mà còn dễ đọc, dễ bảo trì và ít lỗi hơn rất nhiều.

Records (hay "bản ghi") là một tính năng mới và cực kỳ hữu ích được giới thiệu trong **Dart 3**, và do đó nó được dùng rộng rãi trong Flutter.

Hiểu đơn giản, **Record là một kiểu dữ liệu ẩn danh (anonymous), bất biến (immutable) cho phép bạn gom nhiều giá trị (thuộc nhiều kiểu khác nhau) vào chung một đối tượng duy nhất** mà không cần phải tạo một `class` hay `struct` riêng.

-----

## 1\. Tại sao Records ra đời? (Vấn đề nó giải quyết)

Trước khi có Records, nếu bạn muốn một hàm trả về nhiều giá trị (ví dụ: tên và tuổi của người dùng), bạn phải:

1.  **Tạo một lớp (Class):** Viết rất nhiều code thừa (boilerplate) chỉ để chứa 2 biến.
    ```dart
    class UserInfo {
      final String name;
      final int age;
      UserInfo(this.name, this.age);
    }
    UserInfo fetchUser() {
      return UserInfo('John', 30);
    }
    ```
2.  **Dùng `List<dynamic>`:** Mất an toàn về kiểu dữ liệu (bạn không biết `list[0]` là `String` hay `int`) và khó đọc (`list[1]` là gì?).
    ```dart
    List<dynamic> fetchUser() {
      return ['John', 30];
    }
    var user = fetchUser();
    var name = user[0]; // Không rõ ràng
    ```
3.  **Dùng `Map<String, dynamic>`:** Dễ gõ sai key (`"name"` vs `"Name"`), chậm hơn và cũng không an toàn về kiểu.
    ```dart
    Map<String, dynamic> fetchUser() {
      return {'name': 'John', 'age': 30};
    }
    var user = fetchUser();
    var name = user['name']; // Dễ gõ sai
    ```

**Records giải quyết tất cả vấn đề này:** Nó cho phép bạn tạo một "gói" dữ liệu tạm thời, an toàn về kiểu và rất ngắn gọn.

-----

## 2\. Cú pháp và Cách sử dụng

Cú pháp của Record rất đơn giản, chỉ cần dùng dấu ngoặc đơn `()`.

### Khai báo một Record

Bạn chỉ cần đặt các giá trị vào trong `()`, phân cách bởi dấu phẩy.

```dart
// Một record chứa 1 String, 1 int, 1 bool
var myRecord = ('Hello', 123, true);

// Bạn cũng có thể khai báo tường minh kiểu dữ liệu
(String, int, bool) typedRecord = ('Hello', 123, true);
```

### Truy cập phần tử (Positional Fields)

Các phần tử trong record được truy cập bằng cú pháp `$ + vị trí` (bắt đầu từ 1, không phải 0).

```dart
var myRecord = ('Hello', 123, true);

print(myRecord.$1); // Output: Hello
print(myRecord.$2); // Output: 123
print(myRecord.$3); // Output: true
```

### Record với trường được đặt tên (Named Fields)

Đây là cách dùng mạnh mẽ và dễ đọc hơn. Bạn đặt tên cho các trường bằng cú pháp `tên: giá trị`.

```dart
// Khai báo record với các trường được đặt tên
var user = (name: 'Alice', age: 25, isVerified: true);

// Bạn cũng có thể khai báo kiểu tường minh
({String name, int age, bool isVerified}) admin = (name: 'Bob', age: 40, isVerified: false);
```

### Truy cập phần tử (Named Fields)

Bạn truy cập chúng như thuộc tính của một đối tượng, dùng dấu `.`.

```dart
var user = (name: 'Alice', age: 25);

print(user.name); // Output: Alice
print(user.age);  // Output: 25
```

Bạn có thể kết hợp cả hai loại (positional và named) trong cùng một record, nhưng điều này ít phổ biến hơn.

```dart
(String, int, {String role}) admin = ('Bob', 40, role: 'Admin');
print(admin.$1);   // Output: Bob
print(admin.role); // Output: Admin
```

-----

## 3\. Đặc điểm chính của Records

1.  **Ẩn danh (Anonymous):** Bạn không cần đặt tên `class` cho nó. Kiểu dữ liệu của nó được định nghĩa ngay tại chỗ, ví dụ `(String, int)`.
2.  **Bất biến (Immutable):** Bạn **không thể** thay đổi giá trị của các trường sau khi record đã được tạo.
    ```dart
    var user = (name: 'Alice', age: 25);
    // user.age = 26; // LỖI: Không thể gán giá trị mới
    ```
    Nếu muốn thay đổi, bạn phải tạo một record mới.
3.  **So sánh (Equality):** Records đã tích hợp sẵn toán tử `==` và `hashCode`. Hai records bằng nhau nếu tất cả các trường của chúng bằng nhau.
    ```dart
    var r1 = (name: 'Alice', age: 25);
    var r2 = (name: 'Alice', age: 25);

    print(r1 == r2); // Output: true (Điều này không đúng với Class)
    ```

-----

## 4\. Ứng dụng mạnh nhất: Trả về nhiều giá trị từ hàm

Đây là nơi Records tỏa sáng. Nó giúp code của bạn sạch sẽ và an toàn hơn rất nhiều.

```dart
// Hàm trả về 1 record (gồm 1 String và 1 int)
(String, int) fetchUserInfo() {
  // Giả lập lấy dữ liệu
  final name = 'John Doe';
  final age = 30;
  return (name, age);
}

void main() {
  // Lấy kết quả trả về
  var userInfo = fetchUserInfo();
  
  // Truy cập kết quả
  print(userInfo.$1); // John Doe
  print(userInfo.$2); // 30

  // ----------
  
  // Dùng record được đặt tên (named fields) để code dễ đọc hơn
  ({String name, int age}) fetchNamedUser() {
    return (name: 'Jane Doe', age: 28);
  }

  var namedUser = fetchNamedUser();
  print(namedUser.name); // Jane Doe
  print(namedUser.age);  // 28
}
```

### Destructuring (Tách cấu trúc) với Pattern Matching

Bạn có thể "giải nén" record ra các biến riêng lẻ ngay lập tức, làm code cực kỳ gọn gàng.

```dart
// 1. Giải nén khi khai báo biến
var (name, age) = fetchUserInfo(); // Tách (String, int) ra 2 biến
print(name); // John Doe
print(age);  // 30

// 2. Giải nén record có tên
var (:name, :age) = fetchNamedUser(); // Cú pháp :name, :age
print(name); // Jane Doe
print(age);  // 28
```

-----

## 5\. Records KHÔNG phải là gì?

  * **Không phải là Class:** Chúng không thể có phương thức (methods), không có `this`, không thể `extends` hay `implements`. Chúng chỉ là một "cái túi" để chứa dữ liệu.
  * **Không phải là List:** `List` dùng để chứa *nhiều* giá trị *cùng kiểu* (hoặc `dynamic`) và có thể thay đổi kích thước. `Record` dùng để chứa *một nhóm cố định* các giá trị *khác kiểu*.
  * **Không phải là Map:** `Map` là một tập hợp các cặp `key-value`. Key thường là `String` và có thể thêm/bớt. `Record` có số trường cố định, an toàn về kiểu và truy cập qua `.` hoặc `$`.

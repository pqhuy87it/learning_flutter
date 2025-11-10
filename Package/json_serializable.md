`json_serializable` là một package tự động tạo code để chuyển đổi giữa đối tượng Dart và JSON, giúp tiết kiệm thời gian và giảm lỗi so với việc viết code thủ công.

Dưới đây là hướng dẫn từng bước để sử dụng `json_serializable` trong dự án Flutter của bạn.

-----

### 1\. Thêm Dependency

Thêm các package cần thiết vào file `pubspec.yaml` của bạn.

```yaml
dependencies:
  flutter:
    sdk: flutter
  json_annotation: ^4.9.0 # Dùng để đánh dấu các class cần tạo code

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.13 # Công cụ chạy quá trình tạo code
  json_serializable: ^6.9.0 # Package tạo code JSON serialization
```

*Lưu ý: Hãy kiểm tra phiên bản mới nhất trên [pub.dev](https://pub.dev/packages/json_serializable).*

Sau đó, chạy lệnh để tải các package về:

```bash
flutter pub get
```

-----

### 2\. Tạo Model Class

Tạo một file Dart cho model của bạn, ví dụ `user.dart`.

```dart
import 'package:json_annotation/json_annotation.dart';

// Phần này cực kỳ quan trọng. Nó cho phép file này truy cập các thành phần private
// trong file code được tạo ra tự động.
// Tên file phải khớp với tên file hiện tại + .g.dart
part 'user.g.dart';

// Annotation này đánh dấu class User cần được tạo code serialization
@JsonSerializable()
class User {
  // Các trường dữ liệu của bạn
  final String name;
  final String email;
  
  // Bạn có thể tùy chỉnh tên key trong JSON nếu nó khác với tên biến
  @JsonKey(name: 'created_at_timestamp')
  final int createdAt;

  User({required this.name, required this.email, required this.createdAt});

  // Factory constructor để tạo đối tượng User từ một map JSON.
  // Nó sẽ gọi hàm được tạo tự động `_$UserFromJson`.
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);

  // Phương thức để chuyển đối tượng User thành map JSON.
  // Nó sẽ gọi hàm được tạo tự động `_$UserToJson`.
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

Lúc này, bạn sẽ thấy các lỗi báo đỏ ở `part 'user.g.dart'`, `_$UserFromJson`, và `_$UserToJson`. Đừng lo lắng, điều này là bình thường vì file `.g.dart` chưa được tạo ra.

-----

### 3\. Chạy Lệnh Tạo Code

Mở terminal tại thư mục gốc của dự án và chạy lệnh sau:

  * **Tạo code một lần:**

    ```bash
    dart run build_runner build --delete-conflicting-outputs
    ```

    *(Thêm `--delete-conflicting-outputs` để tự động ghi đè nếu có xung đột file cũ)*

  * **Tự động tạo lại code mỗi khi file thay đổi (khuyên dùng trong quá trình phát triển):**

    ```bash
    dart run build_runner watch --delete-conflicting-outputs
    ```

Sau khi chạy xong, bạn sẽ thấy file `user.g.dart` xuất hiện bên cạnh file `user.dart`, và các lỗi báo đỏ sẽ biến mất.

-----

### 4\. Sử Dụng

Bây giờ bạn có thể dễ dàng chuyển đổi giữa JSON và đối tượng Dart.

```dart
import 'dart:convert';
import 'user.dart';

void main() {
  // Giả lập một chuỗi JSON từ server
  String jsonString = '{"name": "Nguyen Van A", "email": "a@example.com", "created_at_timestamp": 1678886400}';

  // 1. Từ JSON String -> Map -> User Object
  Map<String, dynamic> userMap = jsonDecode(jsonString);
  User user = User.fromJson(userMap);

  print('Tên User: ${user.name}'); // Output: Tên User: Nguyen Van A
  print('Email User: ${user.email}'); // Output: Email User: a@example.com

  // 2. Từ User Object -> Map -> JSON String
  Map<String, dynamic> jsonOutput = user.toJson();
  String jsonStringOutput = jsonEncode(jsonOutput);

  print(jsonStringOutput);
  // Output: {"name":"Nguyen Van A","email":"a@example.com","created_at_timestamp":1678886400}
}
```

-----

### Một số tùy chọn nâng cao hữu ích với `@JsonSerializable`

Bạn có thể tùy chỉnh cách `json_serializable` hoạt động bằng cách truyền tham số vào annotation:

  * **`explicitToJson: true`**: Hữu ích khi class của bạn chứa các class con khác cũng dùng `json_serializable`. Nó đảm bảo `toJson()` được gọi đệ quy trên các đối tượng con.
    ```dart
    @JsonSerializable(explicitToJson: true)
    class User { ... }
    ```
  * **`fieldRename: FieldRename.snake`**: Tự động chuyển đổi tên biến từ camelCase (Dart) sang snake\_case (JSON) mà không cần dùng `@JsonKey(name: ...)` cho từng trường.
    ```dart
    @JsonSerializable(fieldRename: FieldRename.snake)
    class User {
      final String firstName; // Sẽ map với key JSON 'first_name'
      ...
    }
    ```
  * **`includeIfNull: false`**: Không bao gồm các trường có giá trị `null` khi tạo JSON.

Bạn có muốn tìm hiểu thêm về cách xử lý các kiểu dữ liệu phức tạp hoặc nested object (đối tượng lồng nhau) không?

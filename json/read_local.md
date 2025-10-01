Chào bạn,

Chắc chắn rồi! Đọc và phân tích (parse) file JSON là một tác vụ cực kỳ phổ biến trong Flutter. Dưới đây là hướng dẫn chi tiết từng bước để bạn có thể đọc một file JSON được lưu cục bộ trong thư mục `assets` của ứng dụng.

Chúng ta sẽ tiếp tục sử dụng model `User` từ câu hỏi trước của bạn.

---

### Các bước tổng quan

1.  **Chuẩn bị file JSON**: Tạo một file `.json` và đặt nó vào thư mục `assets` của dự án.
2.  **Khai báo Assets**: Khai báo thư mục `assets` trong file `pubspec.yaml` để Flutter biết về sự tồn tại của nó.
3.  **Tạo Model Class**: Tạo một lớp Dart (như lớp `User` của bạn) với một factory constructor `fromJson` để chuyển đổi dữ liệu từ JSON sang đối tượng Dart.
4.  **Viết mã đọc file**: Sử dụng `rootBundle` để tải nội dung file JSON dưới dạng chuỗi (String).
5.  **Phân tích (Parse) JSON**: Dùng `dart:convert` để chuyển đổi chuỗi JSON thành đối tượng Dart (`Map` hoặc `List`).
6.  **Chuyển đổi sang Model**: Sử dụng factory `fromJson` để tạo các instance của lớp Model từ dữ liệu đã phân tích.
7.  **Hiển thị lên UI**: Sử dụng `FutureBuilder` để xử lý tác vụ bất đồng bộ và hiển thị dữ liệu lên giao diện người dùng.

---

### Hướng dẫn chi tiết

#### Bước 1: Tạo file JSON và thêm vào dự án

1.  Trong thư mục gốc của dự án Flutter, hãy tạo một thư mục mới tên là `assets`.
2.  Bên trong thư mục `assets`, tạo một file mới tên là `user_data.json`.
3.  Dán nội dung JSON sau vào file `user_data.json`. Đây là ví dụ cho một đối tượng người dùng duy nhất.

    **`assets/user_data.json`**
    ```json
    {
      "id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
      "name": "Nguyễn Văn An",
      "email": "nguyen.van.an@example.com",
      "password": "$2a$10$abcdefghijklmnopqrstuvwx.yzABCDEFGHIJKLMNOPQRSTUVWXYZ012345",
      "address": "123 Đường Lê Lợi, Quận 1, TP. Hồ Chí Minh",
      "type": "user",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6Ik5ndXnhu4VuIFbEg24gQW4iLCJpYXQiOjE1MTYyMzkwMjJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "cart": [
        {
          "productId": "prod_1122",
          "productName": "Áo Thun Cotton",
          "quantity": 2,
          "price": 250000
        }
      ]
    }
    ```

Cấu trúc thư mục của bạn sẽ trông như thế này:

```
your_flutter_project/
├── assets/
│   └── user_data.json
├── lib/
│   └── main.dart
├── pubspec.yaml
└── ... (các file và thư mục khác)
```

#### Bước 2: Khai báo Assets trong `pubspec.yaml`

Mở file `pubspec.yaml` và tìm đến phần `flutter:`. Bỏ ghi chú (uncomment) phần `assets:` và thêm đường dẫn đến thư mục `assets/` của bạn.

**Quan trọng:** Việc thụt lề trong file YAML rất quan trọng. `assets:` phải thẳng hàng với `uses-material-design`.

```yaml
flutter:
  uses-material-design: true

  # Để thêm assets vào ứng dụng, thêm một mục assets như sau:
  assets:
    - assets/
```

Sau khi lưu file `pubspec.yaml`, Flutter IDE (như VS Code hoặc Android Studio) sẽ tự động chạy `flutter pub get`. Nếu không, bạn hãy chạy thủ công trong terminal.

#### Bước 3: Cập nhật Model Class với `fromJson`

Bây giờ, chúng ta sẽ thêm một "factory constructor" tên là `fromJson` vào lớp `User`. Constructor này sẽ nhận một `Map<String, dynamic>` (kết quả của việc parse JSON) và tạo ra một đối tượng `User`.

```dart
class User {
  final String id;
  final String name;
  final String email;
  final String password;
  final String address;
  final String type;
  final String token;
  final List<dynamic> cart;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.password,
    required this.address,
    required this.type,
    required this.token,
    required this.cart,
  });

  // Factory constructor để tạo một instance User từ một map (JSON object)
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
      password: json['password'],
      address: json['address'],
      type: json['type'],
      token: json['token'],
      cart: json['cart'],
    );
  }
}
```

#### Bước 4, 5, 6 & 7: Đọc, Parse và Hiển thị dữ liệu

Bây giờ chúng ta sẽ kết hợp tất cả lại trong file `main.dart`. Chúng ta sẽ tạo một hàm để đọc file JSON, và sau đó sử dụng `FutureBuilder` để hiển thị dữ liệu lên màn hình.

Mở file `lib/main.dart` và thay thế nội dung bằng code sau:

```dart
import 'dart:convert'; // Thư viện để mã hóa và giải mã JSON
import 'package:flutter/material.dart';
import 'package:flutter/services.dart'; // Thư viện để truy cập assets (rootBundle)

// Dán lớp User của bạn vào đây (đã có fromJson)
class User {
  final String id;
  final String name;
  final String email;
  final String password;
  final String address;
  final String type;
  final String token;
  final List<dynamic> cart;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.password,
    required this.address,
    required this.type,
    required this.token,
    required this.cart,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
      password: json['password'],
      address: json['address'],
      type: json['type'],
      token: json['token'],
      cart: List<dynamic>.from(json['cart']),
    );
  }
}

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: UserProfileScreen(),
    );
  }
}

class UserProfileScreen extends StatefulWidget {
  const UserProfileScreen({super.key});

  @override
  State<UserProfileScreen> createState() => _UserProfileScreenState();
}

class _UserProfileScreenState extends State<UserProfileScreen> {
  // Hàm này sẽ đọc và parse file JSON
  // Nó trả về một Future<User> vì đây là một tác vụ bất đồng bộ
  Future<User> readJsonData() async {
    // Đọc nội dung file JSON từ assets
    final String jsonString = await rootBundle.loadString('assets/user_data.json');
    
    // Parse chuỗi JSON thành một Map<String, dynamic>
    final data = jsonDecode(jsonString);
    
    // Chuyển đổi Map thành một đối tượng User bằng factory constructor
    return User.fromJson(data);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Thông tin người dùng từ JSON"),
      ),
      // Sử dụng FutureBuilder để xử lý dữ liệu bất đồng bộ
      body: FutureBuilder<User>(
        future: readJsonData(), // Gọi hàm đọc dữ liệu
        builder: (context, snapshot) {
          // Trường hợp 1: Đang chờ dữ liệu
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          // Trường hợp 2: Có lỗi xảy ra
          else if (snapshot.hasError) {
            return Center(child: Text("Đã xảy ra lỗi: ${snapshot.error}"));
          }
          // Trường hợp 3: Đã có dữ liệu
          else if (snapshot.hasData) {
            final user = snapshot.data!; // Lấy dữ liệu người dùng
            return Padding(
              padding: const EdgeInsets.all(16.0),
              child: ListView(
                children: [
                  ListTile(title: const Text("ID"), subtitle: Text(user.id)),
                  ListTile(title: const Text("Tên"), subtitle: Text(user.name)),
                  ListTile(title: const Text("Email"), subtitle: Text(user.email)),
                  ListTile(title: const Text("Địa chỉ"), subtitle: Text(user.address)),
                  ListTile(title: const Text("Loại tài khoản"), subtitle: Text(user.type)),
                  const SizedBox(height: 10),
                  Text("Giỏ hàng (${user.cart.length} sản phẩm):", style: Theme.of(context).textTheme.titleLarge),
                  // Hiển thị các sản phẩm trong giỏ hàng
                  for (var item in user.cart)
                    Card(
                      margin: const EdgeInsets.symmetric(vertical: 4),
                      child: ListTile(
                        title: Text(item['productName']),
                        subtitle: Text("Số lượng: ${item['quantity']}"),
                        trailing: Text("${item['price']} VND"),
                      ),
                    ),
                ],
              ),
            );
          }
          // Trường hợp 4: Không có dữ liệu (hiếm khi xảy ra nếu file JSON hợp lệ)
          else {
            return const Center(child: Text("Không có dữ liệu"));
          }
        },
      ),
    );
  }
}
```

Bây giờ, hãy chạy ứng dụng của bạn. Màn hình sẽ hiển thị một vòng tròn tải trong giây lát, sau đó hiển thị thông tin chi tiết của người dùng được đọc từ file `user_data.json`.

### Mở rộng: Đọc một danh sách (List) các đối tượng JSON

Nếu file JSON của bạn chứa một mảng các người dùng, bạn chỉ cần thay đổi một chút ở hàm `readJsonData`.

**Ví dụ file `users_list.json`**:
```json
[
  { "id": "1", "name": "Người dùng A", ... },
  { "id": "2", "name": "Người dùng B", ... },
  { "id": "3", "name": "Người dùng C", ... }
]
```

**Hàm đọc dữ liệu sẽ thay đổi như sau**:
```dart
Future<List<User>> readJsonListData() async {
  final String jsonString = await rootBundle.loadString('assets/users_list.json');
  
  // Parse chuỗi JSON thành một List<dynamic>
  final List<dynamic> data = jsonDecode(jsonString);
  
  // Dùng map để chuyển đổi từng item trong list thành một đối tượng User
  return data.map((json) => User.fromJson(json)).toList();
}
```

Và trong UI, bạn sẽ dùng `FutureBuilder<List<User>>` và `ListView.builder` để hiển thị danh sách.

Chúc bạn thành công

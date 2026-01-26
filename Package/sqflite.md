Chào bạn, **`sqflite`** là một trong những package quan trọng và phổ biến nhất trong hệ sinh thái Flutter để xử lý việc lưu trữ dữ liệu cục bộ (Local Storage).

Dưới đây là bản giới thiệu chi tiết từ khái niệm, cách hoạt động đến hướng dẫn sử dụng.

---

### 1. `sqflite` là gì?

**`sqflite`** là một plugin dành cho Flutter, cho phép ứng dụng của bạn tương tác với cơ sở dữ liệu **SQLite**.

* **SQLite** là một hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) nhỏ gọn, được nhúng sẵn trong cả Android và iOS.
* `sqflite` đóng vai trò là cây cầu nối để code Dart gọi xuống native (Android/iOS) để thực thi các lệnh SQL.

### 2. Đặc điểm nổi bật

* **Hỗ trợ đa nền tảng:** Chạy tốt trên Android, iOS và MacOS. (Lưu ý: Web và Windows/Linux cần các gói bổ trợ khác như `sqflite_common_ffi`).
* **Bất đồng bộ (Asynchronous):** Mọi thao tác (đọc, ghi) đều trả về `Future`, đảm bảo không bao giờ chặn UI thread (Main thread), giúp app luôn mượt mà.
* **Mạnh mẽ:** Hỗ trợ đầy đủ các tính năng của SQL chuẩn:
* Giao dịch (Transactions).
* Xử lý hàng loạt (Batch support).
* Truy vấn phức tạp (Joins, Group By, Order By...).
* Hỗ trợ cả Raw SQL (viết câu lệnh SQL thủ công) và Helper methods (hàm hỗ trợ tiện lợi).



---

### 3. Khi nào nên dùng `sqflite`?

Bạn nên chọn `sqflite` thay vì `SharedPreferences` hay `Hive` khi:

1. **Dữ liệu có cấu trúc phức tạp:** Bạn cần lưu các bảng có quan hệ với nhau (Ví dụ: `User` có nhiều `Post`, `Post` có nhiều `Comment`).
2. **Cần truy vấn mạnh mẽ:** Bạn cần lọc, sắp xếp, tìm kiếm phức tạp (VD: "Lấy tất cả nhân viên lương > 1000 và sắp xếp theo tên").
3. **Dữ liệu lớn:** Cần lưu trữ hàng ngàn bản ghi mà vẫn đảm bảo tốc độ truy xuất.

---

### 4. Cài đặt

Thêm vào `pubspec.yaml`. Bạn cũng cần package `path` để xử lý đường dẫn file database.

```yaml
dependencies:
  sqflite: ^2.3.0
  path: ^1.9.0

```

---

### 5. Hướng dẫn sử dụng cơ bản

Quy trình làm việc với `sqflite` thường trải qua 4 bước: **Mở DB -> Tạo Bảng -> CRUD (Thêm/Sửa/Xóa/Đọc) -> Đóng DB**.

#### Bước 1: Khởi tạo và Mở Database

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static Database? _database;

  // Singleton pattern để đảm bảo chỉ có 1 instance của DB
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  _initDatabase() async {
    // Lấy đường dẫn thư mục lưu database trên thiết bị
    String path = join(await getDatabasesPath(), 'my_app.db');

    return await openDatabase(
      path,
      version: 1,
      // Hàm này chỉ chạy khi DB được tạo lần đầu tiên
      onCreate: (db, version) {
        return db.execute(
          'CREATE TABLE users(id INTEGER PRIMARY KEY, name TEXT, age INTEGER)',
        );
      },
    );
  }
}

```

#### Bước 2: Định nghĩa Data Model

Tạo class Dart để ánh xạ với bảng trong Database.

```dart
class User {
  final int? id;
  final String name;
  final int age;

  User({this.id, required this.name, required this.age});

  // Chuyển từ Object sang Map để lưu vào DB
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'age': age,
    };
  }

  // Chuyển từ Map (DB trả về) sang Object
  factory User.fromMap(Map<String, dynamic> map) {
    return User(
      id: map['id'],
      name: map['name'],
      age: map['age'],
    );
  }
}

```

#### Bước 3: Các thao tác CRUD

**A. Create (Thêm dữ liệu)**
Sử dụng hàm `insert`.

```dart
Future<void> insertUser(User user) async {
  final db = await DatabaseHelper().database;
  
  await db.insert(
    'users',
    user.toMap(),
    // conflictAlgorithm: Xử lý khi trùng ID (ở đây là ghi đè)
    conflictAlgorithm: ConflictAlgorithm.replace,
  );
}

```

**B. Read (Đọc dữ liệu)**
Sử dụng hàm `query`.

```dart
Future<List<User>> getUsers() async {
  final db = await DatabaseHelper().database;
  
  // Lấy tất cả bản ghi
  final List<Map<String, dynamic>> maps = await db.query('users');

  return List.generate(maps.length, (i) {
    return User.fromMap(maps[i]);
  });
}

// Truy vấn có điều kiện (WHERE)
Future<List<User>> getUsersOver18() async {
  final db = await DatabaseHelper().database;
  final maps = await db.query(
    'users',
    where: 'age > ?',
    whereArgs: [18], // Dùng ? và whereArgs để tránh SQL Injection
  );
  // ... convert sang List<User>
}

```

**C. Update (Cập nhật)**
Sử dụng hàm `update`.

```dart
Future<void> updateUser(User user) async {
  final db = await DatabaseHelper().database;

  await db.update(
    'users',
    user.toMap(),
    where: 'id = ?',
    whereArgs: [user.id],
  );
}

```

**D. Delete (Xóa)**
Sử dụng hàm `delete`.

```dart
Future<void> deleteUser(int id) async {
  final db = await DatabaseHelper().database;

  await db.delete(
    'users',
    where: 'id = ?',
    whereArgs: [id],
  );
}

```

---

### 6. Các tính năng nâng cao (Nên biết)

#### A. Raw SQL (SQL thuần)

Nếu bạn rành SQL và muốn viết câu lệnh phức tạp, `sqflite` cho phép gọi trực tiếp:

```dart
// Đếm số lượng user
int? count = Sqflite.firstIntValue(await db.rawQuery('SELECT COUNT(*) FROM users'));

// Join bảng
var list = await db.rawQuery('SELECT * FROM users u INNER JOIN posts p ON u.id = p.user_id');

```

#### B. Transaction (Giao dịch)

Dùng khi bạn muốn thực hiện chuỗi hành động: "Thành công cùng thành công, thất bại cùng thất bại" (Atomic).
*Ví dụ: Chuyển tiền (Trừ tiền A, cộng tiền B).*

```dart
await db.transaction((txn) async {
  await txn.rawUpdate('UPDATE account SET balance = balance - 10 WHERE id = 1');
  await txn.rawUpdate('UPDATE account SET balance = balance + 10 WHERE id = 2');
  // Nếu có lỗi xảy ra ở bất kỳ dòng nào, DB sẽ quay lại trạng thái ban đầu (Rollback)
});

```

#### C. Batch (Xử lý hàng loạt)

Dùng khi bạn cần insert 1000 dòng một lúc. Batch nhanh hơn nhiều so với việc gọi `insert` 1000 lần.

```dart
var batch = db.batch();
for (var i = 0; i < 1000; i++) {
  batch.insert('users', {'name': 'User $i'});
}
await batch.commit(); // Thực thi 1 lần duy nhất

```

---

### 7. Ưu và Nhược điểm

| Ưu điểm | Nhược điểm |
| --- | --- |
| **Chuẩn mực:** Dựa trên SQLite, công nghệ đã được kiểm chứng hàng chục năm. | **Nhiều code:** Bạn phải viết nhiều code lặp lại (Boilerplate) như `toMap`, `fromMap`, chuỗi SQL thủ công. |
| **Kiểm soát:** Bạn kiểm soát hoàn toàn câu lệnh SQL, tối ưu index. | **Dễ sai sót:** Viết sai tên cột hoặc câu SQL dạng String dễ gây lỗi Runtime (chạy mới biết lỗi). |
| **Quan hệ:** Xử lý quan hệ bảng (1-n, n-n) tốt. | **Không Web:** Mặc định không hỗ trợ Flutter Web (cần config thêm). |

### 8. Lời khuyên

* Nếu bạn muốn dùng SQLite nhưng ngại viết SQL thủ công và muốn an toàn kiểu dữ liệu (Type-safety), hãy tham khảo thư viện **Drift** (trước đây là Moor). Drift là một lớp ORM bọc lên trên `sqflite` giúp code nhanh và an toàn hơn.
* Nếu dữ liệu đơn giản, không cần quan hệ bảng phức tạp, hãy dùng **Hive** hoặc **Isar** để có hiệu năng cao hơn và code gọn hơn.

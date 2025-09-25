Chắc chắn rồi! Lưu trữ dữ liệu cục bộ (local storage) là một phần không thể thiếu trong hầu hết các ứng dụng di động, và SQLite là một lựa chọn cực kỳ mạnh mẽ, phổ biến cho việc này.

Hãy cùng nhau đi sâu vào cách sử dụng SQLite trong Flutter một cách chi tiết, từ cài đặt cho đến việc thực hiện các thao tác CRUD (Create, Read, Update, Delete) và tích hợp vào giao diện người dùng.

### SQLite là gì và tại sao lại dùng nó?

Hãy tưởng tượng SQLite như một "cuốn sổ tay kỹ thuật số" được tích hợp ngay bên trong ứng dụng của bạn. Nó là một hệ quản trị cơ sở dữ liệu quan hệ, gọn nhẹ, không cần máy chủ (serverless), và lưu toàn bộ dữ liệu vào một file duy nhất trên thiết bị của người dùng.

**Khi nào nên dùng SQLite?**
*   Khi bạn cần lưu trữ một lượng lớn dữ liệu có cấu trúc (ví dụ: danh sách sản phẩm, tin nhắn, ghi chú, thông tin người dùng...).
*   Khi bạn cần truy vấn dữ liệu phức tạp (lọc, sắp xếp, kết hợp nhiều bảng).
*   Khi bạn muốn ứng dụng hoạt động offline.

---

### Bước 1: Cài đặt các gói cần thiết

Bạn sẽ cần 2 package chính từ `pub.dev`:
1.  **`sqflite`**: Thư viện chính để tương tác với cơ sở dữ liệu SQLite.
2.  **`path`**: Giúp xử lý và kết hợp các đường dẫn file một cách đáng tin cậy trên cả Android và iOS.

Mở file `pubspec.yaml` của bạn và thêm vào mục `dependencies`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  sqflite: ^2.3.0 # Luôn kiểm tra phiên bản mới nhất trên pub.dev
  path: ^1.8.3
```
Sau đó, chạy `flutter pub get` trong terminal để cài đặt.

---

### Bước 2: Tạo một "Database Helper" - Quản lý tập trung

Đây là một **best practice** cực kỳ quan trọng. Thay vì gọi các lệnh database rải rác khắp nơi, chúng ta sẽ tạo một class duy nhất để quản lý tất cả các tương tác với database. Điều này giúp code sạch sẽ, dễ bảo trì và tránh lỗi.

Chúng ta sẽ sử dụng **Singleton Pattern** để đảm bảo rằng chỉ có một kết nối duy nhất đến database được mở trong suốt vòng đời của ứng dụng.

Tạo một file mới, ví dụ `database_helper.dart`:
```dart
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class DatabaseHelper {
  // 1. Singleton Pattern
  // Đảm bảo class này chỉ có một instance duy nhất
  DatabaseHelper._privateConstructor();
  static final DatabaseHelper instance = DatabaseHelper._privateConstructor();

  // Chỉ có một kết nối duy nhất đến database
  static Database? _database;

  // Getter để lấy database. Nếu chưa có, sẽ khởi tạo
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  // 2. Khởi tạo Database
  // Hàm này sẽ mở kết nối đến database. Nếu chưa có file db, nó sẽ tạo mới.
  _initDatabase() async {
    // Lấy đường dẫn đến thư mục lưu trữ database của ứng dụng
    String path = join(await getDatabasesPath(), 'mydatabase.db');
    return await openDatabase(
      path,
      version: 1, // Quản lý phiên bản để nâng cấp db sau này
      onCreate: _onCreate, // Hàm sẽ chạy khi database được tạo lần đầu
    );
  }

  // 3. Tạo bảng (Table)
  // Hàm này thực thi câu lệnh SQL để tạo cấu trúc bảng
  Future _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE dogs (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER NOT NULL
      )
    ''');
  }

  // Các hàm CRUD sẽ được thêm vào đây...
}
```

---

### Bước 3: Tạo Model Class

Để làm việc với dữ liệu một cách an toàn và dễ dàng, chúng ta nên tạo một class Dart để đại diện cho đối tượng mà chúng ta lưu trữ (ví dụ: `Dog`). Class này sẽ có các phương thức để chuyển đổi giữa đối tượng Dart và `Map` (định dạng mà `sqflite` sử dụng).

Tạo file `dog_model.dart`:
```dart
class Dog {
  final int? id;
  final String name;
  final int age;

  Dog({this.id, required this.name, required this.age});

  // Chuyển đổi một đối tượng Dog thành một Map.
  // Keys phải khớp với tên các cột trong database.
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'age': age,
    };
  }

  // Tiện ích để debug, in thông tin Dog ra console
  @override
  String toString() {
    return 'Dog{id: $id, name: $name, age: $age}';
  }
}
```

---

### Bước 4: Thực hiện các thao tác CRUD

Bây giờ, hãy quay lại file `database_helper.dart` và thêm các phương thức cho CRUD.

```dart
// ... (bên trong class DatabaseHelper)

// CREATE - Thêm một Dog mới vào database
Future<int> addDog(Dog dog) async {
  Database db = await instance.database;
  return await db.insert(
    'dogs', 
    dog.toMap(),
    // Xử lý xung đột: nếu insert một dog có id đã tồn tại, sẽ thay thế nó
    conflictAlgorithm: ConflictAlgorithm.replace, 
  );
}

// READ - Lấy tất cả các Dog từ database
Future<List<Dog>> getAllDogs() async {
  Database db = await instance.database;
  // db.query sẽ trả về một List<Map<String, dynamic>>
  final List<Map<String, dynamic>> maps = await db.query('dogs');

  // Chuyển List<Map<String, dynamic>> thành List<Dog>
  return List.generate(maps.length, (i) {
    return Dog(
      id: maps[i]['id'],
      name: maps[i]['name'],
      age: maps[i]['age'],
    );
  });
}

// UPDATE - Cập nhật một Dog đã có
Future<int> updateDog(Dog dog) async {
  Database db = await instance.database;
  return await db.update(
    'dogs',
    dog.toMap(),
    // Đảm bảo chỉ update dog có id khớp
    where: 'id = ?',
    whereArgs: [dog.id],
  );
}

// DELETE - Xóa một Dog khỏi database
Future<int> deleteDog(int id) async {
  Database db = await instance.database;
  return await db.delete(
    'dogs',
    where: 'id = ?',
    whereArgs: [id],
  );
}
```

---

### Bước 5: Tích hợp vào giao diện Flutter

Đây là lúc kết hợp tất cả lại. Chúng ta sẽ tạo một màn hình hiển thị danh sách các chú chó, có các nút để thêm, sửa, xóa. `FutureBuilder` là widget hoàn hảo cho việc hiển thị dữ liệu được tải bất đồng bộ từ database.

Đây là một ví dụ hoàn chỉnh cho file `main.dart`:

```dart
import 'package:flutter/material.dart';
// Import các file bạn đã tạo
import 'database_helper.dart';
import 'dog_model.dart';

void main() {
  // Đảm bảo Flutter bindings đã được khởi tạo trước khi chạy app
  WidgetsFlutterBinding.ensureInitialized();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SQLite Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: DogListScreen(),
    );
  }
}

class DogListScreen extends StatefulWidget {
  @override
  _DogListScreenState createState() => _DogListScreenState();
}

class _DogListScreenState extends State<DogListScreen> {
  // Tham chiếu đến database helper
  final dbHelper = DatabaseHelper.instance;
  late Future<List<Dog>> _dogList;

  @override
  void initState() {
    super.initState();
    _refreshDogList();
  }

  // Hàm để lấy dữ liệu mới và cập nhật UI
  void _refreshDogList() {
    setState(() {
      _dogList = dbHelper.getAllDogs();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('My Dogs'),
      ),
      body: FutureBuilder<List<Dog>>(
        future: _dogList,
        builder: (context, snapshot) {
          // 1. Đang tải dữ liệu
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          // 2. Tải xong nhưng bị lỗi
          if (snapshot.hasError) {
            return Center(child: Text('Đã xảy ra lỗi: ${snapshot.error}'));
          }
          // 3. Tải xong và có dữ liệu
          if (snapshot.hasData) {
            if (snapshot.data!.isEmpty) {
              return Center(child: Text('Không có chú chó nào.'));
            }
            // Hiển thị danh sách
            return ListView.builder(
              itemCount: snapshot.data!.length,
              itemBuilder: (context, index) {
                final dog = snapshot.data![index];
                return ListTile(
                  title: Text(dog.name),
                  subtitle: Text('Tuổi: ${dog.age}'),
                  trailing: IconButton(
                    icon: Icon(Icons.delete, color: Colors.red),
                    onPressed: () async {
                      await dbHelper.deleteDog(dog.id!);
                      _refreshDogList(); // Tải lại danh sách sau khi xóa
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(content: Text('Đã xóa ${dog.name}')),
                      );
                    },
                  ),
                );
              },
            );
          }
          // Trường hợp mặc định
          return Center(child: Text('Bắt đầu thêm chó!'));
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () async {
          // Thêm một chú chó mẫu
          final newDog = Dog(name: 'Buddy ${DateTime.now().second}', age: 5);
          await dbHelper.addDog(newDog);
          _refreshDogList(); // Tải lại danh sách sau khi thêm
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Đã thêm ${newDog.name}')),
          );
        },
      ),
    );
  }
}
```

### Tóm tắt luồng hoạt động:

1.  **Khởi tạo:** `main()` đảm bảo Flutter được khởi tạo, sau đó `DogListScreen` được tạo ra.
2.  **`initState()`:** Gọi `_refreshDogList()`, hàm này gán `_dogList` bằng kết quả của `dbHelper.getAllDogs()`. Vì `getAllDogs()` là hàm `async`, nó trả về một `Future`.
3.  **`FutureBuilder`:** Widget này "lắng nghe" `_dogList`.
    *   Trong khi `Future` đang chạy, nó hiển thị `CircularProgressIndicator`.
    *   Khi `Future` hoàn thành, nó kiểm tra dữ liệu và hiển thị `ListView` hoặc thông báo lỗi.
4.  **Tương tác:**
    *   Khi nhấn nút `+`, một `Dog` mới được thêm vào DB, sau đó `_refreshDogList()` được gọi để `FutureBuilder` xây dựng lại UI với dữ liệu mới.
    *   Khi nhấn nút xóa, `dog` tương ứng bị xóa khỏi DB, và `_refreshDogList()` lại được gọi.

Hy vọng hướng dẫn chi tiết này giúp bạn tự tin làm việc với SQLite trong các dự án Flutter của mình

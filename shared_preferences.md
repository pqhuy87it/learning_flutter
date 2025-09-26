Chào bạn! `shared_preferences` là một trong những package đầu tiên mà hầu hết các lập trình viên Flutter đều học và sử dụng. Nó cực kỳ hữu ích và dễ dùng. Hãy cùng nhau tìm hiểu chi tiết về nó nhé!

### `shared_preferences` là gì?

Hãy tưởng tượng `shared_preferences` như một **tấm giấy nhớ kỹ thuật số** siêu nhỏ, được tích hợp sẵn trong ứng dụng của bạn. Nó cho phép bạn lưu trữ các cặp **key-value** (chìa khóa - giá trị) đơn giản và bền bỉ.

*   **Bền bỉ (Persistent)**: Dữ liệu bạn lưu sẽ không bị mất khi người dùng đóng ứng dụng hoặc khởi động lại điện thoại.
*   **Đơn giản (Simple)**: Nó chỉ hỗ trợ các kiểu dữ liệu cơ bản như `String`, `bool`, `int`, `double`, và `List<String>`.
*   **Key-Value**: Mỗi mẩu dữ liệu được lưu dưới một "chìa khóa" (một chuỗi `String` duy nhất) để bạn có thể lấy lại sau này.

Về mặt kỹ thuật, package này là một lớp vỏ (wrapper) bọc quanh các API lưu trữ gốc của nền tảng: `NSUserDefaults` trên iOS/macOS và `SharedPreferences` trên Android.

---

### Khi nào nên dùng `shared_preferences`?

Đây là công cụ hoàn hảo cho việc lưu trữ các mẩu dữ liệu nhỏ, không quá quan trọng về bảo mật.

**Nên dùng cho:**
✅ Lưu cài đặt của người dùng (ví dụ: chế độ tối `isDarkMode`, ngôn ngữ `languageCode`).
✅ Lưu trạng thái đăng nhập đơn giản (ví dụ: `isLoggedIn = true`).
✅ Lưu điểm số cao nhất trong một game đơn giản.
✅ Ghi nhớ liệu người dùng đã xem qua màn hình hướng dẫn (onboarding) hay chưa.

**Không nên dùng cho:**
❌ Lưu trữ dữ liệu phức tạp (như danh sách các đối tượng `User`, `Product`). Hãy dùng `Hive` hoặc `sqflite`.
❌ Lưu trữ dữ liệu nhạy cảm (mật khẩu, token xác thực, thông tin thẻ tín dụng). Hãy dùng `flutter_secure_storage`.
❌ Lưu trữ lượng dữ liệu lớn (hình ảnh, file).

---

### Hướng dẫn sử dụng chi tiết

#### Bước 1: Cài đặt

Thêm package vào file `pubspec.yaml` của bạn:

```yaml
dependencies:
  flutter:
    sdk: flutter
  shared_preferences: ^2.2.2 # Luôn kiểm tra phiên bản mới nhất trên pub.dev
```
Sau đó, chạy lệnh `flutter pub get` trong terminal.

#### Bước 2: Lấy Instance (Đối tượng tham chiếu)

Tất cả các thao tác với `shared_preferences` đều là **bất đồng bộ (asynchronous)**, vì vậy bạn cần sử dụng `async` và `await`. Trước tiên, bạn cần lấy một "instance" của nó.

```dart
import 'package:shared_preferences/shared_preferences.dart';

// Lấy instance của SharedPreferences
final SharedPreferences prefs = await SharedPreferences.getInstance();
```

#### Bước 3: Các thao tác CRUD (Ghi, Đọc, Xóa)

**A. Ghi dữ liệu (Create / Update)**

Sử dụng các phương thức `set<Type>()`. Nếu key đã tồn tại, giá trị cũ sẽ bị ghi đè.

```dart
// Lưu một chuỗi String
await prefs.setString('username', 'PrivateGPT');

// Lưu một số nguyên int
await prefs.setInt('score', 100);

// Lưu một giá trị boolean
await prefs.setBool('isDarkMode', true);

// Lưu một số thực double
await prefs.setDouble('volume', 0.8);

// Lưu một danh sách chuỗi List<String>
await prefs.setStringList('items', <String>['Apple', 'Banana', 'Orange']);
```

**B. Đọc dữ liệu (Read)**

Sử dụng các phương thức `get<Type>()`. Nếu key không tồn tại, nó sẽ trả về `null`. Do đó, việc cung cấp một giá trị mặc định bằng toán tử `??` là một thói quen rất tốt.

```dart
// Đọc một chuỗi String. Nếu không có, mặc định là 'Guest'
final String username = prefs.getString('username') ?? 'Guest';

// Đọc một số nguyên int. Nếu không có, mặc định là 0
final int score = prefs.getInt('score') ?? 0;

// Đọc một giá trị boolean. Nếu không có, mặc định là false
final bool isDarkMode = prefs.getBool('isDarkMode') ?? false;

// Đọc một danh sách chuỗi. Nếu không có, mặc định là một list rỗng
final List<String> items = prefs.getStringList('items') ?? [];

print('Username: $username, Is Dark Mode: $isDarkMode');
```

**C. Xóa dữ liệu (Delete)**

Bạn có thể xóa một cặp key-value cụ thể hoặc xóa tất cả.

```dart
// Xóa một key cụ thể
await prefs.remove('score');

// Xóa TẤT CẢ dữ liệu đã lưu trong SharedPreferences
// Cẩn thận khi sử dụng lệnh này!
// await prefs.clear();
```

---

### Ví dụ thực tế hoàn chỉnh: Lưu trạng thái Dark Mode

Đây là một ví dụ kinh điển, cho thấy cách lưu lựa chọn của người dùng và áp dụng nó khi ứng dụng khởi động lại.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  ThemeMode _themeMode = ThemeMode.light; // Mặc định ban đầu

  @override
  void initState() {
    super.initState();
    _loadTheme(); // Tải theme đã lưu khi ứng dụng bắt đầu
  }

  // Hàm để tải trạng thái theme từ SharedPreferences
  void _loadTheme() async {
    final prefs = await SharedPreferences.getInstance();
    // Đọc giá trị 'isDarkMode'. Nếu không có, mặc định là false.
    final bool isDarkMode = prefs.getBool('isDarkMode') ?? false;
    setState(() {
      _themeMode = isDarkMode ? ThemeMode.dark : ThemeMode.light;
    });
  }

  // Hàm để thay đổi và lưu trạng thái theme
  void _toggleTheme(bool isDark) async {
    setState(() {
      _themeMode = isDark ? ThemeMode.dark : ThemeMode.light;
    });
    // Lưu lựa chọn vào SharedPreferences
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool('isDarkMode', isDark);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SharedPreferences Demo',
      theme: ThemeData.light(), // Theme cho chế độ sáng
      darkTheme: ThemeData.dark(), // Theme cho chế độ tối
      themeMode: _themeMode, // Quyết định theme nào sẽ được hiển thị
      home: HomeScreen(
        isDarkMode: _themeMode == ThemeMode.dark,
        onThemeChanged: _toggleTheme,
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  final bool isDarkMode;
  final ValueChanged<bool> onThemeChanged;

  const HomeScreen({
    super.key,
    required this.isDarkMode,
    required this.onThemeChanged,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Theme Switcher'),
      ),
      body: Center(
        child: Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Light'),
            Switch(
              value: isDarkMode,
              onChanged: onThemeChanged, // Gọi hàm callback để thay đổi
            ),
            const Text('Dark'),
          ],
        ),
      ),
    );
  }
}
```
Khi bạn chạy ứng dụng này, bật công tắc sang Dark Mode rồi đóng ứng dụng hoàn toàn. Mở lại ứng dụng, bạn sẽ thấy nó vẫn ở Dark Mode. Dữ liệu đã được lưu thành công!

### Tóm lại

`shared_preferences` là một công cụ đơn giản nhưng cực kỳ mạnh mẽ cho các nhu cầu lưu trữ cơ bản. Hãy nhớ:
*   Luôn dùng `async`/`await`.
*   Cung cấp giá trị mặc định khi đọc dữ liệu để tránh lỗi `null`.
*   Không dùng nó cho dữ liệu phức tạp hoặc nhạy cảm.

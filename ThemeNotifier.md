Tuyệt vời! Sử dụng `provider` để quản lý việc chuyển đổi Dark Mode là một cách làm cực kỳ phổ biến, sạch sẽ và hiệu quả trong Flutter. Nó giúp bạn tách biệt hoàn toàn logic quản lý theme ra khỏi giao diện.

Hãy cùng nhau xây dựng một ứng dụng có nút chuyển đổi Sáng/Tối bằng `provider` từng bước một nhé!

### Bước 0: Thêm package `provider`

Đầu tiên và quan trọng nhất, hãy đảm bảo bạn đã thêm `provider` vào file `pubspec.yaml` của mình:

```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.0 # Hoặc phiên bản mới nhất
```

Sau đó, chạy `flutter pub get` trong terminal.

---

### Bước 1: Tạo "Bộ Não" - `ThemeNotifier`

Chúng ta cần một class để lưu trữ và quản lý trạng thái theme hiện tại (Sáng hay Tối). Class này sẽ sử dụng `ChangeNotifier` để thông báo cho các widget khác mỗi khi trạng thái thay đổi.

Tạo một file mới, ví dụ: `theme_provider.dart`

```dart
import 'package:flutter/material.dart';

class ThemeNotifier extends ChangeNotifier {
  // Trạng thái ban đầu của theme là Sáng (light)
  ThemeMode _themeMode = ThemeMode.light;

  // Getter để các widget khác có thể đọc được trạng thái hiện tại
  ThemeMode get themeMode => _themeMode;

  // Phương thức để chuyển đổi theme
  void toggleTheme() {
    _themeMode = _themeMode == ThemeMode.light ? ThemeMode.dark : ThemeMode.light;
    
    // Thông báo cho tất cả các widget đang "lắng nghe" rằng trạng thái đã thay đổi
    // để chúng có thể rebuild lại giao diện.
    notifyListeners();
  }
}
```

**Giải thích:**
*   `extends ChangeNotifier`: Biến class này thành một "nhà cung cấp" có khả năng thông báo sự thay đổi.
*   `_themeMode`: Một biến riêng tư để lưu trạng thái (`ThemeMode.light` hoặc `ThemeMode.dark`).
*   `get themeMode`: Cho phép các widget khác đọc giá trị `_themeMode` một cách an toàn.
*   `toggleTheme()`: Logic để đảo ngược trạng thái hiện tại.
*   `notifyListeners()`: Đây là **trái tim** của `ChangeNotifier`. Khi được gọi, nó sẽ báo cho tất cả các `Consumer` hoặc `context.watch` đang theo dõi `ThemeNotifier` phải rebuild.

---

### Bước 2: Cung cấp `ThemeNotifier` cho toàn bộ ứng dụng

Để mọi widget trong ứng dụng có thể truy cập vào `ThemeNotifier`, chúng ta cần "cung cấp" nó ở cấp cao nhất, thường là bên trên `MaterialApp`.

Mở file `main.dart` của bạn:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'theme_provider.dart'; // Import file vừa tạo
import 'home_page.dart';      // Import màn hình chính (sẽ tạo ở bước 4)

void main() {
  runApp(
    // 1. Bọc ứng dụng của bạn bằng ChangeNotifierProvider
    ChangeNotifierProvider(
      // 2. Tạo một instance của ThemeNotifier
      create: (_) => ThemeNotifier(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // 3. Sử dụng Consumer để lắng nghe sự thay đổi của ThemeNotifier
    return Consumer<ThemeNotifier>(
      builder: (context, themeNotifier, child) {
        return MaterialApp(
          title: 'Provider Dark Mode',
          // 4. Kết nối themeMode của MaterialApp với themeMode trong Notifier
          themeMode: themeNotifier.themeMode,
          
          // 5. Định nghĩa theme cho chế độ Sáng
          theme: ThemeData(
            brightness: Brightness.light,
            primarySwatch: Colors.blue,
            useMaterial3: true,
          ),

          // 6. Định nghĩa theme cho chế độ Tối
          darkTheme: ThemeData(
            brightness: Brightness.dark,
            primarySwatch: Colors.indigo,
            useMaterial3: true,
          ),

          home: const HomePage(),
        );
      },
    );
  }
}
```

**Giải thích:**
1.  `ChangeNotifierProvider`: Widget này tạo ra một instance của `ThemeNotifier` và làm cho nó có sẵn cho tất cả các widget con của nó (trong trường hợp này là toàn bộ ứng dụng).
2.  `Consumer<ThemeNotifier>`: Widget này "lắng nghe" `ThemeNotifier`. Bất cứ khi nào `notifyListeners()` được gọi, hàm `builder` của `Consumer` sẽ được chạy lại.
3.  `themeMode: themeNotifier.themeMode`: Đây là sự kết nối quan trọng! Chúng ta nói với `MaterialApp` rằng: "Hãy sử dụng theme mode (Sáng/Tối) được quyết định bởi `themeNotifier`".

---

### Bước 3: Tạo Nút Chuyển Đổi trong Giao diện

Bây giờ, chúng ta sẽ tạo một màn hình `HomePage` có một nút bấm (ví dụ: một `Switch` hoặc `IconButton`) để người dùng có thể chuyển đổi theme.

Tạo file mới `home_page.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'theme_provider.dart'; // Import Notifier

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    // Lấy themeNotifier để kiểm tra trạng thái hiện tại
    // Dùng context.watch để widget này rebuild khi theme thay đổi
    final themeNotifier = context.watch<ThemeNotifier>();
    final isDarkMode = themeNotifier.themeMode == ThemeMode.dark;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Home Page'),
        actions: [
          // Nút chuyển đổi theme
          Switch(
            value: isDarkMode,
            onChanged: (value) {
              // Dùng context.read để gọi một phương thức
              // Nó sẽ không làm widget này rebuild một cách không cần thiết
              context.read<ThemeNotifier>().toggleTheme();
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              isDarkMode ? Icons.nightlight_round : Icons.wb_sunny,
              size: 100,
            ),
            const SizedBox(height: 20),
            Text(
              'Chế độ hiện tại: ${isDarkMode ? "Tối" : "Sáng"}',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Giải thích:**
*   `context.watch<ThemeNotifier>()`: Dùng để **lắng nghe và đọc** dữ liệu. Khi `notifyListeners()` được gọi, widget nào có `watch` sẽ được rebuild. Chúng ta dùng nó để cập nhật giá trị của `Switch` và `Text`.
*   `context.read<ThemeNotifier>()`: Dùng để **chỉ đọc** dữ liệu hoặc **gọi một phương thức** mà không cần lắng nghe sự thay đổi. Nó hiệu quả hơn khi đặt trong các hàm callback như `onChanged` hay `onPressed`, vì bản thân cái nút không cần phải vẽ lại khi theme thay đổi.

### Tóm tắt luồng hoạt động

1.  Người dùng nhấn vào `Switch`.
2.  Sự kiện `onChanged` được kích hoạt, gọi `context.read<ThemeNotifier>().toggleTheme()`.
3.  Phương thức `toggleTheme()` trong `ThemeNotifier` thay đổi giá trị `_themeMode` và gọi `notifyListeners()`.
4.  `notifyListeners()` thông báo cho tất cả các "người nghe".
5.  `Consumer` trong `main.dart` nghe thấy thông báo, nó rebuild lại `MaterialApp` với `themeMode` mới.
6.  `HomePage` (vì có `context.watch`) cũng nghe thấy thông báo, nó rebuild lại để cập nhật trạng thái của `Switch` và `Text`.
7.  Toàn bộ giao diện ứng dụng mượt mà chuyển đổi giữa Sáng và Tối.

Đây là cách làm rất chuẩn và có thể mở rộng. Bạn có thể thêm nhiều tùy chọn theme khác (ví dụ: theme màu xanh, đỏ...) vào `ThemeNotifier` một cách dễ dàng. Chúc bạn thành công

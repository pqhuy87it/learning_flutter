Chắc chắn rồi! `ThemeMode` là một khái niệm cực kỳ quan trọng và mạnh mẽ trong Flutter để quản lý giao diện Sáng/Tối. Hãy cùng nhau khám phá nó một cách chi tiết, từ cơ bản đến nâng cao nhé!

### 1. `ThemeMode` là gì? - Công tắc 3 chế độ

Hãy tưởng tượng bạn có hai bộ "quần áo" cho ứng dụng của mình:
*   Một bộ sáng sủa, thanh lịch (`ThemeData` cho Light Mode).
*   Một bộ tối màu, sang trọng (`ThemeData` cho Dark Mode).

`ThemeMode` chính là **công tắc thông minh** quyết định xem ứng dụng của bạn sẽ mặc bộ quần áo nào. Nó không phải là bản thân bộ quần áo, mà là **quy tắc** để chọn lựa.

Nó là một `enum` (kiểu liệt kê) và có 3 giá trị:

1.  `ThemeMode.system` (Mặc định và được khuyến khích nhất)
2.  `ThemeMode.light`
3.  `ThemeMode.dark`

### 2. Ba giá trị của `ThemeMode` - Chúng làm gì?

#### a. `ThemeMode.system`
Đây là chế độ "tự động" và thông minh nhất. Khi bạn đặt `themeMode: ThemeMode.system`, ứng dụng của bạn sẽ:
*   **Lắng nghe cài đặt của hệ điều hành (OS):** Nếu người dùng bật Dark Mode trên điện thoại (iPhone hoặc Android), ứng dụng của bạn sẽ tự động chuyển sang `darkTheme`.
*   **Tự động chuyển đổi:** Nếu người dùng tắt Dark Mode trên điện thoại, ứng dụng của bạn sẽ tự động quay về `theme`.

=> **Lợi ích:** Mang lại trải nghiệm người dùng nhất quán và liền mạch. Ứng dụng của bạn "tôn trọng" lựa chọn của người dùng trên toàn hệ thống. Đây là lựa chọn mặc định nếu bạn không chỉ định `themeMode`.

#### b. `ThemeMode.light`
Đây là chế độ "ép buộc". Khi bạn đặt `themeMode: ThemeMode.light`, ứng dụng của bạn sẽ:
*   **Luôn luôn** sử dụng `theme` (bộ quần áo sáng màu) mà bạn đã định nghĩa.
*   **Bỏ qua** hoàn toàn cài đặt Dark Mode của hệ điều hành.

=> **Lợi ích:** Hữu ích khi bạn muốn ứng dụng của mình chỉ có một giao diện sáng duy nhất, hoặc khi bạn cung cấp một tùy chọn trong cài đặt để người dùng tự chọn "Luôn ở chế độ Sáng".

#### c. `ThemeMode.dark`
Tương tự như `ThemeMode.light`, đây cũng là chế độ "ép buộc". Khi bạn đặt `themeMode: ThemeMode.dark`, ứng dụng của bạn sẽ:
*   **Luôn luôn** sử dụng `darkTheme` (bộ quần áo tối màu) mà bạn đã định nghĩa.
*   **Bỏ qua** hoàn toàn cài đặt Light Mode của hệ điều hành.

=> **Lợi ích:** Hữu ích khi ứng dụng của bạn được thiết kế đặc biệt cho giao diện tối, hoặc khi bạn cho phép người dùng chọn "Luôn ở chế độ Tối".

### 3. Cách hoạt động: Bộ ba quyền lực (`theme`, `darkTheme`, `themeMode`)

Để `ThemeMode` hoạt động, bạn cần cung cấp đủ "nguyên liệu" cho nó trong `MaterialApp`. Ba thuộc tính này luôn đi cùng nhau:

1.  **`theme`**: `ThemeData` - Định nghĩa giao diện cho chế độ Sáng. **Bắt buộc phải có.**
2.  **`darkTheme`**: `ThemeData` - Định nghĩa giao diện cho chế độ Tối. **Bắt buộc phải có nếu bạn muốn hỗ trợ Dark Mode.**
3.  **`themeMode`**: `ThemeMode` - Quy tắc để chọn giữa `theme` và `darkTheme`.

**Luồng logic của Flutter:**
1.  Flutter nhìn vào thuộc tính `themeMode`.
2.  **Nếu `themeMode` là `.light`:** Nó sẽ dùng `theme`.
3.  **Nếu `themeMode` là `.dark`:** Nó sẽ dùng `darkTheme`.
4.  **Nếu `themeMode` là `.system`:** Nó sẽ hỏi hệ điều hành "Bây giờ đang Sáng hay Tối?".
    *   Nếu hệ điều hành trả lời "Tối", nó sẽ dùng `darkTheme`.
    *   Nếu hệ điều hành trả lời "Sáng", nó sẽ dùng `theme`.

### 4. Ví dụ thực tế: Cho phép người dùng tự chọn `ThemeMode`

Đây là một ví dụ hoàn chỉnh sử dụng `provider` để cho phép người dùng chuyển đổi qua lại giữa 3 chế độ `system`, `light`, và `dark`.

#### Bước 1: Tạo `ThemeNotifier` (đã có ở câu trả lời trước, giờ ta nâng cấp nó)

File `theme_provider.dart`:
```dart
import 'package:flutter/material.dart';

class ThemeNotifier extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system; // Bắt đầu với chế độ hệ thống

  ThemeMode get themeMode => _themeMode;

  String get currentThemeModeName {
    switch (_themeMode) {
      case ThemeMode.system:
        return 'Hệ thống';
      case ThemeMode.light:
        return 'Sáng';
      case ThemeMode.dark:
        return 'Tối';
    }
  }

  // Phương thức để chuyển vòng quanh các chế độ
  void cycleThemeMode() {
    if (_themeMode == ThemeMode.system) {
      _themeMode = ThemeMode.light;
    } else if (_themeMode == ThemeMode.light) {
      _themeMode = ThemeMode.dark;
    } else {
      _themeMode = ThemeMode.system;
    }
    notifyListeners();
  }
}
```

#### Bước 2: Thiết lập `MaterialApp` trong `main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'theme_provider.dart';
import 'home_page.dart';

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => ThemeNotifier(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // Dùng watch ở đây để MaterialApp tự rebuild khi ThemeMode thay đổi
    final themeNotifier = context.watch<ThemeNotifier>();

    return MaterialApp(
      title: 'ThemeMode Demo',

      // ĐÂY LÀ BỘ BA QUYỀN LỰC
      theme: ThemeData(
        brightness: Brightness.light,
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
        appBarTheme: const AppBarTheme(backgroundColor: Colors.blue),
      ),
      darkTheme: ThemeData(
        brightness: Brightness.dark,
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
        appBarTheme: const AppBarTheme(backgroundColor: Colors.black26),
      ),
      themeMode: themeNotifier.themeMode, // <-- KẾT NỐI VỚI PROVIDER

      home: const HomePage(),
    );
  }
}
```

#### Bước 3: Tạo giao diện `HomePage` với nút chuyển đổi

File `home_page.dart`:
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'theme_provider.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    // Dùng read ở đây để gọi hàm, không cần rebuild khi theme thay đổi
    final themeNotifier = context.read<ThemeNotifier>();

    return Scaffold(
      appBar: AppBar(
        title: const Text('ThemeMode Demo'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Chế độ hiện tại:',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            // Dùng watch ở đây vì chỉ có Text này cần rebuild
            Text(
              context.watch<ThemeNotifier>().currentThemeModeName,
              style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () {
          // Gọi phương thức để chuyển đổi ThemeMode
          themeNotifier.cycleThemeMode();
        },
        label: const Text('Đổi chế độ'),
        icon: const Icon(Icons.sync),
      ),
    );
  }
}
```

**Kết quả:**
Khi bạn chạy ứng dụng, mỗi lần nhấn nút "Đổi chế độ", `ThemeMode` sẽ thay đổi theo chu kỳ: Hệ thống -> Sáng -> Tối -> Hệ thống. Giao diện của `MaterialApp` sẽ tự động cập nhật theo quy tắc bạn đã định nghĩa.

### Tóm tắt

*   `ThemeMode` là **quy tắc** để Flutter chọn giữa `theme` (sáng) và `darkTheme` (tối).
*   `ThemeMode.system` là lựa chọn tốt nhất cho trải nghiệm người dùng mặc định.
*   `ThemeMode.light` và `.dark` dùng để ép buộc một theme cụ thể, thường là cho người dùng tự cài đặt trong app.
*   Để `ThemeMode` hoạt động, bạn **phải** cung cấp cả hai thuộc tính `theme` và `darkTheme` cho `MaterialApp`.
*   Kết hợp `ThemeMode` với một giải pháp quản lý trạng thái như `provider` cho phép bạn tạo ra tính năng tùy chỉnh theme một cách mạnh mẽ và chuyên nghiệp.

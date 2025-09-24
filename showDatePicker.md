Chào bạn! Rất vui được hướng dẫn bạn cách sử dụng `showDatePicker` trong Flutter. Đây là một widget cực kỳ hữu ích để người dùng có thể chọn một ngày từ giao diện lịch trực quan.

Dưới đây là hướng dẫn chi tiết từ cơ bản đến nâng cao nhé! 🚀

### 1. Cách Hoạt Động Cơ Bản

`showDatePicker` là một hàm (function) trả về một `Future<DateTime?>`.
*   `Future`: Vì nó sẽ đợi người dùng chọn một ngày (hoặc đóng dialog), đây là một hành động bất đồng bộ. Chúng ta sẽ dùng `async/await` để xử lý.
*   `DateTime?`: Kiểu dữ liệu trả về là `DateTime`. Dấu `?` (nullable) có nghĩa là nó có thể trả về `null` nếu người dùng bấm nút "Cancel" hoặc bấm ra ngoài dialog để đóng nó.

Hàm này yêu cầu 4 tham số bắt buộc:
*   `context`: `BuildContext` của widget hiện tại.
*   `initialDate`: Ngày được hiển thị mặc định khi mở lịch lên.
*   `firstDate`: Ngày sớm nhất mà người dùng có thể chọn.
*   `lastDate`: Ngày muộn nhất mà người dùng có thể chọn.

### 2. Ví dụ Code Hoàn Chỉnh

Đây là một ví dụ đầy đủ về một màn hình có một nút bấm để mở `showDatePicker` và một dòng chữ để hiển thị ngày đã chọn.

**Bước 1: Chuẩn bị Project**

Để định dạng ngày tháng (ví dụ: "22/09/2025"), bạn nên dùng thư viện `intl`. Hãy thêm nó vào file `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  intl: ^0.18.0 # Hoặc phiên bản mới nhất
```

Sau đó, chạy `flutter pub get` trong terminal.

**Bước 2: Viết Code**

```dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart'; // Import thư viện intl

class MyDatePickerPage extends StatefulWidget {
  const MyDatePickerPage({super.key});

  @override
  State<MyDatePickerPage> createState() => _MyDatePickerPageState();
}

class _MyDatePickerPageState extends State<MyDatePickerPage> {
  // Biến để lưu trữ ngày được chọn
  DateTime? _selectedDate;

  // Hàm để hiển thị Date Picker
  void _presentDatePicker() async {
    final DateTime? pickedDate = await showDatePicker(
      context: context,
      initialDate: _selectedDate ?? DateTime.now(), // Ngày ban đầu là ngày đã chọn hoặc ngày hiện tại
      firstDate: DateTime(2000), // Ngày bắt đầu có thể chọn
      lastDate: DateTime(2101), // Ngày cuối cùng có thể chọn
    );

    // Kiểm tra xem người dùng có chọn ngày nào không
    if (pickedDate != null && pickedDate != _selectedDate) {
      // Nếu có, cập nhật lại state để rebuild UI
      setState(() {
        _selectedDate = pickedDate;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ShowDatePicker Demo'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            // Hiển thị ngày đã chọn hoặc một thông báo
            Text(
              _selectedDate == null
                  ? 'Chưa chọn ngày nào!'
                  : 'Ngày đã chọn: ${DateFormat('dd/MM/yyyy').format(_selectedDate!)}', // Định dạng ngày tháng
              style: const TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: _presentDatePicker, // Gọi hàm khi nhấn nút
              child: const Text('Chọn Ngày'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Giải thích code:**
1.  `_selectedDate`: Là một biến state để lưu ngày người dùng chọn.
2.  `_presentDatePicker()`:
    *   Sử dụng `async/await` để đợi kết quả từ `showDatePicker`.
    *   `showDatePicker` được gọi với các tham số cần thiết.
    *   Sau khi có kết quả (`pickedDate`), chúng ta kiểm tra xem nó có `null` hay không.
    *   Nếu người dùng đã chọn một ngày, ta gọi `setState` để cập nhật biến `_selectedDate` và yêu cầu Flutter vẽ lại giao diện với ngày mới.
3.  Trong `build()`:
    *   Chúng ta hiển thị `_selectedDate` đã được định dạng bằng `DateFormat` từ thư viện `intl`.
    *   Nếu `_selectedDate` là `null`, ta hiển thị một thông báo.
    *   `ElevatedButton` có `onPressed` trỏ đến hàm `_presentDatePicker`.

### 3. Tùy Chỉnh Nâng Cao ✨

Bạn có thể tùy chỉnh `showDatePicker` với nhiều tham số khác:

#### a. Thay đổi màu sắc (Theming)

Sử dụng tham số `builder` để bọc `DatePicker` trong một `Theme` widget.

```dart
showDatePicker(
  context: context,
  initialDate: DateTime.now(),
  firstDate: DateTime(2000),
  lastDate: DateTime(2101),
  builder: (context, child) {
    return Theme(
      data: Theme.of(context).copyWith(
        colorScheme: const ColorScheme.light(
          primary: Colors.amber, // Màu header
          onPrimary: Colors.black, // Màu chữ trên header
          onSurface: Colors.blueAccent, // Màu chữ của các ngày
        ),
        textButtonTheme: TextButtonThemeData(
          style: TextButton.styleFrom(
            foregroundColor: Colors.red, // Màu nút OK, CANCEL
          ),
        ),
      ),
      child: child!,
    );
  },
);
```

#### b. Thay đổi ngôn ngữ (Localization)

Để hiển thị lịch bằng Tiếng Việt, bạn cần cài đặt `flutter_localizations`.

1.  Thêm vào `pubspec.yaml`:
    ```yaml
    dependencies:
      flutter:
        sdk: flutter
      flutter_localizations: # Thêm dòng này
        sdk: flutter         # Thêm dòng này
      intl: ^0.18.0
    ```
2.  Cấu hình trong `MaterialApp`:
    ```dart
    import 'package:flutter_localizations/flutter_localizations.dart';

    MaterialApp(
      // ...
      localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      supportedLocales: const [
        Locale('en', ''), // English
        Locale('vi', ''), // Vietnamese
      ],
      locale: const Locale('vi'), // Đặt locale mặc định là Tiếng Việt
      // ...
    );
    ```
3.  Khi gọi `showDatePicker`, bạn cũng có thể truyền `locale`:
    ```dart
    showDatePicker(
      // ...
      locale: const Locale("vi", "VN"),
    );
    ```

#### c. Vô hiệu hóa một số ngày nhất định

Sử dụng `selectableDayPredicate` để quyết định ngày nào có thể được chọn. Hàm này trả về `true` nếu ngày đó hợp lệ, `false` nếu không.

Ví dụ: Chỉ cho phép chọn các ngày trong tuần (vô hiệu hóa Thứ 7, Chủ Nhật).

```dart
showDatePicker(
  // ...
  selectableDayPredicate: (DateTime day) {
    // Không cho chọn Thứ 7 và Chủ Nhật
    if (day.weekday == DateTime.saturday || day.weekday == DateTime.sunday) {
      return false;
    }
    return true;
  },
);
```

Hy vọng hướng dẫn này sẽ giúp bạn sử dụng `showDatePicker` một cách hiệu quả. Chúc bạn code vui vẻ

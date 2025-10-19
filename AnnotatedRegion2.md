`AnnotatedRegion` là một widget trong Flutter cho phép bạn thay đổi giao diện của các lớp phủ giao diện người dùng (UI overlays) của hệ thống, cụ thể là **thanh trạng thái** (status bar - ở trên cùng, chứa giờ và các biểu tượng) và **thanh điều hướng** (navigation bar - ở dưới cùng, trên Android).

Hãy coi nó như một chiếc hộp vô hình mà bạn bọc quanh các phần giao diện của mình. Khi phần giao diện đó hiển thị trên màn hình, Flutter sẽ yêu cầu hệ điều hành thay đổi kiểu của thanh trạng thái hoặc thanh điều hướng để khớp với kiểu bạn đã định nghĩa trong `AnnotatedRegion`.

Điều này rất cần thiết để tạo ra các giao diện người dùng tinh tế, nơi các thanh hệ thống cần phải thích ứng với các nền màn hình khác nhau. Ví dụ, bạn cần văn bản sáng trên nền tối và văn bản tối trên nền sáng.

-----

## Các thuộc tính chính và cách sử dụng

Thuộc tính chính bạn sẽ sử dụng là `value`, nhận một đối tượng `SystemUiOverlayStyle`. Đối tượng này chứa tất cả thông tin về kiểu dáng.

```dart
AnnotatedRegion<SystemUiOverlayStyle>(
  value: SystemUiOverlayStyle(
    // Các thuộc tính kiểu dáng được đặt ở đây
  ),
  child: YourScreenWidget(), // Widget mà kiểu này sẽ được áp dụng
);
```

### Các thuộc tính phổ biến của `SystemUiOverlayStyle`

1.  **`statusBarIconBrightness` (Quan trọng nhất) ✨**:

      * Thuộc tính này kiểm soát màu sắc của các biểu tượng và văn bản trên thanh trạng thái (giờ, pin, Wi-Fi).
      * `Brightness.light`: Các biểu tượng có màu **sáng** (màu trắng). Dùng cho nền tối.
      * `Brightness.dark`: Các biểu tượng có màu **tối** (màu đen). Dùng cho nền sáng.

2.  **`statusBarColor`**:

      * Đặt màu nền cho thanh trạng thái.
      * Sử dụng `Colors.transparent` rất phổ biến để làm cho nền của ứng dụng bạn mở rộng lên khu vực thanh trạng thái.

3.  **`systemNavigationBarColor`** (Android):

      * Đặt màu nền cho thanh điều hướng ở dưới cùng.

4.  **`systemNavigationBarIconBrightness`** (Android):

      * Kiểm soát màu của các biểu tượng trên thanh điều hướng (quay lại, home, ứng dụng gần đây).
      * `Brightness.light`: Biểu tượng sáng.
      * `Brightness.dark`: Biểu tượng tối.

-----

## Các ví dụ thực tế

### Ví dụ 1: Biểu tượng thanh trạng thái tối cho màn hình sáng

Hãy tưởng tượng bạn có một màn hình với nền `Scaffold` màu trắng. Các biểu tượng thanh trạng thái mặc định có màu sáng, sẽ không nhìn thấy được. Bạn cần làm cho chúng tối đi.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class LightScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return AnnotatedRegion<SystemUiOverlayStyle>(
      // Chỉ định kiểu cho màn hình này
      value: const SystemUiOverlayStyle(
        // Làm cho nền thanh trạng thái trong suốt
        statusBarColor: Colors.transparent,
        
        // Làm cho các biểu tượng thanh trạng thái tối đi
        statusBarIconBrightness: Brightness.dark,
        
        // Làm cho nền thanh điều hướng màu trắng
        systemNavigationBarColor: Colors.white,
        
        // Làm cho các biểu tượng thanh điều hướng tối đi
        systemNavigationBarIconBrightness: Brightness.dark,
      ),
      child: Scaffold(
        backgroundColor: Colors.white,
        appBar: AppBar(
          title: Text('Màn hình sáng', style: TextStyle(color: Colors.black)),
          backgroundColor: Colors.white,
          elevation: 0,
        ),
        body: Center(
          child: Text('Nội dung trên nền sáng'),
        ),
      ),
    );
  }
}
```

Trong đoạn code này, `AnnotatedRegion` bọc toàn bộ `Scaffold`. Ngay khi màn hình này hiển thị, Flutter sẽ áp dụng `SystemUiOverlayStyle` đã chỉ định, làm cho các biểu tượng thanh trạng thái tối đi để chúng có thể nhìn thấy trên nền trắng.

### Ví dụ 2: Biểu tượng thanh trạng thái sáng cho màn hình tối

Bây giờ, hãy tạo một màn hình theo chủ đề tối. Chúng ta cần các biểu tượng thanh trạng thái sáng (màu trắng) để chúng có thể nhìn thấy.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class DarkScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return AnnotatedRegion<SystemUiOverlayStyle>(
      // Chỉ định kiểu cho màn hình này
      value: const SystemUiOverlayStyle(
        // Làm cho nền thanh trạng thái trong suốt
        statusBarColor: Colors.transparent,
        
        // Làm cho các biểu tượng thanh trạng thái sáng lên (màu trắng)
        statusBarIconBrightness: Brightness.light,
        
        // Làm cho nền thanh điều hướng tối
        systemNavigationBarColor: Colors.black,

        // Làm cho các biểu tượng thanh điều hướng sáng lên (màu trắng)
        systemNavigationBarIconBrightness: Brightness.light,
      ),
      child: Scaffold(
        backgroundColor: Colors.black,
        appBar: AppBar(
          title: Text('Màn hình tối', style: TextStyle(color: Colors.white)),
          backgroundColor: Colors.black,
          elevation: 0,
        ),
        body: Center(
          child: Text(
            'Nội dung trên nền tối',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}
```

Ở đây, `statusBarIconBrightness: Brightness.light` đảm bảo rằng giờ và các biểu tượng hệ thống có màu trắng và có thể đọc được trên nền tối.

-----

## Khi nào bạn nên sử dụng `AnnotatedRegion`?

1.  **Các kiểu khác nhau cho mỗi màn hình**: Đây là trường hợp sử dụng chính. Màn hình đăng nhập của bạn có thể tối, trong khi màn hình chính lại sáng. Widget gốc của mỗi màn hình nên được bọc trong một `AnnotatedRegion` với kiểu phù hợp.

2.  **Thay đổi kiểu động**: Nếu bạn có một `PageView` hoặc một màn hình cuộn với các phần có màu khác nhau lướt qua dưới thanh trạng thái, bạn có thể sử dụng `AnnotatedRegion` để thay đổi kiểu một cách linh hoạt khi người dùng cuộn.

3.  **Ghi đè kiểu toàn cục**: Bạn có thể đặt một kiểu mặc định cho toàn bộ ứng dụng trong file `main.dart` của mình bằng `AppBarTheme`, nhưng `AnnotatedRegion` được sử dụng để **ghi đè** kiểu mặc định đó cho các widget hoặc màn hình cụ thể.

Tóm lại, `AnnotatedRegion` là cách tiêu chuẩn và hiện đại để kiểm soát các lớp phủ giao diện người dùng hệ thống trong Flutter trên cơ sở từng widget, cho bạn quyền kiểm soát chính xác giao diện và cảm nhận của ứng dụng.

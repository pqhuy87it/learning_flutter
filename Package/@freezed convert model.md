Để chuyển đổi class `CategoryModel` truyền thống của bạn sang sử dụng thư viện `@freezed`, bạn chỉ cần thực hiện theo 3 bước chuẩn dưới đây. 

Sự khác biệt lớn nhất là bạn không cần khai báo biến `final` nữa, mà chỉ cần định nghĩa chúng ngay bên trong `factory` constructor.

### Bước 1: Thêm import và khai báo file `part`
Bắt buộc phải import `freezed_annotation` và khai báo đường dẫn tới file code tự sinh (`.freezed.dart`). Tên file `part` phải trùng khớp với tên file hiện tại của bạn.

*(Giả sử file của bạn tên là `category_model.dart`)*

### Bước 2: Viết lại cấu trúc class
Chuyển class thường thành `class ... with _$TênClass` và sử dụng `factory` kết hợp với `@Default` cho các giá trị mặc định.

Dưới đây là đoạn code đã được chuyển đổi hoàn chỉnh:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

// BẮT BUỘC: Đổi 'category_model' thành tên file thực tế của bạn
part 'category_model.freezed.dart'; 

@freezed
class CategoryModel with _$CategoryModel {
  factory CategoryModel({
    required String name,
    required String slug,
    @Default(false) bool isLocal, // Dùng @Default thay cho this.isLocal = false
    int? timestamp,
    String? imagePath,
  }) = _CategoryModel;
}
```

**💡 Phân tích những thay đổi nhỏ nhưng quan trọng:**
* Không còn từ khóa `this.` bên trong constructor.
* Tham số `isLocal` thay vì gán `= false` thì được bọc bởi `@Default(false)`.
* Không cần viết tường minh các biến `final String name;` nữa, Freezed sẽ tự động "đúc" ra chúng ở dưới background.

### Bước 3: Chạy lệnh sinh code (Build Runner)
Lúc này, code của bạn sẽ báo lỗi đỏ lòm ở chữ `_$CategoryModel` và `_CategoryModel`. Đừng lo, đó là vì file `.freezed.dart` chưa được tạo ra.

Mở Terminal trong Android Studio hoặc VS Code và chạy lệnh sau:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```
*(Hoặc nếu dùng Dart thuần: `dart run build_runner build -d`)*

Chờ vài giây cho lệnh chạy xong, các gạch đỏ sẽ biến mất và bạn đã có một class `CategoryModel` với đầy đủ các tính năng siêu việt như `.copyWith()`, `.toString()`, và so sánh `==` (Deep equality).

---
**Mở rộng (Nếu bạn cần parse JSON từ API):**
Thường thì các class Model như thế này sẽ làm việc với API. Nếu bạn muốn thêm tính năng `fromJson` / `toJson`, bạn chỉ cần thêm 1 dòng `part` và 1 factory nữa như sau:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'category_model.freezed.dart';
part 'category_model.g.dart'; // Thêm dòng này cho JSON

@freezed
class CategoryModel with _$CategoryModel {
  factory CategoryModel({
    required String name,
    required String slug,
    @Default(false) bool isLocal,
    int? timestamp,
    String? imagePath,
  }) = _CategoryModel;

  // Thêm dòng này để parse JSON
  factory CategoryModel.fromJson(Map<String, dynamic> json) => _$CategoryModelFromJson(json);
}
```
Sau đó chạy lại lệnh `build_runner` ở Bước 3 là xong! Bạn có đang dùng thư viện `json_serializable` trong dự án không? Nếu có thì nên thêm luôn để tiện lấy data nhé.

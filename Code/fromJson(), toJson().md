Chào bạn,

Rất vui được hướng dẫn bạn chi tiết cách tạo ra các phương thức `fromJson()` và `toJson()` trong Flutter/Dart. Đây là một kỹ năng thiết yếu khi làm việc với API. Có hai cách tiếp cận chính: **thủ công** và **tự động** (sử dụng code generation).

Chúng ta sẽ bắt đầu với cách thủ công để bạn hiểu rõ bản chất, sau đó sẽ giới thiệu cách tự động để tăng tốc độ làm việc.

Giả sử chúng ta cần làm việc với đối tượng JSON sau từ một API:

```json
{
  "id": 1,
  "name": "Bún Chả Hà Nội",
  "author": {
    "authorId": "user123",
    "authorName": "Esheep Kitchen"
  },
  "tags": [
    "vietnamese",
    "hanoi",
    "grill"
  ],
  "isPublished": true
}
```

---

### Cách 1: Viết thủ công (Manual Serialization)

Cách này giúp bạn hiểu sâu về cách hoạt động của quá trình chuyển đổi.

#### Bước 1: Tạo các lớp Data Model

Đầu tiên, chúng ta cần tạo các lớp Dart tương ứng với cấu trúc JSON. Nhìn vào JSON, chúng ta thấy có 2 đối tượng: một đối tượng chính (công thức) và một đối tượng con (`author`).

**Tạo lớp `Author`:**

```dart
class Author {
  final String authorId;
  final String authorName;

  Author({
    required this.authorId,
    required this.authorName,
  });
}
```

**Tạo lớp `Recipe`:**

```dart
class Recipe {
  final int id;
  final String name;
  final Author author; // Một đối tượng của lớp Author
  final List<String> tags; // Một danh sách các chuỗi
  final bool isPublished;

  Recipe({
    required this.id,
    required this.name,
    required this.author,
    required this.tags,
    required this.isPublished,
  });
}
```

#### Bước 2: Viết Factory Constructor `fromJson()`

Hàm này sẽ nhận một `Map<String, dynamic>` (kết quả từ `json.decode()`) và tạo ra một đối tượng của lớp.

**Thêm `fromJson()` vào lớp `Author`:**

```dart
class Author {
  final String authorId;
  final String authorName;

  Author({
    required this.authorId,
    required this.authorName,
  });

  // Factory constructor để parse JSON
  factory Author.fromJson(Map<String, dynamic> json) {
    return Author(
      authorId: json['authorId'], // Lấy giá trị từ key 'authorId'
      authorName: json['authorName'], // Lấy giá trị từ key 'authorName'
    );
  }
}
```

**Thêm `fromJson()` vào lớp `Recipe`:**

```dart
class Recipe {
  // ... (các thuộc tính và constructor) ...

  factory Recipe.fromJson(Map<String, dynamic> json) {
    return Recipe(
      id: json['id'],
      name: json['name'],
      // Đối với đối tượng lồng nhau, gọi fromJson của lớp tương ứng
      author: Author.fromJson(json['author']),
      // Đối với danh sách, cần ép kiểu và chuyển đổi
      tags: List<String>.from(json['tags']),
      isPublished: json['isPublished'],
    );
  }
}
```
*   **Đối với đối tượng lồng nhau (`author`):** Chúng ta lấy `Map` con từ `json['author']` và truyền nó vào `Author.fromJson()` để tạo ra một đối tượng `Author`.
*   **Đối với danh sách (`tags`):** `json['tags']` sẽ là một `List<dynamic>`. Chúng ta dùng `List<String>.from()` để tạo một danh sách mới có kiểu `List<String>` từ nó.

#### Bước 3: Viết phương thức `toJson()`

Hàm này sẽ làm ngược lại, chuyển đổi đối tượng Dart thành một `Map<String, dynamic>`.

**Thêm `toJson()` vào lớp `Author`:**

```dart
class Author {
  // ... (các thuộc tính, constructor, fromJson) ...

  // Phương thức để chuyển đổi thành Map
  Map<String, dynamic> toJson() {
    return {
      'authorId': authorId,
      'authorName': authorName,
    };
  }
}
```

**Thêm `toJson()` vào lớp `Recipe`:**

```dart
class Recipe {
  // ... (các thuộc tính, constructor, fromJson) ...

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      // Đối với đối tượng lồng nhau, gọi toJson của đối tượng đó
      'author': author.toJson(),
      'tags': tags,
      'isPublished': isPublished,
    };
  }
}
```
*   **Đối với đối tượng lồng nhau (`author`):** Chúng ta gọi phương thức `author.toJson()` để lấy về một `Map` đại diện cho tác giả.

#### Cách sử dụng:

```dart
import 'dart:convert';

void main() {
  String jsonString = '''
  {
    "id": 1,
    "name": "Bún Chả Hà Nội",
    "author": {
      "authorId": "user123",
      "authorName": "Esheep Kitchen"
    },
    "tags": [
      "vietnamese",
      "hanoi",
      "grill"
    ],
    "isPublished": true
  }
  ''';

  // 1. Deserialization: JSON String -> Map -> Object
  Map<String, dynamic> jsonMap = json.decode(jsonString);
  Recipe myRecipe = Recipe.fromJson(jsonMap);
  print('Tên công thức: ${myRecipe.name}');
  print('Tên tác giả: ${myRecipe.author.authorName}');

  // 2. Serialization: Object -> Map -> JSON String
  Map<String, dynamic> recipeMap = myRecipe.toJson();
  String newJsonString = json.encode(recipeMap);
  print('JSON mới: $newJsonString');
}
```

---

### Cách 2: Sử dụng Code Generation (json_serializable) - Khuyến khích

Viết thủ công rất dễ xảy ra lỗi (gõ sai key, quên trường,...) và tốn thời gian. Các thư viện sinh mã (code generation) sẽ tự động làm việc này cho bạn một cách chính xác. Thư viện phổ biến nhất là `json_serializable`.

#### Bước 1: Thêm các dependencies

Mở file `pubspec.yaml` của bạn và thêm các gói sau:

```yaml
dependencies:
  flutter:
    sdk: flutter
  json_annotation: ^4.8.1 # Phiên bản có thể khác

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.6   # Phiên bản có thể khác
  json_serializable: ^6.7.1 # Phiên bản có thể khác
```
Sau đó chạy `flutter pub get` trong terminal.

#### Bước 2: Chuẩn bị các lớp Model

Sửa đổi các lớp của bạn để thêm các "chú thích" (`annotation`) và các "phần" (`part`) cần thiết.

**File `recipe.dart`:**

```dart
import 'package:json_annotation/json_annotation.dart';

// Dòng này rất quan trọng. Tên file phải khớp.
// Nó báo cho Dart biết file này sẽ có một file "bạn đồng hành" được sinh ra.
part 'recipe.g.dart';

// Annotation cho lớp Author
@JsonSerializable()
class Author {
  final String authorId;
  final String authorName;

  Author({
    required this.authorId,
    required this.authorName,
  });

  // Hai dòng này là tất cả những gì bạn cần!
  factory Author.fromJson(Map<String, dynamic> json) => _$AuthorFromJson(json);
  Map<String, dynamic> toJson() => _$AuthorToJson(this);
}


// Annotation cho lớp Recipe
// `explicitToJson: true` cần thiết khi có các đối tượng lồng nhau.
@JsonSerializable(explicitToJson: true)
class Recipe {
  final int id;
  final String name;
  final Author author;
  final List<String> tags;
  final bool isPublished;

  Recipe({
    required this.id,
    required this.name,
    required this.author,
    required this.tags,
    required this.isPublished,
  });

  // Tương tự, chỉ cần hai dòng này
  factory Recipe.fromJson(Map<String, dynamic> json) => _$RecipeFromJson(json);
  Map<String, dynamic> toJson() => _$RecipeToJson(this);
}
```
*   `@JsonSerializable()`: Báo cho `json_serializable` biết rằng nó cần tạo code cho lớp này.
*   `explicitToJson: true`: Cần thiết khi lớp của bạn chứa các đối tượng của các lớp khác cũng được chú thích (`Author` trong trường hợp này). Nó bảo trình sinh mã phải gọi `.toJson()` trên các đối tượng con đó.
*   `part 'recipe.g.dart';`: Liên kết file hiện tại với file sẽ được sinh ra.
*   `_$AuthorFromJson(json)` và `_$AuthorToJson(this)`: Đây là các hàm sẽ được **tự động tạo ra** trong file `recipe.g.dart`. Bạn chỉ cần gọi chúng.

#### Bước 3: Chạy trình sinh mã

Mở terminal trong thư mục gốc của dự án và chạy lệnh sau:

```bash
flutter pub run build_runner build
```
Lệnh này sẽ tìm tất cả các file có annotation và tạo ra các file `.g.dart` tương ứng. Trong trường-hợp này, nó sẽ tạo ra file `recipe.g.dart` chứa đầy đủ logic `fromJson` và `toJson` cho bạn.

Nếu bạn muốn nó tự động chạy lại mỗi khi bạn thay đổi file model, dùng lệnh `watch`:
```bash
flutter pub run build_runner watch
```

#### Bước 4: Sử dụng

Cách sử dụng hoàn toàn giống như cách thủ công, nhưng bạn không cần phải viết và bảo trì logic serialization/deserialization nữa.

### So sánh hai cách

| Tiêu chí | Viết thủ công | Dùng Code Generation (`json_serializable`) |
| :--- | :--- | :--- |
| **Ưu điểm** | - Không cần dependency. <br>- Hiểu rõ bản chất. <br>- Dễ dàng cho các model nhỏ. | - **Tự động 100%**. <br>- **An toàn, ít lỗi**. <br>- Dễ dàng bảo trì khi model thay đổi. <br>- Hỗ trợ các trường hợp phức tạp (đổi tên key, giá trị mặc định...). |
| **Nhược điểm**| - **Rất dễ lỗi** (gõ sai key). <br>- Tốn thời gian. <br>- Khó bảo trì khi JSON thay đổi. | - Cần thiết lập ban đầu. <br>- Cần chạy lệnh build_runner. <br>- Cảm giác "ma thuật" nếu không hiểu. |
| **Khuyến nghị** | Dùng để học hoặc cho các model cực kỳ đơn giản (1-2 trường). | **Cách làm tiêu chuẩn và được khuyến nghị** cho hầu hết các dự án Flutter. |

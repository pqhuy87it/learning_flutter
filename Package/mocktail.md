`mocktail` là một package tạo "đối tượng giả" (mock object) rất phổ biến, được xây dựng trên nền tảng null-safety của Dart. Nó thường được dùng để thay thế cho `mockito`.

Ưu điểm lớn của `mocktail` là nó **không yêu cầu chạy code generation** (giống như `mockito` ở các phiên bản cũ), giúp quá trình viết test nhanh hơn và bớt phức tạp hơn.

Dưới đây là hướng dẫn chi tiết cách sử dụng `mocktail` trong dự án Flutter/Dart.

-----

### 1\. Cài đặt

Thêm `mocktail` vào mục `dev_dependencies` trong file `pubspec.yaml` của bạn, vì package này chỉ dùng cho việc test.

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.3 # Kiểm tra phiên bản mới nhất trên pub.dev
```

Sau đó, chạy lệnh:

```bash
flutter pub get
```

-----

### 2\. Cách sử dụng cơ bản

Giả sử chúng ta có một class `CatService` mà chúng ta muốn "giả lập" (mock) trong khi test một class khác.

```dart
// Class thật mà chúng ta muốn mock
class CatService {
  String meow() => 'Meow';
  Future<String> fetchCatFact() async {
    // Tưởng tượng đây là một lệnh gọi API
    await Future.delayed(Duration(seconds: 1));
    return 'Cats are awesome!';
  }
  bool eat(String food) => food == 'fish';
}
```

#### 1\. Tạo Mock Class

Để tạo một mock cho `CatService`, bạn chỉ cần `extend Mock` và `implements` class đó.

```dart
import 'package:mocktail/mocktail.dart';

// Đặt file này trong thư mục /test
class MockCatService extends Mock implements CatService {}
```

Chỉ vậy thôi\! `MockCatService` giờ đã là một phiên bản "giả" hoàn chỉnh của `CatService`.

#### 2\. Stubbing (Thiết lập hành vi giả)

"Stubbing" là quá trình "dạy" cho mock object phải trả về cái gì khi một phương thức cụ thể được gọi. Chúng ta dùng hàm `when()`.

```dart
void main() {
  test('Stubbing a simple method', () {
    // Sắp xếp (Arrange)
    final mockService = MockCatService();

    // Dạy mock: "KHI (when) ai đó gọi hàm meow(), HÃY TRẢ VỀ (thenReturn) 'Grr'"
    when(() => mockService.meow()).thenReturn('Grr');

    // Hành động (Act)
    final result = mockService.meow();

    // Khẳng định (Assert)
    expect(result, 'Grr');
  });
}
```

  * **Với `Future` (Async):** Dùng `thenAnswer`.
    ```dart
    // Dạy mock: "KHI gọi fetchCatFact(), HÃY TRẢ VỀ một Future chứa 'Fake fact'"
    when(() => mockService.fetchCatFact()).thenAnswer((_) async => 'Fake fact');
    ```
  * **Với tham số:**
    ```dart
    // Chỉ trả về true NẾU tham số chính xác là 'fish'
    when(() => mockService.eat('fish')).thenReturn(true);
    // Trả về false NẾU tham số là bất cứ thứ gì khác 'fish' (dùng 'any()')
    when(() => mockService.eat(any(named: 'food'))).thenReturn(false);
    ```

#### 3\. Verifying (Kiểm tra tương tác)

"Verifying" là để kiểm tra xem một phương thức trên mock có được gọi hay không (và gọi bao nhiêu lần). Chúng ta dùng hàm `verify()`.

```dart
test('Verifying a method call', () {
  // Arrange
  final mockService = MockCatService();
  when(() => mockService.meow()).thenReturn('Grr');

  // Act
  mockService.meow(); // Gọi hàm 1 lần

  // Assert
  // KIỂM TRA (verify) rằng hàm meow() ĐÃ ĐƯỢC GỌI (called) 1 lần.
  verify(() => mockService.meow()).called(1);

  // KIỂM TRA (verify) rằng hàm eat('fish') KHÔNG BAO GIỜ được gọi.
  verifyNever(() => mockService.eat('fish'));
});
```

-----

### 3\. Ví dụ đầy đủ: Test một Repository

Đây là kịch bản phổ biến nhất: bạn có một `Repository` phụ thuộc vào một `Service` (như API client). Bạn muốn test `Repository` mà không cần gọi API thật.

**Các file của bạn:**

```dart
// 1. cat_service.dart (Class thật)
class CatService {
  Future<String> fetchCatFact() async {
    // Giả lập gọi API thật
    throw Exception('No internet!');
  }
}

// 2. cat_repository.dart (Class bạn MUỐN TEST)
class CatRepository {
  final CatService _service;
  CatRepository(this._service);

  Future<String> getFact() async {
    try {
      final fact = await _service.fetchCatFact();
      return fact.toUpperCase(); // Logic nghiệp vụ: viết hoa
    } catch (e) {
      return 'Failed to fetch fact';
    }
  }
}
```

**File test của bạn (`cat_repository_test.dart`):**

```dart
import 'package.flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// Import các file thật
import 'package:your_app/cat_service.dart';
import 'package:your_app/cat_repository.dart';

// 1. Tạo Mock Class
class MockCatService extends Mock implements CatService {}

void main() {
  // Khai báo các biến sẽ dùng trong test
  late CatRepository repository;
  late MockCatService mockService;

  // setUp() chạy trước MỖI test
  setUp(() {
    mockService = MockCatService();
    repository = CatRepository(mockService);
  });

  test('getFact trả về dữ liệu đã xử lý khi service thành công', () async {
    // Arrange (Sắp xếp)
    // Dạy mock service trả về 'fake fact' khi được gọi
    when(() => mockService.fetchCatFact()).thenAnswer((_) async => 'fake fact');

    // Act (Hành động)
    final result = await repository.getFact();

    // Assert (Khẳng định)
    // Kiểm tra xem repository có trả về 'FAKE FACT' (đã viết hoa) không
    expect(result, 'FAKE FACT');
    // Kiểm tra xem service có được gọi 1 lần không
    verify(() => mockService.fetchCatFact()).called(1);
  });

  test('getFact trả về "Failed" khi service báo lỗi', () async {
    // Arrange (Sắp xếp)
    // Dạy mock service ném ra một Exception khi được gọi
    when(() => mockService.fetchCatFact()).thenThrow(Exception('API Error'));

    // Act (Hành động)
    final result = await repository.getFact();

    // Assert (Khẳng định)
    // Kiểm tra xem repository có xử lý lỗi và trả về thông báo 'Failed' không
    expect(result, 'Failed to fetch fact');
    // Kiểm tra xem service vẫn được gọi
    verify(() => mockService.fetchCatFact()).called(1);
  });
}
```

-----

### 4\. Một số trường hợp nâng cao

#### 1\. Đối số (Matchers)

`mocktail` cung cấp các "matchers" (bộ khớp) để bạn không cần truyền giá trị chính xác.

  * `any()`: Khớp với bất kỳ giá trị nào.
    ```dart
    // Sẽ khớp khi gọi eat('fish'), eat('mouse'), v.v.
    when(() => mockService.eat(any())).thenReturn(true);
    ```
  * `argThat()`: Khớp với một điều kiện cụ thể.
    ```dart
    // Chỉ khớp khi tham số bắt đầu bằng 'f'
    when(() => mockService.eat(argThat(startsWith('f')))).thenReturn(true);
    ```

#### 2\. Xử lý `registerFallbackValue` (Rất quan trọng)

Nếu phương thức của bạn nhận một **đối tượng tùy chỉnh** (custom object) làm tham số, `mocktail` sẽ báo lỗi `MissingStubError` khi bạn dùng `any()`.

```dart
class User { ... }
class UserService {
  void saveUser(User user) { ... }
}

class MockUserService extends Mock implements UserService {}

// Dùng `any()` với User sẽ GÂY LỖI
when(() => mockService.saveUser(any())).thenAnswer(...); // LỖI
```

**Cách sửa:** Bạn phải đăng ký một "giá trị dự phòng" (fallback value) cho kiểu `User` đó.

```dart
// 1. Tạo một class User giả
class FakeUser extends Fake implements User {}

void main() {
  setUpAll(() {
    // 2. Đăng ký nó một lần trước khi chạy test
    registerFallbackValue(FakeUser());
  });

  test('...', () {
    final mockService = MockUserService();
    
    // 3. Bây giờ bạn có thể dùng any()
    when(() => mockService.saveUser(any())).thenAnswer((_) async {});
    
    mockService.saveUser(User(name: 'Test'));
    
    verify(() => mockService.saveUser(any())).called(1);
  });
}
```

Bạn có muốn xem thêm ví dụ về cách test Stream, BLoC hoặc một trường hợp cụ thể nào khác không?

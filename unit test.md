Chào bạn, rất vui được hướng dẫn bạn chi tiết về cách tạo và sử dụng **Unit Test** trong Flutter. Đây là một kỹ năng cực kỳ quan trọng giúp đảm bảo chất lượng code, giảm lỗi và giúp bạn tự tin hơn khi tái cấu trúc (refactor) ứng dụng.

### 1. Unit Test là gì và Tại sao nó quan trọng?

**Unit Test** là loại test ở cấp độ thấp nhất, dùng để kiểm tra một "đơn vị" (unit) code nhỏ nhất một cách độc lập. Một "đơn vị" ở đây thường là:
*   Một hàm (function).
*   Một phương thức (method) trong một class.
*   Toàn bộ một class.

**Mục tiêu của Unit Test:** Xác minh rằng một mảnh code cụ thể hoạt động đúng như mong đợi trong mọi tình huống (kể cả các trường hợp ngoại lệ) mà **không phụ thuộc** vào các phần khác của ứng dụng.

**Tại sao phải viết Unit Test?**
1.  **Tìm lỗi sớm:** Giúp bạn phát hiện lỗi logic ngay trong quá trình phát triển.
2.  **Tăng sự tự tin khi Refactor:** Khi bạn thay đổi hoặc tối ưu code, bạn chỉ cần chạy lại bộ test. Nếu tất cả đều pass, bạn có thể yên tâm rằng mình không làm hỏng chức năng hiện có.
3.  **Làm tài liệu sống:** Các bài test mô tả chính xác cách một hàm/class nên hoạt động, giúp người khác (và chính bạn trong tương lai) hiểu code nhanh hơn.
4.  **Thiết kế tốt hơn:** Việc phải viết test cho một class thường buộc bạn phải suy nghĩ về việc thiết kế class đó sao cho dễ kiểm thử, dẫn đến code có tính module cao hơn và ít phụ thuộc hơn.

### 2. Các loại Test trong Flutter

Flutter chia test thành 3 loại chính (tháp kiểm thử):
1.  **Unit Tests:** (Chủ đề chính của bài này) Nhanh nhất, kiểm tra logic Dart thuần túy, không cần trình giả lập hay thiết bị.
2.  **Widget Tests:** Kiểm tra một widget duy nhất. Nhanh hơn Integration Test, chạy trong một môi trường test đặc biệt.
3.  **Integration Tests:** (End-to-end tests) Chậm nhất, kiểm tra toàn bộ ứng dụng hoặc một luồng lớn, chạy trên trình giả lập hoặc thiết bị thật.

---

### Bước 1: Cài đặt và Cấu trúc

Tin tốt là mọi dự án Flutter mới đều được tạo sẵn với mọi thứ bạn cần để bắt đầu.

1.  **Dependencies:** Mở file `pubspec.yaml`, bạn sẽ thấy `flutter_test` đã có sẵn trong `dev_dependencies`:
    ```yaml
    dev_dependencies:
      flutter_test:
        sdk: flutter
      # ... các dependencies khác
    ```
2.  **Thư mục:** Tất cả các file test của bạn phải được đặt trong thư mục `test` ở gốc của dự án.
3.  **Quy ước đặt tên file:** Tên file test nên tương ứng với file code mà nó kiểm thử, và kết thúc bằng `_test.dart`.
    *   Code trong: `lib/utils/calculator.dart`
    *   Test cho nó: `test/utils/calculator_test.dart`

### Bước 2: Các thành phần cơ bản của một Unit Test

Một file unit test được xây dựng từ các hàm và khái niệm sau:

*   **`test(description, body)`:** Hàm chính để định nghĩa một trường hợp kiểm thử (test case).
    *   `description` (String): Mô tả rõ ràng mục đích của bài test. Ví dụ: `'should return 4 when adding 2 and 2'`.
    *   `body` (Function): Một hàm chứa logic của bài test.
*   **`expect(actual, matcher)`:** Hàm cốt lõi để xác thực kết quả.
    *   `actual`: Giá trị thực tế mà code của bạn trả về.
    *   `matcher`: Giá trị bạn mong đợi. Flutter cung cấp rất nhiều `Matcher` hữu ích như `equals()`, `isTrue`, `isFalse`, `throwsA()`, `isNull`,...
*   **`group(description, body)`:** Hàm để nhóm các bài test có liên quan lại với nhau, giúp file test của bạn có tổ chức và dễ đọc hơn.
*   **`setUp(body)` và `tearDown(body)`:**
    *   `setUp`: Một hàm sẽ được chạy **trước mỗi** bài test trong một `group` hoặc file. Thường dùng để khởi tạo đối tượng.
    *   `tearDown`: Một hàm sẽ được chạy **sau mỗi** bài test. Thường dùng để dọn dẹp tài nguyên.

### Bước 3: Viết Unit Test đầu tiên (Ví dụ đơn giản)

Giả sử chúng ta có một class `Calculator` đơn giản.

**File code: `lib/calculator.dart`**
```dart
class Calculator {
  int add(int a, int b) {
    return a + b;
  }

  int subtract(int a, int b) {
    return a - b;
  }
}
```

**File test: `test/calculator_test.dart`**
```dart
// 1. Import thư viện test và file code cần test
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/calculator.dart'; // Thay 'your_app_name'

void main() {
  // 2. Dùng group để nhóm các test cho class Calculator
  group('Calculator', () {
    late Calculator calculator;

    // 3. Dùng setUp để khởi tạo đối tượng trước mỗi test
    setUp(() {
      calculator = Calculator();
    });

    // 4. Viết một test case cho hàm add
    test('should return 4 when 2 is added to 2', () {
      // Arrange (Sắp xếp): Chuẩn bị các biến đầu vào
      const a = 2;
      const b = 2;

      // Act (Hành động): Gọi phương thức cần test
      final result = calculator.add(a, b);

      // Assert (Xác thực): Kiểm tra kết quả
      expect(result, 4);
    });

    // 5. Viết một test case khác cho hàm subtract
    test('should return 0 when 2 is subtracted from 2', () {
      // Arrange
      const a = 2;
      const b = 2;
      
      // Act
      final result = calculator.subtract(a, b);
      
      // Assert
      expect(result, 0);
    });
  });
}
```
**Cấu trúc AAA (Arrange, Act, Assert)** là một quy ước rất phổ biến giúp bài test rõ ràng:
*   **Arrange:** Chuẩn bị mọi thứ cần thiết cho bài test (khởi tạo đối tượng, tạo biến,...).
*   **Act:** Thực thi đoạn code bạn muốn kiểm tra.
*   **Assert:** Dùng `expect` để kiểm tra xem kết quả có đúng như mong đợi không.

### Bước 4: Test Code Bất đồng bộ (Asynchronous)

Nếu hàm của bạn trả về một `Future`, bạn chỉ cần biến hàm `body` của `test` thành `async` và dùng `await`.

**File code: `lib/user_repository.dart`**
```dart
class UserRepository {
  Future<String> fetchUserName(int id) async {
    await Future.delayed(const Duration(seconds: 1));
    if (id == 1) {
      return 'PrivateGPT';
    } else {
      throw Exception('User not found');
    }
  }
}
```

**File test: `test/user_repository_test.dart`**
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/user_repository.dart';

void main() {
  group('UserRepository', () {
    final repository = UserRepository();

    test('should return user name when user is found', () async { // Dùng async
      // Act
      final userName = await repository.fetchUserName(1); // Dùng await

      // Assert
      expect(userName, 'PrivateGPT');

      // Hoặc dùng matcher 'completion'
      // expect(repository.fetchUserName(1), completion(equals('PrivateGPT')));
    });

    test('should throw an exception when user is not found', () {
      // Assert
      // Dùng matcher 'throwsA' để kiểm tra ngoại lệ
      expect(() => repository.fetchUserName(2), throwsA(isA<Exception>()));
    });
  });
}
```

### Bước 5: Mocking - Giả lập các Dependencies

Đây là phần quan trọng nhất trong unit test thực tế. **Unit test phải được cô lập.** Nếu class `A` của bạn phụ thuộc vào class `B` (ví dụ: `B` là một service gọi API), bạn không muốn gọi API thật trong unit test của `A`. Thay vào đó, bạn **giả lập (mock)** class `B`.

Chúng ta sẽ dùng thư viện `mockito`.

1.  **Thêm dependencies:**
    ```yaml
    dev_dependencies:
      mockito: ^5.0.0
      build_runner: ^2.0.0
    ```
2.  **Tạo một class có dependency:**

    **File code: `lib/weather_service.dart`**
    ```dart
    import 'package:http/http.dart' as http;

    // Đây là dependency mà chúng ta sẽ mock
    class ApiClient {
      Future<String> get(String url) async {
        final response = await http.get(Uri.parse(url));
        return response.body;
      }
    }
    
    class WeatherService {
      final ApiClient apiClient;

      WeatherService(this.apiClient);

      Future<String> getCurrentWeather() async {
        final result = await apiClient.get('https://api.weather.com/current');
        if (result == '{"temp": 25, "condition": "Sunny"}') {
          return 'Sunny, 25°C';
        }
        return 'Unknown';
      }
    }
    ```

3.  **Viết file test và tạo mock:**

    **File test: `test/weather_service_test.dart`**
    ```dart
    import 'package:flutter_test/flutter_test.dart';
    import 'package:mockito/annotations.dart';
    import 'package:mockito/mockito.dart';
    import 'package:your_app_name/weather_service.dart';

    // 1. Annotation để bảo mockito tạo mock cho class ApiClient
    @GenerateMocks([ApiClient])
    import 'weather_service_test.mocks.dart'; // File này sẽ được tạo tự động

    void main() {
      late MockApiClient mockApiClient;
      late WeatherService weatherService;

      setUp(() {
        // 2. Khởi tạo mock và class cần test
        mockApiClient = MockApiClient();
        weatherService = WeatherService(mockApiClient);
      });

      test('should return formatted weather when API call is successful', () async {
        // 3. Arrange - Dạy cho mock phải trả về gì khi được gọi
        when(mockApiClient.get('https://api.weather.com/current'))
            .thenAnswer((_) async => '{"temp": 25, "condition": "Sunny"}');

        // 4. Act - Gọi phương thức
        final result = await weatherService.getCurrentWeather();

        // 5. Assert - Kiểm tra kết quả
        expect(result, 'Sunny, 25°C');

        // (Tùy chọn) 6. Verify - Kiểm tra xem phương thức của mock có được gọi không
        verify(mockApiClient.get('https://api.weather.com/current')).called(1);
      });
    }
    ```

4.  **Chạy code generator:**
    Mở terminal và chạy lệnh sau để `mockito` tạo ra file `weather_service_test.mocks.dart`:
    ```bash
    dart run build_runner build
    ```

### Bước 6: Chạy Tests

Có hai cách chính:
1.  **Từ Terminal:**
    ```bash
    # Chạy tất cả các test trong dự án
    flutter test

    # Chạy tất cả các test trong một file cụ thể
    flutter test test/calculator_test.dart
    ```
2.  **Từ IDE (VS Code, Android Studio):**
    *   Bạn sẽ thấy các nút `Run | Debug` nhỏ bên cạnh các hàm `main`, `group`, và `test`.
    *   Bạn có thể nhấn vào chúng để chạy hoặc debug một nhóm test hoặc một test case duy nhất.
    *   IDE cũng cung cấp một giao diện trực quan để xem kết quả test (pass/fail).

Chúc bạn thành công với việc viết unit test cho ứng dụng Flutter của mình

Chào bạn, rất vui được giới thiệu chi tiết về `mockito`, một trong những package **quan trọng và cần thiết nhất** để viết unit test chuyên nghiệp trong Flutter.

### 1. Mockito là gì? - Diễn viên đóng thế cho Code của bạn

Hãy tưởng tượng bạn đang quay một bộ phim hành động. Có một cảnh nguy hiểm mà bạn không muốn diễn viên chính của mình thực hiện. Bạn sẽ làm gì? Bạn thuê một **diễn viên đóng thế (stunt double)**. Người này trông giống diễn viên chính, có thể thực hiện hành động nguy hiểm, nhưng không phải là diễn viên chính thật.

**Mockito trong Flutter chính là công cụ để tạo ra các "diễn viên đóng thế" cho code của bạn.**

Những "diễn viên" này được gọi là **Mocks** (đối tượng giả). Một mock là một đối tượng giả lập, nó "giả vờ" là một class thật nhưng hành vi của nó hoàn toàn nằm dưới sự kiểm soát của bạn trong môi trường test.

### 2. Tại sao chúng ta CẦN Mocking? - Vấn đề của sự Phụ thuộc (Dependency)

Trong một ứng dụng thực tế, các class hiếm khi hoạt động một mình. Chúng thường phụ thuộc vào các class khác để hoàn thành công việc. Ví dụ:
*   Một `ProfileViewModel` phụ thuộc vào một `UserRepository` để lấy thông tin người dùng.
*   Một `UserRepository` lại phụ thuộc vào một `ApiService` để gọi API.
*   Một `ApiService` lại phụ thuộc vào package `http` để thực hiện request mạng.

Khi bạn viết **unit test** cho `ProfileViewModel`, mục tiêu của bạn là chỉ kiểm tra logic của **chính `ProfileViewModel`** mà thôi. Bạn **không** muốn:
*   Thực sự gọi đến `UserRepository`.
*   Thực sự gọi API qua `ApiService`.
*   Thực sự tạo một request mạng qua `http`.

**Tại sao không?**
1.  **Cô lập (Isolation):** Nếu test của bạn thất bại, bạn muốn biết chắc chắn rằng lỗi nằm ở `ProfileViewModel`, chứ không phải do mạng chậm, API server sập, hay lỗi logic trong `UserRepository`.
2.  **Tốc độ (Speed):** Gọi mạng hoặc đọc database thật rất chậm. Unit test phải chạy trong mili giây.
3.  **Khả năng kiểm soát (Control):** Bạn muốn dễ dàng giả lập mọi kịch bản: API trả về thành công, API trả về lỗi 404, API trả về dữ liệu rỗng, mạng bị ngắt,... Điều này rất khó thực hiện với các dependency thật.

**Mockito giải quyết tất cả các vấn đề này** bằng cách cho phép bạn tạo ra một `MockUserRepository` giả. Bạn có thể "dạy" cho mock này phải trả về dữ liệu gì hoặc phải ném ra lỗi gì khi một phương thức cụ thể của nó được gọi, mà không cần thực thi code thật bên trong nó.

---

### 3. Hướng dẫn sử dụng Mockito từng bước

Hãy cùng xem một ví dụ thực tế: test một `WeatherService` phụ thuộc vào một `ApiClient`.

#### Bước 1: Cài đặt

Thêm `mockito` và `build_runner` vào file `pubspec.yaml` của bạn.
*   `mockito`: Thư viện chính.
*   `build_runner`: Công cụ để tự động sinh code mock cho bạn.

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.4 # Luôn dùng phiên bản mới nhất
  build_runner: ^2.4.7 # Luôn dùng phiên bản mới nhất
```
Sau đó chạy `flutter pub get`.

#### Bước 2: Chuẩn bị Code để Test

Giả sử bạn có 2 class sau:

**File: `lib/api_client.dart`** (Dependency cần được mock)
```dart
import 'package:http/http.dart' as http;

class ApiClient {
  Future<String> fetchWeatherData(String city) async {
    // Đây là phần gọi API thật mà chúng ta muốn tránh trong unit test
    final response = await http.get(Uri.parse('https://api.weather.com/data?city=$city'));
    if (response.statusCode == 200) {
      return response.body;
    } else {
      throw Exception('Failed to load weather data');
    }
  }
}
```

**File: `lib/weather_service.dart`** (Class cần được test)
```dart
import 'api_client.dart';
import 'dart:convert';

class WeatherService {
  final ApiClient apiClient;

  // Lớp này phụ thuộc vào ApiClient
  WeatherService(this.apiClient);

  Future<String> getFormattedWeather(String city) async {
    try {
      final jsonData = await apiClient.fetchWeatherData(city);
      final weatherData = jsonDecode(jsonData);
      final temp = weatherData['temperature'];
      final condition = weatherData['condition'];
      return '$condition, ${temp}°C';
    } catch (e) {
      return 'Could not fetch weather';
    }
  }
}
```

#### Bước 3: Viết Test và Sinh Code Mock

**File test: `test/weather_service_test.dart`**

1.  **Import và thêm Annotation:**
    ```dart
    import 'package:flutter_test/flutter_test.dart';
    import 'package:mockito/annotations.dart';
    import 'package:mockito/mockito.dart';
    import 'package:your_app_name/api_client.dart'; // Import class thật
    import 'package:your_app_name/weather_service.dart';

    // 1. Dùng annotation để chỉ định class nào cần được mock
    @GenerateMocks([ApiClient])
    // 2. Import file .mocks.dart (sẽ được tạo ra). Ban đầu nó sẽ báo lỗi.
    import 'weather_service_test.mocks.dart'; 
    
    void main() {
      // ... code test sẽ ở đây
    }
    ```

2.  **Chạy Code Generator:**
    Mở terminal ở thư mục gốc của dự án và chạy lệnh:
    ```bash
    dart run build_runner build
    ```
    Lệnh này sẽ đọc annotation `@GenerateMocks` và tự động tạo ra file `weather_service_test.mocks.dart` chứa class `MockApiClient`. Lỗi import ở trên sẽ biến mất.

3.  **Viết các Test Case:**
    ```dart
    void main() {
      late MockApiClient mockApiClient;
      late WeatherService weatherService;

      // Dùng setUp để khởi tạo các đối tượng trước mỗi bài test
      setUp(() {
        mockApiClient = MockApiClient();
        weatherService = WeatherService(mockApiClient);
      });

      group('getFormattedWeather', () {
        const testCity = 'Tokyo';
        const testJson = '{"temperature": 25, "condition": "Sunny"}';

        test('should return formatted string when API call is successful', () async {
          // Arrange (Sắp xếp) - "Dạy" cho mock phải làm gì
          // Khi phương thức fetchWeatherData với tham số 'Tokyo' được gọi...
          when(mockApiClient.fetchWeatherData(testCity))
              // ...thì trả về một Future hoàn thành với chuỗi JSON giả
              .thenAnswer((_) async => testJson);

          // Act (Hành động) - Gọi phương thức cần test
          final result = await weatherService.getFormattedWeather(testCity);

          // Assert (Xác thực) - Kiểm tra kết quả
          expect(result, 'Sunny, 25°C');
        });

        test('should return error string when API call throws an exception', () async {
          // Arrange - Dạy cho mock ném ra một lỗi
          when(mockApiClient.fetchWeatherData(testCity))
              .thenThrow(Exception('API Error'));

          // Act
          final result = await weatherService.getFormattedWeather(testCity);

          // Assert
          expect(result, 'Could not fetch weather');
        });
      });
    }
    ```

#### Bước 4: Các Hàm Mockito Quan trọng

*   **`when(mockObject.method(arguments))`**:
    Bắt đầu "dạy" cho mock. Bạn đặt vào đây lời gọi phương thức mà bạn mong đợi sẽ xảy ra.

*   **`.thenAnswer((_) async => value)`**:
    Dùng cho các phương thức bất đồng bộ (`Future`). Nó trả về một `Future` hoàn thành với `value` bạn cung cấp.

*   **`.thenReturn(value)`**:
    Dùng cho các phương thức đồng bộ. Nó trả về trực tiếp `value`.

*   **`.thenThrow(Exception())`**:
    Bắt mock ném ra một ngoại lệ.

*   **`verify(mockObject.method(arguments))`**:
    Kiểm tra xem một phương thức trên mock **có được gọi hay không**. Rất hữu ích để đảm bảo code của bạn tương tác đúng với dependency.
    ```dart
    // Kiểm tra xem fetchWeatherData có được gọi đúng 1 lần không
    verify(mockApiClient.fetchWeatherData(testCity)).called(1);
    ```

*   **Argument Matchers (`any`, `argThat`)**:
    Đôi khi bạn không quan tâm chính xác tham số truyền vào là gì.
    ```dart
    // Chấp nhận bất kỳ chuỗi nào được truyền vào
    when(mockApiClient.fetchWeatherData(any)).thenAnswer((_) async => testJson);
    ```

### Kết luận: Lợi ích của Mockito

1.  **Kiểm thử cô lập:** Đảm bảo bạn chỉ đang test logic của một đơn vị duy nhất.
2.  **Tốc độ:** Test chạy cực nhanh vì không có I/O (mạng, đĩa).
3.  **Độ tin cậy:** Test không bị thất bại do các yếu tố bên ngoài (mạng, API).
4.  **Mô phỏng dễ dàng:** Cho phép bạn dễ dàng tạo ra mọi tình huống (thành công, lỗi, dữ liệu trống,...) để kiểm tra code của mình xử lý chúng như thế nào.

Tóm lại, `mockito` là một công cụ không thể thiếu để viết unit test chất lượng cao, giúp bạn xây dựng các ứng dụng Flutter ổn định, dễ bảo trì và đáng tin cậy.

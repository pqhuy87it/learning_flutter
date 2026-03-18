Chào bạn, rất vui được giới thiệu về `dio`, một trong những package HTTP client mạnh mẽ và phổ biến nhất trong hệ sinh thái Flutter/Dart.

Dưới đây là một bài giới thiệu chi tiết về `dio`, từ khái niệm cơ bản đến các tính năng nâng cao và ví dụ thực tế.

### Dio là gì?

**Dio** là một thư viện HTTP client mạnh mẽ dành cho Dart, hỗ trợ các tính năng mà thư viện `http` mặc định không có sẵn, chẳng hạn như Interceptors, cấu hình toàn cục, xử lý FormData, theo dõi tiến trình tải lên/tải xuống, hủy yêu cầu, và nhiều hơn nữa.

Nó giúp việc giao tiếp với các API REST trở nên dễ dàng, linh hoạt và có khả năng mở rộng hơn rất nhiều, đặc biệt là trong các dự án lớn.

### Tại sao nên chọn Dio thay vì package `http` mặc định?

| Tính năng | Package `http` (mặc định) | Package `dio` |
| :--- | :--- | :--- |
| **Interceptors** | ❌ Không có sẵn | ✅ Có (Cho phép chặn và xử lý request, response, error) |
| **Cấu hình toàn cục** | ❌ Khá hạn chế | ✅ Có (Base URL, timeouts, headers có thể cấu hình một lần) |
| **Xử lý lỗi** | Cơ bản, qua `try-catch` | ✅ Mạnh mẽ, có các loại `DioException` cụ thể (timeout, bad response,...) |
| **FormData** | ❌ Phải tự xây dựng phức tạp | ✅ Hỗ trợ sẵn để gửi dữ liệu form và upload file |
| **Tải xuống file** | Có thể làm được nhưng phức tạp | ✅ Có phương thức `download()` riêng với theo dõi tiến trình |
| **Tải lên file** | Phải tự xây dựng `MultipartRequest` | ✅ Đơn giản qua `FormData`, có theo dõi tiến trình |
| **Hủy yêu cầu** | ❌ Không hỗ trợ | ✅ Có, thông qua `CancelToken` |
| **Timeouts** | Có thể cấu hình | ✅ Cấu hình chi tiết (connect, receive, send timeout) |
| **Tự động giải mã JSON** | ❌ Phải tự `json.decode()` | ✅ Tự động giải mã JSON trong response |

### 1. Cài đặt

Thêm `dio` vào file `pubspec.yaml` của bạn:

```yaml
dependencies:
  flutter:
    sdk: flutter
  dio: ^5.4.3+1 # Luôn kiểm tra phiên bản mới nhất trên pub.dev
```

Sau đó, chạy lệnh `flutter pub get` trong terminal.

### 2. Cách sử dụng cơ bản

Import package và tạo một instance của `Dio`.

```dart
import 'package:dio/dio.dart';

final dio = Dio();

void getHttp() async {
  try {
    final response = await dio.get('https://jsonplaceholder.typicode.com/posts/1');
    print(response.data); // Dio tự động giải mã JSON
    print(response.statusCode); // 200
  } catch (e) {
    print(e);
  }
}
```

### 3. Các tính năng chính và ví dụ chi tiết

#### a. Cấu hình toàn cục (BaseOptions)

Thay vì lặp lại URL, headers, hay timeout cho mỗi request, bạn có thể cấu hình một lần.

```dart
import 'package:dio/dio.dart';

// Tạo một instance Dio với cấu hình cơ bản
final dio = Dio(
  BaseOptions(
    baseUrl: 'https://api.yourdomain.com/',
    connectTimeout: Duration(seconds: 5), // 5 giây
    receiveTimeout: Duration(seconds: 3), // 3 giây
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
    },
  ),
);

// Bây giờ bạn chỉ cần gọi endpoint
void fetchUserData() async {
  try {
    // Sẽ gọi đến: https://api.yourdomain.com/users/1
    final response = await dio.get('/users/1'); 
    print(response.data);
  } catch (e) {
    print(e);
  }
}
```

#### b. Interceptors (Bộ chặn)

Đây là tính năng mạnh mẽ nhất của `dio`. Interceptors cho phép bạn "chặn" và can thiệp vào các quá trình của một HTTP request.

*   `onRequest`: Chặn trước khi request được gửi đi (ví dụ: thêm token xác thực).
*   `onResponse`: Chặn sau khi nhận được response thành công.
*   `onError`: Chặn khi có lỗi xảy ra (ví dụ: refresh token khi token hết hạn).

**Ví dụ: Thêm Access Token vào tất cả các request**

```dart
import 'package:dio/dio.dart';

// Giả sử bạn lưu token ở đâu đó
String? accessToken = "your_secret_token_here";

final dio = Dio();

void setupInterceptors() {
  dio.interceptors.add(
    InterceptorsWrapper(
      onRequest: (options, handler) {
        // Thêm access token vào header của mỗi request
        if (accessToken != null) {
          options.headers['Authorization'] = 'Bearer $accessToken';
        }
        print('REQUEST[${options.method}] => PATH: ${options.path}');
        // Phải gọi handler.next(options) để request được tiếp tục
        return handler.next(options); 
      },
      onResponse: (response, handler) {
        print('RESPONSE[${response.statusCode}] => DATA: ${response.data}');
        // Tiếp tục chuỗi response
        return handler.next(response);
      },
      onError: (DioException e, handler) {
        print('ERROR[${e.response?.statusCode}] => MESSAGE: ${e.message}');
        // Xử lý lỗi, ví dụ refresh token nếu lỗi 401
        if (e.response?.statusCode == 401) {
          // Logic refresh token ở đây...
        }
        // Tiếp tục chuỗi lỗi
        return handler.next(e);
      },
    ),
  );
}
```

#### c. Xử lý lỗi (Error Handling)

`Dio` ném ra `DioException` khi có lỗi, chứa rất nhiều thông tin hữu ích.

```dart
void fetchDataWithErrorHandling() async {
  try {
    await dio.get('https://your-api.com/non-existent-endpoint');
  } on DioException catch (e) {
    // Lỗi liên quan đến response từ server (404, 500,...)
    if (e.type == DioExceptionType.badResponse) {
      print('Lỗi response: ${e.response?.statusCode}');
      print('Nội dung lỗi: ${e.response?.data}');
    } 
    // Lỗi kết nối, timeout,...
    else if (e.type == DioExceptionType.connectionTimeout) {
      print('Lỗi: Hết thời gian kết nối.');
    } 
    // Yêu cầu bị hủy
    else if (e.type == DioExceptionType.cancel) {
      print('Yêu cầu đã bị hủy.');
    }
    // Các loại lỗi khác
    else {
      print('Lỗi không xác định: ${e.message}');
    }
  }
}
```

#### d. Gửi dữ liệu (POST, PUT)

Bạn có thể gửi một `Map` và `dio` sẽ tự động chuyển nó thành JSON.

```dart
void createUser() async {
  try {
    final response = await dio.post(
      '/users',
      data: {
        'name': 'John Doe',
        'email': 'john.doe@example.com',
      },
    );
    print('User created: ${response.data}');
  } catch (e) {
    print(e);
  }
}
```

#### e. Tải file lên (Uploading Files) với `FormData`

`FormData` được dùng để gửi dữ liệu dạng `multipart/form-data`, rất hữu ích khi upload file.

```dart
import 'package:dio/dio.dart';
import 'package:image_picker/image_picker.dart'; // Giả sử bạn dùng image_picker

Future<void> uploadProfilePicture(XFile imageFile) async {
  try {
    String fileName = imageFile.path.split('/').last;
    
    FormData formData = FormData.fromMap({
      "file": await MultipartFile.fromFile(imageFile.path, filename: fileName),
      "user_id": 123, // Gửi kèm các dữ liệu khác
    });

    final response = await dio.post(
      '/upload-avatar',
      data: formData,
      onSendProgress: (int sent, int total) {
        // Theo dõi tiến trình upload
        double percentage = (sent / total * 100);
        print('Đã tải lên: ${percentage.toStringAsFixed(2)}%');
      },
    );
    print('Upload thành công: ${response.data}');
  } catch (e) {
    print('Upload thất bại: $e');
  }
}
```

#### f. Tải file xuống (Downloading Files)

`dio` cung cấp một phương thức `download` riêng biệt.

```dart
import 'package:dio/dio.dart';
import 'package:path_provider/path_provider.dart'; // Cần package path_provider

Future<void> downloadFile() async {
  try {
    // Lấy đường dẫn thư mục để lưu file
    final dir = await getApplicationDocumentsDirectory();
    final savePath = '${dir.path}/my_downloaded_file.zip';

    await dio.download(
      'https://link-to-your-file.com/file.zip',
      savePath,
      onReceiveProgress: (received, total) {
        if (total != -1) {
          double percentage = (received / total * 100);
          print('Đã tải xuống: ${percentage.toStringAsFixed(2)}%');
        }
      },
    );
    print('Tải file thành công, đã lưu tại: $savePath');
  } catch (e) {
    print('Tải file thất bại: $e');
  }
}
```

#### g. Hủy yêu cầu (Cancelling Requests)

Đôi khi bạn cần hủy một request đang chạy (ví dụ: khi người dùng rời khỏi màn hình).

```dart
final cancelToken = CancelToken();

void fetchWithCancellation() async {
  try {
    // Bắt đầu một request
    dio.get(
      'https://jsonplaceholder.typicode.com/posts',
      cancelToken: cancelToken,
    );

    // Sau 100ms, hủy request
    Future.delayed(Duration(milliseconds: 100), () {
      if (!cancelToken.isCancelled) {
        cancelToken.cancel("Yêu cầu đã bị người dùng hủy.");
      }
    });

  } on DioException catch (e) {
    if (CancelToken.isCancel(e)) {
      print('Request bị hủy: ${e.message}');
    } else {
      // Xử lý các lỗi khác
    }
  }
}
```

### Kết luận

**Dio** là một công cụ không thể thiếu cho các nhà phát triển Flutter khi làm việc với networking. Nó cung cấp một API mạnh mẽ, linh hoạt và dễ sử dụng, giúp giải quyết hầu hết các tác vụ mạng phức tạp một cách gọn gàng. Nếu dự án của bạn cần nhiều hơn là các request GET/POST đơn giản, việc sử dụng `dio` sẽ giúp bạn tiết kiệm rất nhiều thời gian và công sức.

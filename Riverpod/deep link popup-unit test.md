Chào bạn, để viết unit test cho `appLinkControllerProvider`, chúng ta cần kiểm thử logic bên trong hàm `listen` của Stream.

Vì provider này không trả về giá trị (return `void`) mà chỉ thực hiện các **Side Effects** (gọi hàm khác), nên chiến lược test sẽ là:

1.  Giả lập (Mock) các dependency: `Repository`, `NavigationService`, và `ScreenEController`.
2.  Giả lập `Stream` bằng `StreamController` để chúng ta có thể chủ động bắn sự kiện `Uri` vào.
3.  Kiểm tra (Verify) xem các hàm tương ứng có được gọi đúng theo logic `if/else` không.

Dưới đây là code unit test chi tiết sử dụng `mocktail` và `flutter_riverpod`.

### 1\. Chuẩn bị Mocks

```dart
import 'dart:async';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Import các file thật của bạn
// import 'package:your_app/app_link_controller.dart';
// import 'package:your_app/app_links_repository.dart';
// import 'package:your_app/navigation_service.dart';
// import 'package:your_app/screen_e_controller.dart';

// --- 1. TẠO CÁC MOCK CLASSES ---

class MockAppLinkRepository extends Mock implements AppLinkRepository {}

class MockNavigationService extends Mock implements NavigationService {}

// Vì ScreenEController là StateNotifier, ta cần kế thừa nó để Mock hoạt động trơn tru với Riverpod
class MockScreenEController extends StateNotifier<ScreenEEvent> 
    with Mock 
    implements ScreenEController {
  MockScreenEController() : super(InitialEvent());
}
```

### 2\. Viết Test Case

```dart
void main() {
  late ProviderContainer container;
  late MockAppLinkRepository mockRepo;
  late MockNavigationService mockNavService;
  late MockScreenEController mockScreenEController;
  late StreamController<Uri> uriStreamController;

  setUp(() {
    // 1. Khởi tạo các Mock và StreamController
    mockRepo = MockAppLinkRepository();
    mockNavService = MockNavigationService();
    mockScreenEController = MockScreenEController();
    uriStreamController = StreamController<Uri>.broadcast();

    // 2. Giả lập hành vi cho Repository (trả về stream của chúng ta)
    when(() => mockRepo.uriStream).thenAnswer((_) => uriStreamController.stream);

    // 3. Khởi tạo ProviderContainer với các Override
    container = ProviderContainer(
      overrides: [
        appLinkRepositoryProvider.overrideWithValue(mockRepo),
        navigationServiceProvider.overrideWithValue(mockNavService),
        // Override Controller để bắt được lệnh gọi ref.read(...).onDeepLinkReceived
        screenEControllerProvider.overrideWith((ref) => mockScreenEController),
        // (Nếu code bạn có dùng screenEProvider thì override nó luôn, nếu không thì bỏ qua)
        // screenEProvider.overrideWith((ref) => mockScreenENotifier),
      ],
    );

    // 4. QUAN TRỌNG: Khởi tạo Controller để nó bắt đầu lắng nghe Stream
    // Vì Provider là void nên ta chỉ cần read nó 1 lần để hàm body chạy
    container.read(appLinkControllerProvider);
  });

  tearDown(() {
    uriStreamController.close();
    container.dispose();
  });

  group('appLinkControllerProvider Logic', () {
    
    // TEST CASE 1: Link KHÔNG chứa '/e'
    test('Should DO NOTHING if uri path does not contain "/e"', () async {
      // Arrange
      final uri = Uri.parse('myapp://home');

      // Act
      uriStreamController.add(uri);
      await Future.microtask(() {}); // Chờ stream xử lý

      // Assert
      // Đảm bảo không có hàm nào của logic Screen E được gọi
      verifyNever(() => mockNavService.isAtScreenE);
      verifyNever(() => mockNavService.requestNavigateToE(any()));
      verifyNever(() => mockScreenEController.onDeepLinkReceived(any()));
    });

    // TEST CASE 2: Link chứa '/e' VÀ Đang ở màn hình E (isAtScreenE = true)
    test('Should call onDeepLinkReceived if at Screen E', () async {
      // Arrange
      final uri = Uri.parse('myapp://example.com/e?data=hello_world');
      // Giả lập đang ở màn hình E
      when(() => mockNavService.isAtScreenE).thenReturn(true);

      // Act
      uriStreamController.add(uri);
      await Future.microtask(() {}); // Chờ stream xử lý

      // Assert
      // 1. Kiểm tra xem đã check vị trí chưa
      verify(() => mockNavService.isAtScreenE).called(1);
      
      // 2. Kiểm tra xem hàm onDeepLinkReceived có được gọi với đúng data không
      verify(() => mockScreenEController.onDeepLinkReceived('hello_world')).called(1);
      
      // 3. Đảm bảo KHÔNG gọi hàm chuyển trang
      verifyNever(() => mockNavService.requestNavigateToE(any()));
    });

    // TEST CASE 3: Link chứa '/e' VÀ KHÔNG ở màn hình E (isAtScreenE = false)
    test('Should call requestNavigateToE if NOT at Screen E', () async {
      // Arrange
      final uri = Uri.parse('myapp://example.com/e?data=hello_world');
      // Giả lập KHÔNG ở màn hình E
      when(() => mockNavService.isAtScreenE).thenReturn(false);

      // Act
      uriStreamController.add(uri);
      await Future.microtask(() {});

      // Assert
      verify(() => mockNavService.isAtScreenE).called(1);
      
      // 1. Kiểm tra hàm điều hướng có được gọi đúng data không
      verify(() => mockNavService.requestNavigateToE('hello_world')).called(1);
      
      // 2. Đảm bảo KHÔNG gọi hàm xử lý tại chỗ
      verifyNever(() => mockScreenEController.onDeepLinkReceived(any()));
    });

    // TEST CASE 4: Kiểm tra xử lý data null/empty
    test('Should handle missing query parameters gracefully', () async {
      // Arrange
      final uri = Uri.parse('myapp://example.com/e'); // Không có ?data=...
      when(() => mockNavService.isAtScreenE).thenReturn(false);

      // Act
      uriStreamController.add(uri);
      await Future.microtask(() {});

      // Assert
      // Kiểm tra xem data truyền vào có phải là chuỗi rỗng '' như logic (?? '') không
      verify(() => mockNavService.requestNavigateToE('')).called(1);
    });
  });
}
```

### Giải thích các điểm quan trọng:

1.  **`StreamController.broadcast()`**: Chúng ta dùng cái này để giả lập `repo.uriStream`. Khi ta gọi `.add()`, nó giống như việc `app_links` nhận được một link thực sự từ hệ thống.
2.  **`container.read(appLinkControllerProvider)`**: Vì provider của bạn trả về `void`, nó sẽ không tự chạy trừ khi có ai đó "đọc" hoặc "lắng nghe" nó. Dòng này kích hoạt logic khởi tạo (và dòng `repo.uriStream.listen` bên trong nó).
3.  **`await Future.microtask(() {})`**: Vì Stream hoạt động bất đồng bộ, sau khi `add` sự kiện, chúng ta cần chờ một chút để Event Loop xử lý xong logic bên trong listener trước khi `verify`.
4.  **Overrides**: Việc override `screenEControllerProvider` là rất quan trọng vì bên trong logic bạn có gọi `ref.read(screenEControllerProvider.notifier)`. Nếu không mock, nó sẽ cố gọi controller thật.

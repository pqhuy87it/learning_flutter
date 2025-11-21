Chào bạn, đây là một yêu cầu rất cụ thể về **Side Effect (Tác vụ phụ)** trong UI: Hiển thị Dialog từ một sự kiện bên ngoài (Deep Link).

Để tuân thủ **SOLID** và giải quyết bài toán này bằng **Riverpod**, chúng ta cần tách biệt rõ ràng:

1.  **Logic nhận Deep Link:** (Global Controller).
2.  **Logic nghiệp vụ tại màn hình E:** (ScreenEController - Quản lý trạng thái dialog).
3.  **Giao diện:** (ScreenE - Chỉ việc hiển thị dialog khi được yêu cầu).

Giải pháp sử dụng `ref.listen` của Riverpod là chìa khóa để hiện Dialog.

### Cấu trúc giải pháp

1.  **Domain Layer:** Định nghĩa Event.
2.  **Infrastructure Layer:** `DeepLinkService` (Lắng nghe AppLinks).
3.  **Presentation Layer (Logic):** `ScreenEController` (Quản lý trạng thái popup).
4.  **Presentation Layer (UI):** `ScreenE` (Lắng nghe Controller để show dialog).

-----

### 1\. Định nghĩa Sự kiện (Event State)

Chúng ta cần một State để báo hiệu cho UI biết "Hãy hiện Popup đi".

```dart
// screen_e_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';

// Dùng Freezed hoặc Equatable để so sánh state (Ở đây code chay cho đơn giản)
abstract class ScreenEEvent {}

class InitialEvent extends ScreenEEvent {}

// Sự kiện yêu cầu hiện Popup
class ShowDeepLinkPopupEvent extends ScreenEEvent {
  final String data;
  ShowDeepLinkPopupEvent(this.data);
}
```

### 2\. ScreenE Controller (Business Logic)

Controller này chịu trách nhiệm xử lý logic của màn hình E. Nó tuân thủ **SRP** (chỉ lo việc của màn hình E).

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Provider
final screenEControllerProvider = StateNotifierProvider<ScreenEController, ScreenEEvent>((ref) {
  return ScreenEController();
});

class ScreenEController extends StateNotifier<ScreenEEvent> {
  ScreenEController() : super(InitialEvent());

  // Hàm này được gọi khi nhận được Deep Link
  void onDeepLinkReceived(String data) {
    // 1. Set state để báo UI hiện Popup
    state = ShowDeepLinkPopupEvent(data);
  }

  // Hàm xử lý nghiệp vụ khi người dùng bấm nút trong Popup
  Future<void> processData(String data) async {
    // Thực hiện xử lý (ví dụ gọi API)
    print("Đang xử lý dữ liệu: $data");
    await Future.delayed(const Duration(seconds: 2)); // Giả lập network
    print("Xử lý xong!");
    
    // Reset state sau khi xong (nếu cần)
    state = InitialEvent(); 
  }
}
```

### 3\. Global Deep Link Listener (Coordinator)

Đây là lớp trung gian (Mediator) kết nối `AppLinks` và `ScreenE`. Nó quyết định khi nào thì gọi `ScreenEController`.

```dart
// global_deep_link_listener.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'app_links_repository.dart'; // Giả sử bạn đã có repo từ bài trước
import 'navigation_service.dart';   // Giả sử bạn đã có service từ bài trước

final globalDeepLinkListenerProvider = Provider<void>((ref) {
  final deepLinkRepo = ref.watch(deepLinkRepositoryProvider);
  final navService = ref.watch(navigationServiceProvider);
  
  deepLinkRepo.uriStream.listen((uri) {
    if (uri.path.contains('/e')) {
      final data = uri.queryParameters['data'] ?? '';

      if (navService.isAtScreenE) {
        // CASE 1: Đang ở E -> Gọi Controller của E để kích hoạt Popup
        ref.read(screenEControllerProvider.notifier).onDeepLinkReceived(data);
      } else {
        // CASE 2: Không ở E -> Điều hướng sang E kèm data
        navService.navigateToEWithStackRecursion(data);
      }
    }
  });
});
```

### 4\. Screen E UI (Hiển thị Popup)

Đây là phần quan trọng nhất. Chúng ta sử dụng `ref.listen` để lắng nghe sự thay đổi state và hiển thị `Dialog` (Dialog không thể gọi trong hàm `build` thông thường).

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class ScreenE extends ConsumerStatefulWidget {
  const ScreenE({super.key});

  @override
  ConsumerState<ScreenE> createState() => _ScreenEState();
}

class _ScreenEState extends ConsumerState<ScreenE> {
  @override
  void initState() {
    super.initState();
    // Xử lý trường hợp chuyển trang từ A,B,C,D sang E kèm data (Case 2)
    WidgetsBinding.instance.addPostFrameCallback((_) {
      final args = ModalRoute.of(context)?.settings.arguments;
      if (args is String) {
        // Kích hoạt logic giống như nhận deep link tại chỗ
        ref.read(screenEControllerProvider.notifier).onDeepLinkReceived(args);
      }
    });
  }

  // Hàm hiển thị Popup xử lý
  void _showProcessingDialog(BuildContext context, String data) {
    showDialog(
      context: context,
      barrierDismissible: false, // Bắt buộc người dùng phải thao tác
      builder: (dialogContext) {
        return AlertDialog(
          title: const Text("Nhận được Deep Link"),
          content: Text("Dữ liệu: $data\nBạn có muốn xử lý không?"),
          actions: [
            TextButton(
              onPressed: () => Navigator.of(dialogContext).pop(),
              child: const Text("Hủy"),
            ),
            ElevatedButton(
              onPressed: () async {
                // Đóng dialog confirm
                Navigator.of(dialogContext).pop();
                
                // Gọi Controller xử lý logic
                await ref.read(screenEControllerProvider.notifier).processData(data);
                
                // Có thể hiện thêm 1 dialog thông báo thành công nếu muốn
                if (context.mounted) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text("Đã xử lý thành công!")),
                  );
                }
              },
              child: const Text("Xử lý ngay"),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    // --- LOGIC LẮNG NGHE ĐỂ HIỆN POPUP ---
    ref.listen<ScreenEEvent>(screenEControllerProvider, (previous, next) {
      // Nếu state mới là yêu cầu hiện Popup -> Show Dialog
      if (next is ShowDeepLinkPopupEvent) {
        _showProcessingDialog(context, next.data);
      }
    });
    // --------------------------------------

    return Scaffold(
      appBar: AppBar(title: const Text("Màn hình E")),
      body: const Center(
        child: Text("Nội dung màn hình E"),
      ),
    );
  }
}
```

### Giải thích luồng hoạt động (Ứng dụng SOLID)

1.  **Deep Link đến (`myapp://e?data=XYZ`)**:

      * `GlobalDeepLinkListener` (Lớp điều phối) nhận tín hiệu.
      * Nếu đang ở E: Nó gọi `ScreenEController`.
      * Nếu không ở E: Nó gọi `NavigationService` để chuyển sang E, sau đó `ScreenE.initState` sẽ gọi `ScreenEController`.

2.  **Tại `ScreenEController` (Logic):**

      * Hàm `onDeepLinkReceived` chạy, chuyển state thành `ShowDeepLinkPopupEvent("XYZ")`.
      * Controller không biết UI vẽ gì (Dialog hay Toast), nó chỉ báo hiệu "Cần hiện popup". (**DIP**)

3.  **Tại `ScreenE` (UI):**

      * `ref.listen` phát hiện state thay đổi.
      * Nó gọi hàm `showDialog`.
      * Khi người dùng bấm "Xử lý ngay", UI gọi ngược lại `processData` của Controller để thực hiện logic nghiệp vụ.

### Lợi ích:

  * **Tách biệt:** UI không chứa logic xử lý API. Controller không chứa code UI (Dialog context).
  * **Testable:** Bạn có thể viết unit test cho `ScreenEController` để kiểm tra xem khi gọi `onDeepLinkReceived` thì state có đổi đúng không mà không cần chạy Simulator.
  * **Tái sử dụng:** Logic xử lý `processData` có thể được gọi từ nút bấm khác trên màn hình E chứ không chỉ từ Deep Link.

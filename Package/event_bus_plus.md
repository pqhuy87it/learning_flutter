Chào bạn, tôi sẽ giới thiệu chi tiết về package `event_bus_plus` trong Flutter, một công cụ mạnh mẽ để giao tiếp giữa các thành phần khác nhau trong ứng dụng của bạn một cách tách biệt (decoupled).

### 1. Event Bus là gì?

Hãy tưởng tượng Event Bus như một đài phát thanh trung tâm trong ứng dụng của bạn.

*   **Người phát (Publisher):** Bất kỳ thành phần nào trong ứng dụng (một widget, một service, một BLoC...) muốn thông báo một sự kiện gì đó xảy ra (ví dụ: "Người dùng đã cập nhật hồ sơ!"), nó sẽ "phát sóng" một "sự kiện" (Event) lên đài phát thanh này.
*   **Người nghe (Subscriber):** Các thành phần khác quan tâm đến sự kiện đó (ví dụ: widget hiển thị tên người dùng trên trang chủ, widget avatar ở thanh app bar) sẽ "dò đúng tần số" và "lắng nghe". Khi sự kiện được phát đi, tất cả những người nghe đã đăng ký sẽ nhận được thông báo và có thể hành động tương ứng (ví dụ: cập nhật lại giao diện).

**Lợi ích chính:** Người phát không cần biết người nghe là ai, và người nghe cũng không cần biết người phát là ai. Họ chỉ cần biết về "sự kiện". Điều này giúp các thành phần trong ứng dụng của bạn trở nên độc lập và dễ bảo trì hơn rất nhiều.

`event_bus_plus` là một package hiện thực hóa mô hình này trong Flutter một cách đơn giản, an toàn về kiểu dữ liệu (type-safe) và hiệu quả.

### 2. Tại sao và khi nào nên sử dụng `event_bus_plus`?

Bạn nên cân nhắc sử dụng Event Bus khi:

*   **Giao tiếp giữa các widget không có quan hệ trực tiếp (cha-con):** Ví dụ, một nút bấm ở màn hình Cài đặt cần cập nhật một widget ở màn hình Trang chủ. Thay vì truyền callback qua nhiều lớp widget (gây ra "prop drilling"), bạn chỉ cần bắn một sự kiện.
*   **Tách biệt logic nghiệp vụ khỏi UI:** Một service chạy nền (ví dụ: service tải file) có thể bắn sự kiện về tiến trình (`DownloadProgressEvent`) hoặc khi hoàn thành (`DownloadCompleteEvent`). Các widget UI có thể lắng nghe và cập nhật giao diện mà không cần biết chi tiết về service đó.
*   **Giảm sự phụ thuộc chéo (coupling):** Các module khác nhau trong ứng dụng có thể giao tiếp với nhau qua các sự kiện mà không cần import trực tiếp lẫn nhau.

**Tuy nhiên, không nên lạm dụng:** Đối với giao tiếp cha-con đơn giản, dùng callback vẫn là cách tốt nhất. Đối với quản lý trạng thái toàn cục phức tạp, các giải pháp như Provider, Riverpod, hay BLoC thường phù hợp hơn. Event Bus mạnh nhất cho các "thông báo" hoặc "tác dụng phụ" (side-effects) xuyên suốt ứng dụng.

### 3. Hướng dẫn sử dụng chi tiết

#### Bước 1: Cài đặt

Thêm package vào file `pubspec.yaml` của bạn:

```yaml
dependencies:
  event_bus_plus: ^0.4.1 # Kiểm tra phiên bản mới nhất trên pub.dev
```

Sau đó chạy lệnh: `flutter pub get`

#### Bước 2: Tạo các lớp Event

Mỗi loại sự kiện bạn muốn phát đi nên là một lớp Dart riêng biệt. Điều này giúp `event_bus_plus` đảm bảo an toàn kiểu dữ liệu.

```dart
// event.dart

// Một sự kiện đơn giản không mang dữ liệu
class UserLoggedOutEvent {}

// Một sự kiện mang theo dữ liệu
class UserProfileUpdatedEvent {
  final String newName;
  final String newAvatarUrl;

  UserProfileUpdatedEvent(this.newName, this.newAvatarUrl);
}
```

#### Bước 3: Tạo và Truy cập Event Bus

`event_bus_plus` cung cấp hai cách:

1.  **Sử dụng bus mặc định (Default Event Bus):** Đây là một thực thể singleton, tiện lợi cho việc truy cập từ bất cứ đâu.
    ```dart
    import 'package:event_bus_plus/event_bus_plus.dart';

    final eventBus = EventBus(); // Đây là bus mặc định
    ```
    Bạn có thể đặt biến này trong một file global hoặc inject nó thông qua dependency injection (ví dụ: get_it).

2.  **Tạo một bus tùy chỉnh:** Hữu ích khi bạn muốn có các kênh giao tiếp riêng biệt cho từng module.
    ```dart
    final paymentEventBus = EventBus.custom();
    final userProfileEventBus = EventBus.custom();
    ```

Trong hầu hết các trường hợp, bus mặc định là đủ.

#### Bước 4: Phát (Fire) một sự kiện

Ở nơi bạn muốn thông báo sự kiện, chỉ cần gọi phương thức `fire()`.

```dart
// Trong một hàm onPressed của nút "Lưu thay đổi" chẳng hạn
void saveChanges() {
  // ... logic lưu dữ liệu
  String updatedName = "Nguyễn Văn B";
  String updatedAvatar = "url_to_new_avatar.png";

  // Bắn sự kiện đi
  eventBus.fire(UserProfileUpdatedEvent(updatedName, updatedAvatar));
  print("Đã phát sự kiện UserProfileUpdatedEvent");
}
```

#### Bước 5: Lắng nghe (Listen) một sự kiện

Đây là phần quan trọng nhất. Ở widget hoặc service muốn nhận sự kiện, bạn làm như sau:

1.  Trong `initState()`, sử dụng `eventBus.on<T>()` để lấy về một `Stream` của sự kiện loại `T`.
2.  Sử dụng phương thức `.listen()` trên stream đó để đăng ký một hàm callback sẽ được thực thi mỗi khi có sự kiện.
3.  **Rất quan trọng:** Lưu lại `StreamSubscription` trả về từ `.listen()`.
4.  Trong `dispose()`, gọi `.cancel()` trên `StreamSubscription` đó để hủy đăng ký và tránh rò rỉ bộ nhớ (memory leak).

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:event_bus_plus/event_bus_plus.dart';
// import file chứa định nghĩa event và eventBus

class UserAvatarWidget extends StatefulWidget {
  const UserAvatarWidget({super.key});

  @override
  State<UserAvatarWidget> createState() => _UserAvatarWidgetState();
}

class _UserAvatarWidgetState extends State<UserAvatarWidget> {
  late StreamSubscription<UserProfileUpdatedEvent> _subscription;
  String _avatarUrl = "default_avatar.png"; // Giá trị ban đầu

  @override
  void initState() {
    super.initState();
    // Lắng nghe sự kiện UserProfileUpdatedEvent
    _subscription = eventBus.on<UserProfileUpdatedEvent>().listen((event) {
      // Khi có sự kiện, cập nhật lại UI
      setState(() {
        _avatarUrl = event.newAvatarUrl;
      });
      print("UserAvatarWidget đã nhận được sự kiện và cập nhật avatar!");
    });
  }

  @override
  void dispose() {
    // Hủy đăng ký lắng nghe để tránh memory leak
    _subscription.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      backgroundImage: NetworkImage(_avatarUrl),
    );
  }
}
```

### 4. Ví dụ hoàn chỉnh

Đây là một ứng dụng Flutter đơn giản minh họa toàn bộ luồng.

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:event_bus_plus/event_bus_plus.dart';

// --- Bước 2: Định nghĩa Events và Event Bus ---
class ProfileUpdatedEvent {
  final String newName;
  ProfileUpdatedEvent(this.newName);
}

// --- Bước 3: Tạo Event Bus (đặt global cho tiện) ---
final eventBus = EventBus();

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('EventBus+ Demo'),
          actions: const [
            // Widget lắng nghe sự kiện
            Padding(
              padding: EdgeInsets.only(right: 16.0),
              child: HomeScreenHeader(),
            )
          ],
        ),
        body: const SettingsScreen(), // Widget phát sự kiện
      ),
    );
  }
}

// --- WIDGET LẮNG NGHE (SUBSCRIBER) ---
class HomeScreenHeader extends StatefulWidget {
  const HomeScreenHeader({super.key});

  @override
  State<HomeScreenHeader> createState() => _HomeScreenHeaderState();
}

class _HomeScreenHeaderState extends State<HomeScreenHeader> {
  String _userName = "Người dùng"; // Tên ban đầu
  late StreamSubscription<ProfileUpdatedEvent> _subscription;

  @override
  void initState() {
    super.initState();
    // Bước 5: Lắng nghe sự kiện
    _subscription = eventBus.on<ProfileUpdatedEvent>().listen((event) {
      setState(() {
        _userName = event.newName;
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Header đã nhận được tên mới: ${event.newName}')),
      );
    });
  }

  @override
  void dispose() {
    _subscription.cancel(); // Đừng quên hủy đăng ký!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Center(child: Text('Chào, $_userName!'));
  }
}


// --- WIDGET PHÁT (PUBLISHER) ---
class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final textController = TextEditingController();
    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        children: [
          TextField(
            controller: textController,
            decoration: const InputDecoration(labelText: 'Nhập tên mới'),
          ),
          const SizedBox(height: 20),
          ElevatedButton(
            onPressed: () {
              final newName = textController.text;
              if (newName.isNotEmpty) {
                // Bước 4: Phát sự kiện
                eventBus.fire(ProfileUpdatedEvent(newName));
              }
            },
            child: const Text('Lưu và Phát Sự Kiện'),
          ),
        ],
      ),
    );
  }
}
```

Trong ví dụ này, `SettingsScreen` và `HomeScreenHeader` không hề biết về sự tồn tại của nhau. Chúng giao tiếp hoàn toàn thông qua `eventBus`, giúp code trở nên rất gọn gàng và tách biệt.

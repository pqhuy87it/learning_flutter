Chào bạn,

Chúng ta sẽ cùng tìm hiểu chi tiết về `StreamProvider`, một trong những provider mạnh mẽ và hữu ích nhất của Riverpod, đặc biệt khi làm việc với dữ liệu thay đổi theo thời gian thực.

### `StreamProvider` là gì?

`StreamProvider` là một provider được thiết kế đặc biệt để **lắng nghe một `Stream`** và **cung cấp (expose) giá trị mới nhất** mà stream đó phát ra cho ứng dụng.

Hãy tưởng tượng bạn có một "đường ống" dữ liệu (stream) liên tục chảy, `StreamProvider` sẽ đứng ở cuối đường ống, nhận từng mẩu dữ liệu chảy qua và thông báo cho các widget đang quan tâm để chúng cập nhật giao diện. Nó cũng tự động xử lý việc mở và đóng stream, giúp bạn quản lý tài nguyên hiệu quả.

### Khi nào nên sử dụng `StreamProvider`?

Bạn nên sử dụng `StreamProvider` bất cứ khi nào bạn cần làm việc với nguồn dữ liệu bất đồng bộ và liên tục cập nhật, ví dụ như:

1.  **Firebase Firestore/Realtime Database:** Lắng nghe sự thay đổi của một document hoặc một collection trong thời gian thực. Đây là trường hợp sử dụng phổ biến nhất.
2.  **WebSockets:** Nhận tin nhắn hoặc cập nhật trực tiếp từ server.
3.  **Location Services:** Theo dõi vị trí hiện tại của người dùng.
4.  **Connectivity Status:** Lắng nghe trạng thái kết nối mạng (online/offline).
5.  Bất kỳ API nào trả về một `Stream`.

### Khái niệm quan trọng: `AsyncValue`

Không giống như `Provider` hay `StateProvider` trả về trực tiếp giá trị, `StreamProvider` (và `FutureProvider`) trả về một đối tượng đặc biệt gọi là `AsyncValue`.

`AsyncValue` là một lớp giúp đóng gói trạng thái của một hoạt động bất đồng bộ. Nó có 3 trạng thái chính:

1.  `AsyncData`: Trạng thái thành công, chứa dữ liệu đã nhận được.
2.  `AsyncLoading`: Trạng thái đang chờ, khi stream chưa phát ra giá trị đầu tiên hoặc đang kết nối.
3.  `AsyncError`: Trạng thái lỗi, chứa thông tin về lỗi và stack trace.

Việc sử dụng `AsyncValue` giúp bạn xử lý cả 3 trường hợp (loading, error, data) một cách an toàn và tường minh trong UI.

---

### Cách sử dụng `StreamProvider` chi tiết

#### Bước 1: Tạo một `StreamProvider`

Bạn tạo `StreamProvider` bằng cách trả về một `Stream` từ bên trong nó.

**Ví dụ 1: Một stream số đơn giản**

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// StreamProvider<int>: Chỉ định rằng stream này sẽ phát ra các giá trị kiểu int.
// Hàm bên trong provider phải trả về một Stream<int>.
final numberStreamProvider = StreamProvider<int>((ref) {
  // Trả về một stream phát ra một số nguyên mỗi giây.
  return Stream.periodic(const Duration(seconds: 1), (computationCount) {
    return computationCount;
  });
});
```

**Ví dụ 2: Lắng nghe một document trên Firestore**
(Giả sử bạn đã cài đặt và cấu hình Firebase)

```dart
// final firestoreProvider = Provider((ref) => FirebaseFirestore.instance);

// final userStreamProvider = StreamProvider.autoDispose<DocumentSnapshot>((ref) {
//   final firestore = ref.watch(firestoreProvider);
//   // Lắng nghe sự thay đổi của document 'user123' trong collection 'users'
//   return firestore.collection('users').doc('user123').snapshots();
// });
```

#### Bước 2: Cung cấp Provider

Như thường lệ, hãy đảm bảo ứng dụng của bạn được bọc trong `ProviderScope`.

```dart
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

#### Bước 3: Lắng nghe và Hiển thị dữ liệu trong Widget

Đây là phần quan trọng nhất. Trong widget, bạn sẽ dùng `ref.watch` để lắng nghe `StreamProvider`. Kết quả trả về sẽ là một `AsyncValue`. Cách tốt nhất để xử lý `AsyncValue` là sử dụng phương thức `.when()`.

Phương thức `.when()` buộc bạn phải cung cấp các widget builder cho cả 3 trạng thái: `data`, `loading`, và `error`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class NumberPage extends ConsumerWidget {
  const NumberPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 1. Watch the provider. `numberValue` sẽ là một AsyncValue<int>.
    final AsyncValue<int> numberValue = ref.watch(numberStreamProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('StreamProvider Example')),
      body: Center(
        // 2. Sử dụng .when() để xây dựng UI dựa trên trạng thái của stream.
        child: numberValue.when(
          // Trạng thái thành công: hiển thị dữ liệu
          data: (number) => Text(
            'Số mới nhất: $number',
            style: Theme.of(context).textTheme.headlineMedium,
          ),
          
          // Trạng thái lỗi: hiển thị thông báo lỗi
          error: (error, stackTrace) => Text(
            'Đã xảy ra lỗi: $error',
            style: const TextStyle(color: Colors.red),
          ),

          // Trạng thái đang tải: hiển thị vòng quay loading
          loading: () => const CircularProgressIndicator(),
        ),
      ),
    );
  }
}
```

### Các modifier hữu ích

#### 1. `.autoDispose`

Rất quan trọng khi làm việc với stream. Khi bạn thêm `.autoDispose`, Riverpod sẽ tự động **hủy đăng ký (cancel subscription)** và đóng stream khi không còn widget nào lắng nghe nó. Điều này giúp ngăn ngừa rò rỉ bộ nhớ (memory leaks) và các hoạt động không cần thiết ở background.

```dart
final numberStreamProvider = StreamProvider.autoDispose<int>((ref) {
  // Bạn có thể thực hiện hành động dọn dẹp khi provider bị hủy
  ref.onDispose(() {
    print('Stream đã bị đóng và dọn dẹp!');
  });
  
  return Stream.periodic(const Duration(seconds: 1), (i) => i);
});
```
**Luôn cân nhắc sử dụng `.autoDispose` với `StreamProvider` trừ khi bạn cần stream đó hoạt động toàn cục trong suốt vòng đời ứng dụng.**

#### 2. `.family`

Sử dụng `.family` khi bạn cần tạo một stream phụ thuộc vào một tham số từ bên ngoài. Ví dụ, lắng nghe một document chat cụ thể dựa trên `chatId`.

```dart
// import 'package:cloud_firestore/cloud_firestore.dart';

// final firestore = FirebaseFirestore.instance;

// Tham số thứ hai (String) là kiểu dữ liệu của tham số đầu vào (chatId)
// final chatProvider = StreamProvider.family
//   .autoDispose<QuerySnapshot, String>((ref, String chatId) {
//     return firestore
//         .collection('chats')
//         .doc(chatId)
//         .collection('messages')
//         .orderBy('timestamp')
//         .snapshots();
// });

// Cách sử dụng trong widget
class ChatScreen extends ConsumerWidget {
  final String chatId;
  const ChatScreen({required this.chatId, Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Truyền chatId vào khi watch provider
    final AsyncValue<QuerySnapshot> chatMessages = ref.watch(chatProvider(chatId));

    return chatMessages.when(
      data: (snapshot) {
        // Xây dựng danh sách tin nhắn từ snapshot.docs
        return ListView.builder(...);
      },
      error: (e, st) => const Text('Lỗi tải tin nhắn'),
      loading: () => const Center(child: CircularProgressIndicator()),
    );
  }
}
```

### Kết hợp `StreamProvider` với các Provider khác

Sức mạnh của Riverpod thể hiện khi bạn kết hợp các provider với nhau. Một `StreamProvider` có thể đọc giá trị từ một provider khác để quyết định nên lắng nghe stream nào.

Ví dụ: Lắng nghe thông tin của người dùng đang đăng nhập.

```dart
// Provider chứa ID của người dùng đang đăng nhập (có thể là từ Firebase Auth)
final userIdProvider = StateProvider<String?>((ref) => 'user123');

// StreamProvider này đọc userIdProvider để biết cần lắng nghe document nào
final userProfileProvider = StreamProvider.autoDispose((ref) {
  final userId = ref.watch(userIdProvider);

  // Nếu chưa có user đăng nhập, không làm gì cả
  if (userId == null) {
    return Stream.value(null); // Hoặc ném ra lỗi
  }

  // Lắng nghe document của user đó
  return firestore.collection('users').doc(userId).snapshots();
});
```
Khi `userIdProvider` thay đổi (ví dụ: người dùng đăng xuất và đăng nhập lại bằng tài khoản khác), `userProfileProvider` sẽ tự động bị hủy và tạo lại, lắng nghe vào stream của người dùng mới.

### Tổng kết

*   `StreamProvider` dùng để **lắng nghe `Stream`** và cung cấp dữ liệu theo thời gian thực.
*   Nó trả về một đối tượng `AsyncValue` để quản lý các trạng thái **loading, data, và error**.
*   Sử dụng `ref.watch(myStreamProvider).when(...)` trong UI để xử lý cả ba trạng thái một cách an toàn.
*   Sử dụng `.autoDispose` để tự động đóng stream và tránh rò rỉ bộ nhớ.
*   Sử dụng `.family` để tạo các stream có tham số.
*   Đây là công cụ lý tưởng khi làm việc với các dịch vụ thời gian thực như Firebase hay WebSockets.

Chào bạn,

Tiếp nối giải thích về `Provider`, chúng ta sẽ đi sâu vào `StateProvider`. Đây là một trong những provider được sử dụng thường xuyên nhất trong Riverpod, đặc biệt là cho các trạng thái đơn giản.

### `StateProvider` là gì?

`StateProvider` là một provider được thiết kế để quản lý các **trạng thái đơn giản và có thể thay đổi (mutable state)**.

Hãy nghĩ về nó như một "hộp chứa" một giá trị duy nhất (ví dụ: một số, một chuỗi, một boolean) và cung cấp các phương thức để **thay đổi giá trị đó**, đồng thời tự động **thông báo cho các widget đang lắng nghe để chúng build lại (rebuild)**.

Nó là phiên bản đơn giản hơn của `StateNotifierProvider`, phù hợp cho các trạng thái không có logic phức tạp.

### Khi nào nên sử dụng `StateProvider`?

Bạn nên sử dụng `StateProvider` để quản lý các trạng thái đơn giản như:

1.  **Trạng thái của các thành phần UI:**
    *   Giá trị của một `DropdownButton` (ví dụ: `filterProvider`).
    *   Trạng thái của một `Switch` hoặc `Checkbox` (bật/tắt).
    *   Trang hiện tại trong một `PageView` hoặc `BottomNavigationBar`.
2.  **Dữ liệu nhập từ người dùng trong các form đơn giản:**
    *   Giá trị của một `TextField`.
3.  **Các biến trạng thái đơn giản:**
    *   Một bộ đếm (counter).
    *   Một cờ boolean như `isLoading`.

**Khi nào KHÔNG nên sử dụng `StateProvider`?**
Khi trạng thái của bạn có logic phức tạp đi kèm (ví dụ: cần validation, gọi API để cập nhật trạng thái, xử lý logic nghiệp vụ). Trong những trường hợp đó, hãy sử dụng **`StateNotifierProvider`** hoặc **`NotifierProvider`** (trong Riverpod 2.0+).

---

### Cách sử dụng `StateProvider` chi tiết

Chúng ta sẽ đi qua các bước: **Tạo**, **Cung cấp**, **Đọc/Lắng nghe**, và quan trọng nhất là **Cập nhật** trạng thái.

#### Bước 1: Tạo một `StateProvider`

Giống như `Provider`, `StateProvider` cũng được khai báo dưới dạng một biến `final` toàn cục.

**Cú pháp:**

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// StateProvider<int>: Chỉ định rằng provider này quản lý một trạng thái kiểu int.
// (ref) => 0: Hàm này trả về giá trị ban đầu (initial state) của provider.
// Ở đây, giá trị ban đầu là 0.
final counterProvider = StateProvider<int>((ref) {
  return 0;
});

// Một ví dụ khác với kiểu boolean
final isDarkModeProvider = StateProvider<bool>((ref) => false);
```

#### Bước 2: Cung cấp Provider cho ứng dụng

Bước này hoàn toàn giống với `Provider`. Bạn cần bọc widget gốc của ứng dụng bằng `ProviderScope`.

```dart
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

#### Bước 3: Đọc và Lắng nghe (Watch) State trong Widget

Để đọc và lắng nghe trạng thái, bạn cần chuyển widget của mình thành `ConsumerWidget` (hoặc `ConsumerStatefulWidget`).

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class CounterPage extends ConsumerWidget {
  const CounterPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Sử dụng ref.watch() để lấy giá trị hiện tại của counterProvider.
    // Bất cứ khi nào giá trị này thay đổi, widget này sẽ tự động rebuild.
    final int count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('StateProvider Counter')),
      body: Center(
        child: Text(
          'Giá trị: $count',
          style: Theme.of(context).textTheme.headlineMedium,
        ),
      ),
      // ... Các nút để thay đổi giá trị sẽ được thêm ở bước 4
    );
  }
}
```
Trong ví dụ trên, `ref.watch(counterProvider)` sẽ trả về giá trị `int` hiện tại. Widget `CounterPage` sẽ lắng nghe `counterProvider` và tự động build lại mỗi khi giá trị của nó thay đổi.

#### Bước 4: Cập nhật State

Đây là phần khác biệt cốt lõi so với `Provider`. Để cập nhật trạng thái của một `StateProvider`, bạn cần truy cập vào "notifier" của nó. Notifier ở đây là một đối tượng `StateController`.

Có hai cách chính để cập nhật:

**1. Gán trực tiếp giá trị mới vào `.state`**

```dart
// Lấy notifier bằng ref.read(provider.notifier)
ref.read(counterProvider.notifier).state = 10;
```

**2. Sử dụng phương thức `.update()`**

Phương thức này nhận một callback, cung cấp cho bạn trạng thái cũ (`oldState`) và bạn trả về trạng thái mới. Đây là cách được khuyến khích vì nó an toàn hơn, đặc biệt khi trạng thái mới phụ thuộc vào trạng thái cũ.

```dart
// Tăng giá trị hiện tại lên 1
ref.read(counterProvider.notifier).update((oldState) => oldState + 1);
```

**Quan trọng:** Luôn sử dụng `ref.read(provider.notifier)` bên trong các hàm callback (`onPressed`, `onTap`, v.v.) để thay đổi trạng thái. Đừng dùng `ref.watch` ở đây.

---

### Ví dụ hoàn chỉnh: Một ứng dụng Counter

Hãy kết hợp tất cả các bước trên vào một ví dụ hoàn chỉnh.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Bước 1: Tạo provider
final counterProvider = StateProvider<int>((ref) => 0);

void main() {
  // Bước 2: Cung cấp provider
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: CounterPage());
  }
}

class CounterPage extends ConsumerWidget {
  const CounterPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Bước 3: Lắng nghe trạng thái để hiển thị trên UI
    final int count = ref.watch(counterProvider);

    // Lắng nghe provider để có thể reset về trạng thái ban đầu
    ref.listen<int>(counterProvider, (previous, next) {
      // Ví dụ: hiển thị SnackBar khi giá trị đạt đến 5
      if (next == 5) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Đã đạt đến 5!')),
        );
      }
    });

    return Scaffold(
      appBar: AppBar(
        title: const Text('StateProvider Counter'),
        actions: [
          // Nút để reset counter
          IconButton(
            onPressed: () {
              // Cách 1: Gán lại giá trị ban đầu
              // ref.invalidate(counterProvider) cũng có tác dụng tương tự.
              ref.read(counterProvider.notifier).state = 0;
            },
            icon: const Icon(Icons.refresh),
          ),
        ],
      ),
      body: Center(
        child: Text(
          'Giá trị: $count',
          style: Theme.of(context).textTheme.headlineMedium,
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: () {
              // Bước 4: Cập nhật trạng thái
              // Cách 2: Sử dụng `update` để tính toán giá trị mới dựa trên giá trị cũ.
              ref.read(counterProvider.notifier).update((state) => state + 1);
            },
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 10),
          FloatingActionButton(
            onPressed: () {
              ref.read(counterProvider.notifier).update((state) => state - 1);
            },
            child: const Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

### Các modifier hữu ích

Tương tự `Provider`, `StateProvider` cũng có các modifier `.autoDispose` và `.family`.

#### `.autoDispose`

Khi sử dụng `.autoDispose`, trạng thái của provider sẽ tự động được reset về giá trị ban đầu khi không còn widget nào lắng nghe nó. Điều này rất hữu ích cho trạng thái của các form hoặc các trang tạm thời.

```dart
final usernameProvider = StateProvider.autoDispose<String>((ref) => '');
```
Khi người dùng rời khỏi trang nhập username, giá trị của `usernameProvider` sẽ tự động reset về chuỗi rỗng `''`.

#### `.family`

Dùng để tạo ra các provider có trạng thái độc lập dựa trên một tham số. Ví dụ: quản lý trạng thái "đã chọn" cho từng mục trong một danh sách.

```dart
// Quản lý trạng thái chọn (true/false) cho mỗi item dựa vào itemId (String)
final itemSelectedProvider = StateProvider.family<bool, String>((ref, itemId) {
  return false; // Giá trị ban đầu cho mỗi item là false (chưa được chọn)
});

// Sử dụng trong widget
class ListItem extends ConsumerWidget {
  final String itemId;
  const ListItem({required this.itemId, Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Lắng nghe trạng thái của item cụ thể này
    final isSelected = ref.watch(itemSelectedProvider(itemId));

    return ListTile(
      title: Text('Item $itemId'),
      trailing: Checkbox(
        value: isSelected,
        onChanged: (value) {
          // Cập nhật trạng thái của item cụ thể này
          ref.read(itemSelectedProvider(itemId).notifier).state = value ?? false;
        },
      ),
    );
  }
}
```

### Tóm tắt

*   `StateProvider` dùng để quản lý **trạng thái đơn giản, có thể thay đổi**.
*   Sử dụng `ref.watch(myStateProvider)` để **đọc giá trị** và làm cho UI rebuild khi nó thay đổi.
*   Sử dụng `ref.read(myStateProvider.notifier)` để **lấy controller** và cập nhật giá trị bên trong các hàm callback.
*   Dùng `.state = newValue` hoặc `.update((old) => new)` để thay đổi giá trị.
*   Đối với logic phức tạp, hãy cân nhắc sử dụng `StateNotifierProvider`.

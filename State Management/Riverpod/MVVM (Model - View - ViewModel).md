Chào bạn, sử dụng mô hình **MVVM (Model - View - ViewModel)** kết hợp với **Riverpod** là chuẩn mực ("Industry Standard") hiện nay để xây dựng ứng dụng Flutter dễ bảo trì, dễ test và mở rộng.

Trong Riverpod, kiến trúc MVVM được ánh xạ rất tự nhiên. Dưới đây là giải thích chi tiết từng phần và cách chúng kết nối với nhau.

---

### 1. Tổng quan kiến trúc (Mapping)

Mô hình này chia ứng dụng thành 3 lớp rõ ràng:

1. **Model (M):** Dữ liệu và cấu trúc dữ liệu (Data Class, Entity).
2. **View (V):** Giao diện người dùng (Widgets). Chỉ làm nhiệm vụ hiển thị và bắt sự kiện.
3. **ViewModel (VM):** Cầu nối. Trong Riverpod, đây chính là **Notifier** (hoặc `AsyncNotifier`). Nó giữ trạng thái (State) và chứa logic nghiệp vụ.

---

### 2. Triển khai chi tiết từng bước

Chúng ta sẽ xây dựng ví dụ: **Danh sách công việc (Todo List)**.

#### Bước 1: Model (Dữ liệu)

Đây là lớp đơn giản nhất, chỉ định nghĩa dữ liệu trông như thế nào. Nên dùng thư viện `freezed` để đảm bảo tính bất biến (immutability), nhưng ở đây mình viết class thường cho dễ hiểu.

```dart
// todo_model.dart
class Todo {
  final String id;
  final String title;
  final bool isCompleted;

  Todo({required this.id, required this.title, this.isCompleted = false});

  // Cần hàm copyWith để cập nhật trạng thái bất biến
  Todo copyWith({String? id, String? title, bool? isCompleted}) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}

```

#### Bước 2: ViewModel (Logic & State) - Quan trọng nhất

Trong Riverpod 2.0+, ViewModel được triển khai bằng cách kế thừa `Notifier` (cho state đồng bộ) hoặc `AsyncNotifier` (cho state bất đồng bộ/API).

**Nhiệm vụ của ViewModel:**

1. Khởi tạo state ban đầu trong hàm `build()`.
2. Cung cấp các hàm để thay đổi state (Business Logic).

```dart
// todo_view_model.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'todo_model.dart';

// 1. Định nghĩa ViewModel
class TodoNotifier extends Notifier<List<Todo>> {
  
  // Hàm build: Khởi tạo giá trị ban đầu cho State
  @override
  List<Todo> build() {
    return []; // Ban đầu danh sách rỗng
  }

  // Logic nghiệp vụ: Thêm Todo
  void addTodo(String title) {
    final newTodo = Todo(
      id: DateTime.now().toString(), 
      title: title
    );
    
    // Cập nhật State: Luôn tạo ra list mới, không sửa list cũ
    state = [...state, newTodo];
  }

  // Logic nghiệp vụ: Toggle trạng thái
  void toggleTodo(String id) {
    state = [
      for (final todo in state)
        if (todo.id == id)
          todo.copyWith(isCompleted: !todo.isCompleted)
        else
          todo
    ];
  }
}

// 2. Tạo Provider để View có thể truy cập ViewModel này
final todoProvider = NotifierProvider<TodoNotifier, List<Todo>>(() {
  return TodoNotifier();
});

```

#### Bước 3: View (Giao diện)

View lắng nghe thay đổi từ ViewModel để vẽ lại UI.

* Dùng `ref.watch(provider)` để lấy dữ liệu (State) vẽ UI.
* Dùng `ref.read(provider.notifier)` để gọi hàm logic (Action).

```dart
// todo_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'todo_view_model.dart';

// Phải dùng ConsumerWidget để có 'ref'
class TodoScreen extends ConsumerWidget {
  const TodoScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 1. LẮNG NGHE STATE (Dữ liệu từ ViewModel)
    // Khi state thay đổi, widget này sẽ rebuild
    final todos = ref.watch(todoProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('MVVM Todo List')),
      body: ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) {
          final todo = todos[index];
          return CheckboxListTile(
            title: Text(todo.title),
            value: todo.isCompleted,
            onChanged: (value) {
              // 2. GỌI HÀM LOGIC (Tương tác ngược lại ViewModel)
              // Dùng read(.notifier) để gọi hàm, không dùng watch
              ref.read(todoProvider.notifier).toggleTodo(todo.id);
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Gọi hàm logic thêm mới
          ref.read(todoProvider.notifier).addTodo('New Task');
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}

```

---

### 3. Quy tắc "Vàng" khi làm MVVM với Riverpod

Để không phá vỡ kiến trúc, bạn cần tuân thủ 3 quy tắc sau:

#### Quy tắc 1: View không được chứa Logic nghiệp vụ

* **Sai:** Viết `if (todo.isCompleted) { todo.isCompleted = false }` ngay trong hàm `onTap` của Widget.
* **Đúng:** Widget chỉ gọi `ref.read(provider.notifier).toggleTodo()`. Việc xử lý đúng sai, lưu DB là việc của ViewModel.

#### Quy tắc 2: ViewModel không được chứa UI Code

* **Sai:** Trong class `TodoNotifier` có dòng `BuildContext context` hoặc `ScaffoldMessenger.of(context).showSnackBar(...)`.
* **Đúng:** ViewModel chỉ thay đổi State. View lắng nghe State thay đổi để tự hiện thông báo hoặc chuyển trang (sử dụng `ref.listen`).

#### Quy tắc 3: State phải Bất biến (Immutable)

* **Sai:** `state.add(newItem)` (Sửa trực tiếp list cũ).
* **Đúng:** `state = [...state, newItem]` (Tạo list mới và gán lại). Riverpod chỉ biết state thay đổi khi object state được gán mới hoàn toàn.

---

### 4. Nâng cao: Sử dụng Riverpod Generator (Khuyên dùng)

Code ở bước 2 hơi dài dòng. Hiện nay Riverpod khuyến khích dùng code generation để viết MVVM gọn hơn rất nhiều.

**Cách viết ViewModel kiểu mới (dùng `@riverpod`):**

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'todo_model.dart';

part 'todo_view_model.g.dart'; // File sinh ra tự động

@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() {
    return [];
  }

  void addTodo(String title) {
    final newTodo = Todo(id: DateTime.now().toString(), title: title);
    state = [...state, newTodo];
  }
}

```

*Lưu ý: Bạn cần chạy lệnh `dart run build_runner watch` để tạo file `.g.dart`.*

### Tóm tắt luồng đi (Data Flow)

1. **User** bấm nút cộng trên **View**.
2. **View** gọi `ref.read(todoProvider.notifier).addTodo()`.
3. **ViewModel** xử lý logic, tạo list mới và gán `state = newList`.
4. **Riverpod** phát hiện `state` thay đổi, thông báo cho các **View** đang `watch`.
5. **View** tự động rebuild để hiện danh sách mới.

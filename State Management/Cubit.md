Chào bạn, tất nhiên rồi! Cubit là một phần của thư viện `flutter_bloc` và là một cách tiếp cận đơn giản hơn để quản lý trạng thái. Nếu bạn đã hiểu về BLoC, thì việc học Cubit sẽ rất nhanh chóng.

Hãy tưởng tượng **Cubit là một phiên bản rút gọn và thân thiện hơn của BLoC**.

### Cubit là gì?

Cubit là một lớp đơn giản quản lý một trạng thái (state). Thay vì nhận các sự kiện (Events) và phát ra các trạng thái (States) như BLoC, Cubit chỉ cần bạn **gọi các hàm (functions)** của nó, và nó sẽ **phát ra (emit)** các trạng thái mới.

Luồng hoạt động của Cubit đơn giản hơn rất nhiều: **UI → Call Function → Cubit → emit(newState) → UI**

### So sánh nhanh: Cubit và BLoC

| Tiêu chí | Cubit | BLoC |
| :--- | :--- | :--- |
| **Đầu vào** | Gọi hàm trực tiếp (e.g., `cubit.increment()`) | Gửi các lớp Sự kiện (e.g., `bloc.add(IncrementEvent())`) |
| **Độ phức tạp** | Đơn giản, ít code hơn | Phức tạp hơn, cần định nghĩa các lớp Event |
| **Traceability** | Tốt (biết được state thay đổi khi nào) | Rất tốt (biết được state thay đổi **do Event nào gây ra**) |
| **Trường hợp dùng** | Lý tưởng cho logic đơn giản đến trung bình | Mạnh mẽ cho các state machine phức tạp, nhiều luồng sự kiện |

### Các thành phần cốt lõi của Cubit

1.  **State:** Tương tự như trong BLoC, đây là đối tượng chứa dữ liệu mà UI cần để hiển thị.
2.  **Cubit:** Lớp chứa logic. Nó có:
    *   Một trạng thái ban đầu (initial state).
    *   Các hàm công khai (public functions) mà UI có thể gọi.
    *   Hàm `emit()` để phát ra một trạng thái mới.

---

### Ví dụ thực tế: Ứng dụng Đếm số bằng Cubit

Chúng ta sẽ làm lại ví dụ đếm số để bạn thấy rõ sự khác biệt và đơn giản của Cubit.

#### Bước 1: Tạo Cubit

Tạo một file `counter_cubit.dart`. Đây là nơi chứa cả logic và định nghĩa state (vì state ở đây chỉ là một số nguyên `int`, chúng ta không cần tạo file riêng).

```dart
// counter_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';

// 1. Cubit sẽ quản lý một trạng thái kiểu 'int'
class CounterCubit extends Cubit<int> {
  // 2. Khởi tạo trạng thái ban đầu là 0
  CounterCubit() : super(0);

  // 3. Tạo một hàm public để UI có thể gọi
  void increment() {
    // 4. Dùng emit() để phát ra state mới.
    // 'state' là giá trị hiện tại của Cubit.
    emit(state + 1);
  }

  // 5. Một hàm public khác
  void decrement() {
    emit(state - 1);
  }
}
```

**Bạn thấy không?** Không cần định nghĩa các lớp `Event` hay `State` riêng biệt. Mọi thứ gọn gàng trong một file.

#### Bước 2: Kết nối với UI

Phần giao diện gần như giống hệt khi dùng BLoC. Các widget như `BlocProvider`, `BlocBuilder`, `BlocListener` đều hoạt động với Cubit.

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_cubit.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Cung cấp CounterCubit cho các widget con
    // Widget này hoạt động cho cả BLoC và Cubit
    return BlocProvider(
      create: (context) => CounterCubit(),
      child: const MaterialApp(
        home: CounterPage(),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cubit Counter')),
      body: Center(
        // 2. Dùng BlocBuilder để lắng nghe và rebuild UI
        // Widget này cũng hoạt động cho cả BLoC và Cubit
        child: BlocBuilder<CounterCubit, int>(
          builder: (context, count) {
            // 'count' chính là state (kiểu int) hiện tại của Cubit
            return Text(
              '$count',
              style: Theme.of(context).textTheme.headlineMedium,
            );
          },
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: () {
              // 3. SỰ KHÁC BIỆT LỚN NHẤT LÀ ĐÂY!
              // Thay vì .add(Event()), ta gọi trực tiếp hàm của Cubit
              context.read<CounterCubit>().increment();
            },
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            onPressed: () {
              context.read<CounterCubit>().decrement();
            },
            child: const Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

### Khi nào nên dùng Cubit?

1.  **Khi logic đơn giản:** Nếu các hành động của bạn chỉ là những thay đổi trạng thái trực tiếp (ví dụ: lấy danh sách sản phẩm, bật/tắt một cài đặt, tăng/giảm một giá trị), Cubit là lựa chọn hoàn hảo.
2.  **Khi bạn mới bắt đầu:** Cubit là một điểm khởi đầu tuyệt vời để làm quen với thư viện `flutter_bloc` vì nó rất dễ hiểu và ít code thừa (boilerplate).
3.  **Khi muốn phát triển nhanh:** Việc không phải tạo các file Event riêng biệt giúp tăng tốc độ code.

### Khi nào nên cân nhắc dùng BLoC?

Khi ứng dụng của bạn có những luồng logic phức tạp, nơi một sự kiện có thể dẫn đến nhiều trạng thái khác nhau tùy thuộc vào điều kiện. Việc có các lớp `Event` rõ ràng giúp bạn dễ dàng theo dõi và gỡ lỗi hơn.

### Tóm tắt

*   **Cubit** là một phiên bản đơn giản của BLoC.
*   Nó sử dụng **hàm (function)** thay vì **sự kiện (event)** để kích hoạt thay đổi trạng thái.
*   Nó rất dễ học, dễ viết và hoàn hảo cho phần lớn các trường hợp quản lý trạng thái.
*   Bạn có thể sử dụng tất cả các widget của `flutter_bloc` (`BlocProvider`, `BlocBuilder`, etc.) với cả Cubit và BLoC.

Đối với nhiều nhà phát triển, quy trình làm việc phổ biến là **bắt đầu với Cubit**. Nếu logic trở nên quá phức tạp, họ sẽ tái cấu trúc (refactor) nó thành BLoC.

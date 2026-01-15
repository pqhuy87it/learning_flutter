Chào bạn, đây là câu hỏi cơ bản và quan trọng nhất khi bắt đầu với Riverpod. Sự khác biệt cốt lõi giữa `ref.watch` và `ref.read` nằm ở **"Tính phản ứng" (Reactivity)**.

Dưới đây là bảng so sánh nhanh trước khi đi vào chi tiết:

| Đặc điểm | `ref.watch` (Theo dõi) | `ref.read` (Đọc) |
| --- | --- | --- |
| **Hành vi** | Lấy giá trị **VÀ** đăng ký lắng nghe thay đổi. | Lấy giá trị hiện tại **một lần duy nhất**. |
| **Kết quả** | Khi dữ liệu đổi -> **Vẽ lại (Rebuild)** Widget/Provider. | Khi dữ liệu đổi -> **Không làm gì cả**. |
| **Vị trí dùng** | Trong hàm `build()` hoặc trong body của Provider khác. | Trong các sự kiện: `onTap`, `onPressed`, `initState`. |
| **Mục đích** | Để **hiển thị** dữ liệu lên UI. | Để **thực hiện hành động** (gọi hàm logic). |

---

### 1. `ref.watch` (Dùng để hiển thị)

Hãy tưởng tượng `ref.watch` giống như bạn **đăng ký kênh Youtube**. Khi kênh ra video mới (State thay đổi), bạn sẽ nhận được thông báo ngay lập tức.

* **Cơ chế:** Nó nói với Riverpod rằng: *"Widget này phụ thuộc vào dữ liệu kia. Nếu dữ liệu kia đổi, hãy chạy lại hàm `build` của Widget này để cập nhật giao diện."*
* **Nơi dùng:** Bắt buộc dùng bên trong hàm `build`.

**Ví dụ:**

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // Dùng watch vì ta muốn UI cập nhật số mới khi counter thay đổi
  final count = ref.watch(counterProvider); 

  return Text('$count'); 
}

```

---

### 2. `ref.read` (Dùng để hành động)

Hãy tưởng tượng `ref.read` giống như bạn **mua một tờ báo**. Bạn đọc tin tức tại thời điểm đó. Nếu ngày mai có tin mới, tờ báo cũ trên tay bạn không tự biến đổi nội dung được.

* **Cơ chế:** Nó lấy giá trị hoặc class xử lý (Notifier) tại thời điểm gọi hàm, và sau đó "quên luôn", không quan tâm đến các thay đổi sau này.
* **Nơi dùng:** Dùng trong các hàm xử lý sự kiện (`onPressed`, `onTap`) hoặc vòng đời (`initState`).
* **Lưu ý quan trọng:** Thường dùng kết hợp với `.notifier` để gọi các hàm logic.

**Ví dụ:**

```dart
FloatingActionButton(
  onPressed: () {
    // Dùng read vì ta chỉ cần gọi hàm increment 1 lần khi bấm nút.
    // Ta KHÔNG muốn hàm build chạy lại chỉ vì ta bấm nút.
    ref.read(counterProvider.notifier).increment();
  },
  child: Icon(Icons.add),
)

```

---

### 3. Tại sao không dùng lộn xộn được?

#### A. Tại sao không dùng `ref.read` trong hàm `build`?

```dart
// SAI LẦM PHỔ BIẾN
Widget build(context, ref) {
  final count = ref.read(counterProvider); // SAI
  return Text('$count');
}

```

**Hậu quả:** Khi `count` tăng từ 0 lên 1, Widget này **không biết gì cả** và vẫn hiển thị số 0 mãi mãi. UI bị "đơ".

#### B. Tại sao không dùng `ref.watch` trong `onPressed`?

```dart
// SAI LẦM PHỔ BIẾN
onPressed: () {
  final count = ref.watch(counterProvider); // SAI (Thường sẽ báo lỗi hoặc cảnh báo)
  print(count);
}

```

**Hậu quả:** `onPressed` là một hành động tức thời, nó không phải là một thành phần giao diện cần "lắng nghe" thay đổi. Việc lắng nghe ở đây là vô nghĩa và gây rò rỉ hiệu năng.

---

### 4. Tổng kết bằng Code hoàn chỉnh

```dart
class CounterApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 1. Dùng WATCH để lấy dữ liệu hiển thị (UI tự cập nhật)
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(
        child: Text('$count'), // Hiển thị số
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 2. Dùng READ để thực hiện hành động (Logic)
          // Lưu ý: Thường dùng read với .notifier để gọi hàm
          ref.read(counterProvider.notifier).increment();
        },
        child: Icon(Icons.add),
      ),
    );
  }
}

```

**Mẹo nhớ nhanh:**

* Muốn **thấy** sự thay đổi trên màn hình -> `watch`.
* Muốn **tác động** (bấm nút, gọi API) -> `read`.

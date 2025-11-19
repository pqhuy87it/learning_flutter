Chào bạn\! Trong thư viện `event_bus_plus`, getter `eventBus.inProgress$` đóng một vai trò rất thú vị và hữu ích, giúp bạn quản lý trạng thái của ứng dụng dựa trên các sự kiện (events).

Dưới đây là giải thích chi tiết về mục đích và cách dùng của nó:

### 1\. Định nghĩa và Ý nghĩa

  * **Định nghĩa:** `Stream<List<AppEvent>> get inProgress$` trả về một dòng dữ liệu (Stream) chứa danh sách các `AppEvent`.
  * **Ý nghĩa:** Danh sách này chứa **các sự kiện đang được xử lý (pending/processing)** nhưng chưa hoàn thành.

Khi bạn bắn một event (fire event) và có một listener đang xử lý event đó (ví dụ: gọi API, xử lý tính toán nặng), event đó sẽ nằm trong danh sách `inProgress`. Khi xử lý xong, event đó sẽ được xóa khỏi danh sách này.

### 2\. Mục đích sử dụng chính

`inProgress$` chủ yếu được dùng để **Quản lý UI Loading (Trạng thái chờ)** một cách tự động và tách biệt (decoupled).

Thay vì bạn phải thủ công set biến `isLoading = true` khi bắt đầu gọi hàm và `isLoading = false` khi kết thúc ở từng màn hình, bạn có thể lắng nghe `inProgress$`.

  * **Nếu danh sách rỗng (`[]`):** Nghĩa là không có tác vụ nào đang chạy -\> Ẩn loading.
  * **Nếu danh sách có phần tử:** Nghĩa là có ít nhất một tác vụ đang chạy -\> Hiển thị loading.

### 3\. Ví dụ thực tế

Giả sử bạn có một nút "Đăng nhập". Khi nhấn, bạn bắn ra `LoginEvent`.

**Cách hoạt động:**

1.  Người dùng nhấn nút -\> Bạn gọi `eventBus.fire(LoginEvent(...))`.
2.  Ngay lập tức, `LoginEvent` được thêm vào `inProgress$`.
3.  UI lắng nghe `inProgress$` thấy có event -\> **Tự động hiện vòng xoay Loading**.
4.  Ở một nơi khác (Service/Bloc), code xử lý `LoginEvent` (gọi API đăng nhập).
5.  Khi API trả về kết quả, việc xử lý hoàn tất. `LoginEvent` bị loại bỏ khỏi `inProgress$`.
6.  UI thấy danh sách rỗng -\> **Tự động tắt Loading**.

**Code minh họa với StreamBuilder:**

```dart
StreamBuilder<List<AppEvent>>(
  stream: eventBus.inProgress$,
  builder: (context, snapshot) {
    // Lấy danh sách các event đang chạy
    final events = snapshot.data ?? [];

    // Kiểm tra xem có event nào đang chạy không
    if (events.isNotEmpty) {
      // Bạn thậm chí có thể check cụ thể loại event:
      // if (events.any((e) => e is LoginEvent)) { ... }
      
      return const CircularProgressIndicator(); // Hiện loading
    }

    // Không có gì chạy thì hiện nút bấm
    return ElevatedButton(
      onPressed: () {
        eventBus.fire(LoginEvent());
      },
      child: const Text("Đăng nhập"),
    );
  },
);
```

### 4\. Tại sao lại dùng cách này?

  * **Decoupling (Tách biệt):** Widget hiển thị loading không cần biết *ai* đang xử lý logic hay xử lý trong bao lâu. Nó chỉ cần biết "Event Bus bảo chưa xong".
  * **Global Loading:** Bạn có thể đặt một cái Loading Overlay trùm lên toàn bộ ứng dụng và lắng nghe `inProgress$`. Bất kỳ module nào bắn event, loading toàn cục sẽ hiện ra mà không cần truyền biến `isLoading` qua lại giữa các màn hình.

### Tóm lại

`eventBus.inProgress$` là công cụ để bạn **giám sát sức khỏe và trạng thái hoạt động** của các sự kiện trong app, từ đó hiển thị phản hồi (như loading indicator) cho người dùng một cách thông minh.

Dựa trên các tài liệu chính thức từ pub.dev, dưới đây là bảng tổng hợp và so sánh chi tiết các loại Provider trong Riverpod.

Để dễ hiểu, tôi chia chúng thành 3 nhóm: **Cơ bản (Chỉ đọc)**, **Hiện đại (Riverpod 2.0+)** và **Cũ (Legacy)**.

### Bảng so sánh tổng quan

| Tên Provider | Dữ liệu quản lý | Tính chất State | Khuyên dùng | Mục đích chính |
| --- | --- | --- | --- | --- |
| **Provider** | Mọi loại | Read-only | ✅ Rất cao | Dependency Injection, tính toán giá trị, Cache object. |
| **FutureProvider** | `Future` | Read-only (Async) | ✅ Cao | Gọi API lấy dữ liệu 1 lần (config, static data). |
| **StreamProvider** | `Stream` | Read-only (Async) | ✅ Cao | Lắng nghe dữ liệu realtime (Firebase, WebSocket). |
| **NotifierProvider** | Synchronous | Read/Write | ✅✅ Rất cao | Thay thế `StateNotifier`. Quản lý state đồng bộ phức tạp. |
| **AsyncNotifierProvider** | Asynchronous | Read/Write | ✅✅ Rất cao | Thay thế `FutureProvider`. Quản lý state API cần tương tác (CRUD). |
| **StreamNotifierProvider** | Stream | Read/Write | ✅✅ Rất cao | Thay thế `StreamProvider`. Quản lý Stream cần tương tác. |
| **StateProvider** | Simple (int, bool) | Read/Write | ⚠️ Hạn chế | Chỉ dùng cho UI state cực đơn giản (filter, toggle). |
| **StateNotifierProvider** | Synchronous | Read/Write | ❌ Legacy | Bản cũ của `NotifierProvider`. Hạn chế dùng ở dự án mới. |
| **ChangeNotifierProvider** | Mutable Object | Read/Write | ❌ Tránh dùng | Chỉ dùng khi migrate từ Provider cũ, hiệu năng thấp. |

---

### 1. Nhóm Cơ bản (Chủ yếu để Đọc dữ liệu)

Nhóm này dùng để cung cấp dữ liệu hoặc lắng nghe dữ liệu từ nguồn khác mà **không chứa logic nghiệp vụ phức tạp** để sửa đổi state bên trong.

#### [Provider](https://pub.dev/documentation/hooks_riverpod/latest/hooks_riverpod/Provider-class.html)

* **Đặc điểm:** Là loại cơ bản nhất. Giá trị thường là bất biến hoặc là một Class Service/Repository.
* **Khi nào dùng:**
* Cung cấp Repository, API Client (Dependency Injection).
* Tính toán giá trị phái sinh (VD: Lọc danh sách Todo đã hoàn thành từ `todoListProvider`).
* Lưu cache các giá trị tính toán nặng.



#### [FutureProvider](https://pub.dev/documentation/hooks_riverpod/latest/hooks_riverpod/FutureProvider-class.html)

* **Đặc điểm:** Tương đương với `FutureBuilder` của Flutter nhưng mạnh hơn. Tự động xử lý trạng thái `loading/error/data` vào `AsyncValue`.
* **Khi nào dùng:**
* Đọc file config khi mở app.
* Gọi API lấy thông tin User (nếu không có chức năng sửa User ngay tại đó).


* **Lưu ý:** Không có sẵn hàm để sửa đổi state. Nếu muốn sửa state (VD: thêm/sửa/xóa), hãy dùng `AsyncNotifierProvider`.

#### [StreamProvider](https://pub.dev/documentation/hooks_riverpod/latest/hooks_riverpod/StreamProvider-class.html)

* **Đặc điểm:** Tương đương `StreamBuilder`. Tự động lắng nghe Stream và cập nhật UI.
* **Khi nào dùng:**
* Lắng nghe trạng thái đăng nhập (Auth State) từ Firebase.
* Lắng nghe tin nhắn Chat, vị trí GPS.
* Hiển thị % pin hoặc download progress.



---

### 2. Nhóm Hiện đại (Notifier - Riverpod 2.0+)

Đây là tiêu chuẩn mới của Riverpod. Hỗ trợ **Code Generation** (`@riverpod`), cú pháp gọn gàng hơn và xử lý `ref` tốt hơn các bản cũ.

#### [NotifierProvider](https://pub.dev/documentation/hooks_riverpod/latest/hooks_riverpod/NotifierProvider-class.html)

* **Đặc điểm:** Quản lý state đồng bộ. Class `Notifier` có hàm `build()` để khởi tạo và các hàm logic để cập nhật state.
* **Ưu điểm so với bản cũ:** Có thể dùng `ref` ngay trong mọi hàm của class (không cần truyền qua constructor như `StateNotifier`).
* **Khi nào dùng:**
* Quản lý danh sách Todo, Giỏ hàng.
* Quản lý logic Form đăng nhập.
* Counter App.



#### [AsyncNotifierProvider](https://pub.dev/documentation/hooks_riverpod/latest/hooks_riverpod/AsyncNotifierProvider-class.html)

* **Đặc điểm:** Quản lý state bất đồng bộ (`Future`).
* **Sức mạnh:** Kết hợp sự đơn giản của `FutureProvider` với khả năng thay đổi dữ liệu.
* **Khi nào dùng:**
* Màn hình danh sách sản phẩm có tính năng: Pull-to-refresh, Load more, Search.
* Thực hiện logic "Optimistic Update" (Cập nhật UI trước khi server trả về).



#### [StreamNotifierProvider](https://pub.dev/documentation/hooks_riverpod/latest/hooks_riverpod/StreamNotifierProvider-class.html)

* **Đặc điểm:** Tương tự `AsyncNotifierProvider` nhưng dành cho `Stream`.
* **Khi nào dùng:** Chat App phức tạp, nơi bạn vừa muốn lắng nghe tin nhắn mới, vừa muốn có các hàm logic xử lý tin nhắn đó.

---

### 3. Nhóm Cũ / Legacy (Hạn chế dùng)

#### [StateProvider](https://pub.dev/documentation/hooks_riverpod/latest/legacy/StateProvider-class.html)

* **Đặc điểm:** Cho phép sửa state trực tiếp từ bên ngoài (`ref.read(provider.notifier).state = ...`).
* **Tại sao hạn chế:** Dễ dẫn đến logic bị phân tán lung tung trong Widget thay vì tập trung ở logic layer.
* **Nên dùng khi:** State cực kỳ đơn giản và không có logic đi kèm (VD: `currentIndex` của BottomNavBar, filter type).

#### [StateNotifierProvider](https://pub.dev/documentation/hooks_riverpod/latest/legacy/StateNotifierProvider-class.html)

* **Đặc điểm:** Phiên bản cũ của `NotifierProvider`.
* **Nhược điểm:** Khó khăn khi muốn Inject provider khác vào trong (phải truyền qua constructor rườm rà).
* **Lời khuyên:** Nếu đang dùng thì vẫn ổn, nhưng dự án mới nên dùng `NotifierProvider`.

#### [ChangeNotifierProvider](https://pub.dev/documentation/hooks_riverpod/latest/legacy/ChangeNotifierProvider-class.html)

* **Đặc điểm:** Dùng `ChangeNotifier` của Flutter (mutable state).
* **Nhược điểm:**
* Mutable state dễ gây bug khó tìm.
* Hiệu năng kém hơn Immutable state.
* Khó kết hợp với các tính năng nâng cao của Riverpod.


* **Lời khuyên:** Chỉ dùng khi bạn đang chuyển code cũ từ `provider` package sang `riverpod`.

### Tóm tắt chiến lược chọn Provider

1. Nếu dữ liệu **không đổi** hoặc là **Service**: Dùng `Provider`.
2. Nếu **lấy dữ liệu Async** (API) và có tương tác: Dùng `AsyncNotifierProvider`.
3. Nếu **quản lý logic Sync** phức tạp: Dùng `NotifierProvider`.
4. Nếu là **biến UI đơn giản** (ẩn/hiện): Dùng `StateProvider`.

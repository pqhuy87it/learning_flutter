Trong hệ sinh thái **Riverpod**, `StateNotifierProvider` chỉ là một trong số nhiều "công cụ" (Provider) được thiết kế để giải quyết các bài toán quản lý trạng thái khác nhau. Tùy thuộc vào độ phức tạp của dữ liệu (đồng bộ, bất đồng bộ, đơn giản hay phức tạp), bạn sẽ chọn một công cụ tương ứng.

Dưới đây là danh sách toàn bộ các lớp Provider cốt lõi trong Riverpod, được chia theo từng nhóm công dụng để bạn dễ hình dung:

### 1. Nhóm quản lý dữ liệu đơn giản & Chỉ đọc
* **`Provider`**
    * **Mục đích:** Là loại cơ bản nhất. Nó quản lý một giá trị **chỉ đọc** (không thể thay đổi từ bên ngoài) hoặc được dùng để tính toán một giá trị phái sinh từ các provider khác.
    * **Thường dùng để:** Cung cấp các Dependencies Injection (như cấu hình, class gọi API, class đọc Database).
* **`StateProvider`**
    * **Mục đích:** Quản lý các trạng thái **đơn giản**, có thể thay đổi trực tiếp từ giao diện mà không cần phải viết một class logic riêng.
    * **Thường dùng để:** Lưu trữ các giá trị nguyên thủy như `int`, `String`, `bool`, hoặc `enum` (Ví dụ: Trạng thái đóng/mở của một filter, số trang hiện tại, chế độ sáng/tối).

### 2. Nhóm quản lý Logic nghiệp vụ phức tạp
* **`StateNotifierProvider`** *(Như bạn đã biết)*
    * **Mục đích:** Quản lý một state phức tạp (thường là một Object/Class). Bắt buộc phải đi kèm với một class kế thừa `StateNotifier` để chứa toàn bộ các hàm xử lý logic. Tách biệt hoàn toàn UI và Business Logic.
    * **Thường dùng để:** Quản lý thông tin User, Giỏ hàng, Form đăng nhập.
* **`NotifierProvider`** *(Mới từ Riverpod 2.0+)*
    * **Mục đích:** Đây là phiên bản hiện đại, được khuyên dùng để **thay thế** cho `StateNotifierProvider` trong các dự án mới. Nó khắc phục một số hạn chế của phiên bản cũ và hỗ trợ tốt hơn cho việc tự tạo mã (Code Generation).

### 3. Nhóm xử lý Dữ liệu Bất đồng bộ (Async/API)
Nhóm này cực kỳ mạnh mẽ vì nó tự động bọc dữ liệu của bạn vào một object gọi là `AsyncValue`, giúp UI tự động biết khi nào đang tải (`loading`), khi nào có lỗi (`error`), và khi nào có dữ liệu (`data`) mà không cần bạn tự viết logic check `if/else`.

* **`FutureProvider`**
    * **Mục đích:** Quản lý các tác vụ bất đồng bộ trả về kết quả một lần (`Future`).
    * **Thường dùng để:** Gọi REST API lấy dữ liệu, đọc file từ ổ cứng.
* **`StreamProvider`**
    * **Mục đích:** Quản lý luồng dữ liệu liên tục (`Stream`).
    * **Thường dùng để:** Lắng nghe Firebase Firestore, chat realtime qua WebSockets, đếm ngược thời gian.
* **`AsyncNotifierProvider`** *(Mới từ Riverpod 2.0+)*
    * **Mục đích:** Sự kết hợp hoàn hảo giữa `NotifierProvider` và `FutureProvider`.
    * **Thường dùng để:** Khi bạn muốn gọi API lấy danh sách (giống FutureProvider) nhưng sau đó lại muốn có các hàm để thêm/sửa/xóa phần tử trong danh sách đó ngay trên UI, hoặc làm tính năng Kéo để làm mới (Pull-to-refresh).

### 4. Nhóm Tương thích ngược (Legacy)
* **`ChangeNotifierProvider`**
    * **Mục đích:** Quản lý state thông qua class `ChangeNotifier` truyền thống của Flutter. 
    * **Thường dùng để:** Hỗ trợ các dự án cũ đang dùng thư viện `provider` (của Remi Rousselet) muốn migrate dần sang Riverpod. *Lưu ý: Không được khuyên dùng cho các tính năng viết mới vì hiệu năng không tối ưu bằng các loại trên.*

---

**Tóm tắt cách chọn:**
* Chỉ cần đọc / Cấu hình tĩnh -> **`Provider`**
* Chỉ là True/False, số, chuỗi -> **`StateProvider`**
* Logic phức tạp, nhiều hàm xử lý -> **`NotifierProvider`** (hoặc **`StateNotifierProvider`**)
* Gọi API (đọc 1 lần) -> **`FutureProvider`**
* Gọi API nhưng cần tương tác (kéo làm mới, load thêm) -> **`AsyncNotifierProvider`**

Bạn có muốn tôi kiểm tra thử xem trong source code **Pixiv Artvier** của bạn, tác giả đang sử dụng những loại Provider nào phổ biến nhất và xem một ví dụ thực tế về `FutureProvider` trong đó không?

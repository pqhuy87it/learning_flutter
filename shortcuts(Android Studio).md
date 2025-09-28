Chào bạn, việc sử dụng thành thạo các phím tắt (shortcuts) trong Android Studio là một trong những cách hiệu quả nhất để tăng tốc độ lập trình Flutter. Dưới đây là danh sách chi tiết các phím tắt hữu ích nhất, được phân loại theo chức năng, cùng với ghi chú cho cả Windows/Linux và macOS.

---

### 🚀 Phím tắt quan trọng nhất (Must-Know)

| Chức năng | Windows / Linux | macOS | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Tìm kiếm mọi thứ** | `Shift` + `Shift` (Double Shift) | `Shift` + `Shift` (Double Shift) | Tìm kiếm file, class, action, setting... cực kỳ mạnh mẽ. |
| **Wrap with Widget...** | `Alt` + `Enter` | `⌥` + `Enter` (Option + Enter) | Đặt con trỏ vào một widget và nhấn, sau đó chọn "Wrap with...". |
| **Remove Widget** | `Alt` + `Enter` | `⌥` + `Enter` (Option + Enter) | Đặt con trỏ vào widget cha và chọn "Remove this widget". |
| **Format Code** | `Ctrl` + `Alt` + `L` | `⌘` + `⌥` + `L` (Cmd + Option + L) | Tự động định dạng lại code theo chuẩn Dart. |
| **Show Quick Fixes** | `Alt` + `Enter` | `⌥` + `Enter` (Option + Enter) | Hiển thị các gợi ý sửa lỗi, import thư viện, wrap widget... |
| **Hot Reload** | `Ctrl` + `\` | `⌘` + `\` (Cmd + Backslash) | Tải lại nhanh các thay đổi code mà không mất trạng thái ứng dụng. |
| **Hot Restart** | `Ctrl` + `Shift` + `\` | `⌘` + `Shift` + `\` | Khởi động lại toàn bộ ứng dụng, reset trạng thái. |

---

### 📝 Soạn thảo & Chỉnh sửa Code (Editing)

| Chức năng | Windows / Linux | macOS | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Tự động hoàn thành code** | `Ctrl` + `Space` | `^` + `Space` (Control + Space) | Gợi ý các class, phương thức, biến... |
| **Xóa một dòng** | `Ctrl` + `Y` | `⌘` + `Delete` | Xóa dòng hiện tại nơi con trỏ đang đứng. |
| **Nhân đôi dòng/khối code** | `Ctrl` + `D` | `⌘` + `D` (Cmd + D) | Sao chép dòng hiện tại hoặc khối code đang chọn xuống dưới. |
| **Di chuyển dòng/khối code** | `Ctrl` + `Shift` + `↑` / `↓` | `⌘` + `Shift` + `↑` / `↓` | Di chuyển một dòng hoặc một khối code lên/xuống. |
| **Comment/Uncomment dòng** | `Ctrl` + `/` | `⌘` + `/` (Cmd + Slash) | Thêm/xóa comment `//` cho dòng hiện tại hoặc các dòng đã chọn. |
| **Comment/Uncomment khối** | `Ctrl` + `Shift` + `/` | `⌘` + `⌥` + `/` (Cmd + Option + Slash) | Thêm/xóa comment khối `/* ... */`. |
| **Chọn khối code thông minh** | `Ctrl` + `W` | `⌥` + `↑` (Option + Up Arrow) | Mở rộng vùng chọn ra khối code lớn hơn (từ biến -> câu lệnh -> hàm...). |
| **Thu nhỏ vùng chọn** | `Ctrl` + `Shift` + `W` | `⌥` + `↓` (Option + Down Arrow) | Ngược lại với phím tắt trên. |
| **Đổi tên (Refactor)** | `Shift` + `F6` | `⇧` + `F6` (Shift + F6) | Đổi tên biến, hàm, class... ở tất cả mọi nơi nó được sử dụng. |
| **Extract Widget/Method...** | `Ctrl` + `Alt` + `W` / `M` | `⌘` + `⌥` + `W` / `M` | Tách một khối code ra thành một Widget hoặc Method riêng. |

---

### 🧭 Điều hướng & Tìm kiếm (Navigation & Search)

| Chức năng | Windows / Linux | macOS | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Đi tới định nghĩa** | `Ctrl` + `Click` hoặc `Ctrl` + `B` | `⌘` + `Click` hoặc `⌘` + `B` | Nhảy tới nơi class, hàm, biến được định nghĩa. |
| **Xem nhanh định nghĩa** | `Ctrl` + `Shift` + `I` | `⌥` + `Space` | Hiển thị một cửa sổ popup chứa code định nghĩa mà không cần rời đi. |
| **Tìm nơi sử dụng** | `Alt` + `F7` | `⌥` + `F7` (Option + F7) | Tìm tất cả các nơi mà một class, hàm, biến được sử dụng trong dự án. |
| **Quay lại vị trí trước đó** | `Ctrl` + `Alt` + `←` | `⌘` + `[` hoặc `⌘` + `⌥` + `←` | Nhảy về vị trí con trỏ trước đó. |
| **Tiến tới vị trí sau đó** | `Ctrl` + `Alt` + `→` | `⌘` + `]` hoặc `⌘` + `⌥` + `→` | Ngược lại với phím tắt trên. |
| **Tìm trong file hiện tại** | `Ctrl` + `F` | `⌘` + `F` (Cmd + F) | Tìm kiếm văn bản trong file đang mở. |
| **Tìm và thay thế trong file** | `Ctrl` + `R` | `⌘` + `R` (Cmd + R) | |
| **Tìm trong toàn bộ dự án** | `Ctrl` + `Shift` + `F` | `⌘` + `Shift` + `F` | Tìm kiếm văn bản trong tất cả các file của project. |
| **Tìm và thay thế trong dự án** | `Ctrl` + `Shift` + `R` | `⌘` + `Shift` + `R` | |
| **Hiển thị cấu trúc file** | `Ctrl` + `F12` | `⌘` + `F12` | Hiển thị popup chứa danh sách các hàm, biến trong file hiện tại để nhảy nhanh. |
| **Đi tới dòng...** | `Ctrl` + `G` | `⌘` + `L` (Cmd + L) | Nhảy tới một số dòng cụ thể. |

---

### 🐛 Gỡ lỗi & Chạy ứng dụng (Debugging & Running)

| Chức năng | Windows / Linux | macOS | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Chạy ứng dụng** | `Shift` + `F10` | `^` + `R` (Control + R) | Chạy ứng dụng ở chế độ bình thường. |
| **Chạy ở chế độ Debug** | `Shift` + `F9` | `^` + `D` (Control + D) | Chạy ứng dụng với trình gỡ lỗi (debugger) được đính kèm. |
| **Toggle Breakpoint** | `Ctrl` + `F8` | `⌘` + `F8` | Đặt hoặc xóa điểm dừng (breakpoint) tại dòng hiện tại. |
| **Step Over** | `F8` | `F8` | Thực thi dòng hiện tại và đi tới dòng tiếp theo (không đi vào bên trong hàm). |
| **Step Into** | `F7` | `F7` | Nếu dòng hiện tại là một lời gọi hàm, sẽ đi vào bên trong hàm đó. |
| **Resume Program** | `F9` | `⌥` + `⌘` + `R` | Tiếp tục chạy chương trình cho đến khi gặp breakpoint tiếp theo. |

---

### 💻 Quản lý Cửa sổ & Giao diện (Window & UI Management)

| Chức năng | Windows / Linux | macOS | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Mở/Đóng cây thư mục** | `Alt` + `1` | `⌘` + `1` | Hiển thị hoặc ẩn cửa sổ Project. |
| **Mở/Đóng Terminal** | `Alt` + `F12` | `⌥` + `F12` | Mở cửa sổ dòng lệnh tích hợp. |
| **Mở/Đóng TODO** | `Alt` + `6` | `⌘` + `6` | Hiển thị các comment `// TODO:`. |
| **Mở/Đóng Flutter Inspector** | `Ctrl` + `Shift` + `P` (trong DevTools) | `⌘` + `Shift` + `P` | Mở công cụ Flutter Inspector. |
| **Ẩn tất cả các cửa sổ** | `Ctrl` + `Shift` + `F12` | `⌘` + `Shift` + `F12` | Tối đa hóa cửa sổ soạn thảo code. |

### Lời khuyên

*   **Bắt đầu với những phím tắt quan trọng nhất:** Đừng cố gắng học tất cả cùng một lúc. Hãy tập trung vào nhóm "Must-Know" trước, đặc biệt là `Alt + Enter` và `Ctrl + Alt + L`.
*   **Sử dụng "Find Action"**: Nếu bạn quên một phím tắt, hãy nhấn `Ctrl + Shift + A` (Windows/Linux) hoặc `Cmd + Shift + A` (macOS) và gõ tên hành động bạn muốn làm (ví dụ: "format code"). Nó sẽ hiển thị phím tắt tương ứng.
*   **In ra và dán lên tường:** In ra một danh sách các phím tắt bạn hay dùng và dán nó ở nơi bạn có thể dễ dàng nhìn thấy.

Chúc bạn lập trình hiệu quả

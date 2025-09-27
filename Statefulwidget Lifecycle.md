Tất nhiên, đây là danh sách các phương thức chính trong vòng đời của một `StatefulWidget` theo đúng thứ tự và ý nghĩa của chúng.



---
### **1. `createState()`**
* **Thứ tự:** #1
* **Ý nghĩa:** Đây là phương thức đầu tiên được gọi khi một `StatefulWidget` được đưa vào cây widget. Nó **tạo ra đối tượng State** sẽ được liên kết với widget này. Framework sẽ gọi phương thức này một lần duy nhất cho mỗi `StatefulWidget`.

---
### **2. `mounted` is `true`**
* **Thứ tự:** #2
* **Ý nghĩa:** Đây không phải là một phương thức mà là một thuộc tính (`property`). Ngay sau khi `createState()` chạy và đối tượng State được liên kết với `BuildContext`, thuộc tính `mounted` sẽ trở thành `true`. Điều này báo hiệu rằng widget đã có một vị trí trong cây widget và có thể truy cập `context`. Bạn nên kiểm tra `if (mounted)` trước khi gọi `setState()` trong các hàm bất đồng bộ.

---
### **3. `initState()`**
* **Thứ tự:** #3
* **Ý nghĩa:** Được gọi **một lần duy nhất** ngay sau khi widget được tạo ra. Đây là nơi để thực hiện các công việc khởi tạo chỉ cần làm một lần, chẳng hạn như:
    * Khởi tạo dữ liệu, biến.
    * Đăng ký (subscribe) vào các `Stream`, `ChangeNotifier`, hoặc `AnimationController`.
    * **Lưu ý:** Bạn không thể sử dụng `BuildContext` trong phương thức này.

---
### **4. `didChangeDependencies()`**
* **Thứ tự:** #4 (và có thể gọi lại)
* **Ý nghĩa:** Được gọi ngay sau `initState()` lần đầu tiên. Quan trọng hơn, nó sẽ được **gọi lại mỗi khi một đối tượng `InheritedWidget` mà widget này phụ thuộc vào thay đổi**. Đây là nơi thích hợp để:
    * Thực hiện các tác vụ cần `BuildContext`.
    * Tải dữ liệu từ API hoặc cơ sở dữ liệu khi dữ liệu đó phụ thuộc vào `InheritedWidget` (ví dụ: `Provider.of(context)`).

---
### **5. `build()`**
* **Thứ tự:** #5 (và được gọi lại thường xuyên)
* **Ý nghĩa:** Đây là phương thức quan trọng nhất, chịu trách nhiệm **xây dựng và trả về cây widget con** để hiển thị lên giao diện. `build()` sẽ được gọi lại mỗi khi:
    * `setState()` được gọi.
    * `didChangeDependencies()` được gọi.
    * Widget cha của nó được xây dựng lại.
    * **Lưu ý:** Phương thức này phải chạy thật nhanh và không nên chứa các logic xử lý phức tạp.

---
### **6. `didUpdateWidget()`**
* **Thứ tự:** #6 (chỉ được gọi khi widget được cập nhật)
* **Ý nghĩa:** Được gọi khi widget cha được xây dựng lại và cung cấp một phiên bản mới của widget hiện tại (với các thuộc tính có thể đã thay đổi). Framework sẽ so sánh `widget` cũ và `newWidget` mới. Đây là nơi để bạn phản ứng với sự thay đổi đó, ví dụ như reset lại một animation hoặc đăng ký lại một `Stream` nếu một `ID` nào đó thay đổi.

---
### **7. `setState()`**
* **Thứ tự:** Được gọi bởi lập trình viên
* **Ý nghĩa:** Đây là phương thức bạn gọi để thông báo cho framework rằng **trạng thái nội tại của widget đã thay đổi**. Việc gọi `setState()` sẽ lên lịch cho một lần chạy lại của phương thức `build()`, giúp cập nhật giao diện người dùng theo trạng thái mới.

---
### **8. `deactivate()`**
* **Thứ tự:** #7 (khi State bị xóa khỏi cây)
* **Ý nghĩa:** Được gọi khi đối tượng State bị xóa khỏi cây widget, nhưng có thể nó sẽ được chèn lại vào một vị trí khác (ví dụ khi di chuyển một widget trong danh sách bằng `GlobalKey`). Phương thức này ít khi được sử dụng trực tiếp.

---
### **9. `dispose()`**
* **Thứ tự:** #8 (khi State bị xóa vĩnh viễn)
* **Ý nghĩa:** Được gọi khi đối tượng State bị **xóa vĩnh viễn** khỏi cây widget. Đây là nơi bạn phải thực hiện các công việc "dọn dẹp" để tránh rò rỉ bộ nhớ, chẳng hạn như:
    * Hủy đăng ký (unsubscribe) các `Stream`, `ChangeNotifier`.
    * Hủy các `Timer`.
    * Giải phóng tài nguyên của các `AnimationController`.

---
### **10. `mounted` is `false`**
* **Thứ tự:** #9
* **Ý nghĩa:** Sau khi `dispose()` được gọi, thuộc tính `mounted` sẽ chuyển thành `false`. Widget không còn trong cây widget và không bao giờ được build lại nữa.

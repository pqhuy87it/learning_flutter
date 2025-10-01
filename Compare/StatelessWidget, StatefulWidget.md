Chắc chắn rồi! Đây là so sánh chi tiết giữa `StatelessWidget` và `StatefulWidget`, hai khối xây dựng cơ bản và quan trọng nhất trong Flutter. Hiểu rõ sự khác biệt giữa chúng là chìa khóa để xây dựng ứng dụng hiệu quả.

---

### Tổng quan

Trong Flutter, mọi thứ bạn thấy trên màn hình đều là Widget. Chúng được chia thành hai loại chính dựa trên khả năng quản lý **trạng thái (state)**.

*   **State (Trạng thái)**: Là bất kỳ dữ liệu nào có thể thay đổi theo thời gian trong ứng dụng của bạn. Ví dụ: giá trị của một thanh trượt, nội dung của một ô nhập liệu, một sản phẩm đã được thêm vào giỏ hàng, trạng thái bật/tắt của một công tắc.

Sự khác biệt cốt lõi là:
*   `StatelessWidget`: **Không** có trạng thái nội tại. Một khi đã được tạo, nó không thể tự thay đổi.
*   `StatefulWidget`: **Có** trạng thái nội tại. Nó có thể tự thay đổi và vẽ lại giao diện của mình nhiều lần trong suốt vòng đời.

---

### 1. StatelessWidget (Widget không trạng thái)

#### Khái niệm cốt lõi

`StatelessWidget` là một widget **bất biến (immutable)**. Điều này có nghĩa là tất cả các thuộc tính của nó (các biến được khai báo với `final`) được thiết lập một lần duy nhất khi widget được tạo (thông qua constructor) và không bao giờ thay đổi trong suốt vòng đời của widget đó.

Giao diện của nó chỉ được vẽ lại khi widget cha của nó xây dựng lại và truyền vào các thông tin cấu hình mới.

#### Đặc điểm

*   **Vòng đời đơn giản**: Chỉ có hai giai đoạn chính: constructor để khởi tạo và phương thức `build()` để vẽ giao diện.
*   **Không có State Object**: Nó không có đối tượng `State` đi kèm.
*   **Hiệu suất cao**: Vì chúng bất biến và không cần quản lý trạng thái, chúng nhẹ hơn và có hiệu suất tốt hơn cho các thành phần giao diện tĩnh.
*   **Phụ thuộc vào dữ liệu từ bên ngoài**: Giao diện của nó được quyết định hoàn toàn bởi các tham số được truyền vào từ widget cha.

#### Khi nào nên sử dụng `StatelessWidget`?

Sử dụng nó cho bất kỳ phần giao diện nào **không cần thay đổi** sau khi đã được hiển thị lần đầu.

*   Các icon, nhãn (labels), nút bấm đơn giản.
*   Văn bản tĩnh (`Text('Xin chào')`).
*   Các thẻ hiển thị thông tin không đổi (ví dụ: `Card` hiển thị chi tiết một sản phẩm tĩnh).
*   Bất kỳ widget nào mà giao diện của nó chỉ phụ thuộc vào thông tin cấu hình được truyền vào.

#### Ví dụ về mã nguồn

```dart
// Một card hiển thị thông tin người dùng một cách tĩnh
class UserProfileCard extends StatelessWidget {
  final String name;
  final String email;

  // Dữ liệu được truyền vào qua constructor và không bao giờ thay đổi bên trong widget này
  const UserProfileCard({
    super.key,
    required this.name,
    required this.email,
  });

  @override
  Widget build(BuildContext context) {
    // Phương thức build chỉ chạy khi UserProfileCard được tạo
    // hoặc khi widget cha của nó rebuild và truyền vào name/email mới.
    return Card(
      child: ListTile(
        leading: Icon(Icons.person),
        title: Text(name),
        subtitle: Text(email),
      ),
    );
  }
}
```

---

### 2. StatefulWidget (Widget có trạng thái)

#### Khái niệm cốt lõi

`StatefulWidget` là một widget **động (dynamic)**. Nó có thể thay đổi giao diện của mình nhiều lần để phản hồi lại các sự kiện (người dùng tương tác, dữ liệu từ mạng về, v.v.).

Nó được tạo thành từ hai lớp:
1.  **Lớp `StatefulWidget`**: Lớp này cũng bất biến, chứa thông tin cấu hình được truyền từ widget cha. Nhiệm vụ duy nhất của nó là tạo ra một đối tượng `State`.
2.  **Lớp `State`**: Đây là nơi chứa tất cả trạng thái có thể thay đổi và logic để xây dựng giao diện. Đối tượng `State` tồn tại lâu dài qua nhiều lần widget được xây dựng lại.

#### Đặc điểm

*   **Vòng đời phức tạp**: Có nhiều phương thức vòng đời quan trọng để quản lý trạng thái, như `initState()`, `didChangeDependencies()`, `build()`, `didUpdateWidget()`, và `dispose()`.
*   **Tự vẽ lại giao diện**: Có thể gọi phương thức `setState()` để thông báo cho Flutter framework rằng trạng thái đã thay đổi và widget cần được xây dựng lại (gọi lại phương thức `build()`) với dữ liệu mới.
*   **Linh hoạt**: Phù hợp cho các giao diện cần tương tác và thay đổi.

#### Vòng đời quan trọng của `State`

1.  `createState()`: (Trong lớp Widget) Được gọi để tạo đối tượng `State`.
2.  `initState()`: Được gọi **một lần duy nhất** khi đối tượng `State` được tạo và chèn vào cây widget. Đây là nơi hoàn hảo để khởi tạo dữ liệu, đăng ký listeners, hoặc gọi API lần đầu.
3.  `build()`: Được gọi mỗi khi widget cần được vẽ lên màn hình. Nó có thể được gọi nhiều lần.
4.  `setState(VoidCallback fn)`: Phương thức quan trọng nhất. Khi bạn gọi nó, hàm `fn` bên trong sẽ được thực thi, sau đó Flutter sẽ lên lịch để **gọi lại phương thức `build()`**, cập nhật giao diện với trạng thái mới.
5.  `dispose()`: Được gọi khi đối tượng `State` bị xóa vĩnh viễn khỏi cây widget. Đây là nơi bạn phải dọn dẹp tài nguyên, như hủy bỏ các `StreamSubscription`, `AnimationController`, `TextEditingController` để tránh rò rỉ bộ nhớ.

#### Khi nào nên sử dụng `StatefulWidget`?

Sử dụng nó khi giao diện của widget cần thay đổi dựa trên **tương tác của người dùng** hoặc **dữ liệu thay đổi theo thời gian**.

*   Các ô checkbox, radio button, switch.
*   Các form nhập liệu (`TextField`).
*   Một màn hình hiển thị dữ liệu được tải từ internet.
*   Bất kỳ widget nào có chứa animation.
*   Một bộ đếm số lần nhấn nút.

#### Ví dụ về mã nguồn

```dart
// Một nút bấm đếm số lần được nhấn
class CounterButton extends StatefulWidget {
  const CounterButton({super.key});

  @override
  State<CounterButton> createState() => _CounterButtonState();
}

class _CounterButtonState extends State<CounterButton> {
  // Biến _counter là "trạng thái" của widget này
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    // Khởi tạo trạng thái ban đầu
    _counter = 0;
    print("Widget được khởi tạo!");
  }

  void _incrementCounter() {
    // Gọi setState để thông báo cho Flutter rằng trạng thái đã thay đổi
    setState(() {
      // Logic thay đổi trạng thái được đặt bên trong hàm của setState
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    // Phương thức build được gọi lại mỗi khi _counter thay đổi
    print("Widget được build lại!");
    return ElevatedButton(
      onPressed: _incrementCounter,
      child: Text('Bạn đã nhấn $_counter lần'),
    );
  }
  
  @override
  void dispose() {
    print("Widget đã bị hủy!");
    super.dispose();
  }
}
```

---

### Bảng tóm tắt so sánh

| Tiêu chí | StatelessWidget | StatefulWidget |
| :--- | :--- | :--- |
| **Trạng thái (State)** | Bất biến (Immutable), không có trạng thái nội tại. | Khả biến (Mutable), có trạng thái nội tại được quản lý bởi đối tượng `State`. |
| **Vòng đời** | Rất đơn giản (constructor, `build`). | Phức tạp hơn (`initState`, `build`, `setState`, `dispose`, v.v.). |
| **Phương thức chính** | `build()` | `createState()` và `setState()` trong đối tượng `State`. |
| **Tự cập nhật** | Không thể tự cập nhật. Được vẽ lại khi widget cha thay đổi. | Có thể tự cập nhật và vẽ lại giao diện bằng cách gọi `setState()`. |
| **Hiệu suất** | Nhẹ hơn, nhanh hơn cho các UI tĩnh. | Nặng hơn một chút do phải quản lý đối tượng `State`. |
| **Trường hợp sử dụng** | Giao diện tĩnh, không thay đổi: icon, text, card thông tin. | Giao diện động, có tương tác: form, checkbox, animation, dữ liệu thay đổi. |

### Lời khuyên

*   **Ưu tiên `StatelessWidget`**: Luôn bắt đầu với `StatelessWidget`. Chỉ chuyển sang `StatefulWidget` khi bạn thực sự cần quản lý một trạng thái sẽ thay đổi bên trong widget đó.
*   **Giữ trạng thái ở mức thấp nhất**: Cố gắng đặt trạng thái ở widget lá (widget con nhất có thể) trong cây widget của bạn. Điều này giúp hạn chế việc phải xây dựng lại các phần không cần thiết của giao diện, giúp tối ưu hóa hiệu suất.

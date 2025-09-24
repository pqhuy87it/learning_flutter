Chắc chắn rồi! `BottomSheet` là một thành phần giao diện cực kỳ hữu ích trong Flutter, được sử dụng để hiển thị một menu các tùy chọn hoặc thông tin bổ sung từ cạnh dưới của màn hình. Nó mang lại trải nghiệm người dùng mượt mà và hiện đại.

Có hai loại `BottomSheet` chính, và cách sử dụng chúng hoàn toàn khác nhau. Hãy cùng phân tích chi tiết từng loại.

---

### Tổng quan: 2 loại BottomSheet

1.  **Modal Bottom Sheet (Phổ biến nhất)**:
    *   Đây là một panel **tạm thời** trượt lên từ phía dưới.
    *   Nó **chặn tương tác** với phần còn lại của màn hình. Người dùng phải chọn một tùy chọn hoặc đóng nó (bằng cách vuốt xuống hoặc nhấn ra ngoài) để tiếp tục.
    *   Được gọi bằng hàm `showModalBottomSheet()`.
    *   **Trường hợp sử dụng**: Menu chia sẻ, chọn ảnh từ thư viện/camera, danh sách các hành động (sửa, xóa, sao chép)...

2.  **Persistent Bottom Sheet (Ít phổ biến hơn)**:
    *   Đây là một panel **cố định** hiển thị bên trong nội dung chính của màn hình.
    *   Nó **không chặn tương tác**. Người dùng có thể tương tác với cả bottom sheet và phần còn lại của màn hình cùng một lúc.
    *   Được hiển thị bằng cách gọi phương thức `showBottomSheet()` của `ScaffoldState`.
    *   **Trường hợp sử dụng**: Trình phát nhạc (như Spotify), hiển thị thông tin ngữ cảnh bổ sung mà không làm gián đoạn luồng chính.

---

### Phần 1: Modal Bottom Sheet (Cách sử dụng chi tiết)

Đây là loại bạn sẽ sử dụng trong 95% các trường hợp.

#### a. Cách gọi `showModalBottomSheet`

Đây là một hàm toàn cục, bạn có thể gọi nó từ bất cứ đâu có `BuildContext`.

```dart
showModalBottomSheet(
  context: context, // Bắt buộc
  builder: (BuildContext context) {
    // Trả về widget bạn muốn hiển thị bên trong bottom sheet
    return Container(
      height: 200,
      child: Center(
        child: Text('This is a Modal Bottom Sheet'),
      ),
    );
  },
);
```

-   **`context`**: `BuildContext` của widget gọi hàm.
-   **`builder`**: Một hàm nhận vào `BuildContext` và trả về `Widget`. Đây chính là nơi bạn xây dựng giao diện cho bottom sheet của mình.

#### b. Ví dụ 1: Một menu lựa chọn đơn giản

```dart
// Trong một StatefulWidget hoặc StatelessWidget
ElevatedButton(
  child: Text('Show Modal Bottom Sheet'),
  onPressed: () {
    showModalBottomSheet(
      context: context,
      builder: (BuildContext context) {
        return Container(
          // Thêm padding cho các cạnh
          padding: EdgeInsets.all(16.0),
          child: Wrap( // Wrap giúp các item tự xuống dòng nếu không đủ chỗ
            children: <Widget>[
              ListTile(
                leading: Icon(Icons.share),
                title: Text('Share'),
                onTap: () {
                  print('Share tapped');
                  Navigator.pop(context); // Đóng bottom sheet sau khi chọn
                },
              ),
              ListTile(
                leading: Icon(Icons.link),
                title: Text('Get link'),
                onTap: () {
                  print('Get link tapped');
                  Navigator.pop(context);
                },
              ),
              ListTile(
                leading: Icon(Icons.edit),
                title: Text('Edit name'),
                onTap: () {
                  print('Edit name tapped');
                  Navigator.pop(context);
                },
              ),
            ],
          ),
        );
      },
    );
  },
)
```

#### c. Trả về dữ liệu từ BottomSheet

`showModalBottomSheet` trả về một `Future`. Bạn có thể dùng `await` để nhận giá trị được trả về khi bottom sheet được đóng bằng `Navigator.pop(context, value)`.

```dart
// Hàm gọi
final result = await showModalBottomSheet<String>( // Chỉ định kiểu dữ liệu trả về
  context: context,
  builder: (BuildContext context) {
    return Container(
      child: Wrap(
        children: <Widget>[
          ListTile(
            title: Text('Option 1'),
            onTap: () => Navigator.pop(context, 'Option 1 Selected'),
          ),
          ListTile(
            title: Text('Option 2'),
            onTap: () => Navigator.pop(context, 'Option 2 Selected'),
          ),
        ],
      ),
    );
  },
);

// Sau khi bottom sheet đóng, bạn có thể sử dụng kết quả
if (result != null) {
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(content: Text('You selected: $result')),
  );
}
```

#### d. Các thuộc tính tùy chỉnh quan trọng

`showModalBottomSheet` có rất nhiều thuộc tính để bạn tùy chỉnh giao diện và hành vi.

-   **`shape`**: Bo tròn các góc trên của bottom sheet.
    ```dart
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.vertical(top: Radius.circular(20.0)),
    ),
    ```
-   **`isScrollControlled: true`**: Rất quan trọng! Thuộc tính này cho phép bottom sheet chiếm nhiều hơn một nửa chiều cao màn hình, thậm chí là toàn màn hình. Cần thiết khi nội dung của bạn dài hoặc khi có bàn phím hiện lên.
    ```dart
    // Ví dụ về một bottom sheet toàn màn hình
    showModalBottomSheet(
      context: context,
      isScrollControlled: true, // Bắt buộc
      builder: (context) => Padding(
        padding: EdgeInsets.only(
          // Thêm padding cho phần bị che bởi bàn phím
          bottom: MediaQuery.of(context).viewInsets.bottom,
        ),
        child: Container(
          height: MediaQuery.of(context).size.height * 0.8, // 80% chiều cao màn hình
          child: ListView( /* Nội dung dài ở đây */ ),
        ),
      ),
    );
    ```
-   **`backgroundColor`**: Màu nền của bottom sheet.
-   **`isDismissible: false`**: Ngăn người dùng đóng bottom sheet bằng cách nhấn ra ngoài.
-   **`enableDrag: false`**: Ngăn người dùng vuốt xuống để đóng bottom sheet.
-   **`useSafeArea: true`**: (Mới) Tự động thêm padding để tránh các khu vực hệ thống như "tai thỏ" hoặc Dynamic Island ở đầu màn hình khi `isScrollControlled` là `true`.

---

### Phần 2: Persistent Bottom Sheet (Cách sử dụng chi tiết)

Loại này phức tạp hơn một chút vì nó gắn liền với `Scaffold`.

#### a. Yêu cầu

1.  Bạn phải có một `Scaffold`.
2.  Bạn cần một cách để truy cập `ScaffoldState` của nó. Cách phổ biến là dùng `GlobalKey`.

#### b. Ví dụ

```dart
class PersistentSheetExample extends StatefulWidget {
  @override
  _PersistentSheetExampleState createState() => _PersistentSheetExampleState();
}

class _PersistentSheetExampleState extends State<PersistentSheetExample> {
  // 1. Tạo một GlobalKey cho Scaffold
  final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();
  PersistentBottomSheetController? _controller; // Để có thể đóng nó

  void _toggleBottomSheet() {
    if (_controller == null) {
      // Nếu chưa mở, thì mở nó ra
      _controller = _scaffoldKey.currentState?.showBottomSheet(
        (context) => Container(
          height: 200,
          color: Colors.amber,
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                const Text('This is a Persistent Bottom Sheet'),
                ElevatedButton(
                  child: const Text('Close'),
                  onPressed: () {
                    _controller?.close(); // Đóng bottom sheet
                  },
                ),
              ],
            ),
          ),
        ),
      );
      // Khi bottom sheet được đóng (bằng cách vuốt hoặc gọi close),
      // đặt lại _controller thành null
      _controller?.closed.then((_) {
        setState(() {
          _controller = null;
        });
      });
    } else {
      // Nếu đã mở, thì đóng nó lại
      _controller?.close();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // 2. Gán key cho Scaffold
      key: _scaffoldKey,
      appBar: AppBar(title: Text('Persistent Bottom Sheet')),
      body: Center(
        child: ElevatedButton(
          // 3. Nút để mở/đóng bottom sheet
          onPressed: _toggleBottomSheet,
          child: Text('Toggle Persistent Sheet'),
        ),
      ),
    );
  }
}
```

-   **`_scaffoldKey.currentState?.showBottomSheet(...)`**: Đây là cách gọi để hiển thị.
-   **`PersistentBottomSheetController`**: Hàm `showBottomSheet` trả về một controller. Bạn cần lưu nó lại để có thể gọi phương thức `close()` sau này.
-   **`controller.closed.then(...)`**: Dùng để lắng nghe sự kiện khi bottom sheet đã được đóng hoàn toàn để cập nhật lại trạng thái.

### Bảng so sánh nhanh

| Tính năng | Modal Bottom Sheet | Persistent Bottom Sheet |
| :--- | :--- | :--- |
| **Cách gọi** | `showModalBottomSheet()` | `Scaffold.of(context).showBottomSheet()` |
| **Chặn UI** | **Có**, chặn tương tác với nền | **Không**, vẫn tương tác được |
| **Cách đóng** | Vuốt xuống, nhấn ra ngoài, `Navigator.pop()` | Chỉ có thể vuốt xuống hoặc đóng bằng code |
| **Bối cảnh** | Bên trên tất cả UI | Bên trong `Scaffold` |
| **Use Case** | Menu hành động, lựa chọn nhanh | Trình phát nhạc, thông tin bổ sung |

Hy vọng giải thích chi tiết này giúp bạn nắm vững cách sử dụng cả hai loại `BottomSheet` trong Flutter

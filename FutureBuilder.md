Chào bạn! `FutureBuilder` là một trong những widget tiện ích và được sử dụng nhiều nhất trong Flutter để xử lý giao diện người dùng bất đồng bộ (asynchronous UI). Nó giúp bạn giải quyết một bài toán rất phổ biến: **"Hiển thị một thứ gì đó trong khi đang chờ dữ liệu, sau đó hiển thị dữ liệu khi nó có, hoặc hiển thị lỗi nếu có sự cố."**

Hãy cùng phân tích chi tiết, từ khái niệm cơ bản đến các lỗi thường gặp nhé.

### 1. `FutureBuilder` là gì? - Một ví dụ đời thường

Hãy tưởng tượng bạn đặt một món hàng online. Quá trình này có các trạng thái:
1.  **Chưa đặt hàng:** Bạn chưa làm gì cả.
2.  **Đang chờ shipper:** Bạn đã đặt hàng và đang đợi (`Future` đang chạy). Màn hình ứng dụng hiển thị "Đang tìm shipper...".
3.  **Shipper đã giao hàng thành công:** Bạn nhận được món hàng (`Future` hoàn thành với dữ liệu). Màn hình hiển thị "Giao hàng thành công! Món hàng của bạn là...".
4.  **Giao hàng thất bại:** Shipper báo không tìm thấy địa chỉ (`Future` hoàn thành với lỗi). Màn hình hiển thị "Giao hàng thất bại: Không tìm thấy địa chỉ".

`FutureBuilder` chính là widget giúp bạn xây dựng giao diện cho tất cả các trạng thái trên một cách gọn gàng và có tổ chức.

### 2. Các thành phần chính của `FutureBuilder`

`FutureBuilder` có hai thuộc tính quan trọng nhất bạn cần cung cấp:

1.  **`future`:** Đây là "đơn hàng" của bạn. Nó là một đối tượng `Future` mà `FutureBuilder` sẽ "lắng nghe". Ví dụ: một hàm gọi API, một truy vấn cơ sở dữ liệu, đọc một file...
2.  **`builder`:** Đây là "người xây dựng giao diện". Nó là một hàm sẽ được gọi lại mỗi khi trạng thái của `future` thay đổi. Hàm `builder` này cung cấp cho bạn một đối tượng cực kỳ quan trọng là `AsyncSnapshot`.

### 3. "Trái tim" của `FutureBuilder`: `AsyncSnapshot`

`AsyncSnapshot` là một đối tượng chứa tất cả thông tin về trạng thái hiện tại của `Future`. Nó cho bạn biết: "Đơn hàng của bạn đang ở trạng thái nào rồi?".

Các thuộc tính chính của `AsyncSnapshot`:

*   **`connectionState`:** Trạng thái kết nối hiện tại. Đây là thuộc tính quan trọng nhất để bạn quyết định hiển thị widget nào. Nó có các giá trị trong enum `ConnectionState`:
    *   `ConnectionState.none`: `future` là `null` (bạn chưa đặt hàng).
    *   `ConnectionState.waiting`: `future` đang chạy (đang chờ shipper). Đây là lúc bạn hiển thị `CircularProgressIndicator`.
    *   `ConnectionState.active`: Dùng cho `StreamBuilder`, ít dùng với `FutureBuilder`.
    *   `ConnectionState.done`: `future` đã hoàn thành, dù thành công hay thất bại (shipper đã kết thúc công việc).
*   **`hasData`:** `true` nếu `future` hoàn thành thành công và trả về dữ liệu (không phải `null`).
*   **`data`:** Dữ liệu thực tế mà `future` trả về khi thành công. Nếu chưa có dữ liệu, nó sẽ là `null`.
*   **`hasError`:** `true` nếu `future` hoàn thành với một lỗi.
*   **`error`:** Đối tượng lỗi (exception) nếu có lỗi xảy ra.

### 4. Ví dụ chi tiết: Lấy dữ liệu từ một API giả lập

Hãy xây dựng một màn hình lấy một câu trích dẫn từ một API giả.

#### Bước 1: Tạo hàm `Future`

Hàm này sẽ giả lập việc gọi API mất 2 giây.

```dart
// Hàm này trả về một Future<String>
Future<String> fetchQuote() async {
  // Giả lập độ trễ mạng
  await Future.delayed(const Duration(seconds: 2));

  // Giả lập trường hợp có lỗi (ví dụ: mất mạng)
  // Bạn có thể bỏ comment dòng này để xem giao diện lỗi
  // if (DateTime.now().second % 2 == 0) {
  //   throw Exception("Không thể kết nối đến server!");
  // }

  // Trả về dữ liệu thành công
  return "Học, học nữa, học mãi. - V.I. Lenin";
}
```

#### Bước 2: Sử dụng `FutureBuilder` trong Widget

```dart
import 'package:flutter/material.dart';

class QuoteScreen extends StatefulWidget {
  const QuoteScreen({super.key});

  @override
  State<QuoteScreen> createState() => _QuoteScreenState();
}

class _QuoteScreenState extends State<QuoteScreen> {
  // RẤT QUAN TRỌNG: Khai báo Future ở đây!
  late final Future<String> _quoteFuture;

  @override
  void initState() {
    super.initState();
    // Gọi hàm fetchQuote MỘT LẦN DUY NHẤT trong initState
    _quoteFuture = fetchQuote();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FutureBuilder Demo'),
      ),
      body: Center(
        child: FutureBuilder<String>(
          // 1. Cung cấp future
          future: _quoteFuture,

          // 2. Cung cấp builder để xây dựng UI
          builder: (BuildContext context, AsyncSnapshot<String> snapshot) {
            // Dựa vào connectionState để quyết định UI
            if (snapshot.connectionState == ConnectionState.waiting) {
              // TRẠNG THÁI 1: Đang chờ
              return const CircularProgressIndicator();
            } else if (snapshot.hasError) {
              // TRẠNG THÁI 2: Có lỗi
              return Text(
                'Lỗi: ${snapshot.error}',
                style: const TextStyle(color: Colors.red),
                textAlign: TextAlign.center,
              );
            } else if (snapshot.hasData) {
              // TRẠNG THÁI 3: Thành công và có dữ liệu
              return Padding(
                padding: const EdgeInsets.all(16.0),
                child: Text(
                  '"${snapshot.data}"',
                  style: const TextStyle(fontSize: 24, fontStyle: FontStyle.italic),
                  textAlign: TextAlign.center,
                ),
              );
            } else {
              // TRẠNG THÁI 4: Thành công nhưng không có dữ liệu (snapshot.data là null)
              return const Text('Không có trích dẫn nào được tìm thấy.');
            }
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Nút để tải lại dữ liệu
          setState(() {
            _quoteFuture = fetchQuote();
          });
        },
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

### 5. Lỗi thường gặp nhất: Đặt `Future` sai chỗ!

Đây là sai lầm mà 99% người mới dùng `FutureBuilder` đều mắc phải.

**Cách làm SAI:**
```dart
@override
Widget build(BuildContext context) {
  return FutureBuilder<String>(
    future: fetchQuote(), // GỌI HÀM TRỰC TIẾP TRONG BUILD
    builder: (context, snapshot) {
      // ...
    },
  );
}
```

**Tại sao sai?**
Hàm `build()` có thể được gọi lại rất nhiều lần (khi xoay màn hình, khi bàn phím hiện lên, khi widget cha rebuild...). Mỗi lần `build()` chạy, nó sẽ gọi lại `fetchQuote()`, tạo ra một `Future` mới và bắt đầu lại quá trình lấy dữ liệu. Kết quả là bạn sẽ thấy `CircularProgressIndicator` nhấp nháy liên tục và không bao giờ thấy dữ liệu.

**Cách làm ĐÚNG (Như trong ví dụ chi tiết ở trên):**
1.  Khai báo một biến `Future` trong `State` của bạn (`late final Future<String> _quoteFuture;`).
2.  Khởi tạo `Future` đó **một lần duy nhất** trong `initState()`.
3.  Truyền biến đó vào thuộc tính `future` của `FutureBuilder`.

Bằng cách này, `Future` chỉ được tạo một lần khi widget được tạo ra. Các lần `build()` sau đó sẽ sử dụng lại cùng một `Future` đó, cho phép `FutureBuilder` theo dõi đúng trạng thái của nó.

### Tổng kết

*   `FutureBuilder` là widget lý tưởng để xây dựng UI dựa trên kết quả của một tác vụ bất đồng bộ (một `Future`).
*   Nó quản lý các trạng thái: **đang chờ, lỗi, thành công** một cách tự động.
*   Sử dụng `AsyncSnapshot` trong hàm `builder` để kiểm tra trạng thái và lấy dữ liệu/lỗi.
*   **Luôn luôn** khởi tạo `Future` trong `initState()` hoặc một nơi nào đó bên ngoài hàm `build()` để tránh việc gọi lại tác vụ bất đồng bộ một cách không cần thiết.

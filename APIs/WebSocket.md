**WebSocket** là một giao thức giao tiếp cung cấp kênh truyền thông hai chiều (bi-directional), toàn thời gian (full-duplex) qua một kết nối TCP duy nhất.

Nếu như HTTP (REST API) giống như việc bạn gửi một bức thư và chờ người ta gửi thư hồi âm (Client hỏi -> Server trả lời -> Ngắt kết nối), thì **WebSocket giống như một cuộc gọi điện thoại**: kết nối được giữ liên tục, cả bạn (App Flutter) và đầu dây bên kia (Server) có thể nói chuyện qua lại bất cứ lúc nào mà không cần phải gọi lại từ đầu.

### 1. Khi nào nên dùng WebSocket thay vì REST API?

Bạn nên dùng WebSocket khi ứng dụng yêu cầu tính **thời gian thực (Real-time)** và dữ liệu thay đổi liên tục:

* **Ứng dụng Chat / Nhắn tin:** (Zalo, Messenger) Tin nhắn phải nảy lên màn hình ngay lập tức khi người khác bấm gửi.
* **Bản đồ / Theo dõi vị trí:** (Grab, Uber) Nhìn thấy icon tài xế di chuyển liên tục trên bản đồ.
* **Biểu đồ tài chính / Chứng khoán / Crypto:** Giá cả thay đổi từng mili-giây.
* **Game Multiplayer:** Đồng bộ thao tác của các người chơi cùng lúc.

Với những tính năng này, nếu dùng HTTP truyền thống (Polling - cứ 1 giây lại gọi API hỏi server 1 lần xem có gì mới không), server sẽ bị quá tải nhanh chóng và ứng dụng sẽ rất hao pin.

### 2. Cách triển khai WebSocket trong Flutter

Trong Flutter, package chính thức và phổ biến nhất do chính team Dart cung cấp là **`web_socket_channel`**.

**Bước 1: Cài đặt package**
Thêm vào file `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  web_socket_channel: ^3.0.0 # Kiểm tra version mới nhất trên pub.dev

```

**Bước 2: Cách sử dụng cơ bản**

Đây là 4 thao tác cốt lõi khi làm việc với WebSocket: Kết nối, Lắng nghe, Gửi đi và Đóng.

```dart
import 'package:flutter/material.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

class WebSocketExample extends StatefulWidget {
  const WebSocketExample({super.key});

  @override
  State<WebSocketExample> createState() => _WebSocketExampleState();
}

class _WebSocketExampleState extends State<WebSocketExample> {
  final TextEditingController _controller = TextEditingController();
  
  // 1. Tạo kết nối tới Server
  // Ở đây dùng server test wss://echo.websocket.events (gửi gì nó trả về nấy)
  late final WebSocketChannel _channel = WebSocketChannel.connect(
    Uri.parse('wss://echo.websocket.events'),
  );

  @override
  void dispose() {
    // 4. Đóng kết nối khi rời khỏi màn hình để giải phóng tài nguyên
    _channel.sink.close();
    _controller.dispose();
    super.dispose();
  }

  // 3. Hàm gửi dữ liệu lên Server
  void _sendMessage() {
    if (_controller.text.isNotEmpty) {
      _channel.sink.add(_controller.text); // Dùng sink.add để gửi
      _controller.clear();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('WebSocket Demo')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          children: [
            TextField(
              controller: _controller,
              decoration: const InputDecoration(labelText: 'Nhập tin nhắn...'),
            ),
            const SizedBox(height: 20),
            
            // 2. Lắng nghe dữ liệu từ Server trả về
            // WebSocketChannel cung cấp một Stream (dòng chảy dữ liệu)
            Expanded(
              child: StreamBuilder(
                stream: _channel.stream,
                builder: (context, snapshot) {
                  // Kiểm tra có dữ liệu không
                  if (snapshot.hasData) {
                    return Text(
                      'Server phản hồi: ${snapshot.data}',
                      style: const TextStyle(fontSize: 18, color: Colors.blue),
                    );
                  }
                  if (snapshot.hasError) {
                    return Text('Lỗi: ${snapshot.error}');
                  }
                  return const Text('Đang chờ tin nhắn...');
                },
              ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _sendMessage,
        child: const Icon(Icons.send),
      ),
    );
  }
}

```

### 3. Những thư viện thay thế (Tùy thuộc Backend)

* **`web_socket_channel`**: Dùng khi backend của bạn chạy WebSocket thuần (Native WebSocket - URL bắt đầu bằng `ws://` hoặc `wss://`).
* **`socket_io_client`**: Nếu backend của bạn (thường là Node.js) đang sử dụng thư viện **Socket.IO**. Lưu ý: Socket.IO *không phải* là WebSocket thuần, nó là một lớp bọc (wrapper) bên ngoài có thêm các tính năng như auto-reconnect, broadcast. Nếu backend dùng Socket.IO, app Flutter **bắt buộc** phải dùng `socket_io_client`, không dùng `web_socket_channel` được.
* **Firebase Realtime Database / Firestore**: Thực chất bên dưới chúng cũng sử dụng cơ chế WebSockets để đồng bộ dữ liệu thời gian thực, nhưng Firebase đã bọc lại thành các SDK rất dễ dùng, bạn không cần phải tự quản lý kết nối.

Bạn đang định làm tính năng gì với WebSocket? Có muốn tôi hướng dẫn cách xử lý việc tự động kết nối lại (auto-reconnect) khi app bị rớt mạng không?

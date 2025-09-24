Chào bạn, tôi rất sẵn lòng giải thích chi tiết về cách sử dụng `CircularProgressIndicator` trong Flutter. Đây là một widget cực kỳ hữu ích để thông báo cho người dùng rằng ứng dụng đang thực hiện một tác vụ nào đó.

### `CircularProgressIndicator` là gì?

`CircularProgressIndicator` là một widget Material Design, hiển thị một vòng tròn quay để cho biết một tiến trình đang diễn ra. Nó có hai chế độ hoạt động chính:

1.  **Indeterminate (Vô định):** Vòng tròn sẽ quay liên tục. Chế độ này được sử dụng khi bạn không biết chính xác khi nào tác vụ sẽ hoàn thành (ví dụ: đang chờ phản hồi từ server). Đây là chế độ mặc định.
2.  **Determinate (Xác định):** Vòng tròn sẽ được lấp đầy dần theo một giá trị tiến trình cụ thể (từ 0.0 đến 1.0). Chế độ này được sử dụng khi bạn biết rõ tiến độ của tác vụ (ví dụ: tải xuống một tệp tin).

---

### 1. Cách sử dụng cơ bản (Chế độ Vô định)

Đây là cách đơn giản nhất để hiển thị một chỉ báo tải. Bạn chỉ cần đặt widget `CircularProgressIndicator` vào cây widget của mình. Thông thường, nó được đặt bên trong một `Center` để căn giữa màn hình.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('CircularProgressIndicator Cơ Bản'),
        ),
        body: const Center(
          child: CircularProgressIndicator(),
        ),
      ),
    );
  }
}
```

Trong ví dụ trên, một vòng tròn sẽ xuất hiện và quay liên tục ở giữa màn hình.

---

### 2. Tùy chỉnh `CircularProgressIndicator`

Bạn có thể tùy chỉnh giao diện của nó thông qua các thuộc tính quan trọng sau:

*   `value`: Giá trị kiểu `double` từ `0.0` đến `1.0`. Nếu thuộc tính này là `null` (mặc định), nó sẽ ở chế độ vô định. Nếu bạn cung cấp một giá trị, nó sẽ chuyển sang chế độ xác định.
*   `color`: Màu của phần vòng tròn đang quay hoặc đang được lấp đầy.
*   `backgroundColor`: Màu của phần nền (phần chưa được lấp đầy) của vòng tròn.
*   `strokeWidth`: Độ dày của nét vẽ vòng tròn. Giá trị mặc định là `4.0`.
*   `strokeAlign`: Căn chỉnh vị trí của nét vẽ so với bán kính. Có thể là `strokeAlignCenter` (mặc định), `strokeAlignInside`, hoặc `strokeAlignOutside`.
*   `valueColor`: Cho phép tùy chỉnh màu sắc nâng cao hơn, thường dùng với `Animation<Color>`. Cách dùng phổ biến là `AlwaysStoppedAnimation<Color>(Colors.red)` để đặt một màu cố định.

**Ví dụ tùy chỉnh:**

```dart
CircularProgressIndicator(
  color: Colors.red,
  backgroundColor: Colors.grey,
  strokeWidth: 8.0,
)
```

---

### 3. Cách sử dụng (Chế độ Xác định)

Để sử dụng chế độ xác định, bạn cần:
1.  Sử dụng một `StatefulWidget` để có thể lưu và cập nhật trạng thái tiến trình.
2.  Tạo một biến để lưu trữ giá trị tiến trình (ví dụ: `double _progress = 0.0;`).
3.  Cập nhật biến này khi tác vụ đang diễn ra và gọi `setState()` để giao diện được vẽ lại với giá trị mới.
4.  Truyền biến này vào thuộc tính `value` của `CircularProgressIndicator`.

Dưới đây là một ví dụ hoàn chỉnh mô phỏng quá trình tải trong 5 giây.

### Ví dụ hoàn chỉnh: Kết hợp cả hai chế độ

Ví dụ này sẽ hiển thị cả hai loại chỉ báo. Khi bạn nhấn nút "Bắt đầu tải", chỉ báo xác định sẽ bắt đầu chạy từ 0% đến 100% và hiển thị cả phần trăm ở giữa.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const ProgressIndicatorDemo(),
    );
  }
}

class ProgressIndicatorDemo extends StatefulWidget {
  const ProgressIndicatorDemo({super.key});

  @override
  State<ProgressIndicatorDemo> createState() => _ProgressIndicatorDemoState();
}

class _ProgressIndicatorDemoState extends State<ProgressIndicatorDemo> {
  bool _isLoading = false;
  double _progress = 0.0;
  Timer? _timer;

  void _startLoading() {
    // Nếu đang tải thì không làm gì cả
    if (_isLoading) return;

    setState(() {
      _isLoading = true;
      _progress = 0.0;
    });

    // Mô phỏng việc tải dữ liệu bằng Timer
    _timer = Timer.periodic(const Duration(milliseconds: 100), (timer) {
      setState(() {
        _progress += 0.02; // Tăng tiến trình lên 2% mỗi 100ms
        if (_progress >= 1.0) {
          _progress = 1.0;
          _timer?.cancel(); // Dừng timer khi hoàn thành
          _isLoading = false;
          // Có thể thêm thông báo hoàn thành ở đây
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(content: Text('Tải xuống hoàn tất!')),
          );
        }
      });
    });
  }

  @override
  void dispose() {
    _timer?.cancel(); // Hủy timer khi widget bị hủy để tránh rò rỉ bộ nhớ
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('CircularProgressIndicator Demo'),
      ),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(20.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const Text(
                '1. Chế độ Vô định (Indeterminate)',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 20),
              // Chỉ báo quay liên tục
              const CircularProgressIndicator(
                strokeWidth: 6.0,
                color: Colors.amber,
              ),
              const SizedBox(height: 50),
              const Text(
                '2. Chế độ Xác định (Determinate)',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 20),
              // Dùng Stack để hiển thị text phần trăm bên trong vòng tròn
              Stack(
                alignment: Alignment.center,
                children: [
                  SizedBox(
                    width: 100,
                    height: 100,
                    child: CircularProgressIndicator(
                      value: _progress, // Giá trị tiến trình
                      strokeWidth: 8.0,
                      backgroundColor: Colors.grey.shade300,
                      color: Colors.blue,
                    ),
                  ),
                  // Hiển thị phần trăm
                  Text(
                    '${(_progress * 100).toInt()}%',
                    style: const TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
              const SizedBox(height: 40),
              ElevatedButton(
                // Vô hiệu hóa nút khi đang tải
                onPressed: _isLoading ? null : _startLoading,
                child: const Text('Bắt đầu tải'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Tóm tắt các điểm chính:

1.  **Sử dụng đơn giản:** Chỉ cần gọi `CircularProgressIndicator()` cho chế độ mặc định (vô định).
2.  **Hai chế độ:** `value: null` (vô định) và `value: 0.0...1.0` (xác định).
3.  **Tùy chỉnh:** Dùng các thuộc tính như `color`, `backgroundColor`, `strokeWidth` để thay đổi giao diện.
4.  **Chế độ xác định:** Cần một `StatefulWidget` để quản lý và cập nhật giá trị tiến trình thông qua `setState()`.
5.  **Kết hợp với `Stack`:** Một kỹ thuật phổ biến là đặt `CircularProgressIndicator` và một `Text` widget bên trong `Stack` để hiển thị phần trăm tiến trình.

Hy vọng giải thích chi tiết này sẽ giúp bạn sử dụng thành thạo `CircularProgressIndicator` trong các dự án Flutter của mình

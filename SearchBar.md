Chào bạn! `SearchBar` là một trong những widget "cool" và hiện đại nhất được giới thiệu trong Material 3 của Flutter. Nó không chỉ là một ô nhập liệu, mà là cả một trải nghiệm tìm kiếm hoàn chỉnh. Hãy cùng nhau "mổ xẻ" nó một cách chi tiết nhé!

### 1. `SearchBar` là gì và tại sao nó "xịn"?

Trước đây, để tạo chức năng tìm kiếm, bạn thường phải tự kết hợp một `TextField`, một `IconButton`, quản lý `FocusNode`, và xây dựng một overlay để hiển thị kết quả. Nó khá phức tạp.

`SearchBar` (và người bạn đồng hành của nó là `SearchAnchor`) ra đời để đóng gói tất cả logic đó vào một widget duy nhất, cung cấp một trải nghiệm tìm kiếm mượt mà, quen thuộc như bạn thấy trong các ứng dụng của Google.

**Trải nghiệm người dùng:**
1.  **Trạng thái nghỉ:** Người dùng thấy một thanh tìm kiếm gọn gàng (widget `SearchBar`).
2.  **Kích hoạt:** Khi nhấn vào, thanh tìm kiếm sẽ mở ra một "chế độ xem tìm kiếm" (search view) mới, thường là toàn màn hình, với bàn phím tự động hiện lên.
3.  **Gợi ý:** Khi người dùng gõ, một danh sách các gợi ý (suggestions) sẽ hiện ra ngay bên dưới.
4.  **Hoàn tất:** Khi người dùng chọn một gợi ý hoặc đóng view, họ sẽ được đưa trở lại màn hình ban đầu.

### 2. Các thành phần cốt lõi: `SearchAnchor` và `SearchBar`

Đây là cặp đôi không thể tách rời:

*   **`SearchAnchor`**: Đây là widget "bộ não", là người quản lý toàn bộ trạng thái. Nó vô hình, nhưng nó điều khiển việc khi nào mở/đóng "chế độ xem tìm kiếm" và xây dựng danh sách gợi ý.
*   **`SearchBar`**: Đây là widget "bộ mặt", là thanh tìm kiếm mà người dùng nhìn thấy và tương tác ban đầu. Nó chỉ là một "cái cò súng" (trigger) để kích hoạt `SearchAnchor`.

Bạn sẽ luôn đặt `SearchBar` bên trong `builder` của `SearchAnchor`.

### 3. Hướng dẫn chi tiết qua ví dụ: Ứng dụng tìm kiếm trái cây

Hãy xây dựng một ứng dụng đơn giản cho phép người dùng tìm kiếm tên một loại trái cây.

#### Bước 1: Chuẩn bị dữ liệu và `StatefulWidget`

Chúng ta cần một danh sách dữ liệu để tìm kiếm và một `StatefulWidget` để quản lý trạng thái.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const SearchBarDemo(),
    );
  }
}

class SearchBarDemo extends StatefulWidget {
  const SearchBarDemo({super.key});

  @override
  State<SearchBarDemo> createState() => _SearchBarDemoState();
}

class _SearchBarDemoState extends State<SearchBarDemo> {
  // Dữ liệu gốc của chúng ta
  final List<String> allFruits = [
    'Apple', 'Banana', 'Orange', 'Mango', 'Pineapple', 'Strawberry',
    'Blueberry', 'Raspberry', 'Grapes', 'Watermelon', 'Kiwi',
  ];

  // Biến để lưu kết quả tìm kiếm được chọn
  String? selectedFruit;

  @override
  Widget build(BuildContext context) {
    // Phần UI sẽ được xây dựng ở bước tiếp theo
    return Scaffold(
      appBar: AppBar(
        title: const Text('Search Bar Demo'),
      ),
      body: // ...
    );
  }
}
```

#### Bước 2: Xây dựng `SearchAnchor` và `SearchBar`

Đây là phần chính. Chúng ta sẽ đặt `SearchAnchor` vào `body` của `Scaffold`.

```dart
// ... bên trong class _SearchBarDemoState

@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: const Text('Search Bar Demo'),
    ),
    body: Padding(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        children: [
          // 1. SEARCHANCHOR: BỘ NÃO ĐIỀU KHIỂN
          SearchAnchor(
            // 2. BUILDER: Xây dựng thanh tìm kiếm ban đầu (SearchBar)
            builder: (BuildContext context, SearchController controller) {
              return SearchBar(
                controller: controller, // Kết nối controller
                hintText: 'Tìm kiếm trái cây...',
                // Bắt sự kiện khi người dùng nhấn vào thanh tìm kiếm
                onTap: () {
                  controller.openView(); // Lệnh mở chế độ xem tìm kiếm
                },
                // Bắt sự kiện khi nội dung thay đổi (tùy chọn)
                onChanged: (_) {
                  controller.openView();
                },
                leading: const Icon(Icons.search),
              );
            },

            // 3. SUGGESTIONSBUILDER: Xây dựng danh sách gợi ý
            suggestionsBuilder:
                (BuildContext context, SearchController controller) {
              // Lấy từ khóa người dùng đang gõ
              final String query = controller.text;

              // Lọc danh sách trái cây dựa trên từ khóa
              final List<String> filteredFruits = allFruits.where((fruit) {
                return fruit.toLowerCase().contains(query.toLowerCase());
              }).toList();

              // Trả về một danh sách các ListTile
              return List.generate(filteredFruits.length, (int index) {
                final String item = filteredFruits[index];
                return ListTile(
                  title: Text(item),
                  // Bắt sự kiện khi người dùng chọn một gợi ý
                  onTap: () {
                    setState(() {
                      // Cập nhật UI với kết quả đã chọn
                      selectedFruit = item;
                      // ĐÓNG chế độ xem tìm kiếm và trả về kết quả
                      controller.closeView(item); 
                    });
                  },
                );
              });
            },
          ),

          const SizedBox(height: 24),

          // Hiển thị kết quả đã chọn
          if (selectedFruit != null)
            Text(
              'Bạn đã chọn: $selectedFruit',
              style: Theme.of(context).textTheme.headlineSmall,
            )
        ],
      ),
    ),
  );
}
```

### Phân tích chi tiết các phần quan trọng

1.  **`SearchAnchor`**:
    *   Nó bao bọc toàn bộ logic tìm kiếm.
    *   **`builder`**: Thuộc tính này yêu cầu một hàm trả về một widget. Widget này chính là `SearchBar` mà người dùng thấy. Hàm này cung cấp một `SearchController` để bạn kết nối.
    *   **`suggestionsBuilder`**: Đây là nơi "phép thuật" xảy ra. Hàm này cũng nhận `SearchController`.
        *   Bạn lấy văn bản người dùng đang gõ qua `controller.text`.
        *   Dựa vào văn bản đó, bạn lọc dữ liệu và trả về một danh sách các widget gợi ý (thường là `ListTile` trong một `ListView` hoặc `List.generate`).
        *   **Quan trọng nhất:** Bên trong `onTap` của mỗi gợi ý, bạn phải gọi `controller.closeView(value)` để đóng chế độ xem tìm kiếm và (tùy chọn) truyền giá trị được chọn trở lại.

2.  **`SearchBar`**:
    *   **`controller`**: Bắt buộc phải gán `controller` nhận được từ `builder` của `SearchAnchor` vào đây. Đây là sợi dây liên kết giữa "bộ mặt" và "bộ não".
    *   **`onTap` / `onChanged`**: Bạn phải gọi `controller.openView()` trong các callback này để ra lệnh cho `SearchAnchor` mở chế độ xem tìm kiếm.
    *   **`leading`, `trailing`**: Dùng để thêm các icon như kính lúp, micro, hoặc nút xóa.

### 4. Tùy chỉnh nâng cao

`SearchAnchor` cung cấp nhiều thuộc tính để bạn tùy chỉnh trải nghiệm:
*   `isFullScreen`: `bool` (mặc định là `true`). Nếu đặt là `false`, chế độ xem tìm kiếm sẽ không chiếm toàn màn hình mà chỉ là một overlay bên dưới thanh tìm kiếm.
*   `searchViewBackgroundColor`: Thay đổi màu nền của chế độ xem tìm kiếm.
*   `searchViewHintText`: Đặt một `hintText` khác cho chế độ xem tìm kiếm so với `SearchBar` ban đầu.
*   `searchViewLeading`, `searchViewTrailing`: Tùy chỉnh các icon trên thanh tìm kiếm khi nó ở trạng thái mở.

### Tóm tắt

1.  Bắt đầu với `SearchAnchor`, widget điều khiển chính.
2.  Trong `builder` của `SearchAnchor`, trả về một `SearchBar` và kết nối `controller` của nó.
3.  Trong `onTap` của `SearchBar`, gọi `controller.openView()`.
4.  Trong `suggestionsBuilder` của `SearchAnchor`, lọc dữ liệu dựa trên `controller.text` và trả về một danh sách các widget gợi ý.
5.  Trong `onTap` của mỗi gợi ý, gọi `controller.closeView(value)` để đóng và trả về kết quả.

Sử dụng cặp đôi `SearchAnchor` và `SearchBar` là cách làm hiện đại, đúng chuẩn Material 3, giúp bạn tạo ra chức năng tìm kiếm mạnh mẽ, mượt mà với ít code và công sức hơn rất nhiều.

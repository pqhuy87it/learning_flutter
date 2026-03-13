Thực ra `Future.wait` **đã hoạt động đúng như bạn muốn rồi** — nó đợi **tất cả** xong mới trả kết quả. Bạn đang hiểu nhầm cơ chế của nó.

## Future.wait đã đợi tất cả

```dart
// Cả 2 request chạy SONG SONG, nhưng dòng code DỪNG LẠI ở đây
// cho đến khi CẢ HAI đều xong
final results = await Future.wait([getUser(), getPosts()]);

// Dòng này CHỈ CHẠY khi cả getUser() VÀ getPosts() đã hoàn thành
final user = results[0];  // User data
final posts = results[1]; // Posts data

// Hiển thị cả 2 cùng lúc
emit(HomeLoaded(user: user, posts: posts));
```

Giả sử `getUser()` mất 1 giây, `getPosts()` mất 3 giây — `await Future.wait` sẽ đợi đủ 3 giây rồi mới chạy tiếp. Không có chuyện cái nào xong thì hiện trước.

## So sánh trực quan

```dart
// ❌ Cái bạn đang TƯỞNG Future.wait làm (nhưng không phải):
// t=0s:  bắt đầu cả 2
// t=1s:  getUser() xong → hiện user ngay
// t=3s:  getPosts() xong → hiện posts

// ✅ Cái Future.wait THỰC SỰ làm:
// t=0s:  bắt đầu cả 2 đồng thời
// t=1s:  getUser() xong → chờ...
// t=3s:  getPosts() xong → BÂY GIỜ mới trả cả 2 kết quả
```

## Vậy khi nào mới xảy ra kiểu "cái nào xong hiện trước"?

Đó là khi bạn **không dùng** `Future.wait`, mà gọi riêng lẻ:

```dart
// Kiểu này MỚI hiện từng cái một
getUser().then((user) => emit(UserLoaded(user)));    // Xong lúc nào hiện lúc đó
getPosts().then((posts) => emit(PostsLoaded(posts))); // Xong lúc nào hiện lúc đó
```

Hoặc dùng `Stream`/`BLoC` pattern để update từng phần UI độc lập — đó là thiết kế có chủ đích, không phải hành vi mặc định của `Future.wait`.

## Tóm lại

| Cách gọi | Hành vi |
|---|---|
| `await Future.wait([a(), b()])` | Đợi **tất cả** xong → trả kết quả 1 lần |
| `await a(); await b();` | Chạy **tuần tự**, a xong rồi mới bắt đầu b |
| `a.then(...); b.then(...)` | Chạy song song, **cái nào xong xử lý cái đó** |

`Future.wait` đã đúng là thứ bạn cần — đợi tất cả rồi mới hiển thị.

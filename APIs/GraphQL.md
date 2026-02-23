**GraphQL** là một ngôn ngữ truy vấn (query language) dành cho API, được Facebook phát triển. Khi kết hợp với **Flutter**, nó mang lại một cách tiếp cận cực kỳ linh hoạt và tối ưu để ứng dụng của bạn giao tiếp với server.

Thay vì dựa vào cấu trúc URL cố định như REST API, GraphQL cho phép ứng dụng Flutter **chỉ yêu cầu chính xác những dữ liệu nó cần**, không thừa và không thiếu.

---

### 1. Tại sao nên dùng GraphQL thay vì REST API trong Flutter?

* **Tránh Over-fetching (Thừa dữ liệu):** Trong REST, nếu bạn gọi API `/users/1`, server có thể trả về 20 trường dữ liệu (tên, tuổi, địa chỉ, sở thích...). Nếu màn hình Flutter của bạn chỉ cần hiển thị `avatar` và `name`, 18 trường còn lại là lãng phí băng thông. Với GraphQL, bạn chỉ cần yêu cầu `avatar` và `name`.
* **Tránh Under-fetching (Thiếu dữ liệu):** Trong REST, để lấy thông tin User và danh sách Bài viết của họ, bạn có thể phải gọi 2 API riêng biệt (`/users/1` và `/users/1/posts`). Với GraphQL, bạn lấy tất cả trong 1 lần gọi duy nhất.
* **Chỉ có một Endpoint duy nhất:** Thay vì quản lý hàng tá URL (`/login`, `/get_posts`, `/update_profile`), GraphQL chỉ giao tiếp qua một URL duy nhất (thường là `https://your-domain.com/graphql`).
* **Type-safe (Định kiểu chặt chẽ):** Schema của GraphQL định nghĩa rõ ràng kiểu dữ liệu (String, Int, Boolean, Custom Object). Điều này giúp việc sinh code (code generation) các class Dart (Model) rất dễ dàng và tránh lỗi runtime.

---

### 2. Các khái niệm cốt lõi của GraphQL

Khi làm việc với GraphQL trong Flutter, bạn sẽ dùng 3 loại thao tác chính:

* **Query:** Dùng để LẤY dữ liệu (tương đương với lệnh GET trong REST).
* **Mutation:** Dùng để THÊM, SỬA, XÓA dữ liệu (tương đương POST, PUT, DELETE trong REST).
* **Subscription:** Dùng để nhận dữ liệu theo thời gian thực (Real-time) qua WebSockets (giống như Firebase Realtime/Firestore listen).

---

### 3. Các thư viện (Packages) phổ biến cho Flutter

Để sử dụng GraphQL trong Flutter, bạn không cần phải tự viết code gọi HTTP thủ công. Cộng đồng đã xây dựng sẵn các package rất mạnh mẽ:

1. **`graphql_flutter` (Phổ biến nhất):** Cung cấp sẵn các Widget như `Query`, `Mutation`, `Subscription` để bạn nhúng trực tiếp vào giao diện. Hỗ trợ caching (lưu trữ tạm) rất tốt.
2. **`ferry`:** Một GraphQL Client cực kỳ mạnh mẽ, tập trung vào caching nâng cao và khả năng offline (dùng khi mất mạng). Phù hợp cho các app lớn, phức tạp.
3. **`artemis`:** Công cụ giúp tự động sinh code (auto-generate) các class Dart từ file cấu hình GraphQL Schema, giúp bạn code an toàn (type-safe) 100%.

---

### 4. Cách hoạt động cơ bản (Ví dụ với `graphql_flutter`)

Quy trình tích hợp thường gồm 3 bước:

**Bước 1: Khởi tạo Client và bọc App lại**
Bạn tạo một `GraphQLClient` chứa link API của bạn, sau đó dùng `GraphQLProvider` bọc bên ngoài `MaterialApp`.

```dart
final HttpLink httpLink = HttpLink('https://api.github.com/graphql');

final ValueNotifier<GraphQLClient> client = ValueNotifier(
  GraphQLClient(
    link: httpLink,
    cache: GraphQLCache(store: HiveStore()), // Caching
  ),
);

// Bọc toàn bộ app
GraphQLProvider(
  client: client,
  child: MyApp(),
);

```

**Bước 2: Viết câu Query**
Thay vì viết URL, bạn viết một chuỗi (String) GraphQL.

```dart
String readRepositories = """
  query ReadRepositories(\$nRepositories: Int!) {
    viewer {
      repositories(last: \$nRepositories) {
        nodes {
          id
          name
        }
      }
    }
  }
""";

```

**Bước 3: Dùng Widget để gọi và hiển thị giao diện**

```dart
Query(
  options: QueryOptions(
    document: gql(readRepositories),
    variables: const {'nRepositories': 50},
  ),
  builder: (QueryResult result, { VoidCallback? refetch, FetchMore? fetchMore }) {
    if (result.hasException) return Text(result.exception.toString());
    if (result.isLoading) return CircularProgressIndicator();

    // Dữ liệu trả về ở định dạng Map/JSON
    List repositories = result.data?['viewer']['repositories']['nodes'];
    
    return ListView.builder(
      itemCount: repositories.length,
      itemBuilder: (context, index) => Text(repositories[index]['name']),
    );
  },
);

```

---

Bạn có muốn tôi hướng dẫn chi tiết cách cài đặt và cấu hình package **`graphql_flutter`** từ đầu cho dự án hiện tại của bạn không?

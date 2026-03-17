Để minh họa thêm về sức mạnh của Clean Architecture trong dự án này, tôi sẽ vẽ cho bạn một luồng dữ liệu (Data Flow) rất thú vị và phức tạp hơn một chút: **Luồng Thêm Sản Phẩm Vào Giỏ Hàng (Add To Cart Flow)**. 

Luồng này đặc biệt ở chỗ nó có xử lý **logic rẽ nhánh (Offline-first / Guest Mode)** ngay tại Data Layer: 
- Nếu người dùng chưa đăng nhập (không có token) hoặc mất mạng $\rightarrow$ Chỉ lưu giỏ hàng ở bộ nhớ tạm cục bộ (Local).
- Nếu người dùng đã đăng nhập và có mạng $\rightarrow$ Đẩy lên Server (API) rồi mới lưu vào bộ nhớ cục bộ.

Dưới đây là sơ đồ Mermaid mô tả chi tiết luồng này:

```mermaid
graph TD
    %% --- Lớp Presentation (Giao diện & State) ---
    subgraph PRESENTATION ["Presentation Layer (UI & State)"]
        View["ProductDetailsView (UI)"]
        Event("AddProduct (CartEvent)")
        Bloc["CartBloc (BLoC)"]
        State("CartLoaded (CartState)")
    end

    %% --- Lớp Domain (Logic nghiệp vụ) ---
    subgraph DOMAIN ["Domain Layer (Business Logic)"]
        UseCase["AddCartUseCase"]
        DomainRepo["CartRepository (Interface)"]
        Entity("CartItem (Domain Entity)")
    end

    %% --- Lớp Data (Dữ liệu & Nguồn) ---
    subgraph DATA ["Data Layer (Repository Impl & Data Sources)"]
        RepoImpl["CartRepositoryImpl"]
        UserLocalDS["UserLocalDataSource (Get Token)"]
        AuthCheck{"Has Token & Internet?"}
        
        RemoteDS["CartRemoteDataSourceImpl"]
        LocalDS["CartLocalDataSourceImpl"]
        
        API[("REST API (/carts)")]
        Cache[("SharedPreferences (Local Cart)")]
    end

    %% --- Luồng Yêu cầu (Request Flow) ---
    View -.->|"1. User clicks 'Add to Cart'"| Event
    Event -.->|"2. add(AddProduct(cartItem))"| Bloc
    
    Bloc -->|"3. call(cartItem)"| UseCase
    UseCase -->|"4. addCartItem(cartItem)"| DomainRepo
    DomainRepo -.->|"5. implements"| RepoImpl
    
    RepoImpl -->|"6. getToken()"| UserLocalDS
    RepoImpl -->|"7. isConnected?"| AuthCheck
    
    %% --- Nhánh 1: Online & Có Token (Đã đăng nhập) ---
    AuthCheck -->|"8a. True (Online & Có Token)"| RemoteDS
    RemoteDS -->|"9a. http.post(/carts)"| API
    API --x|"10a. Return CartItem JSON"| RemoteDS
    RemoteDS --x|"11a. Return CartItemModel"| RepoImpl
    RepoImpl -->|"12a. saveCartItem(Remote Data)"| LocalDS
    LocalDS -->|"13a. SharedPreferences.setString()"| Cache
    
    %% --- Nhánh 2: Offline hoặc Chưa Đăng Nhập (Guest Mode) ---
    AuthCheck .->|"8b. False (Offline / No Token)"| LocalDS
    LocalDS .->|"9b. saveCartItem() locally"| Cache
    
    %% --- Luồng Trả Về (Return Flow) ---
    Cache --x|"14. Success"| LocalDS
    LocalDS --x|"15. Return Right(CartItem)"| RepoImpl
    
    RepoImpl --x|"16. Return Right(CartItem)"| UseCase
    UseCase --x|"17. Return Either<Failure, CartItem>"| Bloc
    
    Bloc --x|"18. emit(CartLoaded(newCartList))"| State
    State --x|"19. Update UI (Cart Badge, SnackBar)"| View

    %% Styling
    style View fill:#ff9,stroke:#333,stroke-width:2px
    style Bloc fill:#ccf,stroke:#333,stroke-width:2px
    style UseCase fill:#eff,stroke:#333,stroke-width:2px
    style RepoImpl fill:#efe,stroke:#333,stroke-width:2px
    style RemoteDS fill:#efe,stroke:#333
    style LocalDS fill:#efe,stroke:#333
    style UserLocalDS fill:#efe,stroke:#333
    style AuthCheck fill:#fdf,stroke:#333
    style API fill:#eee,stroke:#333,stroke-dasharray: 5 5
    style Cache fill:#eee,stroke:#333,stroke-dasharray: 5 5
```

### 📝 Phân tích chi tiết các bước trong Data Flow:

**1. Hành động từ UI (Presentation Layer)**
* Người dùng ở màn hình `ProductDetailsView`, chọn Price Tag và bấm nút "Add to Cart".
* Nút này gọi sự kiện: `context.read<CartBloc>().add(AddProduct(cartItem: ...))`.
* `CartBloc` nhận Event `AddProduct`, lấy danh sách giỏ hàng hiện tại, sau đó gọi `_addCartUseCase(event.cartItem)`.

**2. Qua cầu nối nghiệp vụ (Domain Layer)**
* `AddCartUseCase` nhận tham số `CartItem`. Nó không quan tâm dữ liệu sẽ được lưu thế nào, nó chỉ đơn giản ra lệnh cho `CartRepository` thực thi hàm `addCartItem()`.

**3. Xử lý Logic thông minh tại Data Layer**
Tại `CartRepositoryImpl`, logic của Clean Architecture tỏa sáng:
* Nó gọi `UserLocalDataSource` để lấy Token (kiểm tra xem đã login chưa).
* Kiểm tra `NetworkInfo.isConnected` (xem có mạng không).
* **Rẽ nhánh 8a (Online & Authenticated):** Nếu có mạng VÀ có Token hợp lệ, nó gọi `CartRemoteDataSourceImpl` để bắn API `POST /carts`. Khi API lưu thành công trên Server và trả kết quả về, nó lấy Model đó đưa cho `CartLocalDataSource` để lưu tiếp vào Local (cho việc dùng offline).
* **Rẽ nhánh 8b (Guest Mode / Offline):** Nếu không có mạng HOẶC chưa đăng nhập (token rỗng), code bỏ qua việc gọi API, trực tiếp dùng `CartLocalDataSourceImpl` để lưu sản phẩm vào `SharedPreferences` (sẽ đồng bộ lên server ở lần đăng nhập tiếp theo).

**4. Luồng trả kết quả (Return Flow)**
* Dù rẽ vào nhánh nào, `CartRepositoryImpl` đều gom kết quả thành công vào `Right(CartItem)` (sử dụng thư viện Dartz).
* Kết quả truyền ngược lại qua `AddCartUseCase` về `CartBloc`.
* `CartBloc` gộp `CartItem` mới vào mảng `cart` hiện tại và emit ra State `CartLoaded(cart: newCartList)`.
* Giao diện nhận thấy sự thay đổi State sẽ update UI (Ví dụ: Số lượng badge trên giỏ hàng ở thanh Navigation tăng lên).

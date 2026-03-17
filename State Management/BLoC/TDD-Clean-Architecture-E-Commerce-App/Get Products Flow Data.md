Dựa trên source code của dự án **Flutter-TDD-Clean-Architecture-E-Commerce-App** mà bạn cung cấp, kiến trúc được sử dụng là Clean Architecture kết hợp với BLoC làm State Management. 

Để minh họa rõ nhất luồng dữ liệu (Data Flow), tôi sẽ lấy ví dụ về **Luồng lấy danh sách sản phẩm (Get Products Flow)** (từ `HomeView` -> `ProductBloc` -> `GetProductUseCase` -> API/Cache), vì đây là luồng cốt lõi và thể hiện đầy đủ nhất các thành phần của kiến trúc này.

Dưới đây là sơ đồ Mermaid mô tả chính xác Data Flow của dự án:

```mermaid
graph TD
    %% --- Lớp Presentation (Giao diện & State) ---
    subgraph PRESENTATION ["Presentation Layer (UI & State Management)"]
        View["HomeView (UI)"]
        Event("GetProducts (ProductEvent)")
        Bloc["ProductBloc (BLoC)"]
        State("ProductLoaded / Loading / Error (ProductState)")
    end

    %% --- Lớp Domain (Logic nghiệp vụ) ---
    subgraph DOMAIN ["Domain Layer (Business Logic)"]
        UseCase["GetProductUseCase"]
        DomainRepo["ProductRepository (Interface)"]
        Entity("ProductResponse / Product (Domain Entity)")
    end

    %% --- Lớp Data (Dữ liệu & Nguồn) ---
    subgraph DATA ["Data Layer (Repository Impl & Data Sources)"]
        RepoImpl["ProductRepositoryImpl"]
        NetworkInfo{"NetworkInfo (Check Internet)"}
        RemoteDS["ProductRemoteDataSourceImpl"]
        LocalDS["ProductLocalDataSourceImpl"]
        Mapper("ProductModel (Model & Mapper)")
        API[("REST API (http.Client)")]
        Cache[("SharedPreferences (Local Cache)")]
    end

    %% --- Luồng Gọi Dữ Liệu (Request Flow) ---
    View -.->|"1. dispatch event"| Event
    Event -.->|"2. add(GetProducts)"| Bloc
    
    Bloc -->|"3. call(params)"| UseCase
    UseCase -->|"4. getRemoteProducts()"| DomainRepo
    DomainRepo -.->|"5. implements"| RepoImpl
    
    RepoImpl -->|"6. isConnected?"| NetworkInfo
    NetworkInfo -->|"7. True (Online)"| RemoteDS
    RemoteDS -->|"8. http.get(.../products)"| API
    
    %% --- Luồng Trả Về & Caching (Return & Cache Flow) ---
    API --x|"9. Return JSON"| RemoteDS
    RemoteDS --x|"10. Return ProductResponseModel"| RepoImpl
    
    RepoImpl -->|"11. saveProducts(Remote Data)"| LocalDS
    LocalDS -->|"12. SharedPreferences.setString()"| Cache
    
    RepoImpl --x|"13. Convert Model -> Entity (Implicit)"| Mapper
    Mapper --x|"14. Return Domain Entity"| RepoImpl
    
    RepoImpl --x|"15. Return Right(ProductResponse)"| UseCase
    UseCase --x|"16. Return Either<Failure, ProductResponse>"| Bloc
    
    Bloc --x|"17. emit(ProductLoaded)"| State
    State --x|"18. BlocBuilder rebuilds UI"| View

    %% --- Luồng phụ khi Offline (Fallback / Get Local) ---
    NetworkInfo .->|"19. False (Offline / Error)"| RepoImpl
    RepoImpl .->|"20. getLocalProducts()"| LocalDS
    LocalDS .->|"21. Read JSON"| Cache

    %% Styling
    style View fill:#ff9,stroke:#333,stroke-width:2px
    style Bloc fill:#ccf,stroke:#333,stroke-width:2px
    style UseCase fill:#eff,stroke:#333,stroke-width:2px
    style RepoImpl fill:#efe,stroke:#333,stroke-width:2px
    style RemoteDS fill:#efe,stroke:#333
    style LocalDS fill:#efe,stroke:#333
    style NetworkInfo fill:#fdf,stroke:#333
    style API fill:#eee,stroke:#333,stroke-dasharray: 5 5
    style Cache fill:#eee,stroke:#333,stroke-dasharray: 5 5
```

### Giải thích chi tiết các bước trong Flow:

**1. Presentation Layer (Luồng yêu cầu):**
* `HomeView` sử dụng `BlocBuilder`. Khi user mở app hoặc pull-to-refresh, UI gọi `context.read<ProductBloc>().add(GetProducts(...))`.
* `ProductBloc` nhận Event, lập tức emit state `ProductLoading` ra UI và gọi hàm thực thi của UseCase.

**2. Domain Layer (Xử lý logic nghiệp vụ):**
* `GetProductUseCase` nhận `FilterProductParams` (keyword, category, minPrice...). Nó đóng vai trò cầu nối, gọi hàm `getRemoteProducts()` từ interface `ProductRepository`.
* Domain Layer hoàn toàn **không biết** dữ liệu lấy từ API hay Database cục bộ.

**3. Data Layer (Lấy dữ liệu và Caching):**
* `ProductRepositoryImpl` (được tiêm qua Dependency Injection - `get_it`) thực thi gọi dữ liệu.
* Bước đầu tiên nó làm là dùng `NetworkInfo` kiểm tra xem thiết bị có internet không.
* **Nếu có mạng (Online):** * Gọi `ProductRemoteDataSourceImpl` để bắn request HTTP GET tới REST API bằng `http.Client`.
  * Khi API trả về JSON, RemoteDataSource map nó thành `ProductResponseModel`.
  * Gửi trả Model về cho `ProductRepositoryImpl`.
  * `ProductRepositoryImpl` sẽ gọi `ProductLocalDataSourceImpl` để lưu (cache) đoạn dữ liệu này vào `SharedPreferences` cho các lần dùng offline sau này.
* **Nếu mất mạng (Offline / Error):** * (Trong một số context như get local/cart/categories, flow sẽ chẽ nhánh lấy dữ liệu cache từ `SharedPreferences`).

**4. Luồng trả về (Return Flow lên UI):**
* Dữ liệu nhận được là các Model (nằm ở lớp Data), nó sẽ được ngầm định coi như là Entity (do `ProductModel extends Product`) hoặc được map sang Domain Entity (`Product`).
* Repository bọc dữ liệu thành công trong đối tượng `Right` của package `dartz` (Ví dụ: `Right(ProductResponse)`). Nếu lỗi sẽ bọc trong `Left(Failure)`.
* Trả ngược kết quả về `ProductBloc`.
* `ProductBloc` kiểm tra kết quả (fold Either), emit ra `ProductLoaded` kèm theo danh sách sản phẩm.
* `BlocBuilder` trên `HomeView` nhận State mới và tiến hành Rebuild lại UI, vẽ các `ProductCard` ra màn hình.

Dựa vào cấu trúc mã nguồn của dự án **Pixiv Artvier** mà bạn đã cung cấp (đặc biệt là các tệp trong `pages/login`, `api_app/oauth.dart`, `global/provider/current_account_provider.dart` và `preferences/account_storage.dart`), tôi đã điều chỉnh sơ đồ luồng dữ liệu (Data Flow) cho phù hợp với kiến trúc thực tế của ứng dụng này. 

Dự án này sử dụng mô hình **Riverpod** (quản lý state) kết hợp với cấu trúc phân lớp chức năng rõ ràng. Dưới đây là sơ đồ luồng đăng nhập (đặc biệt là qua Refresh Token hoặc Web Login):

```mermaid
graph TD
    %% --- Lớp Presentation (Giao diện & State) ---
    subgraph PRESENTATION ["Presentation Layer (UI & State)"]
        View["LoginWizardPage / TokenLoginDialog (UI)"]
        Event("Submit Login (Action)")
        AccountProvider["CurrentAccountProvider (Riverpod)"]
        State("AccountProfileState")
    end

    %% --- Lớp Domain/Logic (Logic nghiệp vụ) ---
    subgraph DOMAIN ["Service Layer (Business Logic)"]
        OAuthApi["OAuthApi (api_app/oauth.dart)"]
        Entity("AccountProfile (Freezed Model)")
    end

    %% --- Lớp Data (Dữ liệu & Nguồn) ---
    subgraph DATA ["Data Layer (Network & Storage)"]
        HttpReq["HttpRequester (request/http_requester.dart)"]
        AccountStore["AccountStorage (preferences/account_storage.dart)"]
        
        API[("Pixiv OAuth API (/auth/token)")]
        Cache[("SharedPreferences (Local Keys)")]
    end

    %% --- Luồng Yêu cầu (Request Flow) ---
    View -.->|"1. User inputs Refresh Token\nhoặc Login qua Web"| Event
    Event -.->|"2. Gọi hàm login(token)"| OAuthApi
    
    OAuthApi -->|"3. Gửi request lấy AccessToken"| HttpReq
    HttpReq -->|"4. HTTPS POST (kèm Client Hash)"| API
    
    %% --- Luồng Trả Về & Xử lý (Return Flow) ---
    API --x|"5. Trả về JSON (AccessToken,\n RefreshToken, UserInfo)"| HttpReq
    HttpReq --x|"6. Parse JSON thành Model"| Entity
    
    Entity --x|"7. Trả đối tượng AccountProfile"| OAuthApi
    
    %% --- Lưu trữ cục bộ ---
    OAuthApi -->|"8. Lưu Token & User Info"| AccountStore
    AccountStore -->|"9. setString(token)"| Cache
    Cache --x|"10. Confirm Saved"| AccountStore
    AccountStore --x|"11. Success"| OAuthApi
    
    %% --- Cập nhật State ---
    OAuthApi --x|"12. Trả về kết quả Login (Success)"| Event
    Event -->|"13. Cập nhật thông tin User"| AccountProvider
    
    AccountProvider --x|"14. state = new AccountProfile"| State
    State --x|"15. Rebuild UI, Điều hướng sang\nMainNavigationTabPage"| View

    %% Styling
    style View fill:#ff9,stroke:#333,stroke-width:2px
    style AccountProvider fill:#ccf,stroke:#333,stroke-width:2px
    style OAuthApi fill:#eff,stroke:#333,stroke-width:2px
    style HttpReq fill:#efe,stroke:#333,stroke-width:2px
    style AccountStore fill:#efe,stroke:#333
    style Entity fill:#fdf,stroke:#333
    style API fill:#eee,stroke:#333,stroke-dasharray: 5 5
    style Cache fill:#eee,stroke:#333,stroke-dasharray: 5 5
```

### Giải thích ánh xạ với Source Code của bạn:
1. **Lớp Presentation**: Giao diện (ví dụ `login_wizard_page.dart` hoặc `token_login.dart`) nhận tương tác của người dùng. Trạng thái sau khi đăng nhập được quản lý bởi `current_account_provider.dart` (biến toàn cục lưu trữ người dùng hiện tại).
2. **Lớp Service (Domain)**: Thay vì BLoC/UseCase, project này gọi thẳng các API thông qua file `oauth.dart` nằm trong `api_app`. Dữ liệu trả về sẽ được map vào model có sẵn như `account_profile.dart`.
3. **Lớp Data**: Tầng mạng gọi qua file `http_requester.dart` (cấu hình sẵn interceptors), và dữ liệu sau khi nhận về (AccessToken, RefreshToken) sẽ được lưu xuống ổ cứng nội bộ thông qua `account_storage.dart` (sử dụng Shared Preferences).

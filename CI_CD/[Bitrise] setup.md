Có, bạn hoàn toàn có thể cấu hình và quản lý builds trên Bitrise từ terminal macOS bằng **Bitrise CLI**. Có hai cách chính:

## 1. Bitrise CLI (chạy local)

Cài đặt qua Homebrew:

```bash
brew install bitrise
```

Sau đó bạn có thể chạy workflow ngay trên máy local:

```bash
bitrise run <workflow_name>
```

File cấu hình chính là `bitrise.yml` trong project của bạn. Bạn có thể tạo, chỉnh sửa file này rồi chạy thử trước khi push lên Bitrise.io.

## 2. Bitrise API (điều khiển builds trên server Bitrise.io)

Bạn dùng API token để trigger và quản lý builds từ xa. Lấy API token tại **Account Settings > Security** trên bitrise.io.

**Trigger một build:**

```bash
curl -X POST "https://api.bitrise.io/v0.1/apps/YOUR_APP_SLUG/builds" \
  -H "Authorization: token YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "hook_info": { "type": "bitrise" },
    "build_params": {
      "branch": "main",
      "workflow_id": "primary"
    }
  }'
```

**Xem danh sách builds:**

```bash
curl -H "Authorization: token YOUR_API_TOKEN" \
  "https://api.bitrise.io/v0.1/apps/YOUR_APP_SLUG/builds"
```

**Xem danh sách apps:**

```bash
curl -H "Authorization: token YOUR_API_TOKEN" \
  "https://api.bitrise.io/v0.1/apps"
```

## Tóm lại

Bitrise CLI phù hợp khi bạn muốn chạy và debug workflow trên máy local. Bitrise API phù hợp khi bạn muốn trigger build, kiểm tra trạng thái, hoặc tích hợp vào script tự động mà vẫn build trên server của Bitrise.io. Hai cách này có thể kết hợp với nhau — bạn test local bằng CLI, rồi dùng API để trigger build chính thức trên cloud.

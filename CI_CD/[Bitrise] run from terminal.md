Hoàn toàn được. Bạn chỉ cần làm 3 bước setup một lần, sau đó mỗi lần build chỉ cần gõ 1 lệnh.

## Bước 1: Lấy API Token

Vào **https://app.bitrise.io** → click avatar góc phải → **Account Settings** → **Security** → mục **Personal Access Token** → tạo token mới và copy lại.

## Bước 2: Lấy App Slug

App slug là ID của project. Chạy lệnh này để liệt kê tất cả app bạn có quyền truy cập:

```bash
curl -s -H "Authorization: token YOUR_API_TOKEN" \
  "https://api.bitrise.io/v0.1/apps" | python3 -m json.tool
```

Kết quả sẽ trả về danh sách app, mỗi app có dạng:

```json
{
  "slug": "abc123def456",
  "title": "MyApp-iOS",
  ...
}
```

Copy lại giá trị `slug` của project bạn cần build.

## Bước 3: Trigger build

```bash
curl -X POST "https://api.bitrise.io/v0.1/apps/APP_SLUG/builds" \
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

Thay `APP_SLUG`, `YOUR_API_TOKEN`, `branch` và `workflow_id` cho đúng project của bạn.

---

## Gợi ý: Tạo script để dùng lại cho tiện

Tạo file `~/bitrise-build.sh`:

```bash
#!/bin/bash

API_TOKEN="your_personal_access_token"
APP_SLUG="your_app_slug"
BRANCH="${1:-main}"
WORKFLOW="${2:-primary}"

echo "🚀 Triggering build..."
echo "   Branch:   $BRANCH"
echo "   Workflow: $WORKFLOW"

RESPONSE=$(curl -s -X POST "https://api.bitrise.io/v0.1/apps/$APP_SLUG/builds" \
  -H "Authorization: token $API_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"hook_info\": { \"type\": \"bitrise\" },
    \"build_params\": {
      \"branch\": \"$BRANCH\",
      \"workflow_id\": \"$WORKFLOW\"
    }
  }")

BUILD_URL=$(echo "$RESPONSE" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('build_url',''))" 2>/dev/null)
STATUS=$(echo "$RESPONSE" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('status','unknown'))" 2>/dev/null)

if [ -n "$BUILD_URL" ]; then
  echo "✅ Build triggered!"
  echo "   URL: $BUILD_URL"
else
  echo "❌ Failed to trigger build"
  echo "$RESPONSE" | python3 -m json.tool
fi
```

Cấp quyền chạy:

```bash
chmod +x ~/bitrise-build.sh
```

Sau đó mỗi lần cần build chỉ cần gõ:

```bash
# Build branch main, workflow primary (mặc định)
~/bitrise-build.sh

# Build branch develop, workflow staging
~/bitrise-build.sh develop staging
```

---

## Xem danh sách workflow có sẵn

Nếu bạn không nhớ tên workflow, chạy:

```bash
curl -s -H "Authorization: token YOUR_API_TOKEN" \
  "https://api.bitrise.io/v0.1/apps/APP_SLUG/build-workflows" | python3 -m json.tool
```

Vậy là bạn không cần mở trình duyệt nữa, mọi thứ làm từ terminal.

# Kết nối Claude Code với Workshop

## Chỉ dẫn cho agent setup

Thực hiện trong **project đang mở**, không cài lại server và không sửa global config (`~/.claude/settings.json`, `~/.claude.json`).

1. Merge cấu hình bên dưới vào `.claude/settings.local.json` và `.mcp.json`; giữ nguyên các key/MCP khác. Cài default model và các alias đúng JSON mẫu đã được người dùng yêu cầu; không tự chọn model khác, nới quyền, commit hoặc push.
2. Để `YOUR_WORKSHOP_TOKEN` ở các trường token chưa được cấu hình; người dùng tự điền sau khi setup. Không hỏi/yêu cầu token hay đường dẫn file token, không tự tìm token từ môi trường, credential hoặc lịch sử.
3. Bảo đảm `.claude/settings.local.json` không được Git track và được ignore; dùng `.git/info/exclude` nếu cần. Nếu file đã được track, không ghi secret hoặc tự untrack; báo người dùng cần giữ file riêng trước khi tự điền token. Đặt quyền `0600` khi hệ điều hành hỗ trợ. Không in token ra terminal/chat/log.
4. `.mcp.json` chỉ chứa `${WORKSHOP_TOKEN}`, không chứa token thật. Token được khai báo trong `env` của settings local và áp dụng sau khi người dùng trust workspace. Không dùng `${ANTHROPIC_AUTH_TOKEN}` trong MCP header: Claude Code cố ý không expand biến credential này cho remote MCP.
5. Parse JSON để kiểm tra cấu hình; không gọi endpoint cần xác thực khi token vẫn là placeholder, không gửi prompt/gọi model. Không tự bỏ qua workspace trust hay xác nhận cấp quyền.
6. Báo file đã thay đổi, nhắc người dùng tự điền token rồi khởi động lại Claude Code. Không coi việc chưa điền token là blocker của bước tạo cấu hình; không tuyên bố đã kết nối nếu chưa kiểm tra thực tế.

## Endpoint

| Dịch vụ | URL |
| --- | --- |
| AI cho Claude Code — Anthropic-compatible | `https://workshop-ai.vihn.dev` |
| AI cho OpenAI-compatible client | `https://workshop-ai.vihn.dev/v1` |
| MCP Workshop | `https://workshop-mcp.vihn.dev/mcp` |

Sau khi setup, người dùng tự thay `YOUR_WORKSHOP_TOKEN` bằng token riêng có quyền `ai` và `mcp`; agent không hỏi hoặc yêu cầu cung cấp token. Không commit hoặc chia sẻ token với người khác.

## 1. AI: merge vào `.claude/settings.local.json` của project

Đây là settings local theo project, không phải `~/.claude/settings.json`. Giữ nguyên các thiết lập khác. Base URL cho Claude Code **không có `/v1` ở cuối**.

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://workshop-ai.vihn.dev",
    "ANTHROPIC_AUTH_TOKEN": "YOUR_WORKSHOP_TOKEN",
    "ANTHROPIC_API_KEY": "",
    "WORKSHOP_TOKEN": "YOUR_WORKSHOP_TOKEN",
    "ANTHROPIC_MODEL": "opus[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "opus[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "sonnet[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "haiku[1m]"
  }
}
```

Sau khi setup, người dùng tự điền cùng token riêng ở hai giá trị `YOUR_WORKSHOP_TOKEN`. `ANTHROPIC_API_KEY: ""` vô hiệu hóa API key cũ cho project này, tránh gửi đồng thời `Authorization` và `x-api-key`. Không sửa credential global. Managed settings hoặc môi trường do launcher cấp có thể ghi đè cấu hình; không bypass chúng.

## 2. MCP: tạo `.mcp.json` ở thư mục gốc dự án

**Không đặt `mcpServers` vào `settings.json`.** Claude Code dùng `.mcp.json` cho MCP theo dự án; MCP dùng chung mọi dự án nằm trong `~/.claude.json`.

```json
{
  "mcpServers": {
    "workshop": {
      "type": "http",
      "url": "https://workshop-mcp.vihn.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${WORKSHOP_TOKEN}"
      }
    }
  }
}
```

Nếu `.mcp.json` đã tồn tại, chỉ merge entry `workshop` vào `mcpServers`. File này không chứa secret nên có thể chia sẻ; `.claude/settings.local.json` chứa token thật phải giữ private và không được commit.

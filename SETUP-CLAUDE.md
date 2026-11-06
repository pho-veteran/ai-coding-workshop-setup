# Kết nối Claude Code với Workshop

## Chỉ dẫn cho agent setup

Thực hiện trong **project đang mở**, không cài lại server và không sửa global config (`~/.claude/settings.json`, `~/.claude.json`).

1. Merge cấu hình bên dưới vào `.claude/settings.local.json` và `.mcp.json`; giữ nguyên các key/MCP khác. Cài default model và các alias đúng JSON mẫu đã được người dùng yêu cầu; không tự chọn model khác, nới quyền, commit hoặc push.
2. Lấy token từ biến môi trường `WORKSHOP_TOKEN` được cung cấp rõ ràng hoặc file token riêng do người dùng chỉ định. Nếu thiếu, hỏi đường dẫn file; không yêu cầu dán token vào chat, không quét credential/history.
3. Trước khi ghi token, bảo đảm `.claude/settings.local.json` không được Git track và được ignore; dùng `.git/info/exclude` nếu cần. Nếu file đã được track, dừng bước ghi secret và hỏi cách lưu riêng, không tự untrack file. Đặt quyền `0600` khi hệ điều hành hỗ trợ. Không in token ra terminal/chat/log.
4. `.mcp.json` chỉ chứa `${WORKSHOP_TOKEN}`, không chứa token thật. Token được khai báo trong `env` của settings local và áp dụng sau khi người dùng trust workspace. Không dùng `${ANTHROPIC_AUTH_TOKEN}` trong MCP header: Claude Code cố ý không expand biến credential này cho remote MCP.
5. Parse JSON, kiểm tra AI `/v1/models` và MCP initialize/tools-list bằng token, **không gửi prompt/gọi model**. Chỉ kiểm tra server `workshop`, không khởi chạy MCP khác. Không tự bỏ qua workspace trust hay xác nhận cấp quyền.
6. Báo file đã thay đổi, kết quả thực tế và việc cần khởi động lại Claude Code. Nếu thiếu token, bị managed settings/launch environment ghi đè hoặc chờ approval, báo đúng blocker; không tuyên bố đã kết nối.

## Endpoint

| Dịch vụ | URL |
| --- | --- |
| AI cho Claude Code — Anthropic-compatible | `https://workshop-ai.vihn.dev` |
| AI cho OpenAI-compatible client | `https://workshop-ai.vihn.dev/v1` |
| MCP Workshop | `https://workshop-mcp.vihn.dev/mcp` |

Thay `YOUR_WORKSHOP_TOKEN` trong các mẫu dưới đây bằng token riêng được cấp cho bạn, có quyền `ai` và `mcp`. Không commit hoặc chia sẻ token với người khác.

### Token lấy ở đâu?

Token được operator cấp riêng, không nằm trên GitHub và không có API public để tải token. Người tham gia nhận file token có quyền `ai` và `mcp` qua kênh riêng từ operator.

Trên máy host, token operator hiện có ở `.private/public-host/operator.token` tính từ thư mục repo `ai-coding-workshop`, quyền `0600`. Chủ máy có thể chỉ định file này khi setup trên chính máy host. Không dùng chung token operator cho người tham gia, không đưa file hoặc giá trị token lên repo public. Trên máy khác, dùng file token riêng được cấp hoặc biến `WORKSHOP_TOKEN`; đường dẫn trên host không tự tồn tại trên máy client.

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

Dùng cùng token riêng ở hai giá trị `YOUR_WORKSHOP_TOKEN`. `ANTHROPIC_API_KEY: ""` vô hiệu hóa API key cũ cho project này, tránh gửi đồng thời `Authorization` và `x-api-key`. Không sửa credential global. Managed settings hoặc môi trường do launcher cấp có thể ghi đè cấu hình; không bypass chúng.

### Model alias đã cấu hình sẵn

Bốn biến model trong JSON được lấy từ `~/.claude/settings.json` của máy host hiện tại. Mặc định cho phiên mới là `opus[1m]`; `/model opus`, `/model sonnet`, `/model haiku` dùng các override tương ứng. Giữ nguyên các alias gateway này, không tự thay bằng model ID Anthropic hoặc model của Codex.

Máy host còn lưu `"model": "sonnet"` ở settings, nhưng `ANTHROPIC_MODEL` có ưu tiên cao hơn trường `model`, nên mẫu chỉ đặt mặc định qua biến môi trường. `--model` hoặc lựa chọn trong phiên có thể ghi đè mặc định này. Không suy ra model của phiên đang chạy chỉ từ settings.

Catalog upstream có các alias gốc `opus`, `sonnet`, `haiku`, không liệt kê các giá trị có hậu tố `[1m]`. Hậu tố được giữ theo cấu hình máy host; chưa gọi model Claude để xác minh inference hoặc dung lượng context thực tế.

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

## 3. Sử dụng

Khởi động lại Claude Code trong project, tự xem và chấp thuận workspace/MCP nếu được hỏi, rồi mở `/mcp` kiểm tra server `workshop`. Không tự ghi cờ trust/approval để bỏ qua bước này.

Nhắn Claude: **“Dùng workshop MCP để bắt đầu session và ghi nhận buổi làm bài này.”** MCP không tự thu toàn bộ hội thoại; các message/artifact do client gửi được đánh dấu là client-reported.

Thiếu hoặc sai token Workshop sẽ trả `401`. Nếu MCP initialize thành công nhưng AI `/v1/models` trả `401`, operator cần kiểm tra upstream credential trong cấu hình host; không tự đổi token Workshop hoặc bỏ auth. Máy host, mạng và AI upstream phải đang chạy. AI streaming từng được kiểm chứng bằng một request OpenAI-compatible; chưa chạy model Claude để kiểm thử.

Tài liệu chính thức: [Settings env và thời điểm áp dụng](https://code.claude.com/docs/en/settings-reference#env) · [MCP project scope](https://code.claude.com/docs/en/mcp#project-scope) · [Biến môi trường trong MCP](https://code.claude.com/docs/en/mcp#environment-variable-expansion-in-mcp-json) · [Model và thứ tự ưu tiên](https://code.claude.com/docs/en/model-config#setting-your-model).

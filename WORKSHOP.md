# Bài thực hành: Xây bảng công việc nhóm cùng AI

## Đề bài

Sử dụng Claude Code để xây một ứng dụng bảng công việc nhỏ cho nhóm: tạo và sửa công việc, chuyển giữa ba trạng thái **To do / Doing / Done**, lọc theo trạng thái và lưu dữ liệu trên trình duyệt để không mất khi tải lại trang.

Bạn được chọn stack quen thuộc và tự quyết các chi tiết chưa được mô tả trong đề.

**Prompt bắt đầu phiên Workshop MCP:**

```text
Dùng server MCP workshop của project này để bắt đầu phiên thực hành: hỏi tên hiển thị nếu chưa biết, gọi start_session và dùng session_id trả về cho các lần ghi tiếp theo. Trong quá trình làm bài, chủ động dùng record_visible_event và record_artifact để lưu những trao đổi, quyết định và kết quả kiểm chứng liên quan; không ghi key/token, dữ liệu riêng hoặc suy luận ẩn. Chỉ gọi freeze_session khi tôi yêu cầu kết thúc. Nếu MCP chưa kết nối hoặc tool lỗi, báo đúng trạng thái, không giả vờ đã ghi.
```

## Mục đích

Thực hành một workflow làm việc với AI có chủ đích: hiểu vấn đề, xác định phạm vi, lập kế hoạch, chia việc, cung cấp ngữ cảnh phù hợp, triển khai từng bước, kiểm tra và điều chỉnh.

Không chỉ đưa một yêu cầu kiểu **“code app này cho tôi”** rồi nhận kết quả. Bạn cần chủ động định hướng, xem xét các lựa chọn và kiểm chứng những gì AI làm.

## Tiêu chí grading chính

- **Áp dụng workflow tốt:** làm rõ yêu cầu và phạm vi trước khi triển khai; chia công việc thành các bước hợp lý, có kiểm tra và điều chỉnh.
- **Phối hợp với AI có chủ đích:** cung cấp ngữ cảnh cần thiết, giao việc rõ ràng và đưa ra quyết định thay vì chấp nhận mọi đề xuất.
- **Kiểm chứng thực tế:** rà soát code, chạy ứng dụng hoặc kiểm thử phù hợp; phân biệt kết quả quan sát được với lời khẳng định của AI.
- **Debug và cải tiến dựa trên bằng chứng:** mô tả lỗi cụ thể, khoanh vùng nguyên nhân, sửa và kiểm tra lại thay vì yêu cầu AI sửa mò.
- **Hiểu các lựa chọn và giới hạn:** giải thích được những quyết định chính, đánh đổi và phần chưa được kiểm chứng.

Grading tập trung vào chất lượng quy trình làm việc với AI, không chỉ việc app chạy được hay giao diện đẹp. Không chấm theo độ dài prompt, số lượt chat hoặc số lần gọi tool.

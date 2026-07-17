# CLAUDE.md — Your Eyes VisionCare dev-env

## Mục đích

Repository chạy môi trường phát triển/tích hợp local cho `Your-eyes-visioncare-host`, `Your-eyes-visioncare-mobile-app`, `Your-eyes-visioncare-smart-glasses` và chính `Your-eyes-visioncare-dev-env` (§9.4).

## Luật gốc

`PROJECT_ARCHITECTURE.md` nằm ở repository workspace. File này trích § thay vì chép; thay đổi luật phải sửa kiến trúc và mở ADR (§19).

## Ranh giới workspace và dev-env

Workspace lấy code về; dev-env chạy code (§5, ADR-008). `bootstrap/validate-workspace` chỉ kiểm tra bốn repo sibling và báo lỗi khi thiếu, không clone. Cấu trúc chính thức nằm ở §9.4; không chép cây tại đây.

## Quy tắc Claude phải theo

- Không copy source của bốn repo vào dev-env (§9.4).
- Không tạo business logic trong script hoặc container phụ (§9.4).
- Không commit `.env` thật, API key, device secret hoặc certificate (§14.2).
- Chỉ commit `.env.example` với giá trị giả được đánh dấu hoặc để trống (§14.2).
- Pin version image; không dùng `latest` để bảo đảm tái lập.
- Mỗi service phải có healthcheck trước khi integration test chạy.
- Script phải chạy lại an toàn khi có thể; `reset` phải yêu cầu xác nhận.
- Không dùng `sleep` cố định thay readiness check khi có health endpoint.
- Không dùng dev-env làm cấu hình production (§9.4).
- Không bắt buộc Git submodule; bốn repo clone cạnh nhau theo ADR-008.
- Không làm Mobile phụ thuộc Docker; app chạy trên emulator/thiết bị thật với Host URL từ config (§9.4).
- Mọi mock response phải tuân thủ contract Host; mock lệch là rủi ro R2 (§13).
- Không sửa source của sibling repo từ dev-env.
- Không biến workspace/dev-env thành monorepo (§5).
- Integration test mặc định không phụ thuộc provider trả phí (N9, §17).

## Giai đoạn hiện tại

Repository là skeleton; chưa có manifest, script hay mock implementation.

## Definition of Done

- Thành viên mới làm theo README và khởi động được môi trường.
- Không có secret; `.env.example` bao phủ biến cần thiết.
- `start`, `stop`, `status` hoạt động đúng và báo exit code rõ khi được triển khai.
- Healthcheck/readiness bảo vệ thứ tự khởi động; smoke test pass.
- Mock khớp cùng artifact contract với Host.
- Không làm hỏng khả năng chạy độc lập của từng repo.
- Tài liệu cập nhật cùng thay đổi vận hành.

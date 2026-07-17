# scripts

Script dự kiến theo §9.4: `setup`, `start`, `stop`, `status`, `reset`, `seed`, `test`. **Chưa có script nào**.

Script phải fail rõ khi thiếu prerequisite, không nuốt lỗi, trả exit code đúng và chạy lại an toàn khi có thể. `reset` xóa dữ liệu nên phải yêu cầu xác nhận. Không dùng `sleep` cố định thay readiness check khi có health endpoint.

`setup` của dev-env không clone repo; clone thuộc `workspace/scripts/setup.ps1` (§5, ADR-008).

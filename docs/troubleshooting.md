# Troubleshooting

| Triệu chứng | Nguyên nhân | Xử lý |
|---|---|---|
| Port bind thất bại | Port đã bị chiếm | Dùng `status`/công cụ hệ điều hành tìm process hoặc đổi port local chưa chốt |
| Test chạy trước DB | Postgres chưa healthy | Chờ readiness/healthcheck; không thêm `sleep` cố định |
| Android emulator không gọi Host | Dùng sai địa chỉ loopback | Dùng `10.0.2.2` cho máy host |
| Thiết bị thật không gọi Host | Không cùng LAN hoặc firewall chặn | Dùng LAN IP, kiểm firewall và binding Host |
| Contract test đỏ với mock | Mock trả response lệch artifact Host | Cập nhật mock từ cùng artifact version (§13), không sửa test để chấp nhận drift |

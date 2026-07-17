# Luồng tích hợp local

Quy trình kiến trúc nằm ở §9.4 và chín luồng test bắt buộc nằm ở §17; tài liệu này không nhân bản.

Thứ tự khởi động thật: Postgres/Redis → Host → OCR/VLM/payment/OTP mock → glasses simulator. Mobile chạy **ngoài** container trên emulator hoặc thiết bị thật và kết nối Host local. Readiness phải pass trước khi chuyển bước.

# CLAUDE.md — Your-eyes-visioncare-dev-env

## 1. Mục đích repository

`Your-eyes-visioncare-dev-env` là môi trường phát triển và kiểm thử tích hợp tại local cho ba repository sản phẩm:

```text
Your-eyes-visioncare-host
Your-eyes-visioncare-mobile
Your-eyes-visioncare-glasses
```

Repository này cung cấp cấu hình, script và tài liệu để các thành viên có thể khởi động các dependency, Host, glasses simulator và integration test theo cùng một cách.

Repository này **không phải một thành phần sản phẩm** và **không chứa business logic**.

---

## 2. Vai trò trong workspace

Cấu trúc local khuyến nghị:

```text
your-eyes-workspace/
├── Your-eyes-visioncare-host/
├── Your-eyes-visioncare-mobile/
├── Your-eyes-visioncare-glasses/
└── Your-eyes-visioncare-dev-env/
```

`compose.yaml` có thể sử dụng các build context dạng sibling path:

```text
../Your-eyes-visioncare-host
../Your-eyes-visioncare-glasses
```

Mobile app thường chạy ngoài container bằng emulator hoặc thiết bị thật và kết nối tới Host local.

---

## 3. Phạm vi trách nhiệm

- Khởi động PostgreSQL, Redis và các dependency local khác.
- Build/run Host local.
- Build/run glasses simulator.
- Cung cấp mock services khi provider thật không được sử dụng.
- Seed dữ liệu development.
- Chạy smoke test và end-to-end integration test.
- Chuẩn hóa environment variables qua `.env.example`.
- Cung cấp hướng dẫn kết nối Android emulator, iOS simulator, điện thoại thật và glasses simulator.
- Dọn dữ liệu local và reset môi trường.
- Tùy chọn profile observability local nếu cần.

Không chứa:

- Source code nghiệp vụ của Host.
- Source code mobile.
- Source code kính.
- API key thật.
- Cấu hình production.
- Dữ liệu người dùng thật.

---

## 4. Nguyên tắc bắt buộc

1. **Không copy source code của ba repo vào dev-env.**
2. **Không tạo business logic trong script hoặc container phụ.**
3. **Không commit `.env` thật, API key, device secret hoặc certificate.**
4. **Chỉ commit `.env.example` với giá trị giả hoặc trống.**
5. **Phiên bản image phải được pin**, không phụ thuộc mơ hồ vào `latest` khi cần tính tái lập.
6. **Các service phải có healthcheck** trước khi integration test chạy.
7. **Script phải chạy an toàn nhiều lần** khi có thể.
8. **Lệnh reset dữ liệu phải được đặt tên rõ ràng và yêu cầu xác nhận nếu có nguy cơ xóa dữ liệu.**
9. **Không dùng dev-env như cấu hình production deployment.**
10. **Không bắt buộc Git submodule.** Mặc định hỗ trợ clone bốn repo cạnh nhau.
11. **Không làm mobile phụ thuộc vào Docker.** Mobile có thể chạy bằng emulator/device và dùng Host URL được cấu hình.
12. **Mọi mock response phải tuân thủ API contract của Host.**

---

## 5. Các service local đề xuất

```text
postgres
redis
host
ocr-mock
vlm-mock
glasses-simulator
mail/otp-mock tùy chọn
payment-webhook-mock tùy chọn
```

Các profile Docker Compose có thể gồm:

```text
core          # postgres, redis, host
simulator     # glasses simulator
mocks         # OCR/VLM/payment/OTP mock
observability # logs/metrics local nếu cần
```

Không thêm service chỉ để làm môi trường phức tạp hơn. Mỗi service cần có mục đích phát triển hoặc kiểm thử rõ ràng.

---

## 6. Environment variables

`.env.example` nên mô tả tối thiểu:

```text
HOST_PORT
DATABASE_PORT
REDIS_PORT
DATABASE_URL
REDIS_URL
MOBILE_API_BASE_URL
GLASSES_API_BASE_URL
DEVICE_SERIAL
DEVICE_SECRET_DEV_ONLY
OCR_PROVIDER_MODE=mock
VLM_PROVIDER_MODE=mock
LOG_LEVEL
```

Các giá trị development phải được đánh dấu rõ là không dùng cho production.

Không ghi secret thật trong README, compose file hoặc script.

---

## 7. Script chuẩn hóa

Các script đề xuất:

```text
bootstrap     # kiểm tra prerequisite và tạo file local cần thiết
start         # khởi động môi trường
stop          # dừng môi trường
status        # xem trạng thái service
logs          # xem log tổng hợp
seed          # tạo dữ liệu development
reset         # xóa volume/dữ liệu local có kiểm soát
smoke-test    # test health và endpoint cơ bản
e2e-test      # chạy integration test
```

Script phải:

- Thất bại rõ ràng nếu thiếu prerequisite.
- Không nuốt lỗi.
- Trả exit code đúng.
- Hoạt động trên hệ điều hành mục tiêu hoặc có phiên bản tương ứng.
- Không tự tải/chạy binary không đáng tin cậy.

---

## 8. Quy tắc khi Claude Code làm việc trong repo

Claude phải:

1. Đọc `CLAUDE.md`, `README.md` và cấu trúc workspace trước khi sửa.
2. Không thêm business logic vào dev-env.
3. Không sửa source code của sibling repo từ repository này.
4. Không giả định đường dẫn tuyệt đối theo máy cá nhân.
5. Dùng biến môi trường cho port và path có thể thay đổi.
6. Không commit secret hoặc dữ liệu thật.
7. Pin version image/dependency khi cần reproducibility.
8. Thêm healthcheck và dependency condition hợp lý.
9. Không dùng `sleep` cố định thay cho readiness check nếu có thể kiểm tra health.
10. Giữ lệnh start đơn giản và có tài liệu.
11. Cập nhật `.env.example` và README khi thêm biến hoặc service.
12. Giữ mock service đúng API contract.
13. Không phá khả năng chạy từng repo độc lập.
14. Không biến repo này thành monorepo chứa cả ba codebase.
15. Ghi rõ khác biệt giữa local, staging và production.

---

## 9. Quy trình chạy local đề xuất

```text
1. Clone bốn repo vào cùng một thư mục cha.
2. Sao chép .env.example thành .env local.
3. Chạy bootstrap.
4. Khởi động core services.
5. Seed user/device/subscription development.
6. Chạy glasses simulator.
7. Chạy mobile app bằng emulator hoặc thiết bị thật.
8. Chạy smoke test hoặc e2e test.
```

Mobile local cần chú ý:

```text
Android emulator → có thể dùng 10.0.2.2 để gọi Host trên máy
Thiết bị thật → dùng LAN IP của máy chạy Host
Production/staging → dùng HTTPS domain tương ứng
```

Không hard-code các địa chỉ trên trong source mobile.

---

## 10. Kiểm thử tích hợp tối thiểu

Dev-env phải hỗ trợ các flow:

```text
OTP login mock hoặc sandbox
Link device bằng serial + activation code
Kích hoạt subscription development
Glasses simulator gửi READ_TEXT
Host authorize và gọi OCR mock
Host trả result_text
Glasses simulator nhận kết quả
Subscription expired bị từ chối
Device locked/lost bị từ chối
Payment webhook mock kích hoạt subscription idempotent
```

Integration test không phụ thuộc API provider trả phí theo mặc định.

---

## 11. Definition of Done

Một thay đổi dev-env chỉ hoàn thành khi:

- Thành viên mới có thể làm theo README để chạy môi trường.
- Không chứa secret hoặc dữ liệu thật.
- `start`, `stop`, `status` hoạt động đúng.
- Service có healthcheck phù hợp.
- `.env.example` đầy đủ.
- Smoke test vượt qua.
- Mock response đúng API contract.
- Không làm hỏng khả năng chạy repo riêng lẻ.
- Tài liệu được cập nhật.

---

## 12. Cấu trúc thư mục định hướng

```text
Your-eyes-visioncare-dev-env/
├── CLAUDE.md
├── README.md
├── compose.yaml
├── compose.override.example.yaml
├── .env.example
├── docs/
│   ├── local-network.md
│   ├── mobile-emulator.md
│   ├── troubleshooting.md
│   └── integration-flow.md
├── scripts/
│   ├── bootstrap.sh
│   ├── start.sh
│   ├── stop.sh
│   ├── status.sh
│   ├── logs.sh
│   ├── seed.sh
│   ├── reset.sh
│   └── test.sh
├── mocks/
│   ├── ocr/
│   ├── vlm/
│   ├── payment/
│   └── otp/
├── seed/
│   ├── devices/
│   ├── users/
│   └── subscriptions/
├── integration-tests/
│   ├── smoke/
│   └── e2e/
└── config/
```

Có thể dùng PowerShell hoặc cross-platform task runner thay cho shell script nếu nhóm phát triển chủ yếu trên Windows. Không cần duy trì hai hệ script nếu chưa có nhu cầu thực tế.

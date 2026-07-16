# Your Eyes VisionCare — Project Architecture

## 1. Mục tiêu thiết kế

Tài liệu này định nghĩa cấu trúc thư mục và ranh giới trách nhiệm cho bốn repository của hệ thống **Your Eyes VisionCare**.

Kiến trúc được thiết kế theo các nguyên tắc:

- Chia hệ thống theo **chức năng và trách nhiệm**, không chia theo framework.
- Mỗi thư mục phải có mục đích rõ ràng và cần thiết.
- `bootstrap` chịu trách nhiệm khởi tạo và lắp ráp hệ thống.
- `routing` chịu trách nhiệm phân loại input và điều phối đến đúng module.
- Business logic nằm trong `modules/` hoặc `features/`, không nằm trong routing hay bootstrap.
- Giao tiếp giữa các repository phải thông qua `contracts` đã thống nhất.
- Không khóa dự án vào React Native, Flutter, FastAPI, NestJS, Spring Boot, C++, Rust hoặc bất kỳ công nghệ cụ thể nào.
- Chỉ bổ sung thư mục phụ thuộc framework sau khi công nghệ được quyết định.

---

## 2. Tổng thể hệ thống

```text
Your Eyes VisionCare
│
├── Your-eyes-visioncare-host/      # Backend trung tâm, cấp quyền và điều phối OCR/VLM
├── Your-eyes-visioncare-mobile/    # Ứng dụng quản lý tài khoản, thiết bị và thuê bao
├── Your-eyes-visioncare-glasses/   # Phần mềm chạy trên kính, xử lý STT/TTS và giao tiếp Host
└── Your-eyes-visioncare-dev-env/   # Môi trường chạy và kiểm thử tích hợp trên máy local
```

### Luồng giao tiếp chính

```text
Mobile App cho cả Android và IOS
    │
    │ Đăng nhập, liên kết kính, thanh toán, quản lý thiết bị
    ▼
VisionCare Host
    │
    ├── Xác thực người dùng và thiết bị
    ├── Kiểm tra thuê bao, quyền chức năng và quota
    ├── Điều phối OCR/VLM
    └── Trả kết quả dạng text
    ▲
    │ Gửi text + ảnh, trạng thái thiết bị; nhận text phản hồi từ host
    │
Smart Glasses
```

### Nguyên tắc quan trọng

- App và kính không truy cập database trực tiếp.
- App và kính không giữ API key của OCR/VLM provider.
- App và kính không giao tiếp trực tiếp với nhau, thuê bao và quyền sử dụng được Host quản lý theo `device_id`.
- Host là nơi duy nhất lưu provider API key và điều phối OCR/VLM.
- STT/TTS chạy trực tiếp trên kính.

---

## 3. Workspace local

```text
your-eyes-workspace/                         # Thư mục làm việc chung trên máy phát triển
│
├── Your-eyes-visioncare-host/              # Git repository độc lập của Host
├── Your-eyes-visioncare-mobile/            # Git repository độc lập của Mobile App
├── Your-eyes-visioncare-glasses/           # Git repository độc lập của phần mềm kính
└── Your-eyes-visioncare-dev-env/           # Git repository độc lập cho môi trường tích hợp local
```

Bốn thư mục là bốn Git repository độc lập. `dev-env` không chứa source code của ba repository sản phẩm.

---

# 4. Your-eyes-visioncare-host

## 4.1. Vai trò

Host là hệ thống trung tâm chịu trách nhiệm:

- Quản lý tài khoản và xác thực người dùng.
- Quản lý kính và liên kết kính với tài khoản.
- Quản lý gói dịch vụ, thuê bao và thanh toán.
- Cấp quyền chức năng cho từng thiết bị.
- Kiểm tra quota và lịch sử sử dụng.
- Nhận request từ kính.
- Điều phối request đến OCR hoặc VLM.
- Trả kết quả text cho kính.
- Cung cấp API riêng cho Mobile App và Smart Glasses.

## 4.2. Cấu trúc thư mục

```text
Your-eyes-visioncare-host/
│
├── CLAUDE.md                              # Context và quy tắc làm việc dành cho Claude Code
├── README.md                              # Giới thiệu repo, cách thiết lập và cách chạy
│
├── docs/                                  # Tài liệu kiến trúc và nghiệp vụ của Host
│   ├── system-context/                    # Bối cảnh hệ thống, actor và quan hệ với app/kính/provider
│   ├── business-rules/                    # Quy tắc thuê bao, thanh toán, thiết bị và quyền sử dụng
│   ├── api/                               # Tài liệu API dành cho Mobile và Glasses
│   ├── flows/                             # Sequence flow cho login, linking, payment và AI task
│   └── decisions/                         # Ghi lại các quyết định kiến trúc quan trọng
│
├── src/                                   # Toàn bộ source code của Host
│   │
│   ├── main.*                             # Entry point duy nhất; chỉ gọi bootstrap và chạy ứng dụng
│   │
│   ├── bootstrap/                         # Khởi tạo, kiểm tra và lắp ráp toàn bộ hệ thống
│   │   ├── application-bootstrap.*        # Tạo application và kết nối các thành phần cấp cao
│   │   ├── module-registry.*              # Đăng ký các module nghiệp vụ được sử dụng
│   │   ├── dependency-registry.*          # Kết nối implementation với interface cần dùng
│   │   ├── startup-checks.*               # Kiểm tra config, dependency và kết nối trước khi chạy
│   │   └── README.md                      # Mô tả thứ tự và nguyên tắc khởi động Host
│   │
│   ├── routing/                           # Nhận input và chuyển đến đúng public interface của module
│   │   ├── mobile-routing/                # Điều phối request đến từ Mobile App
│   │   ├── glasses-routing/               # Điều phối request đến từ Smart Glasses
│   │   ├── admin-routing/                 # Điều phối chức năng quản trị và hỗ trợ nội bộ
│   │   ├── health-routing/                # Endpoint kiểm tra trạng thái sống và sẵn sàng của Host
│   │   └── README.md                      # Quy tắc routing, validation đầu vào và format đầu ra
│   │
│   ├── modules/                           # Các module nghiệp vụ độc lập của Host
│   │   ├── authentication/                # OTP, đăng nhập, token và phiên người dùng
│   │   ├── users/                         # Hồ sơ người dùng, vai trò và thông tin tài khoản
│   │   ├── devices/                       # Thông tin, trạng thái và vòng đời của kính
│   │   ├── device-linking/                # Liên kết/hủy liên kết user với kính bằng serial và activation code
│   │   ├── plans/                         # Định nghĩa gói tháng, năm, dùng thử và quyền của từng gói
│   │   ├── subscriptions/                 # Thuê bao hiện tại, gia hạn, hết hạn và hủy thuê bao
│   │   ├── payments/                      # Tạo giao dịch, nhận webhook và đối soát thanh toán
│   │   ├── entitlements/                  # Xác định device được phép sử dụng chức năng nào
│   │   ├── usage-quota/                   # Theo dõi và giới hạn số lần sử dụng theo gói
│   │   ├── glasses-tasks/                 # Điều phối toàn bộ request chức năng gửi từ kính
│   │   ├── ocr-processing/                # Chuẩn hóa input/output và giao tiếp OCR provider
│   │   ├── vlm-processing/                # Chuẩn hóa prompt/input/output và giao tiếp VLM provider
│   │   ├── notifications/                 # Thông báo hết hạn, thanh toán và trạng thái thiết bị
│   │   └── support/                       # Yêu cầu hỗ trợ, phản hồi lỗi và thông tin hotline
│   │
│   ├── contracts/                         # Quy ước giao tiếp ổn định trong và ngoài Host
│   │   ├── mobile-api/                    # Request/response contract giữa Mobile và Host
│   │   ├── glasses-api/                   # Request/response contract giữa Glasses và Host
│   │   ├── provider-api/                  # Interface chung cho OCR/VLM/payment/OTP provider
│   │   ├── schemas/                       # Schema dữ liệu được chia sẻ giữa nhiều module
│   │   ├── task-types/                    # Danh sách loại task như READ_TEXT, DESCRIBE_SCENE
│   │   ├── statuses/                      # Device, subscription, payment và task status
│   │   └── error-codes/                   # Bộ mã lỗi thống nhất cho app và kính
│   │
│   ├── shared/                            # Thành phần kỹ thuật dùng chung, không chứa business logic
│   │   ├── errors/                        # Base error và cơ chế ánh xạ lỗi
│   │   ├── logging/                       # Ghi log có cấu trúc và correlation/request ID
│   │   ├── validation/                    # Validation primitive dùng bởi nhiều module
│   │   ├── security/                      # Hashing, token utilities và quy tắc bảo mật chung
│   │   ├── serialization/                 # Chuyển đổi dữ liệu qua boundary
│   │   ├── time/                          # Clock abstraction và xử lý thời gian dùng chung
│   │   ├── types/                         # Kiểu dữ liệu kỹ thuật dùng chung
│   │   └── constants/                     # Hằng số kỹ thuật toàn ứng dụng
│   │
│   └── config/                            # Định nghĩa, đọc và kiểm tra cấu hình runtime
│       ├── environment.*                  # Đọc biến môi trường
│       ├── application-config.*           # Cấu hình tổng thể của Host
│       ├── feature-flags.*                # Bật/tắt chức năng có kiểm soát
│       └── README.md                      # Danh sách cấu hình và nguồn cung cấp giá trị
│
├── tests/                                 # Test ở cấp repository và nhiều module
│   ├── unit/                              # Test logic độc lập của từng đơn vị
│   ├── contract/                          # Kiểm tra implementation tuân thủ API contract
│   ├── integration/                       # Test nhiều module và dependency cùng hoạt động
│   └── end-to-end/                        # Test luồng hoàn chỉnh từ request đến response
│
├── scripts/                               # Script thiết lập, kiểm tra, migrate và hỗ trợ phát triển
├── .env.example                           # Danh sách biến môi trường mẫu; không chứa secret thật
├── .gitignore                             # Loại trừ secret, build output và file local
└── project-config-files                   # File cấu hình build/lint/test sau khi chọn công nghệ
```

## 4.3. Luồng xử lý Host

```text
main
  → bootstrap
      → đọc config
      → khởi tạo dependency
      → đăng ký module
      → đăng ký routing
      → kiểm tra startup
  → application chạy
      → routing nhận request
      → routing gọi public interface của module
      → module xử lý business logic
      → response trả theo contract
```

---

# 5. Your-eyes-visioncare-mobile

## 5.1. Vai trò

Mobile App chịu trách nhiệm:

- Đăng ký và đăng nhập bằng số điện thoại/OTP.
- Liên kết kính bằng serial number và activation code.
- Xem và quản lý thiết bị đã liên kết.
- Chọn gói dịch vụ và thực hiện thanh toán.
- Xem trạng thái thuê bao và lịch sử giao dịch.
- Khóa kính, báo mất hoặc yêu cầu đổi kính.
- Cung cấp hướng dẫn và kênh hỗ trợ.
- Đảm bảo khả năng tiếp cận cho người khiếm thị và người hỗ trợ.

## 5.2. Cấu trúc thư mục

```text
Your-eyes-visioncare-mobile/
│
├── CLAUDE.md                              # Context và quy tắc làm việc dành cho Claude Code
├── README.md                              # Giới thiệu app, cách thiết lập và cách chạy
│
├── docs/                                  # Tài liệu sản phẩm và kiến trúc Mobile
│   ├── user-flows/                        # Luồng đăng nhập, linking, payment và quản lý kính
│   ├── accessibility/                     # Quy chuẩn screen reader, contrast, font và thao tác
│   ├── api-integration/                   # Quy tắc giao tiếp với Host
│   ├── screens/                           # Mô tả vai trò và trạng thái của từng màn hình
│   └── decisions/                         # Các quyết định kiến trúc và UX quan trọng
│
├── src/                                   # Toàn bộ source code của Mobile App
│   │
│   ├── main.*                             # Entry point duy nhất; gọi bootstrap và khởi chạy app
│   │
│   ├── bootstrap/                         # Khởi tạo các context cần thiết trước khi hiển thị app
│   │   ├── application-bootstrap.*        # Tạo root application và kết nối feature cấp cao
│   │   ├── session-bootstrap.*            # Khôi phục token, user session và trạng thái đăng nhập
│   │   ├── configuration-bootstrap.*      # Đọc API URL, environment và feature flag
│   │   ├── accessibility-bootstrap.*      # Khởi tạo tùy chọn accessibility toàn ứng dụng
│   │   ├── startup-checks.*               # Kiểm tra điều kiện cần thiết trước khi mở luồng chính
│   │   └── README.md                      # Mô tả thứ tự khởi động Mobile App
│   │
│   ├── routing/                           # Điều phối màn hình và luồng dựa trên state ứng dụng
│   │   ├── application-routing/           # Quyết định luồng gốc: auth, linking, subscription hay home
│   │   ├── authentication-routing/        # Điều phối các bước phone, OTP và hoàn tất đăng nhập
│   │   ├── device-routing/                # Điều phối liên kết, chi tiết, khóa và báo mất kính
│   │   ├── subscription-routing/          # Điều phối chọn gói, checkout và trạng thái thuê bao
│   │   ├── deep-link-routing/             # Xử lý callback thanh toán hoặc liên kết ngoài
│   │   └── README.md                      # Quy tắc navigation và điều kiện chuyển flow
│   │
│   ├── features/                          # Các chức năng độc lập mà người dùng tương tác
│   │   ├── authentication/                # Số điện thoại, OTP, đăng nhập và đăng xuất
│   │   ├── onboarding/                    # Hướng dẫn ban đầu và thiết lập trải nghiệm
│   │   ├── device-linking/                # Nhập/quét serial và activation code
│   │   ├── device-management/             # Thông tin, trạng thái, khóa, báo mất và đổi kính
│   │   ├── subscription-plans/            # Danh sách và chi tiết gói dịch vụ
│   │   ├── subscription-status/           # Còn hạn, sắp hết hạn, quá hạn và gia hạn
│   │   ├── payments/                      # Tạo checkout và xử lý kết quả thanh toán
│   │   ├── transaction-history/           # Danh sách giao dịch, hóa đơn và trạng thái
│   │   ├── user-profile/                  # Hồ sơ, tùy chọn người dùng và người hỗ trợ
│   │   ├── notifications/                 # Hiển thị thông báo liên quan gói và thiết bị
│   │   └── support/                       # Hotline, hướng dẫn và phản hồi lỗi
│   │
│   ├── contracts/                         # Contract dữ liệu Mobile sử dụng để giao tiếp Host
│   │   ├── host-api/                      # Request/response model của Host API
│   │   ├── schemas/                       # Schema dữ liệu dùng chung giữa nhiều feature
│   │   ├── statuses/                      # Trạng thái device, payment và subscription
│   │   └── error-codes/                   # Error code cần ánh xạ thành thông báo UI
│   │
│   ├── shared/                            # Thành phần dùng chung giữa nhiều feature
│   │   ├── ui/                            # Component giao diện tái sử dụng
│   │   ├── accessibility/                 # Helper và component hỗ trợ accessibility
│   │   ├── api/                           # HTTP client, auth header, retry và response parsing
│   │   ├── storage/                       # Lưu token và dữ liệu local không thuộc riêng feature
│   │   ├── errors/                        # Chuẩn hóa lỗi kỹ thuật thành lỗi ứng dụng
│   │   ├── validation/                    # Validation dùng chung cho input người dùng
│   │   ├── formatting/                    # Format ngày, tiền tệ và trạng thái hiển thị
│   │   ├── types/                         # Kiểu dữ liệu dùng chung
│   │   └── constants/                     # Hằng số dùng toàn app
│   │
│   └── config/                            # Cấu hình runtime và build environment của Mobile
│       ├── environment.*                  # API URL và tên environment
│       ├── application-config.*           # Cấu hình hành vi toàn app
│       ├── feature-flags.*                # Bật/tắt feature có kiểm soát
│       └── README.md                      # Mô tả nguồn và mục đích từng cấu hình
│
├── assets/                                # Tài nguyên tĩnh đóng gói cùng ứng dụng
│   ├── images/                            # Hình ảnh giao diện
│   ├── icons/                             # Icon ứng dụng và chức năng
│   ├── audio/                             # Âm thanh hướng dẫn hoặc phản hồi nếu cần
│   └── localization/                      # Nội dung đa ngôn ngữ nếu được hỗ trợ
│
├── tests/                                 # Test cấp ứng dụng
│   ├── unit/                              # Test logic và state độc lập
│   ├── integration/                       # Test nhiều feature/shared component
│   ├── accessibility/                     # Test nhãn đọc, thứ tự focus và contrast
│   └── end-to-end/                        # Test luồng người dùng hoàn chỉnh
│
├── scripts/                               # Script build, test và hỗ trợ phát triển
├── .env.example                           # Biến môi trường mẫu; không chứa secret
├── .gitignore                             # Loại trừ build output và file local
└── platform-and-project-config-files      # Android/iOS/framework config sau khi chọn công nghệ
```

## 5.3. Luồng routing Mobile

```text
App khởi động
  → bootstrap khôi phục session
  → routing kiểm tra trạng thái
      ├── chưa đăng nhập → authentication
      ├── chưa liên kết kính → device-linking
      ├── thuê bao hết hạn → subscription-status/payments
      └── hợp lệ → màn hình chính
```

---

# 6. Your-eyes-visioncare-glasses

## 6.1. Vai trò

Phần mềm kính chịu trách nhiệm:

- Thu nhận âm thanh từ microphone.
- Chạy STT trực tiếp trên thiết bị.
- Xác định ý định/lệnh cơ bản của người dùng.
- Điều khiển camera và chụp ảnh khi cần.
- Gửi text, ảnh và metadata đến Host.
- Nhận kết quả text từ Host.
- Chạy TTS trực tiếp trên thiết bị.
- Quản lý kết nối, trạng thái, pin và lỗi thiết bị.
- Cung cấp phản hồi offline cho các trường hợp không có mạng.

Phần mềm kính không quản lý thanh toán, thuê bao hoặc gọi trực tiếp OCR/VLM provider.

## 6.2. Cấu trúc thư mục

```text
Your-eyes-visioncare-glasses/
│
├── CLAUDE.md                              # Context và quy tắc làm việc dành cho Claude Code
├── README.md                              # Giới thiệu phần mềm kính, thiết lập và cách chạy
│
├── docs/                                  # Tài liệu luồng thiết bị và tích hợp phần cứng
│   ├── device-flow/                       # Luồng voice → STT → task → Host → TTS
│   ├── hardware-abstraction/              # Quy ước tách module khỏi driver/OS cụ thể
│   ├── host-communication/                # Request, authentication, retry và timeout
│   ├── offline-behavior/                  # Hành vi khi mất mạng hoặc Host không phản hồi
│   ├── resource-constraints/              # Giới hạn RAM, CPU, pin, nhiệt độ và storage
│   └── decisions/                         # Các quyết định kỹ thuật và kiến trúc quan trọng
│
├── src/                                   # Toàn bộ source code chạy trên kính
│   │
│   ├── main.*                             # Entry point duy nhất; gọi bootstrap và chạy vòng đời kính
│   │
│   ├── bootstrap/                         # Khởi tạo software module và hardware adapter
│   │   ├── application-bootstrap.*        # Lắp ráp application và luồng xử lý chính
│   │   ├── hardware-bootstrap.*           # Khởi tạo camera, microphone, loa, nút và sensor
│   │   ├── audio-bootstrap.*              # Khởi tạo STT, TTS và audio pipeline
│   │   ├── network-bootstrap.*            # Khởi tạo kết nối và Host client
│   │   ├── startup-checks.*               # Kiểm tra phần cứng, credential và cấu hình
│   │   └── README.md                      # Mô tả trình tự khởi động của kính
│   │
│   ├── routing/                           # Điều phối lệnh, task, response và lỗi
│   │   ├── intent-routing/                # Ánh xạ text từ STT thành ý định chức năng
│   │   ├── task-routing/                  # Chọn workflow cần chạy cho từng loại task
│   │   ├── response-routing/              # Chọn cách xử lý và phát kết quả nhận từ Host
│   │   ├── error-routing/                 # Ánh xạ error code thành hành vi và câu TTS phù hợp
│   │   └── README.md                      # Quy tắc phân loại và điều phối input/output
│   │
│   ├── modules/                           # Các chức năng sản phẩm độc lập của kính
│   │   ├── voice-input/                   # Ghi âm, VAD và chuẩn bị audio cho STT
│   │   ├── stt/                           # Chuyển giọng nói thành text trên thiết bị
│   │   ├── intent-recognition/            # Nhận diện ý định cơ bản từ text
│   │   ├── camera-capture/                # Chụp, resize và nén ảnh trước khi gửi
│   │   ├── task-execution/                # Điều phối workflow hoàn chỉnh của một yêu cầu
│   │   ├── host-communication/            # Xác thực, gửi request và nhận response từ Host
│   │   ├── response-processing/           # Chuẩn hóa text và metadata Host trả về
│   │   ├── tts/                           # Chuyển text thành âm thanh trên thiết bị
│   │   ├── device-authentication/         # Quản lý device credential và token phiên
│   │   ├── connectivity/                  # Theo dõi mạng, retry, reconnect và timeout
│   │   ├── device-state/                  # Trạng thái hoạt động, busy, idle, error và locked
│   │   ├── telemetry/                     # Thu thập pin, nhiệt độ, mạng và lỗi để gửi Host
│   │   └── offline-mode/                  # Phản hồi local khi không thể gọi Host
│   │
│   ├── platform/                          # Adapter phụ thuộc phần cứng, OS hoặc SDK cụ thể
│   │   ├── audio/                         # Implementation microphone, speaker và audio driver
│   │   ├── camera/                        # Implementation camera driver hoặc camera SDK
│   │   ├── network/                       # Implementation Wi-Fi/4G và network stack
│   │   ├── storage/                       # Implementation lưu credential, config và cache
│   │   ├── controls/                      # Implementation nút bấm hoặc cảm biến thao tác
│   │   ├── power/                         # Implementation pin, sạc và chế độ tiết kiệm năng lượng
│   │   ├── indicators/                    # Implementation LED, âm báo hoặc rung
│   │   └── system/                        # OS process, clock, device info và system lifecycle
│   │
│   ├── contracts/                         # Contract giao tiếp với Host và giữa các boundary
│   │   ├── host-api/                      # Request/response model gửi và nhận từ Host
│   │   ├── device-events/                 # Event nội bộ như button pressed, network lost
│   │   ├── task-types/                    # READ_TEXT, DESCRIBE_SCENE, FIND_OBJECT...
│   │   ├── response-types/                # Success, partial, denied và failed response
│   │   └── error-codes/                   # Error code từ Host và lỗi local của kính
│   │
│   ├── shared/                            # Thành phần kỹ thuật dùng chung giữa các module
│   │   ├── errors/                        # Base error và lỗi kỹ thuật dùng chung
│   │   ├── logging/                       # Log phù hợp với tài nguyên hạn chế
│   │   ├── validation/                    # Validation cho config và dữ liệu giao tiếp
│   │   ├── serialization/                 # Encode/decode request, response và telemetry
│   │   ├── timing/                        # Timeout, delay và clock abstraction
│   │   ├── types/                         # Kiểu dữ liệu dùng chung
│   │   └── constants/                     # Hằng số toàn ứng dụng kính
│   │
│   └── config/                            # Cấu hình runtime của kính
│       ├── device-config.*                # Device ID, feature setting và thông tin firmware
│       ├── audio-config.*                 # STT/TTS, microphone và speaker setting
│       ├── network-config.*               # Host URL, retry và timeout
│       ├── camera-config.*                # Resolution, compression và capture setting
│       └── README.md                      # Giải thích nguồn và mục đích từng cấu hình
│
├── simulator/                             # Chạy và kiểm thử luồng kính khi chưa có phần cứng thật
│   ├── mock-platform/                     # Fake camera, microphone, speaker, network và button
│   ├── sample-images/                     # Ảnh mẫu dùng để gửi task
│   ├── sample-audio/                      # Audio mẫu dùng để test STT
│   ├── scenarios/                         # Kịch bản online, offline, timeout và lỗi thuê bao
│   └── simulator.*                        # Entry point của simulator
│
├── tests/                                 # Test cấp repository kính
│   ├── unit/                              # Test logic của module độc lập
│   ├── platform/                          # Test adapter và abstraction phần cứng
│   ├── integration/                       # Test nhiều module kết hợp
│   └── end-to-end/                        # Test voice/image → Host → text → TTS
│
├── scripts/                               # Script build, flash, test và thu log thiết bị
├── .env.example                           # Cấu hình mẫu cho simulator/local; không chứa secret
├── .gitignore                             # Loại trừ build output và credential local
└── platform-and-project-config-files      # Toolchain/board/framework config khi chọn công nghệ
```

## 6.3. Luồng routing trên kính

```text
Microphone
  → voice-input
  → STT
  → intent-routing
  → task-routing
      ├── task local → xử lý local
      └── task AI → camera-capture → host-communication
  → response-routing
  → TTS
```

---

# 7. Your-eyes-visioncare-dev-env

## 7.1. Vai trò

`dev-env` là repository hỗ trợ phát triển và tích hợp local. Nó chịu trách nhiệm:

- Chuẩn bị môi trường local cho Host và Glasses Simulator.
- Quản lý địa chỉ, port và network giữa các service.
- Khởi động dependency như database, cache hoặc mock provider.
- Seed dữ liệu người dùng, thiết bị, gói và thuê bao.
- Chạy integration test xuyên nhiều repository.
- Cung cấp mock OTP, payment, OCR và VLM provider.

Repository này không chứa business logic sản phẩm và không thay thế production infrastructure repository.

## 7.2. Cấu trúc thư mục

```text
Your-eyes-visioncare-dev-env/
│
├── CLAUDE.md                              # Context và quy tắc làm việc dành cho Claude Code
├── README.md                              # Cách clone, setup và chạy toàn bộ hệ thống local
│
├── bootstrap/                             # Chuẩn bị workspace và dependency local
│   ├── setup-workspace.*                  # Kiểm tra vị trí và sự tồn tại của các repo sản phẩm
│   ├── validate-environment.*             # Kiểm tra tool, port, env và prerequisite
│   ├── initialize-dependencies.*          # Khởi tạo database, cache và mock provider
│   ├── initialize-seed-data.*             # Nạp dữ liệu ban đầu phục vụ phát triển
│   └── README.md                          # Trình tự setup và khởi động local
│
├── routing/                               # Quy định cách các service tìm và gọi nhau trong local
│   ├── service-routing/                   # Tên service và địa chỉ nội bộ
│   ├── proxy-routing/                     # Reverse proxy hoặc gateway local nếu cần
│   ├── port-mapping/                      # Mapping port giữa host machine và service
│   └── README.md                          # Bản đồ network và quy tắc routing local
│
├── manifests/                             # Khai báo service và dependency theo công cụ được chọn
│   ├── local/                             # Cấu hình chạy phục vụ phát triển hằng ngày
│   ├── testing/                           # Cấu hình chạy integration/end-to-end test
│   └── dependencies/                      # Khai báo database, cache và mock service
│
├── env/                                   # Biến môi trường mẫu cho từng repository
│   ├── host.env.example                   # Cấu hình Host local
│   ├── mobile.env.example                 # Cấu hình Mobile kết nối Host local
│   ├── glasses.env.example                # Cấu hình Glasses Simulator kết nối Host
│   └── shared.env.example                 # Cấu hình dùng chung giữa nhiều service
│
├── scripts/                               # Lệnh thao tác với toàn bộ workspace
│   ├── setup.*                            # Thiết lập môi trường lần đầu
│   ├── start.*                            # Khởi động các service cần thiết
│   ├── stop.*                             # Dừng các service
│   ├── status.*                           # Kiểm tra trạng thái service
│   ├── reset.*                            # Xóa và tạo lại môi trường local
│   ├── seed.*                             # Nạp dữ liệu mẫu
│   └── test.*                             # Chạy integration test liên repository
│
├── mocks/                                 # Mock các dịch vụ bên ngoài để phát triển độc lập
│   ├── otp-provider/                      # Mock gửi và xác minh OTP
│   ├── payment-provider/                  # Mock checkout, webhook và trạng thái giao dịch
│   ├── ocr-provider/                      # Mock kết quả nhận diện văn bản
│   ├── vlm-provider/                      # Mock kết quả mô tả ảnh và tìm vật thể
│   └── glasses-device/                    # Mock request và trạng thái của kính
│
├── seed/                                  # Dữ liệu mẫu phục vụ phát triển và test
│   ├── users/                             # Người dùng và người hỗ trợ mẫu
│   ├── devices/                           # Kính, serial và activation code mẫu
│   ├── plans/                             # Gói tháng, năm và dùng thử mẫu
│   └── subscriptions/                     # Thuê bao active, expiring và expired mẫu
│
├── integration-tests/                     # Test luồng xuyên qua nhiều repository/service
│   ├── smoke/                             # Kiểm tra service khởi động và kết nối được
│   ├── device-linking/                    # Test app liên kết user với kính
│   ├── subscription-activation/           # Test thanh toán và cấp quyền cho device
│   ├── glasses-task/                      # Test kính gửi task và nhận kết quả
│   ├── subscription-expiration/           # Test từ chối task khi gói hết hạn
│   └── access-denied/                     # Test khóa, báo mất và thiếu quyền chức năng
│
├── docs/                                  # Tài liệu vận hành môi trường local
│   ├── workspace-setup.md                 # Hướng dẫn clone và sắp xếp bốn repo
│   ├── service-map.md                     # Sơ đồ service, địa chỉ và port
│   ├── integration-flow.md                # Luồng test end-to-end giữa app, Host và kính
│   ├── contract-sync.md                   # Quy trình đồng bộ API contract giữa các repo
│   └── troubleshooting.md                 # Lỗi thường gặp và cách xử lý
│
├── .env.example                           # Cấu hình tổng quát mẫu; không chứa secret
├── .gitignore                             # Không commit env thật, volume và output local
└── project-config-files                   # File cấu hình công cụ local khi được lựa chọn
```

---

# 8. Cấu trúc chuẩn bên trong một module/feature

Cấu trúc module phải phản ánh trách nhiệm, không áp đặt framework.

## 8.1. Module nhỏ

```text
example-module/
├── README.md              # Context, trách nhiệm, input/output và dependency của module
├── public.*               # Public interface duy nhất module khác được phép sử dụng
├── module.*               # Điểm kết nối các thành phần nội bộ của module
├── rules/                 # Business rule thuộc sở hữu của module
├── models/                # Dữ liệu và trạng thái nghiệp vụ của module
└── tests/                 # Test đặt gần module nếu phù hợp với ngôn ngữ được chọn
```

## 8.2. Module lớn

```text
example-module/
├── README.md              # Mô tả boundary và trách nhiệm của module
├── public.*               # Public API/interface của module
├── workflows/             # Luồng nghiệp vụ gồm nhiều bước
├── rules/                 # Business rule thuần của module
├── models/                # Model, value object và trạng thái nghiệp vụ
├── interfaces/            # Interface module cần từ dependency bên ngoài
├── adapters/              # Adapter kết nối interface với implementation
├── internal/              # Implementation không được import từ bên ngoài
└── tests/                 # Test module
```

Không bắt buộc mọi module có toàn bộ thư mục trên. Chỉ tạo thư mục khi có trách nhiệm và code thực tế.

---

# 9. Quy tắc phụ thuộc thống nhất

## 9.1. Luồng phụ thuộc cấp cao

```text
main
  → bootstrap
      → config
      → routing
      → module public interfaces
      → shared/platform implementations

routing
  → module public interface

module
  → public interface của module khác
  → contracts
  → shared
  → interface do chính module định nghĩa

platform/provider adapter
  → implements interface của module
```

## 9.2. Các quy tắc bắt buộc

1. `main.*` chỉ được gọi `bootstrap` và khởi chạy application.
2. `bootstrap` chỉ khởi tạo và kết nối thành phần, không chứa business logic.
3. `routing` chỉ phân loại input, validate boundary và gọi module phù hợp.
4. Routing không truy cập database hoặc provider trực tiếp.
5. Module khác chỉ được sử dụng `public interface` của module.
6. Không import file sâu trong `internal/` của module khác.
7. `shared/` không chứa business rule của một module cụ thể.
8. `contracts/` chỉ định nghĩa dữ liệu, interface, trạng thái và error code; không chứa implementation.
9. Repo kính chỉ truy cập phần cứng thông qua `platform/` abstraction.
10. App và kính không gọi trực tiếp OCR/VLM/payment provider.
11. Provider API key chỉ tồn tại trong Host runtime hoặc secret manager.
12. Không sao chép business rule từ Host sang Mobile hoặc Glasses.
13. Mọi thay đổi API phải cập nhật contract và contract test.
14. Không tạo thư mục rỗng chỉ để “đủ kiến trúc”.
15. Mỗi thư mục mới phải có code thực tế hoặc `README.md` giải thích trách nhiệm.

---

# 10. Quy tắc đặt tên

- Tên thư mục phản ánh chức năng: `device-linking`, `usage-quota`, `camera-capture`.
- Không đặt tên theo công nghệ khi chưa chọn công nghệ.
- Tránh tên mơ hồ: `helpers`, `misc`, `common2`, `manager-stuff`.
- `shared` chỉ dành cho thành phần thật sự được nhiều module sử dụng.
- `routing` là điều phối input, không phải nơi chứa toàn bộ workflow.
- `bootstrap` là khởi tạo/lắp ráp, không phải nơi xử lý nghiệp vụ.
- Tên file tuân theo convention của ngôn ngữ sau khi công nghệ được chọn.

---

# 11. Những thành phần chưa được quyết định

Cấu trúc này chưa quyết định và không được tự giả định:

- Ngôn ngữ lập trình.
- Backend framework.
- Mobile framework.
- Hệ điều hành hoặc board của kính.
- Database và cache.
- Cloud provider.
- Container/runtime technology.
- Message broker.
- CI/CD platform.
- OCR/VLM provider cụ thể.

Khi một công nghệ được chọn, file cấu hình và implementation tương ứng được bổ sung vào đúng boundary đã định nghĩa, không làm thay đổi trách nhiệm cấp cao của repository.

---

# 12. Trình tự phát triển đề xuất

1. Chốt context, trách nhiệm và boundary của bốn repository.
2. Chốt task type, status, error code và API contract.
3. Tạo skeleton `main`, `bootstrap`, `routing`, `modules/features`, `contracts`, `shared`, `config`.
4. Chỉ tạo module đang thực sự được phát triển.
5. Xây Host mock hoặc mock Host trong `dev-env`.
6. Xây Glasses Simulator trước khi phần cứng hoàn thiện.
7. Phát triển Mobile và Glasses song song dựa trên contract.
8. Chọn framework dựa trên yêu cầu thực tế.
9. Bổ sung implementation phụ thuộc framework vào đúng boundary.
10. Viết contract test và integration test cho các luồng quan trọng.

---

# 13. Các luồng tích hợp bắt buộc phải kiểm thử

```text
1. User đăng nhập → liên kết kính → thanh toán → subscription active.

2. Kính xác thực → gửi READ_TEXT → Host kiểm tra quyền → gọi OCR → trả text → kính TTS.

3. Kính gửi DESCRIBE_SCENE → Host gọi VLM → trả text → kính TTS.

4. Subscription expired → Host từ chối task → kính đọc thông báo gia hạn.

5. Device locked/lost → Host từ chối mọi task.

6. OCR/VLM timeout → Host trả error code chuẩn → kính phát phản hồi phù hợp.

7. Mất mạng → kính chuyển sang offline-mode và không treo luồng tương tác.
```

# Service map

Port cụ thể **CHƯA QUYẾT**. Mỗi service phải có mục đích rõ; không thêm service chỉ để tăng độ phức tạp.

| Service | Mục đích | Port | Healthcheck | Profile |
|---|---|---|---|---|
| postgres | Nguồn dữ liệu nghiệp vụ local | **CHƯA QUYẾT** | readiness DB | `core` |
| redis | Cache/queue/idempotency local | **CHƯA QUYẾT** | ping/readiness | `core` |
| host | Backend local | **CHƯA QUYẾT** | liveness + readiness | `core` |
| ocr-mock | OCR không trả phí | **CHƯA QUYẾT** | readiness | `mocks` |
| vlm-mock | VLM không trả phí | **CHƯA QUYẾT** | readiness | `mocks` |
| glasses-simulator | Phần mềm kính giả lập | **CHƯA QUYẾT** | **CHƯA QUYẾT** | `simulator` |
| observability | Công cụ quan sát nếu thật sự cần | **CHƯA QUYẾT** | **CHƯA QUYẾT** | `observability` |

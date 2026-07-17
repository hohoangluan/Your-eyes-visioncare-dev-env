# Your Eyes VisionCare dev-env

Repository cung cấp môi trường phát triển và tích hợp local cho bốn repo VisionCare (§9.4). Nó không chứa business logic và không thay thế production infrastructure.

**`PROJECT_ARCHITECTURE.md` đã chuyển sang [repository workspace](../PROJECT_ARCHITECTURE.md) theo ADR-008.** Người clone dev-env từ v1.0 phải đọc bản ở workspace.

Clone bốn repo không thuộc dev-env; dùng `workspace/scripts/setup.ps1` (§5). Dev-env chỉ validate sibling rồi chạy môi trường.

## Luồng local

Quy trình chuẩn nằm ở §9.4 và [docs/integration-flow.md](docs/integration-flow.md).

```text
bootstrap/          # validate và init
routing/            # service/proxy/port
manifests/          # service local
mocks/              # provider/client mocks
seed/               # dữ liệu dev
integration-tests/  # luồng xuyên repo
env/                # local override templates
```

Cây đầy đủ nằm ở §9.4. Trạng thái hiện tại: chỉ tài liệu và skeleton; chưa có manifest, script, mock hay business code.

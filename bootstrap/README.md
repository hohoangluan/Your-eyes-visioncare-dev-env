# bootstrap

Chuẩn bị môi trường local. Thành phần dự kiến theo §9.4: `validate-workspace`, `validate-environment`, `init-dependencies`, `init-seed-data`.

`validate-workspace` **không clone repository**; nó chỉ kiểm tra bốn repo sibling, fail rõ khi thiếu và trỏ sang `your-eyes-visioncare-workspace/scripts/setup.ps1` (§5, §9.4, ADR-008). Ranh giới: workspace lấy code về, dev-env chạy code.

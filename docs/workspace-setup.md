# Thiết lập workspace cho dev-env

Phần clone thuộc `your-eyes-visioncare-workspace/docs/workspace-setup.md` theo ADR-008; không chép lại ở đây.

Sau khi đủ bốn repo sibling, prerequisite chạy môi trường gồm container runtime **CHƯA QUYẾT**, Git cho build context và PowerShell 7 cho script tương lai. Thứ tự khởi động: dependency → Host → mock → simulator; Mobile chạy ngoài container.

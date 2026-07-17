# Integration tests

Danh sách chín luồng bắt buộc nằm ở §17; không chép lại. Ánh xạ trách nhiệm:

| Luồng §17 | Thư mục phủ |
|---|---|
| 1 | `device-linking/`, `subscription-activation/` |
| 2, 3, 7 | `glasses-task/` |
| 4 | `subscription-expiration/` |
| 5, 6 | `access-denied/` |
| 8 | `subscription-activation/` |
| 9 | `glasses-task/` phối hợp simulator offline |

Test không phụ thuộc provider trả phí (N9). **Chưa có test nào**.

# Đồng bộ contract

Contract được đồng bộ bằng **lệnh codegen**, không copy tay (§13): Host publish OpenAPI và JSON schema từ `src/contracts/` ra `docs/api/`, sau đó Mobile/Glasses codegen từ cùng artifact version.

Cả ba repo chạy contract test trên đúng version đó; lệch là fail build (§13).

## CHƯA QUYẾT

- Publish artifact bằng release asset hay package registry?
- Quy tắc version artifact là gì?
- Codegen tool nào cho Mobile sau ADR-005?
- Codegen tool nào cho Smart Glasses sau ADR-006?

Vì các lựa chọn này chưa chốt, lệnh cụ thể chưa tồn tại.

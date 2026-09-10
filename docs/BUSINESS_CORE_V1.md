# BUSINESS CORE V1 — BETA CONTRACT

Status: `BETA_IMPLEMENTATION`

## Nguồn thiết kế

Business Core V1 bám các quyết định đã chốt cho DC Core và strong-reference model Pick Pack 1291: D1 là authority; Google Sheets chỉ là projection; mutation về sau phải dùng immutable event + idempotency + entity version; correction/reversal là event mới, không rewrite raw event.

Không migrate dữ liệu employee/resource/history từ hệ thống Pick Pack cũ. BETA mới chỉ nhận synthetic/controlled test data khi mutation/auth layer được mở.

## Schema baseline

Migration `worker/migrations/0001_business_core.sql` tạo các nhóm dữ liệu:

- cluster + shift definitions;
- employee + cluster membership;
- resource master: PDA, USER_PICK, BAN_PACK, USER_PACK;
- work session + session/resource bindings;
- labor records;
- dropped goods / nhận hàng rớt;
- document metadata, binary tiếp tục nằm Drive;
- immutable domain events;
- conflict/correction state;
- D1 projection outbox + workbook catalog;
- import audit.

`domain_events` bị chặn UPDATE và DELETE bằng database trigger. Idempotency key và `(device_id, device_seq)` có unique index khi có giá trị.

## Public API hiện tại

Các endpoint không chứa PII/nghiệp vụ:

- `GET /health`
- `GET /health/deep`
- `GET /health/integrations`
- `GET /api/v1/meta`
- `GET /api/v1/capabilities`

Mọi route dưới `/api/v1/data/*` và `/api/v1/admin/*` trả `401 AUTH_REQUIRED` cho tới khi session/permission contract được triển khai. Không mở anonymous mutation và chưa bật cross-origin API.

## Health semantics

D1 là canonical business authority nên D1/schema là dependency critical của core health. Google Gateway/Sheets là projection/archive bất đồng bộ; lỗi tạm thời ở Google được báo `degraded` nhưng không làm core write path fail. `projection_outbox` giữ backlog để retry/ack/checkpoint thay vì buộc mutation phụ thuộc đồng bộ vào Google.

BETA đã quan sát một trường hợp Google Apps Script trả HTTP 403 cho subrequest phát ra từ Cloudflare Worker trong khi cùng deployment vẫn truy cập được từ GitHub runner. Vì vậy direct Worker→GAS không được dùng làm availability gate cho D1 core. Transport projection sẽ được kiểm chứng riêng trước khi mở dữ liệu thật.

## Projection

Schema có `projection_outbox` và `projection_catalog` để chuẩn bị one-writer/batch projection sang workbook theo `environment + cluster_id + quarter`. Việc tạo workbook/append projection chưa được kích hoạt trong migration này.

## Next

Khóa session/auth/permission contract theo Master Spec, sau đó mở mutation BETA có idempotency/version/device-sequence và triển khai Google projection worker/gateway theo outbox batch. Root login cụ thể không được tự suy đoán nếu chưa đọc lại đúng semantics từ Master Spec.

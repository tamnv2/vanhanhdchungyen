# DECISIONS

## D-001 — Repository PUBLIC

Repo giữ PUBLIC để dùng GitHub-hosted Actions không phụ thuộc quota private repo hoặc self-hosted runner. Hệ quả: không được commit bất kỳ secret/private key nào.

## D-002 — Tách môi trường

BETA và STABLE có token, Apps Script, Drive runtime, D1, Worker và signing key riêng.

## D-003 — Public hostnames

- BETA: `beta.supra.cc.cd`
- STABLE: `supra.cc.cd`

## D-004 — LAN isolation

LAN hostname không có public DNS record. LAN routing/resolution sẽ được cấu hình riêng trong mạng nội bộ.

## D-005 — Cloudflare runtime

Worker là public service layer; D1 là structured store chính. Durable Objects dành cho lock/rate-limit/concurrency. R2 mặc định OFF.

## D-006 — Google runtime

Drive/Sheets được truy cập qua Google Gateway/Apps Script. Hai Apps Script project giữ deployment ID cố định và CI cập nhật source/version vào deployment đó.

## D-007 — Stable promotion

STABLE chỉ deploy khi Owner chốt và GitHub Environment `stable` yêu cầu approval.

## D-008 — Authority và continuity

GitHub lưu bootstrap/current-state/decisions/task-ledger/changelog để AI đọc lại ở mọi phiên.

## D-009 — Changelog append-only

Mỗi thay đổi version/release phải thêm entry mới; không sửa/xóa lịch sử để làm sạch bề ngoài.

## D-010 — DC Core + cluster rollout

Nền tảng phục vụ toàn DC; cluster là operational boundary có `cluster_id`, không hard-code một cluster thành toàn bộ nghiệp vụ. Pick Pack 1291 là cluster BETA đầu tiên để build/test/stress/soak trước khi promote STABLE.

## D-011 — Legacy Pick Pack chỉ là strong reference

Được đọc/reuse có chọn lọc logic/schema/UI/test từ Pick Pack 1291 cũ. Không write/deploy/runtime fallback sang tài nguyên cũ và không migrate employee/resource/history cũ vì dữ liệu đó là test.

## D-012 — Immutable event + correction

Canonical mutation phải có immutable event, idempotency, entity version và device sequence khi áp dụng. Raw event không UPDATE/DELETE; correction/tombstone/reversal là event mới và conflict phải lưu evidence + resolver decision.

## D-013 — Google Sheets projection only

Sheets là human-readable projection/archive/đối soát, không phải business authority. Projection dùng D1 outbox + batch one-writer; app/web/PDA không ghi trực tiếp nhiều tab.

## D-014 — Free-first

Tối ưu request/CPU/D1/Drive/Sheets trước khi Paid. Không polling liên tục; dùng indexed query, delta, batch và archive verified. Free capacity được quyết định bằng stress/soak thực tế.

## D-015 — Beta/Stable runtime isolation

BETA/STABLE tách D1/Worker/GAS/OAuth/Drive/signing/runtime state. Promotion là source/schema/config đã duyệt, không copy wholesale runtime data.

## D-016 — Không public business data trước auth

Repo và Worker public không đồng nghĩa dữ liệu nghiệp vụ public. Trước khi session/permission layer hoàn tất, chỉ health/meta/capabilities được anonymous; data/admin routes bị đóng và anonymous mutation bị cấm.

## D-017 — Projection integration không chặn canonical core

D1 là authority và mutation phải commit được độc lập với availability tức thời của Google. Google Sheets/Gateway là async projection/archive; lỗi integration được báo `degraded` và xử lý bằng outbox/retry/ack/checkpoint, không biến thành lỗi canonical write. Direct Worker→GAS chỉ là probe/transport candidate, không phải dependency critical cho health hoặc transaction.

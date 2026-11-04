# Quy trình phối hợp và tích hợp — v1.5

Ngày chốt: 04/11/2026. Hạn nộp: 22/11/2026. Đây là quyết định thiết kế và kế hoạch, chưa phải triển khai code.

## 1. Owner và hỗ trợ

| Người | Owner chính | Reviewer / kiểm tra tích hợp |
|---|---|---|
| TV1 — Quốc Thịnh | Auth, common-auth/errors, Forum API/UI, Profile, test plan, PO | Review Core của TV3; xác nhận tiêu chí nghiệm thu |
| TV2 — Tùng | Run/Judge/worker/sandbox, Leaderboard, SM | Review Auth TV1; FE gọi Run/Submit TV4; bảo mật execution |
| TV3 — Hiếu | Core/catalog/levels/history, common-db/levels, migration/seed; Git Lead | Review TV2 về DB/transaction; Compose/CI và cấu hình chung |
| TV4 — Huy | Next.js, shared UI, editor, Admin, các trang trừ Forum/Profile | Review UI của TV1; contract frontend/backend |

**Hỗ trợ được phân công:** Hiếu (TV3) hỗ trợ Tùng (TV2) về DB/transaction, shared package và tích hợp Core–Judge; Quốc Thịnh (TV1) hỗ trợ Huy (TV4) về Auth, phân quyền, API user và tích hợp UI. Owner chính giữ trách nhiệm đầu ra và cập nhật trạng thái task. Người tham gia viết phần thay đổi cần một thành viên khác review trước khi hoàn thành.

AI hỗ trợ toàn dự án nhưng mỗi lượt làm bám một task/owner cụ thể: đọc yêu cầu, kiểm tra phụ thuộc, triển khai trong phạm vi, kiểm thử, đưa kết quả review. Không tự nhận task của thành viên khác là đã hoàn tất. Trước sửa shared package hoặc config phải kiểm tra các bên dùng nó. Nếu phát hiện xung đột, báo mã task, tệp/hợp đồng bị ảnh hưởng, owner và cách xử lý. Không ghi đè thay đổi chưa commit của người khác. Không đóng vai reviewer con người hoặc PO nghiệm thu thay team.

## 2. Hợp đồng phải thống nhất

- OpenAPI theo service, request/response/error/pagination được version cùng PR. Mock FE dựa đúng hợp đồng; client type sinh từ OpenAPI. API thay đổi phải cập nhật cả mock, client và acceptance tests liên quan.
- Một Go module, một bộ phiên bản backend; frontend một package.json và lockfile. CI kiểm tra gofmt, go vet, golangci-lint, go test, TypeScript, lint và next build. Go test -race cho logic concurrency trên runner hỗ trợ; PostgreSQL/Redis thật cho integration tests.
- Biến môi trường thống nhất trong .env.example không có secret: AUTH_DATABASE_URL, JUDGE_DATABASE_URL, REDIS_URL, JWT_ISSUER, JWT_AUDIENCE, JWT_PRIVATE_KEY_PATH, JWT_PUBLIC_KEY_PATH, INTERNAL_SERVICE_TOKEN, AUTH_INTERNAL_URL, CORE_INTERNAL_URL, RUN_INTERNAL_URL, JUDGE_INTERNAL_URL, WEB_ORIGIN. NEXT_PUBLIC_* chỉ chứa dữ liệu công khai; tuyệt đối không DB URL/secret.
- Port dev nội bộ đề xuất: Next 3000; Auth 8081; Core 8082; Run 8083; Judge API 8084; PostgreSQL 5432; Redis 6379. Docker dùng service DNS, không localhost để liên lạc giữa container. Chỉ Next public ở môi trường demo, backend giữ mạng nội bộ.
- Next proxy theo route map cụ thể: /auth/*, /users/me*, /admin/users* → Auth; /problems/{slug}/run và /runs/* → Run; POST /problems/{slug}/submissions, GET /submissions/jobs/* → Judge; còn lại Core. Không chuyển GET /submissions/{id} sang Judge. /internal/* không public. Cookie Set-Cookie/Origin/Authorization phải được bảo toàn và test.
- TV3 quản lý migration Judge, TV1 Auth. Migration đã merge không sửa lại; thêm migration mới. Unique/check/FK baseline giữ nguyên. Test dựng DB sạch và nâng cấp DB có dữ liệu trước merge.
- Sandbox nhận input qua stdin, không mount DB/secret/output mong đợi. Giới hạn CPU/memory/pids/file/output, network none, non-root, read-only, seccomp, cleanup. Java có cấu hình JVM phù hợp memory limit.
- Job và run có user_id, chỉ chủ sở hữu xem; queue giao lại không tạo AC trùng. Giới hạn timeout, retry, TTL, concurrency phải dùng config chung và kiểm thử khi Redis/worker lỗi.

## 3. Lịch và cổng nghiệm thu

| Ngày | TV1 — Quốc Thịnh | TV2 — Tùng | TV3 — Hiếu | TV4 — Huy | Điều kiện tích hợp |
|---|---|---|---|---|---|
| 04–05/11 | Auth schema, error/JWT contract, middleware | Sandbox/queue contract, Python runner | Go module, baseline SQL/Goose, pgx/sqlc, Compose/CI | Next.js/TS, routes/layout, shared UI, Axios/mock | Khung chạy Compose; version/env/API thống nhất |
| 06–07/11 | Register/login/refresh/logout; user lookup/admin | Python sandbox và test cô lập | Catalog/language/template, levels, AC repository | Auth/Landing, route guards | Login thật; auth middleware dùng chung; Python chạy an toàn |
| 08–11/11 | Admin user/test plan; soạn bài | C++/wrapper, Run, Judge API/Asynq | Admin sample/test/template, publish guard, seed 6 bài | Monaco/gói/bài/Run; Admin nội dung | Run Python/C++; không ghi DB; không lộ hidden tests |
| 12–15/11 | Forum CRUD, hồ sơ khi kịp | Java, verdict, AC transaction/idempotency, levels/IE | History, ownership, seed 18 bài; hỗ trợ concurrency | Submit/polling/history | Submit đầy đủ 3 runtime; duplicate không tăng count; level lock đúng |
| 16–19/11 | Forum comments/likes/moderation/XSS và UI | Leaderboard/cache, E2E | Seed ≥30 bài, audit Core; Landing Should | Leaderboard, Admin moderation, responsive | Must đạt; đóng tính năng 19/11 lúc 21:00 |
| 20–21/11 | PO nghiệm thu, tài liệu/demo | Sandbox attack/load/regression | Security/migration sạch/seed/backup | Cross-browser/build/regression | CI xanh, không lỗi nghiêm trọng; chạy demo từ DB sạch |
| 22/11 | Nộp dự án trước giờ giảng viên quy định | Trực hỗ trợ Judge | Trực DB/deploy | Trực frontend | Không thêm tính năng mới |

Giữ 67 task và 72 story; phân bổ chi tiết ngày trước đó vẫn dùng, thay các đầu ra Python backend bằng Go tương ứng. Không thêm runtime Go cho bài nộp. Nếu 11/11 Run chưa đạt hoặc 15/11 AC/levels chưa đạt: dừng Should, dồn hỗ trợ vào Must và báo PO; không hạ chuẩn sandbox để chạy kịp. Chưa biết số giờ/ngày nên lịch là mục tiêu, không phải bảo đảm khả thi.

## 4. Trước và sau mỗi task

1. Đọc SRS v1.5 và acceptance story; kiểm tra git status, nhánh, owner; không overwrite việc dở của người khác.
2. Kiểm tra task chặn: API đã có hay mock, package/config/migration đã merge chưa; ghi blocker cụ thể.
3. Thực hiện task trên feature/<task-id>-<slug>; tránh đổi dependency/contract ngoài phạm vi không có lý do.
4. Kiểm thử logic task và integration bị ảnh hưởng. FE đổi phải build; DB đổi phải migrate trên DB sạch; shared package đổi phải test các consumer.
5. PR ghi task, thay đổi, validation, breaking changes và owner bị ảnh hưởng; một người khác review. Chỉ tick khi merge/CI/PO đạt DoD.

Các workbook v1.4 là lịch tham chiếu, còn đề cập stack cũ; khi dùng để giao task phải áp dụng mapping v1.5: Alembic→Goose, SQLAlchemy→pgx/sqlc, FastAPI→Go/chi, Celery/RQ→Asynq, React skeleton→Next.js/TS. Không sửa cột ngày và điểm chỉ vì đổi stack.

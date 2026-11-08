# Coding Judge Platform — tài liệu hiện hành

Đồng bộ ngày 08/11/2026 từ backend `code-automatic/codjingrank-backend`,
`main` tại `68b0244`. Các tài liệu SRS cũ ở thư mục gốc đã được thay bằng bản
trong backend; đường dẫn tương đối được điều chỉnh cho repository này.

## Tài liệu dùng để nghiệm thu và làm frontend

- [SRS 2.1](01_SRS_Coding_Judge_Platform.md): phạm vi, yêu cầu F01–F13.
- [Frontend contract](docs/FRONTEND_API_CONTRACT.md): endpoint, ownership,
  compatibility và các giới hạn tích hợp.
- [OpenAPI đầy đủ](docs/openapi.yaml) / [learner](docs/openapi_fe.yaml): request,
  response, quyền, lỗi và ví dụ để frontend mock.
- [Response envelope](docs/fe/RESPONSE_CONTRACT.md) và
  [auth flow](docs/fe/AUTH_FLOW.md).
- [Quyết định đã chốt](docs/DECISIONS.md), [AI context](docs/AI_CONTEXT.md).
- [Schema](schema.sql) và
  [thiết kế database](docs/superpowers/specs/2026-11-06-database-schema-design.md).
- [Kế hoạch tích hợp](04_KE_HOACH_TICH_HOP.md).

## Trạng thái nghiệm thu

Envelope đã triển khai và merge qua PR #1. Ownership và compatibility đã có
contract, nhưng ownership enforcement và Gateway `/api/v1` chưa hoàn tất.
OpenAPI là target để mock, không phải danh sách API live. Avatar/bio đã có
trong schema; Go DTO/query/handler còn cần tích hợp UUID/schema mới.

`schema.sql` là bootstrap phá dữ liệu, chỉ chạy trên database mới dành riêng.
[PROJECT_STATE](PROJECT_STATE.md) là snapshot của backend, không mô tả Git của
repo tài liệu. [Reality audit](docs/BACKEND_REALITY_AUDIT.md) có phân tích legacy;
đọc cảnh báo cleanup và context hiện hành trước khi dùng các bảng lịch sử.

## Phân công và backlog

[02](02_PHAN_CONG_NHIEM_VU.md) và [03](03_BACKLOG_THEO_SPRINT.md) là bản kế hoạch
cũ được sao chép từ backend; chưa đồng bộ với phân công Excel đã chốt
(BE/DevOps: Trung Hiếu; FE/QA: Hoàng Huy, Quốc Thịnh, Thanh Tùng).
Các tiêu chí cũ như lỗi nested, Asynq hoặc DB Auth riêng không thay thế
DECISIONS, schema và OpenAPI hiện hành. Không coi bảng kế hoạch là bằng chứng
đã triển khai. Lần đồng bộ này không đổi phân công hay lịch của Excel.

## Mock

```bash
npx @stoplight/prism-cli mock docs/openapi_fe.yaml -p 4010
```

Chỉ đồng bộ file local; chưa commit, push hay mở PR trong repository SRS.

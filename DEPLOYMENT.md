# Thông tin deploy — CP5

> Cập nhật các mục trạng thái, URL và kết quả sau khi deploy thật. Không ghi
> giá trị API key, password, token hoặc credential vào file này.

## Thông tin học viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Nguyen Cong Duan (theo tên repository; xác nhận/cập nhật cách viết chính thức nếu cần) |
| Mã học viên | 2A202602716 |
| Repository | https://github.com/duanhap/K4-L3B-DAY12-NguyenCongDuan-2A202602716-CloudServicesAndDeployment |

## Service Railway

| Mục | Trạng thái |
|---|---|
| Public URL | Chưa deploy — cập nhật sau khi Railway tạo domain |
| Platform | Railway (đã chọn; deployment chưa xác minh) |
| Ngày deploy | Chưa deploy |

## Biến môi trường trên Railway

Ghi trạng thái và nguồn của biến sau khi cấu hình. Chỉ ghi tên biến và nguồn,
không ghi giá trị bí mật.

| Biến | Trạng thái | Nguồn/ghi chú |
|---|---|---|
| `PORT` | Chờ deploy | Railway tự cấp; không tự đặt giá trị cố định |
| `AGENT_API_KEY` | Chưa cấu hình/xác minh | Tạo giá trị riêng và lưu trong Railway Variables |
| `REDIS_URL` | Chưa cấu hình/xác minh | Tham chiếu biến kết nối của Redis service Railway |
| `RATE_LIMIT_PER_MINUTE` | Chưa cấu hình/xác minh | Khuyến nghị giá trị lab: 10 |
| `MONTHLY_BUDGET_USD` | Chưa cấu hình/xác minh | Khuyến nghị giá trị lab: 10.0 |
| `LOG_LEVEL` | Chưa cấu hình/xác minh | Khuyến nghị: INFO |

## Các bước triển khai

1. Push nhánh chứa code CP1–CP5 lên repository GitHub ở trên.
2. Tạo project Railway và thêm service từ repository; Railway sẽ phát hiện Dockerfile ở thư mục gốc.
3. Thêm Redis service riêng trong cùng project. Railway không chạy trực tiếp toàn bộ `docker-compose.yml`; cấu hình Redis local trong Compose không tự tạo database trên Railway.
4. Trong Variables của service agent, cấu hình `AGENT_API_KEY`, `REDIS_URL`, `RATE_LIMIT_PER_MINUTE`, `MONTHLY_BUDGET_USD`, `LOG_LEVEL`. Dùng reference tới URL Redis của service; để Railway tự cấp `PORT`.
5. Đợi deployment build và chạy thành công. Kiểm tra build/runtime logs; trong Settings → Networking, tạo public domain.
6. Ghi domain thật vào bảng Service, cập nhật trạng thái biến, lưu ảnh dashboard và health vào `screenshots/`.

`railway.toml` dùng Dockerfile ở repo root, chạy Uvicorn trên `$PORT` và đặt healthcheck path `/health`.

## Kiểm tra sau deploy

Trong PowerShell, nhập public URL thật khi được nhắc; dùng API key lưu trong biến môi trường cục bộ `DEPLOY_API_KEY` (không commit biến này).

```powershell
$baseUrl = (Read-Host "Nhập public URL Railway").TrimEnd('/')

# Liveness: mong đợi 200 và status=ok
Invoke-RestMethod "$baseUrl/health"

# Readiness/Redis: mong đợi 200 và status=ready
Invoke-RestMethod "$baseUrl/ready"

# Không có API key: mong đợi 401
Invoke-RestMethod -Uri "$baseUrl/ask" -Method Post `
  -ContentType "application/json" -Body '{"question":"Hello"}'

# Có key: mong đợi 200 cùng câu trả lời
$headers = @{ "X-API-Key" = $env:DEPLOY_API_KEY; "X-User-Id" = "sv-test" }
$body = @{ question = "Deploy là gì?" } | ConvertTo-Json -Compress
Invoke-RestMethod -Uri "$baseUrl/ask" -Method Post `
  -ContentType "application/json" -Headers $headers -Body $body
```

Kiểm tra rate limit bằng cách gửi nhiều request có xác thực trong một phút. Ghi
status code thực tế; các request vượt hạn mức phải nhận `429`.

## Kết quả chạy thật

Chưa deploy; chưa có URL hoặc output kiểm tra thực tế. Sau deploy, lưu output
thật của `/health`, `/ready`, request thiếu key, request có key và rate limit ở đây.

## Ảnh minh chứng

- `screenshots/dashboard.png` — trang service/deployment Railway.
- `screenshots/health.png` — kết quả gọi `/health` sau deploy.

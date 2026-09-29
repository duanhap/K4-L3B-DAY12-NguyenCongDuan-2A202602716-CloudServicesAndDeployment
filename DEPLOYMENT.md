# Thông tin deploy — Checkpoint 5

> Cập nhật trạng thái, URL và kết quả sau deployment thật. Không ghi giá trị
> API key, password, token hoặc credential vào file này.

## Thông tin học viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Nguyen Cong Duan (theo tên repository; xác nhận/cập nhật cách viết chính thức nếu cần) |
| Mã học viên | 2A202602716 |
| Repository | https://github.com/duanhap/K4-L3B-DAY12-NguyenCongDuan-2A202602716-CloudServicesAndDeployment |

## Service Render

| Mục | Trạng thái |
|---|---|
| Public URL | Chưa deploy — lấy URL onrender.com sau khi Blueprint tạo web service |
| Platform | Render Blueprint (đã chọn; deployment chưa xác minh) |
| Ngày deploy | Chưa deploy |

## Biến môi trường

Chỉ ghi tên biến và nguồn/trạng thái; không ghi giá trị secret.

| Biến | Trạng thái | Nguồn/ghi chú |
|---|---|---|
| `PORT` | Chờ deploy | Render cấp runtime port; ứng dụng đọc biến này |
| `AGENT_API_KEY` | Chưa cấu hình/xác minh | Nhập trực tiếp vào Render khi Blueprint yêu cầu; không lưu trong Git |
| `REDIS_URL` | Chờ tạo service | Blueprint tham chiếu connection string của Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | Blueprint đặt mặc định | 10 |
| `MONTHLY_BUDGET_USD` | Blueprint đặt mặc định | 10.0 |
| `LOG_LEVEL` | Blueprint đặt mặc định | INFO |

## Các bước triển khai

1. Push code CP1–CP5 cùng `render.yaml` lên nhánh GitHub cần deploy.
2. Trong Render Dashboard, chọn **New → Blueprint** và kết nối repository trên.
3. Kiểm tra Blueprint nhận diện web service `day12-agent` và Key Value `day12-redis`; chọn Free plan nếu tài khoản cho phép.
4. Khi được hỏi giá trị `AGENT_API_KEY`, tạo/dán khóa riêng trong Render Dashboard. Không ghi khóa vào repository.
5. Apply Blueprint và theo dõi build/deploy logs. Blueprint sẽ cấp `REDIS_URL` cho web service từ Key Value; Render tự cấp `PORT` lúc runtime.
6. Chờ health check `/ready` thành công. Mở web service để lấy public `onrender.com` URL, rồi cập nhật bảng Service ở trên.
7. Lưu ảnh dashboard và kết quả health vào `screenshots/`.

`render.yaml` build web service từ `Dockerfile`, đọc `PORT`, lấy `REDIS_URL` từ Key Value và dùng `/health` làm platform health check. Sau deploy, gọi riêng `/ready` để xác minh Redis.

Với Free plan, web service có thể ngủ sau 15 phút không có traffic và mất khoảng một phút để thức lại. Render Key Value Free lưu dữ liệu trong bộ nhớ; nếu instance Redis restart thì history, rate-limit và cost counters có thể bị xóa. Phù hợp cho lab/demo; không xem đó là lưu trữ bền vững.

## Kiểm tra sau deploy

Trong PowerShell, nhập URL public thật khi được nhắc. Để kiểm tra request có key, đặt giá trị `AGENT_API_KEY` của Render vào biến môi trường cục bộ `DEPLOY_API_KEY`; không commit biến này.

```powershell
$baseUrl = (Read-Host "Nhập public URL của Render").TrimEnd('/')

# Liveness: mong đợi 200, status=ok
Invoke-RestMethod "$baseUrl/health"

# Readiness/Redis: mong đợi 200, status=ready
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

Gửi nhiều request có xác thực trong một phút để kiểm tra rate limit; ghi lại
status code thực tế. Các request vượt hạn mức phải nhận `429`.

## Kết quả chạy thật

Chưa deploy; chưa có URL hoặc output kiểm tra thực tế. Sau deployment, lưu output
thật của `/health`, `/ready`, request thiếu key, request có key và rate limit ở đây.

## Ảnh minh chứng

- `screenshots/dashboard.png` — trang Blueprint/service/deployment Render.
- `screenshots/health.png` — kết quả gọi `/health` sau deployment.

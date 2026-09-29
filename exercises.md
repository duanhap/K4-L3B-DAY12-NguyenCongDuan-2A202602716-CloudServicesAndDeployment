# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Công Duẩn  Mã học viên: 2A202602716

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ khi em chạy docker compose up nhưng quên cấu hình AGENT_API_KEY, nếu có mặc định "changeme" thì container vẫn chạy bình thường làm em tưởng cấu hình đã đúng. Còn khi không có giá trị mặc định, app sẽ chết ngay lúc khởi động và log báo thiếu agent_api_key, nó giúp em phát hiện và sửa cấu hình ngay trước khi tiếp tục test hệ thống.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log thu được khi gọi `/ask` thành công: {"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T14:58:16.180167+00:00", "user_id": "sv01", "tokens_in": 3, "tokens_out": 35, "cost_usd": 2.145e-05}  So với print("đã trả lời xong"), log JSON giúp em lọc/thống kê theo từng field như user, token, chi phí và tạo cảnh báo tự động khi token hoặc chi phí vượt ngưỡng. Vì dữ liệu có cấu trúc rõ ràng nên hệ thống monitoring có thể đọc và xử lý tự động.


---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~287.7 MB |
| Multi-stage | ~271.16 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Image 1-stage của em khoảng 287.7 MB, còn multi-stage khoảng 271.16 MB, giảm khoảng 16.54 MB. Phần dung lượng giảm chủ yếu là các file và công cụ chỉ cần trong quá trình build/cài dependency như build cache, `wheel`, `setuptools` hoặc các thành phần build khác. Với multi-stage, các phần này được giữ ở builder stage, còn image cuối chỉ lấy những thứ cần thiết để chạy app nên nhẹ hơn.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile hiện tại, khi em chỉ sửa app/main.py thì requirements.txt không đổi nên Docker vẫn dùng cache cho bước pip install, chỉ các bước copy source phía sau phải chạy lại. Nếu để COPY . . trước RUN pip install thì chỉ cần sửa một file code cũng làm cache bị mất và pip install phải chạy lại, khiến build lâu hơn dù dependencies không thay đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python của em có lỗ hổng và bị khai thác, attacker có thể chạy lệnh bên trong container. Nếu app đang chạy bằng root thì họ cũng có quyền rất cao trong container và nếu khai thác thêm được lỗ hổng container escape thì có thể ảnh hưởng tới host. `USER appuser` cắt chuỗi này ở chỗ giới hạn quyền: dù app bị khai thác thì attacker cũng chỉ có quyền của `appuser`, giảm khả năng sửa file hệ thống hoặc leo quyền tiếp.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Nếu đếm theo từng phút cố định, user có thể gửi 10 request ở cuối phút trước và ngay sau khi counter reset lại gửi thêm 10 request ở đầu phút sau, tức có thể đạt 20 request chỉ trong khoảng 2 giây. Sliding window tránh được trường hợp này vì hệ thống luôn kiểm tra 60 giây gần nhất, nên 10 request vừa gửi vẫn được tính và các request tiếp theo sẽ bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong một khoảng thời gian, còn cost guard giới hạn tổng chi phí sử dụng. Ví dụ em gửi ít request nên không vượt rate limit, nhưng mỗi request dùng nhiều token làm hết ngân sách thì cost guard vẫn chặn. Ngược lại, em chưa tốn nhiều chi phí nhưng gửi quá nhiều request liên tục trong thời gian ngắn thì rate limit sẽ chặn trước, dù cost guard vẫn còn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp `/health` và `/ready`, khi Redis mất kết nối thì cả 3 agent đều có thể bị đánh dấu unhealthy và restart dù FastAPI vẫn đang chạy bình thường. Restart cũng không giải quyết được vì lỗi nằm ở Redis. Khi tách riêng, `/health` vẫn trả 200 để báo app còn sống, còn `/ready` trả 503 để báo tạm thời chưa sẵn sàng. Khi Redis hoạt động lại thì `/ready` tự về 200 mà không cần restart các agent.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi scale thành 3 agent, nếu lưu history bằng dict Python thì mỗi container sẽ có bộ nhớ riêng, nên `history_length` có thể thành `0, 0, 0, 1, 1, 1...` tùy request được chuyển vào agent nào. Nếu dùng Redis thì cả 3 agent cùng đọc/ghi một history chung nên lịch sử vẫn tăng liên tục. Ngoài ra nếu một container restart thì dict trong RAM sẽ mất, còn dữ liệu lưu ở Redis không phụ thuộc vào container agent đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi test /ask thiếu API key trên PowerShell, em nhận được kết quả là 422 thay vì 401 vì JSON body bị format sai, khiến FastAPI không parse được request trước khi kiểm tra API key. Sau khi đổi từ '{"question":"Hello"}' sang '{\"question\":\"Hello\"}', JSON được gửi đúng và server trả 401 Unauthorized như thiết kế. 

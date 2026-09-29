# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng giữ chỗ bằng câu trả lời thực tế.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Văn Duy
>
> Mã học viên: 2A202602729

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Một tình huống cụ thể là deploy lên Railway nhưng quên tạo biến
> `AGENT_API_KEY`. Nếu code có mặc định `"changeme"`, container vẫn báo healthy
> và endpoint `/ask` được bảo vệ bằng một khóa ai cũng đoán được. Khi
> `agent_api_key` là trường bắt buộc, Pydantic dừng process ngay trong build hoặc
> startup log. Lỗi xuất hiện lúc tôi còn đang quan sát deployment, trước khi URL
> công khai nhận request.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log tôi nhận được khi gọi `/ask`:
>
> ```json
> {"event":"ask_completed","level":"info","timestamp":"2026-09-29T02:46:12.091595+00:00","user_id":"2A202602729","tokens_in":9,"tokens_out":43,"cost_usd":2.715e-05}
> ```
>
> Với JSON này tôi có thể lọc tổng chi phí theo `user_id` và dựng cảnh báo khi
> `cost_usd` tăng bất thường. Tôi cũng có thể tính latency/tần suất lỗi theo mốc
> thời gian. Dòng `print("đã trả lời xong")` không có trường dữ liệu ổn định để
> máy tổng hợp hai thống kê đó.

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
| 1 stage (bản đầu) | 1.7 GB |
| Multi-stage | 305 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build lại hai bản và đo bằng `docker images`: bản một stage dùng
> `python:3.11` là 1.7 GB, bản multi-stage dùng `python:3.11-slim` là 305 MB,
> giảm khoảng 82%. Phần chênh lệch chủ yếu là Debian đầy đủ, compiler, header,
> cache và công cụ build chỉ cần khi cài package. Runtime stage chỉ giữ Python
> slim, virtualenv đã cài và source cần chạy.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile của tôi copy `requirements.txt`, tạo virtualenv và cài dependency
> trước khi copy `app/` và `utils/`. Khi chỉ sửa một ký tự trong `app/main.py`,
> các layer base image, copy requirements và pip install được dùng lại; chỉ
> layer copy source cùng các layer sau nó phải chạy lại. Nếu đặt `COPY . .`
> trước `RUN pip install`, mọi thay đổi source làm invalid cache và buộc tải/cài
> lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗ hổng thực thi lệnh, mã của kẻ tấn công sẽ chạy với
> user của process trong container. Process chạy root có thể sửa file hệ thống,
> cài công cụ và khai thác tiếp kernel hoặc volume mount để tác động host. Lệnh
> `USER appuser` chuyển process sang UID không đặc quyền trước khi Uvicorn chạy,
> nên lỗ hổng ứng dụng không tự động đem lại quyền root trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng hai giây: gửi 10 request ở cuối phút cũ,
> ví dụ 10:00:59, rồi gửi tiếp 10 request ngay đầu phút mới, 10:01:00. Bộ đếm
> theo phút đã reset dù 20 request nằm sát nhau. Sliding window luôn nhìn 60
> giây gần nhất nên request thứ 11 trong chuỗi đó bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong thời gian ngắn, còn cost guard giới hạn
> tổng tiền một user được phép tiêu trong tháng. Một request rất dài có thể vẫn
> nằm trong 10 request/phút nhưng làm tổng chi phí vượt 10 USD, nên cost guard
> phải chặn. Ngược lại, user có thể gửi 11 request cực ngắn khi ngân sách còn
> gần như nguyên; rate limiter chặn request thứ 11 dù cost guard vẫn cho phép.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp hai endpoint và `/health` cũng ping Redis, khi Redis mất kết nối cả ba
> container cùng trả health 503. Orchestrator coi cả ba process đã hỏng và
> restart đồng loạt. Container mới vẫn không kết nối được Redis nên tiếp tục bị
> restart, tạo restart loop và làm toàn bộ service mất khả dụng. Tách `/health`
> giúp process vẫn sống; `/ready` trả 503 để load balancer tạm ngừng gửi traffic
> cho tới khi Redis phục hồi.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Trong kiểm thử HTTP, request đầu trả `history_length=0`; request tiếp theo của
> cùng `X-User-Id` trả `history_length=2` vì Redis giữ cả message user và
> assistant. Hai `ConversationStore` khác nhau cùng nhìn thấy dữ liệu đó. Nếu
> dùng dict Python, ba container có ba dict riêng: request bị load balancer đưa
> sang instance khác sẽ thấy 0 hoặc một con số thấp không ổn định, và lịch sử
> biến mất khi instance restart.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Tôi thử Railway trước nhưng dashboard báo subscription đang `unpaid`, nên
> workspace có nguy cơ gián đoạn và không phù hợp làm evidence. Tôi xác định đây
> là vấn đề tài khoản chứ không phải code bằng banner billing trên dashboard.
> Tôi chuyển sang Render, dùng Blueprint từ `render.yaml`, khai báo
> `AGENT_API_KEY` trong secret prompt và để `REDIS_URL` lấy từ Render Key Value.
> Deployment sau đó lên trạng thái Live; `/health` và `/ready` đều trả 200.

# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder bên dưới mỗi câu hỏi bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đào Đức Hải  Mã học viên: 2A202602752

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> *để mặc định "changeme" thì kẻ tấn công có thể truy cập vào hệ thống, hoặc các ai và bot có thể thử các khóa phổ biến. cuối tháng khi check lại hóa đơn sẽ ra một số tiền khổng lồ/tài khoản cạn tiền*

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> *{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T10:21:15.123456+00:00", "user_id": "sv-test", "tokens_in": 4, "tokens_out": 37, "cost_usd": 0.0000234}

2 việc làm được: 
truy vấn và tổng hợp số liệu bằng máy
thiết lập cảnh báo tự động*

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
| 1 stage (bản đầu) | khoảng 1020 MB |
| Multi-stage | khoảng 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *image đầy đủ chứa toàn bộ hệ thống với các trình biên dịch, thư viện phát triển,tài liệu hướng dẫn... không cần thiết cho quá trình vận hành, trong khi đó multi-stage chỉ giữ lại kernel và runtime tối thiểu

multi stage build tách biệt hoàn toàn công đoạn cài đặt và chạy, chỉ có các package python sau khi cài đặt ở /install được copy qua image cuối cùng, các dependency không cần thiết sẽ bị loại bỏ, giúp image nhẹ hơn *

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> *các layer từ đầu đến bước cài dependency đều được dùng lại từ cache vì file requirement.txt không thay đổi, chỉ các layer từ COPY app ./app trở đi mới mất cache và phải chạy lại

Nếu đặt COPY . . lên trước RUN pip install
khi sửa bất kỳ ký tự nào trong app/main.py, layer COPY . . sẽ bị thay đổi hash, theo nguyên lý của Docker thì toàn bộ các layer phía sau nó đều bị vô hiệu hóa cache*

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> *1. mã nguồn Python có lỗ hổng (ví dụ lỗi RCE - Remote Code Execution từ thư viện phụ thuộc hoặc hàm eval/pickle).
2. kẻ tấn công gửi payload khai thác thành công và chiếm được shell thực thi lệnh bên trong container.
3. do container chạy dưới quyền root (UID 0), kẻ tấn công bên trong container sở hữu toàn quyền cao nhất (root privilege).
4. chúng lợi dụng các lỗ hổng nhân Linux (kernel escape), container breakout (như khai thác mount socket /var/run/docker.sock, cgroups, hoặc các đặc quyền Linux capabilities chưa bị drop) để thoát ra khỏi namespace của container.
5. khi đã thoát ra ngoài máy host, vì UID trong container ánh xạ trực tiếp tới root trên host, kẻ tấn công lập tức có quyền root toàn bộ máy chủ vật lý/máy ảo host.

lệnh USER cắt đứt chuỗi tấn công này ở bước 3: thay vì chạy bằng root, container chạy với UID bị cô lập, do đó dù kẻ tấn công chiếm được shell bên trong container, quyền hạn của chúng bị giới hạn ở mức user thường (không có quyền quản trị kernel hoặc host)*

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> *người dùng có thể gửi tối đa 20 request trong 2 giây liên tiếp
cách đạt được số đó: tại giây cuối cùng của phút N, người dùng gửi 10 request, và tại giây đầu tiên của phút N+1, người dùng tiếp tục gửi 10 request. trong 2 giây liên tiếp, hệ thống ghi nhận 20 request, vượt quá 10 request/phút*

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> *rate limit: giới hạn tần suất/số lượng request trong 1 đơn vị thời gian ngắn chống nghẽn mạng và tấn công DoS/spam
cost guard: giới hạn về số tiền/ngân sách tiêu thị trong khoảng thời gian dài dựa trên lượng token thực tế mà LLM xử lý

rate limit cho qua nhưng cost guard chặn: người dùng gửi 1 request nhưng đã tiêu tốn hết ngân sách
cost guard cho qua nhưng rate limit chặn: người dùng gửi 1 request nhưng lượng request trong 1 phút đã đầy*

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> *tại giây 0: redis mất kết nối
tại khoảng 5-10 giây: orchestrator gửi liveness probe tới /health của 3 container agent, kiểm redis thất bại nên 3 container phản hồi lỗi
tại giây 15: liveness check fail liên tiếp vượt số lần retry, orchestrator kết luận 3 container hỏng tiến trình và kích hoạt restart/kill 3 container
khoảng giây 20-30: 3 container đang bị tắt và khởi động lại từ đầu, khiến toàn cụm agent biến mất, không có container phục vụ người dùng
tại giây 30: redis trở lại bình thường, nhưng các container agent chưa kịp khởi động xong hoặc bị rơi vào vòng lặp restart, khiến sập hoàn toàn hệ thống*

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> *mỗi container sở hữu một vùng nhớ RAM độc lập, load balancer phân phối request theo cơ chế round-robin nên history_length sẽ bị nhảy giật cục, không ổn định dẫn đến agent bị mất trí nhớ ngẫu nhiên, không nắm được ngữ cảnh của câu hỏi vừa hỏi trước đó *

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *lỗi kết nối readiness probe thất bại
thông báo lỗi: redis.exceptions.ConnectionError: Error connecting to redis://localhost:6379/0. Connection refused.
tôi tìm ra nguyên nhân bằng cách mở Logs trên render dashboard, thấy ứng dụng đang cố gắng kết nối tới redis://localhost:6379/0
trên môi trừng cloud của render, service redis là instance tách biệt, không nằm chung với container của agent, trỏ vào localhost là trỏ vào chính container của agent nên không tìm thấy redis
tôi sửa bằng cách kiểm tra biến REDIS_URL và dùng fromService trong render.yaml để lấy chuối connectionString do render cấp phát cho service redis. sau cập nhật, service tự động deploy lại và /ready trả về 200 OK*

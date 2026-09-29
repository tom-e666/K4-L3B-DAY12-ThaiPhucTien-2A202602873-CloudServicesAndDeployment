# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Thái Phúc Tiến  Mã học viên: 2A202602873

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Việc agent chết sớm do agent_api_key chưa được setup giúp ngăn ngừa tình trạng bỏ qua hoặc quên thiết lập API key. Điều này rất quan trọng cho lớp phòng thủ này. Kẻ xấu có thể dò default_api_key và khai thác, gây mất mát tài nguyên mà chúng ta không nhận ra.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> $ curl -i -X POST https://day12-agent-zu2j.onrender.com/ask   -H "Content-Type: application/json"   -H "X-API-Key: $AGENT_API_KEY"   -H "X-User-Id: sv-test"   -d '{"question":"Deploy l\u00e0 g\u00ec?"}'
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 14:20:59 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 49a29712-6290-44a2
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a42ba1b8aa2f8b69-HKG
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud. (Mình đang nhớ 6 lượt trao đổi trước đó.)","user_id":"sv-test","history_length":6,"cost_usd":4.785e-05,"tokens":{"in":139,"out":45}}

Việc dữ liệu trả về có cấu trúc dạng jsson có lợi cho việc truy vấn dữ liệu tự động, hoặc là khám phá phân tích dữ liệu, truy vết bug. Việc chỉ xuất thông tin dưới dạng văn bản thuần túy sẽ khiến chúng ta khó phân biệt đâu là nội dung từ system, đâu là nội dung do LLM tạo ra. Bên cạch đó, việc có các metric thống kê, cảnh báo còn giúp xây dựng các hệ thống cảnh báo rủi ro tự động.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f Dockerfile.single  -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.18 GB |
| Multi-stage | 210 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch  bao gồm các tools và thư viện phụ trợ dùng trong quá trình build của phiên bản 1-stage. Chẳng hạn như Python 3.11, các file dependencies, và các công cụ liên quan đến quá trình build. Trong khi đó, phiên bản multi-stage đã loại bỏ toàn bộ các thành phần này sau khi build xong, chỉ giữ lại runtime cần thiết.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa file main.py, các layer trước đó gồm copy requirements.txt, run pip install sẽ được dùng lại từ cache. Các lệnh sau đó thì phải chạy lại. Nếu đặt COPY .. lên trước RUN pip install, thì lệnh RUN pip install sẽ chạy lại mỗi khi thay đổi code. Điều này gây lãng phí tài nguyên và thời gian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Container chạy bằng root. Hacker sử dụng nhiều kĩ thuật khác nhau, như khai thác lỗi hỗng trong code, hoặc supply chain attack mở được terminal của container. Hacker sẽ chạy mọi lệnh với quyền root, có thể khai thác kernel bug để chiếm luôn root của máy host. Việc dùng lệnh USER appuser có tác dụng hạ quyền chạy container xuống mức user thường. Nhờ đó, dẫu có chiếm được app thì hacker cũng không có quyền root của máy host.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Mỗi lần reset cung cấp 10 request. Trước reset có tối đa 10 request (gửi trong giây cuối cùng của chu kì trước) và ngay sau khi reset, người dùng có thể gửi thêm 10 request (trong giây đầu tiên của chu kì mới). Tổng cộng có thể gửi 20 request trong 2 giây liên tiếp. 

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate Limit kiểm tra tần suất gọi, từ đó chống quá tải server; Trong khi đó, cost guard kiểm tra tổng chi phí tích lũy để ngăn vượt ngân sách phí API

TÌnh huống 1: User 1 phút gửi 1 request, về cơ bản là không bao giờ vượt rate limit, nhưng mỗi request rất dài, rất nhiều token, nhanh chóng vượt ngân sách, đến một lúc thì bị cost guard cảnh báo và chặn.
Tình huống 2: User mới dùng, không có lịch sử sử dụng, nhưng lại spam 15 request trong 5 giây, buộc rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Endpoint /health kiểm tra redis thất bại, trả HTTP 503.
Orchestrator thấy /health trả 503 nên kết luận container đơ, tự động kill & restart toàn bộ 3 container 'agent'
Container mới khởi động, tiếp tục gọi /health nhưng redis vẫn fail, trả 503, bị resart. 
Hệ thống bị reset liên tục vì lỗi redis tạm thời.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> history length sẽ thay đổi bất thường do mỗi container lưu 1 phiên bản, có tổng cộng 3 phiên bản và load balancer sẽ truyền tùy ý request của mọi người vào các container khác nhau trong các thời điểm khác nhau.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> `HTTP 503 Service Unavailable` tại endpoint `/ready` (`{"status":"not ready","redis":false}`)
Nguyên nhân là do redis và agent kết nối không thành công khi deploy trên nền tảng railway
Cách sửa là đổi sang render.

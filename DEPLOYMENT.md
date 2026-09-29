# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Thái Phúc Tiến |
| Mã học viên | 2A202602873 |
| Repo | https://github.com/tom-e666/K4-L3B-DAY12-ThaiPhucTien-2A202602873-CloudServicesAndDeployment |
## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-zu2j.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của platform |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-zu2j.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-zu2j.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-zu2j.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-zu2j.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy l\u00e0 g\u00ec?"}'
# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-zu2j.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
curl -i https://day12-agent-zu2j.onrender.com/health
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 14:03:49 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 687b9b3c-0a2b-4165
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a42b8890182ea0b0-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}


$ curl -i https://day12-agent-zu2j.onrender.com/ready
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 14:04:18 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 6683d772-f099-4c4a
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a42b89444a35d62b-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}

$ curl -i -X POST https://day12-agent-zu2j.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
HTTP/1.1 401 Unauthorized
Date: Tue, 29 Sep 2026 14:04:32 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 6dd45b4c-ff3a-4603
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a42b899c9f6d06ff-HKG
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing API key"}

$ curl -i -X POST https://day12-agent-zu2j.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy l\u00e0 g\u00ec?"}'
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 14:04:54 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 1367ef4a-3b44-4189
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a42b8a280bca088f-HKG
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud. (Mình đang nhớ 4 lượt trao đổi trước đó.)","user_id":"sv-test","history_length":4,"cost_usd":4.05e-05,"tokens":{"in":90,"out":45}}

$ for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-zu2j.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -d '{"question":"test"}'
done; echo
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 

```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl




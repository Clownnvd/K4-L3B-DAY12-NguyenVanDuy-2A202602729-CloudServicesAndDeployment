# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Nguyễn Văn Duy |
| Mã học viên | 2A202602729 |
| Repo | https://github.com/Clownnvd/K4-L3B-DAY12-NguyenVanDuy-2A202602729-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | https://day12-agent-em9j.onrender.com |
| Platform | Render Blueprint — Web Service + Key Value |
| Ngày deploy | 29/09/2026 |
| Trạng thái | Live, free instance |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn, không ghi giá trị secret.

| Biến | Đã set | Nguồn |
|---|---|---|
| `PORT` | Có | Render tự gán |
| `AGENT_API_KEY` | Có | Render Blueprint secret prompt |
| `REDIS_URL` | Có | Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | Có | `render.yaml`, giá trị 10 |
| `MONTHLY_BUDGET_USD` | Có | `render.yaml`, giá trị 10.0 |
| `LOG_LEVEL` | Có | `render.yaml`, giá trị INFO |

## Lệnh Kiểm Tra

```bash
URL=https://day12-agent-em9j.onrender.com

curl -i $URL/health
curl -i $URL/ready

curl -i -X POST $URL/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

curl -i -X POST $URL/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $DEPLOY_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
```

## Kết Quả Chạy Thật

```text
GET  /health                  -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready                   -> 200 {"status":"ready","redis":true}
POST /ask không có API key    -> 401
POST /ask có API key hợp lệ   -> 200; response có answer, user_id, history_length, cost_usd, tokens
Rate limit 11 request         -> 200 200 200 200 200 200 200 200 200 200 429
```

Free instance có thể sleep khi không hoạt động; request đầu tiên sau cold start
có thể mất khoảng 50 giây theo thông báo của Render.

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — Render service ở trạng thái Live.
- `screenshots/health.png` — `/health` trả HTTP 200.

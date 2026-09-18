# 机械钟表擒纵调校API

纯后端零依赖Node服务，使用 `data/db.json` 持久化钟表档案、调校记录和复测记录。

## 启动

```bash
PORT=3021 node server.js
```

## 主要接口

- `GET /health`
- `GET /clocks`
- `POST /clocks`
- `GET /clocks/not-qualified`
- `GET /clocks/:id/history`
- `POST /clocks/:id/adjustments`
- `POST /clocks/:id/retests`
- `GET /clocks/:id/latest-retest`
- `GET /adjustments?clockId=`
- `GET /retests?clockId=&qualified=`

## 过冲保护

复测落库后，取当前调校周期内（实测时间不早于最近一次调校）按实测时间排序的最近两条日差：
前一条在一侧、后一条跨过零点到反向（一正一负，0 不算任一侧）时，钟表立即进入**过冲待重调**状态：

- 合格状态失效（`qualified` 强制为 `false`，自动进入 `/clocks/not-qualified`）
- 后续复测返回 `409` 且不落库
- 登记新调校后状态解除，过冲事件保留在 `overshoots` 历史中

`GET /clocks`、`GET /clocks/:id/history`、`GET /clocks/:id/latest-retest` 三个查询入口均返回一致的
`overshoot` 状态（`{ active, event }`）；history 接口额外返回完整 `overshoots` 历史列表。

## 闭环示例

```bash
curl http://127.0.0.1:3021/clocks/not-qualified
curl -X POST http://127.0.0.1:3021/clocks/clock_demo/retests \
  -H 'Content-Type: application/json' \
  -d '{"dailyRateSeconds":12,"amplitude":252,"note":"复测进入目标范围"}'
```

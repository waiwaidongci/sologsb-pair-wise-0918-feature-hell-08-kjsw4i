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

## 闭环示例

```bash
curl http://127.0.0.1:3021/clocks/not-qualified
curl -X POST http://127.0.0.1:3021/clocks/clock_demo/retests \
  -H 'Content-Type: application/json' \
  -d '{"dailyRateSeconds":12,"amplitude":252,"note":"复测进入目标范围"}'
```

## 过冲保护

当前调校后的复测按 `testedAt` 取最近两条日差：前一条在零点一侧、后一条跨过零点到反向（两值异号）时，判定为**过冲**：

- 触发过冲的复测仍落库，但 `qualified` 强制为 `false`（合格失效），钟表进入「待重调」状态。
- 待重调期间再提交复测直接返回 **409**，且**不落库**；必须先 `POST /clocks/:id/adjustments` 登记新调校才能解除。
- 登记新调校会把过冲记录标记为 `resolved`，过冲历史保留在 `overshoots` 中。
- 三个查询入口返回一致的过冲状态（`overshoot` 字段，待重调时为对象、否则为 `null`）：
  - `GET /clocks`（含 `/clocks/not-qualified`，待重调钟表 `qualified` 一律为 `false`）
  - `GET /clocks/:id/history`（同时返回全部 `overshoots` 历史）
  - `GET /clocks/:id/latest-retest`


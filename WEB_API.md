# Hummingbot 内置 Web API 说明

Hummingbot 启动后会在**后台**启动一个 HTTP API 服务，供 Web UI 或脚本调用，对接所有主要 CLI 命令。

## 启用与端口

- **默认**：自动启动，监听 `http://127.0.0.1:8080`
- **环境变量**：
  - `HUMmingBOT_API_HOST`：监听地址（默认 `127.0.0.1`）
  - `HUMmingBOT_API_PORT`：端口（默认 `8080`）
  - `HUMmingBOT_API_ENABLED=0`：关闭 API 服务

## 通用响应格式

- 成功：`{"output": ["line1", "line2", ...]}`，`output` 为命令的逐行输出（与 CLI 一致）
- 错误：HTTP 4xx/5xx，或 `output` 中含 `Error: ...`
- 部分接口返回 JSON 结构（如 `/api/info`、`/api/history/trades`）

## 接口列表

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/health` | 健康检查 |
| GET | `/api/info` | 当前策略名、实例 ID 等 |
| GET | `/api/strategies` | 列出 conf 下策略配置文件（供 import 下拉） |
| GET | `/api/gateway/status` | Gateway 状态 |
| POST | `/api/start` | 启动策略，Body: `{"script": "xxx", "conf": "yyy"}` 可选 |
| POST | `/api/stop` | 停止策略，Query: `skip_order_cancellation=false` |
| GET | `/api/status` | 状态检查，Query: `live=false` |
| GET | `/api/balance` | 余额，Query: `option=limit|paper` 可选 |
| GET | `/api/config` | 列出配置 |
| POST | `/api/config` | 设置单项，Body: `{"key": "xxx", "value": "yyy"}` |
| GET | `/api/history` | 历史表现，Query: `days=0&verbose=false&precision=` |
| GET | `/api/history/trades` | 历史成交 JSON，Query: `days=0` |
| POST | `/api/import` | 导入策略，Body: `{"file_name": "conf_xxx.yml"}` |
| GET | `/api/connect` | 连接列表（交易所 Keys 状态） |
| GET | `/api/help` | 帮助，Query: `command=all` 或具体命令名 |
| POST | `/api/exit` | 退出，Query: `force=false` |
| GET | `/api/ticker` | 行情，Query: `live=false&exchange=&market=` |
| GET | `/api/rate` | 汇率，Query: `pair=ETH-USDT` 或 `token=ETH` |

## 示例

```bash
# 健康检查
curl http://127.0.0.1:8080/api/health

# 当前信息
curl http://127.0.0.1:8080/api/info

# 启动策略（使用已导入策略）
curl -X POST http://127.0.0.1:8080/api/start

# 导入并检查状态
curl -X POST http://127.0.0.1:8080/api/import -H "Content-Type: application/json" -d '{"file_name": "conf_pure_mm_1.yml"}'
curl http://127.0.0.1:8080/api/status

# 余额
curl http://127.0.0.1:8080/api/balance

# 历史成交 JSON
curl "http://127.0.0.1:8080/api/history/trades?days=1"
```

## 交互式文档

启动 Hummingbot 后访问 **http://127.0.0.1:8080/docs** 可查看 Swagger UI，直接在线调试接口。

## 与 Web UI 对接

前端可用任意 HTTP 客户端（fetch/axios）请求上述接口，将 `output` 数组拼接为文本展示，或解析 `/api/info`、`/api/history/trades` 等 JSON 做表格/图表。CORS 已开放 `*`，支持跨域。

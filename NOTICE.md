# 项目注意事项

## 启动前必看

本项目最后提交或演示前，请先阅读：

- 需求与本机运行踩坑说明：[requirements-analysis.md](requirements-analysis.md)
- 答辩汇报 PPT：[technical-supervision-defense-v3.pptx](docs/presentations/technical-supervision-defense-v3.pptx)

其中 `requirements-analysis.md` 末尾的 **第 16 节：本机运行注意事项与常见故障** 记录了本机启动、AI Gateway、Knowledge runtime、SiliconFlow API Key、文档解析、问答和报告生成的排障方法。

## 技术栈概览

| 模块 | 技术 |
| --- | --- |
| 前端 | React、TypeScript、Vite、Bun |
| 前端状态/请求 | TanStack Query、TanStack Router、Zustand |
| UI/样式 | Tailwind CSS、Radix UI、Base UI、lucide-react、ECharts |
| 后端 | Go 微服务 |
| 后端服务 | gateway、auth、file、knowledge、qa、document、ai-gateway |
| 知识库运行时 | Python Knowledge runtime |
| 数据库 | PostgreSQL |
| 缓存/任务队列 | Redis |
| 对象存储 | MinIO |
| 检索索引 | Elasticsearch |
| 本地基础设施 | Docker Compose |
| AI 接入 | AI Gateway 统一管理 chat / embedding / rerank |
| 当前模型 | DeepSeek-V4-Flash、BAAI/bge-m3、BAAI/bge-reranker-v2-m3 |

## 本地启动入口

标准本地启动建议使用：

```bash
./scripts/local/start.sh --china
```

前端单独启动：

```powershell
cd apps/web
bun run dev --host 127.0.0.1
```

访问地址：

```text
http://localhost:5173
```

默认账号：

```text
admin / LocalDemoAdmin#12345
superadmin / LocalDemoAdmin#12345
```

## 启动后必须确认的服务

```text
Gateway:           http://localhost:8080/readyz
Auth:              http://localhost:8001/healthz
File:              http://localhost:8082/healthz
Knowledge adapter: http://localhost:8083/readyz
QA:                http://localhost:8084/readyz
Document:          http://localhost:8085/readyz
AI Gateway:        http://localhost:8086/readyz
Knowledge runtime: http://localhost:9380/api/v1/system/ping
```

注意：页面能打开不代表 AI 功能可用。问答、文档解析、报告生成都依赖 AI Gateway。

## 重要配置提醒

- `.env.local` 是本地私有配置，不要提交到 GitHub。
- 如果以后重新 clone / pull 到一个新的目录，`.env.local` 不会自动回来，需要重新执行 `cp .env.example .env.local`，再把自己的 SiliconFlow API Key 和 AI Gateway 模型配置填进去。
- SiliconFlow API Key 不要写进公开文档。
- 更换 API Key 后，需要刷新 AI Gateway 本地 profile，避免数据库里仍使用旧 key。
- 报告生成建议保持：

```env
DOCUMENT_AI_GATEWAY_MODEL=
DOCUMENT_AI_GATEWAY_PROFILE_ID=default-chat
```

`DOCUMENT_AI_GATEWAY_MODEL` 留空后，Document 服务会自动使用 `default-chat` 对应的真实模型，避免和 AI Gateway profile 不匹配。

## 本机路径说明

`requirements-analysis.md` 第 16 节里有一些 `C:\Users\Lenovo\Desktop\Software_Teamwork` 这样的本机路径，这是按当前电脑记录的启动路径，主要给自己复现踩坑时使用。

如果以后把项目放到其他位置，不需要照抄这个路径，只要进入新的项目根目录后再运行启动命令即可。

## 常见故障速查

| 现象 | 优先检查 |
| --- | --- |
| 文档解析失败 | `Knowledge adapter`、`Knowledge runtime worker`、AI Gateway embedding |
| 问答无响应 | AI Gateway chat、LLM 配置是否发布 |
| 报告生成失败 | Document 日志、`DOCUMENT_AI_GATEWAY_MODEL` 是否为空 |
| AI Gateway ready 但调用失败 | SiliconFlow key、profile credential 是否刷新 |
| 端口占用 | 是否有旧进程仍监听 8080/8083/8085/8086/9380/5173 |
| `uv: command not found` | Knowledge runtime 启动时 `.local/bin` 没进 PATH |

日志优先看：

```powershell
Get-Content .local/logs/ai-gateway.restart.log -Tail 160
Get-Content .local/logs/document.restart.log -Tail 160
Get-Content .local/logs/knowledge-runtime-worker.restart.err.log -Tail 160
```

## 提交 Git 前检查

提交前建议确认：

```powershell
git status --short
```

应该提交的材料包括：

- `requirements-analysis.md`
- `NOTICE.md`
- `docs/presentations/technical-supervision-defense-v3.pptx`

不要提交：

- `.env.local`
- `.local/`
- `node_modules/`
- API Key、密码、个人账号凭据

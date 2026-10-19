# Memory Warehouse v1.3.0

## 下载

- **完整版**：下载 `memory-warehouse-1.3.0-full.part1.rar`、`part2.rar`、`part3.rar` 到同一目录，用 WinRAR / 7-Zip 解压 `part1.rar`，得到完整包（含约 90MB 本地向量模型）。
- **精简版**：下载 `memory-warehouse-1.3.0-lite.zip`，不含模型；运行 `python download_model.py` 下载，或直接使用纯 FTS。

## 本版新增

- **按 agent 隔离**：`events` / `digests` / `import_state` 增加 `agent_id`，新增 `session_agents` 记录会话归属。
- hooks 从 Claude Code 进程的 `--add-dir ...\Data\Agents\<uuid>` 自动解析 agent；watcher 按会话映射打标，并在 hook 首次看到历史会话时自动回填 `legacy` 事件。
- MCP server 注册时必须显式传 `--agent-id <uuid>`；每个 agent 使用独立的 server，默认只能读写自己的记忆。
- 新增 CLI：`agent current` / `agent list` / `agent assign`；检索、brief、timeline、show、stats、summarize、export 都支持 `--agent`，只有 `--agent '*'` 才跨 agent。
- 旧数据升级时归入 `legacy`，不会被任意 agent 默认看到；确认归属后可用 `agent assign` 迁移。
- 自动迁移旧库：启动时增加 `agent_id` 列和索引，无需重建数据库。

## 升级说明

- 已有历史会落在 `legacy`。用 `python mw.py agent current` 查看当前 agent，再用 `python mw.py agent assign --session <前缀> --agent <uuid>` 迁移确认的会话。
- 已注册的 MCP server 需要补上 `--agent-id`；不传时只会读写 `legacy`。
- 建议完整重启 Cherry Studio / Claude Code，让 hooks 和 MCP 配置生效。

## 校验

文件 SHA256 见 `SHA256SUMS.txt`。

## 安装与使用

见 `README.md`。

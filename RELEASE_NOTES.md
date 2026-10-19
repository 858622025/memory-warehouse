# Memory Warehouse v1.2.0

## 下载

- **完整版**：下载 `memory-warehouse-1.2.0-full.part1.rar`、`part2.rar`、`part3.rar` 到同一目录，用 WinRAR / 7-Zip 解压 `part1.rar`，得到完整包（含约 90MB 本地向量模型）。
- **精简版**：下载 `memory-warehouse-1.2.0-lite.zip`，不含模型；运行 `python download_model.py` 下载，或直接使用纯 FTS。

## 本版新增

- MCP watcher：`mw.py mcp` 默认在进程内轮询 transcript，不依赖 hooks。
- 新增 MCP 工具 `sync` / `watch_status` / `ingest`，以及 `memory://brief` / `memory://stats` resources。
- 新增 CLI `watch --once` / `watch --interval N` / `watch-status`。
- transcript 重写检测（offset + 首尾 4KB 指纹），重写时保留旧事件和 turn 编号。
- 现有 SessionStart / Stop / PreCompact / SessionEnd hooks 全部保留。

## 校验

文件 SHA256 见 `SHA256SUMS.txt`。

## 安装与使用

见 `README.md`。

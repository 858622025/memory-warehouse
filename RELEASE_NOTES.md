# Memory Warehouse v1.3.1

## 下载

- **完整版**：下载 `memory-warehouse-1.3.1-full.part1.rar`、`part2.rar`、`part3.rar` 到同一目录，用 WinRAR / 7-Zip 解压 `part1.rar`，得到完整包（含约 90MB 本地向量模型）。
- **精简版**：下载 `memory-warehouse-1.3.1-lite.zip`，不含模型；运行 `python download_model.py` 下载，或直接使用纯 FTS。

## 本版修复

- **Windows hooks 不再闪脚本窗口**：
  - Stop / PreCompact / SessionEnd 改用 `pythonw.exe` 启动；SessionStart 保留 `python.exe` 以输出摘要 JSON。
  - `warehouse/identity.py` 调用 wmic / PowerShell 解析 agent 时加 `CREATE_NO_WINDOW` + `SW_HIDE`。
- **agent 身份跨进程缓存**：新增 `data/agent_cache.json`，按 Claude 会话 / PID 缓存 12 小时；同一会话只有第一次 hook 需要查进程链，后续直接命中缓存。
- `install_hooks.py` 会自动选择 `pythonw.exe`（存在时）；升级后重新运行安装脚本即可。
- 其余检索、隔离和发布行为与 1.3.0 相同。

## 升级说明

- 重新运行 `python install.py`（或 `python install_hooks.py`），然后完整重启 Cherry Studio / Claude Code，让新的 hook 命令生效。
- 已有数据库、agent 归属和 MCP 配置不需要迁移；版本号会自动更新。

## 校验

文件 SHA256 见 `SHA256SUMS.txt`。

## 安装与使用

见 `README.md`。

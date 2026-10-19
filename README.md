# Memory Warehouse（v1.1.0）

把 agent 的对话历史增量落盘到本地 SQLite。上下文被压缩（compact）后，仍然能按关键词或语义找回压缩前说过的话、跑过的命令、改过的文件和当时的结论。

> 数据只保存在本机。默认不联网；只有开启 LLM 语义摘要时，才会把当轮对话发送给你配置的 API。

## 功能

- **V1 原始记忆**：用户消息、助手回复、思考、工具调用、工具结果增量入库；SQLite FTS5，中文按三元组检索。
- **V2 语义摘要**：每轮生成 `summary` / `decisions` / `open_questions` / `must_keep`（可选，需要 Anthropic 兼容 API）。
- **V3 向量检索**：本地中文 embedding（`bge-small-zh-v1.5`，512 维），按句切块，和 BM25 用 RRF 融合。
- **Hooks**：SessionStart / Stop / PreCompact / SessionEnd 自动落盘、压缩前快照、启动注入摘要。
- **CLI + MCP**：`recall` / `show` / `timeline` / `remember` / `checkpoint` / `summarize` / `embed` / `stats`。
- **可选 Skill**：让 agent 在"还记得吗 / 上次 / 之前"时主动检索。

## 环境要求

- Python **3.10+**
- Cherry Studio（Claude Agent）或 Claude Code
- 可选：任意 Anthropic Messages 兼容 API（用于 V2 语义摘要）
- Windows 测试最充分；macOS / Linux 使用 `python` 命令即可，hooks 机制相同

## 安装

> 两个 zip 使用同一份文档：**完整版（full）**内含模型，按「完整版」步骤安装；**精简版（lite）**不含模型，按「精简版」步骤安装，或先运行 `download_model.py`。

### 完整版（zip 内含约 90MB 的本地模型）

```bash
# 1. 解压后进入目录
python -m pip install -r requirements.txt

# 2. 安装 hooks + skill（会自动检测 Cherry Studio / Claude Code 的配置路径）
python install.py

# 3. 完整重启 Cherry Studio / Claude Code

# 4. 导入已有历史 transcript
python mw.py sync
```

### 精简版（不含模型）

```bash
python -m pip install -r requirements.txt
python download_model.py       # 可选；不下载也能用纯 FTS
python install.py
```

> 不想装 `fastembed`？只装 `numpy` + `requests` 也可以，FTS、hooks、摘要都正常，向量检索会自动跳过。

如果脚本检测不到 Cherry Studio / Claude Code 的配置路径，可以手动指定：

```bash
python install.py --settings "C:\path\to\settings.json" --skills-dir "C:\path\to\skills"
```

只装 hooks 或只装 skill：

```bash
python install.py --no-skill
python install.py --no-hooks
```

## LLM 配置（V2，可选）

`install.py` 会尝试从以下环境变量生成 `llm.local.json`：

```
ANTHROPIC_BASE_URL
ANTHROPIC_API_KEY
ANTHROPIC_AUTH_TOKEN
ANTHROPIC_DEFAULT_HAIKU_MODEL / ANTHROPIC_MODEL
ANTHROPIC_CUSTOM_HEADERS
```

也可以手动创建 `llm.local.json`：

```json
{
  "enabled": true,
  "auto_summarize": true,
  "disable_thinking": true,
  "base_url": "https://your-api.example.com/anthropic",
  "api_key": "YOUR_KEY",
  "model": "your-model",
  "max_tokens": 2500,
  "timeout": 60
}
```

说明：

- `disable_thinking` 建议保持 `true`。部分模型会把思考 token 计入输出上限，导致最终没有文本。
- 不配置 LLM 也完全可用：确定性回合摘要、FTS、向量检索都不依赖它，只是没有语义摘要。
- 开启 LLM 摘要后，每轮对话内容会发送到你配置的 API。请只在信任的端点上开启。
- `llm.local.json` 含密钥，不要分享或提交。

## 常用命令

```bash
python mw.py recall "DPI 缩放" -n 8
python mw.py recall "上下文丢了怎么恢复" --explain
python mw.py recall "关键词" --no-semantic     # 只走 FTS
python mw.py recall "关键词" --semantic        # 强制语义

python mw.py show 12345                        # 看某条事件的完整原文
python mw.py timeline -n 20                    # 回合时间线
python mw.py stats                             # 事件/回合/摘要/向量统计
python mw.py lost --session <id>               # 某会话被压缩掉的事件

python mw.py remember "长期事实" --pinned --tags 项目
python mw.py checkpoint --summary "当前进度" --must-keep "路径, 命令, 报错"

python mw.py summarize --limit 20              # 补历史 LLM 摘要
python mw.py embed                             # 增量向量索引
python mw.py vacuum                            # 压缩数据库文件
```

Windows 下也可以用 `mw.cmd` 代替 `python mw.py`。

## 它是怎么工作的

| 组件 | 作用 |
|---|---|
| `SessionStart` hook | 注入最近进展、检查点、上次压缩摘要（约 3.5k 字符） |
| `Stop` hook | 每轮结束增量导入 transcript；异步起摘要/向量子进程 |
| `PreCompact` hook | 压缩前先同步，再写 checkpoint（压缩会重写 transcript，这是抢救窗口） |
| `SessionEnd` hook | 收尾同步 |
| `mw.py mcp` | stdio MCP server，提供 recall/remember/checkpoint/brief/stats/summarize/embed |

transcript 来源：

- Cherry Studio：`%APPDATA%\CherryStudio\Data\Agents\.claude\projects\**\*.jsonl`
- Claude Code：`~/.claude/projects/**/*.jsonl`

可在 `config.json` 的 `transcript_roots` 中增减。

## 数据与隐私

- 数据库：`data/memory.db`
- 日志：`logs/hook.log`
- LLM 凭证：`llm.local.json`
- 向量模型：`data/models/bge-small-zh-v1.5`

所有内容默认只在本机。入库前会脱敏常见密钥（`sk-*`、`ghp_*`、Bearer、`api_key=`、`password=` 等），工具结果会截断保存，完整原文仍可从 transcript 用 `show` 回取。

## 卸载

```bash
python install_hooks.py --uninstall          # 移除 hooks
python install_hooks.py --uninstall-skill    # 移除 skill
```

然后删除整个目录即可。注意：`data/memory.db` 是历史记忆的唯一副本，删除不可恢复。

## 常见问题

**hooks 没生效？**
完整重启 Cherry Studio / Claude Code，然后看 `logs/hook.log` 是否有 `stop sync ...`。

**中文搜不到？**
先试更短的关键词、`--since 90d`、`--any`；再对比 `--no-semantic` 和默认结果，判断是关键词问题还是语义模型未覆盖。

**LLM 摘要为空？**
检查 `llm.local.json` 的 `base_url` / 凭证 / `model`；保持 `disable_thinking: true`；先用 `python mw.py summarize --session <id> --turn <n> --force` 单条调试。

**模型缺失？**
运行 `python download_model.py`，或继续使用纯 FTS。语义检索不可用不会影响其他功能。

**数据库太大？**
`python mw.py vacuum`。也可以调低 `config.json` 里的 `caps` 后重建数据库。

**安装前已经被压缩掉的内容能找回吗？**
不能。Claude Code 压缩时会重写 transcript，被丢弃的消息在磁盘上也不存在了。仓库只覆盖安装之后的时间。

## License

MIT。第三方组件见 `NOTICE`。

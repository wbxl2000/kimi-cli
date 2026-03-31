# Kimi Code CLI 仓库解析

## 📦 仓库概览

**项目名称**: `kimi-cli`
**当前版本**: `1.28.0`
**开发者**: MoonshotAI（月之暗面）
**语言**: Python（要求 >= 3.12）
**构建工具**: `uv`（使用 `uv_build` 作为 build backend）
**许可证**: 有 LICENSE 文件

> Kimi Code CLI 是一个**运行在终端中的 AI 编码代理 (Agent)**，可以帮助完成软件开发任务和终端操作——包括读写代码、执行 shell 命令、搜索/抓取网页，并在执行过程中自主规划和调整行为。

---

## 🏗️ 项目架构

### Monorepo / Workspace 结构

这是一个 **uv workspace** 多包项目，包含以下子包：

| 子包 | 路径 | 说明 |
|---|---|---|
| **kimi-cli** (主包) | `src/kimi_cli/` | CLI 主程序，197 个 Python 文件 |
| **kosong** | `packages/kosong/` | LLM 对话/工具调用核心框架（v0.47.0） |
| **kaos (pykaos)** | `packages/kaos/` | 辅助库 |
| **kimi-code** | `packages/kimi-code/` | kimi-code 扩展包 |
| **kimi-sdk** | `sdks/kimi-sdk/` | SDK |

### 主包 `kimi_cli` 核心模块

| 模块 | 职责 |
|---|---|
| `__main__.py` / `app.py` | 入口 & 应用主逻辑 |
| `cli/` | CLI 命令定义（基于 typer） |
| `tools/` | **工具系统**：file（读/写/grep/glob/replace）、shell、web（fetch/search）、agent、plan、think、ask_user、todo、background 等 |
| `ui/` | 用户界面：shell 交互（prompt/keyboard/approval/echo）、ACP 协议、打印可视化 |
| `llm.py` | LLM 调用层 |
| `agents/` / `subagents/` | 多 Agent 编排 |
| `prompts/` | 系统提示词模板 |
| `skill/` / `skills/` | 技能系统 |
| `hooks/` | 钩子机制 |
| `auth/` | 认证（含 OAuth） |
| `background/` | 后台任务管理（worker、store、agent_runner） |
| `config.py` | 配置管理 |
| `session.py` / `session_state.py` | 会话状态 |
| `web/` | Web UI（FastAPI + WebSocket） |
| `acp/` | Agent Client Protocol 支持 |
| `approval_runtime/` | 权限审批运行时 |
| `wire/` | 数据传输/序列化 |
| `plugin/` | 插件系统 |
| `notifications/` | 通知系统 |

---

## 🔑 核心功能特性

1. **AI 编码代理** — 通过 LLM 驱动的 Agent 自动完成编码任务
2. **Shell 命令模式** — `Ctrl-X` 切换直接运行 shell 命令
3. **MCP 支持** — Model Context Protocol，支持 stdio/HTTP 传输
4. **ACP 支持** — Agent Client Protocol，可集成到 Zed、JetBrains 等 IDE
5. **VS Code 扩展** — 通过专用扩展集成
6. **多工具系统** — 文件读写、grep 搜索、glob、网页抓取/搜索、计划模式等
7. **后台任务** — 异步 Agent 任务管理
8. **Web UI** — 基于 FastAPI + WebSocket 的 Web 界面
9. **技能系统** — 可扩展的 Skill 机制
10. **Hook 系统** — 事件钩子

---

## 🛠️ 技术栈

| 类别 | 技术 |
|---|---|
| CLI 框架 | `typer` |
| 终端 UI | `prompt-toolkit`, `rich` |
| Web 框架 | `FastAPI` + `uvicorn` + `websockets` |
| HTTP 客户端 | `aiohttp`, `httpx` |
| 异步 I/O | `aiofiles` |
| 数据验证 | `pydantic` |
| MCP | `fastmcp` |
| 搜索 | `ripgrepy` (ripgrep) |
| 网页解析 | `trafilatura` |
| 代码质量 | `ruff`（lint/format）, `pyright` / `ty`（类型检查） |
| 测试 | `pytest` + `pytest-asyncio` |
| 打包 | `PyInstaller`（独立二进制），`uv_build`（Python 包） |

---

## 📁 其他目录

| 目录 | 说明 |
|---|---|
| `tests/` | 单元测试（17 个子目录/文件） |
| `tests_e2e/` | 端到端测试 |
| `tests_ai/` | AI 相关测试 |
| `docs/` | 文档 |
| `examples/` | 示例 |
| `scripts/` | 脚本工具 |
| `web/` | Web 前端 |
| `vis/` | 可视化相关 |
| `klips/` | 代码片段/剪辑 |
| `.agents/` | Agent 配置 |
| `.github/` | CI/CD 配置 |

---

## 总结

这是一个**功能完整、架构成熟的 AI 终端编码代理项目**，由月之暗面（MoonshotAI）开发。它类似 Claude Code / Cursor 等 AI 编码工具的 CLI 版本，采用 monorepo 架构，核心是 LLM 驱动的 Agent + 丰富的工具系统，支持多种 IDE 集成和协议（MCP/ACP）。代码规模约 197+ Python 文件，已达到 v1.28.0 版本，是一个活跃维护的项目。

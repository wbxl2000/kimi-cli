# Kimi Code CLI

## 常用命令（使用 uv）

- `make prepare`（同步工作区内各包的依赖并安装 git hooks）
- `make format`
- `make check`
- `make test`
- `make ai-test`
- `make build` / `make build-bin`

若直接运行工具，请使用 `uv run ...`。

## 项目概述

Kimi Code CLI 是一个面向软件工程工作流的 Python CLI agent。支持交互式 shell UI、用于 IDE 集成的 ACP server 模式，以及 MCP 工具加载。

## 技术栈

- Python 3.12+（工具链面向 3.14）
- CLI 框架：Typer
- 异步运行时：asyncio
- LLM 框架：kosong
- MCP 集成：fastmcp
- 日志：loguru
- 包管理/构建：uv + uv_build；二进制使用 PyInstaller
- 测试：pytest + pytest-asyncio；lint/格式化：ruff；类型：pyright + ty

## 架构概览

- **CLI 入口**：`src/kimi_cli/cli/__init__.py`（Typer）解析标志位（UI 模式、agent 规格、配置、MCP），并路由到 `src/kimi_cli/app.py` 中的 `KimiCLI`。
- **应用/运行时初始化**：`KimiCLI.create` 加载配置（`src/kimi_cli/config.py`）、选择模型/供应商（`src/kimi_cli/llm.py`）、构建 `Runtime`（`src/kimi_cli/soul/agent.py`）、加载 agent 规格、恢复 `Context`，再构造 `KimiSoul`。
- **Agent 规格**：`src/kimi_cli/agents/` 下的 YAML，由 `src/kimi_cli/agentspec.py` 加载。规格可 `extend` 基础 agent、按 import 路径选用工具，并通过 `subagents` 字段注册内置子 agent 类型。子 agent 实例单独持久化在 session 目录下，可通过 `agent_id` 恢复。系统提示词与规格同目录；内置参数包括 `KIMI_NOW`、`KIMI_WORK_DIR`、`KIMI_WORK_DIR_LS`、`KIMI_AGENTS_MD`、`KIMI_SKILLS`（本文件通过 `KIMI_AGENTS_MD` 注入）。
- **工具**：`src/kimi_cli/soul/toolset.py` 按 import 路径加载工具、注入依赖并执行 tool call。内置工具位于 `src/kimi_cli/tools/`（agent、shell、file、web、todo、background、dmail、think、plan）。MCP 工具通过 `fastmcp` 加载；CLI 侧管理在 `src/kimi_cli/mcp.py`，配置存放在 share 目录。
- **子 agent**：`src/kimi_cli/soul/agent.py` 中的 `LaborMarket` 注册内置子 agent 类型。`Agent` 工具（`src/kimi_cli/tools/agent/`）创建或恢复子 agent 实例；`SubagentStore` 将会话元数据、提示词、wire 日志与上下文持久化在 `session/subagents/<agent_id>/`。
- **核心循环**：`src/kimi_cli/soul/kimisoul.py` 为主 agent 循环：接收用户输入、处理斜杠命令（`src/kimi_cli/soul/slash.py`）、追加到 `Context`（`src/kimi_cli/soul/context.py`）、调用 LLM（kosong）、运行工具，并在需要时执行压缩（`src/kimi_cli/soul/compaction.py`）。
- **审批**：`src/kimi_cli/soul/approval.py` 为面向工具的封装。`src/kimi_cli/approval_runtime/` 中的 `ApprovalRuntime` 是会话级待审批来源，审批请求会投影到根 wire 流，供 Shell/Web 类 UI 使用。
- **UI/Wire**：`src/kimi_cli/soul/run_soul` 将 `KimiSoul` 接到 `Wire`（`src/kimi_cli/wire/`），使 UI 循环可流式消费事件。UI 位于 `src/kimi_cli/ui/`（shell/print/acp/wire）。
- **Shell UI**：`src/kimi_cli/ui/shell/` 处理交互式 TUI 输入、shell 命令模式与斜杠命令补全；为默认交互体验。
- **斜杠命令**：Soul 级命令在 `src/kimi_cli/soul/slash.py`；shell 级命令在 `src/kimi_cli/ui/shell/slash.py`。shell UI 同时暴露两者并按注册表分发。标准 skill 注册 `/skill:<skill-name>` 并将 `SKILL.md` 作为用户 prompt；flow skill 注册 `/flow:<skill-name>` 并执行内嵌 flow。

## 主要模块与接口

- `src/kimi_cli/app.py`：`KimiCLI.create(...)` 与 `KimiCLI.run(...)` 为程序化主入口；各 UI 层依赖于此。
- `src/kimi_cli/soul/agent.py`：`Runtime`（配置、session、内置项）、`Agent`（系统提示词 + toolset）、`LaborMarket`（内置子 agent 类型注册表）。
- `src/kimi_cli/soul/kimisoul.py`：`KimiSoul.run(...)` 为循环边界；发出 Wire 消息并通过 `KimiToolset` 执行工具。
- `src/kimi_cli/soul/context.py`：对话历史与检查点；DMail 用于基于检查点的回复。
- `src/kimi_cli/soul/toolset.py`：加载工具、执行 tool call、桥接 MCP 工具。
- `src/kimi_cli/ui/*`：shell/print/acp 前端；消费 `Wire` 消息。
- `src/kimi_cli/wire/*`：soul 与 UI 之间的事件类型与传输。

## 仓库地图

- `src/kimi_cli/agents/`：内置 agent 的 YAML 规格与提示词
- `src/kimi_cli/prompts/`：共享提示词模板
- `src/kimi_cli/soul/`：核心运行时/循环、context、压缩、审批
- `src/kimi_cli/tools/`：内置工具
- `src/kimi_cli/ui/`：UI 前端（shell/print/acp/wire）
- `src/kimi_cli/acp/`：ACP server 组件
- `packages/kosong/`、`packages/kaos/`：工作区依赖
  + Kosong 是为现代 AI agent 应用设计的 LLM 抽象层，统一消息结构、异步工具编排与可插拔 chat 供应商，便于构建 agent 并降低供应商锁定。
  + PyKAOS 是轻量 Python 库，为 agent 与操作系统交互提供抽象层。通过 KAOS 的文件操作与命令执行可在本地环境与 SSH 远端之间切换。
- `tests/`、`tests_ai/`：测试套件
- `klips`：Kimi Code CLI Improvement Proposals

## 规范与质量

- Python >=3.12（ty 配置使用 3.14）；行宽 100。
- Ruff 负责 lint + 格式化（规则：E、F、UP、B、SIM、I）；pyright + ty 做类型检查。
- 测试使用 pytest + pytest-asyncio；文件命名为 `tests/test_*.py`。
- CLI 入口：`kimi` / `kimi-cli` → `src/kimi_cli/__main__.py`（再路由到 `src/kimi_cli/cli/__init__.py`）。
- 用户配置：`~/.kimi/config.toml`；日志、session 与 MCP 配置位于 `~/.kimi/`。

## Git 提交信息

采用 Conventional Commits 格式：

```
<type>(<scope>): <subject>
```

允许的 type：
`feat`、`fix`、`test`、`refactor`、`chore`、`style`、`docs`、`perf`、`build`、`ci`、`revert`。

## 版本策略

项目采用**仅升次版本号**的版本方案（`MAJOR.MINOR.PATCH`）：

- **PATCH** 恒为 `0`，不要提升。
- **MINOR** 在任何变更时递增：新功能、改进、缺陷修复等。
- **MAJOR** 仅在有明确人工决策时修改；日常开发保持不变。

示例：`0.68.0` → `0.69.0` → `0.70.0`；不要出现 `0.68.1`。

此规则适用于仓库内所有包（根目录、`packages/*`、`sdks/*`）以及发布与 skill 相关工作流。

## 发布流程

1. 确保 `main` 为最新（拉取远端）。
2. 创建发布分支，例如 `bump-0.68` 或 `bump-pykaos-0.5.3`。
3. 更新 `CHANGELOG.md`：将 `[Unreleased]` 重命名为 `[0.68] - YYYY-MM-DD`。
4. 更新 `pyproject.toml` 中的版本。
5. 运行 `uv sync` 对齐 `uv.lock`。
6. 提交分支并开 PR。
7. 合并 PR 后切回 `main` 并拉取最新。
8. 打标签并推送：
   - `git tag 0.68` 或 `git tag pykaos-0.5.3`
   - `git push --tags`
9. 推送标签后由 GitHub Actions 处理发布。

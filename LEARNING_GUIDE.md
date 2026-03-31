# Kimi CLI 学习路线指南

> 写给同时是 Python 新手和项目新接触者的核心贡献者

---

## 先说残酷的现实

这个项目用到了 Python 里**比较高阶的东西**：

- 大量 `async/await`（异步编程），这不是 Python 入门内容
- 类型注解 + `pydantic` 做数据校验，对新手有认知门槛
- Monorepo 多包管理，依赖关系复杂
- 抽象层很多（tools → agents → subagents → LLM → wire）

你不可能"先学完 Python 再来看代码"，因为学完也不够。正确的路径是**边做边学，但要选对切入点**。

---

## 第一阶段：别急着看核心，先能跑起来（1-2 周）

**目标：建立"改一行代码 → 看到效果"的反馈循环**

### 1. 跑通开发环境

```sh
make prepare
uv run kimi
make test
```

能跑起来、能跑测试，就算成功。遇到报错就解决报错，这个过程本身就是学习。

### 2. 学这些 Python 基础就够启动了（不要贪多）

- 函数、类、模块导入
- `dict` / `list` 推导式
- `with` 语句（上下文管理）
- `type hints`（看懂 `def foo(x: str) -> bool:` 就行）
- **不要现在学 async**，遇到了先跳过

### 3. 读测试代码而不是源码

- `tests/` 里的文件比 `src/` 好懂得多
- 测试告诉你"这个函数应该做什么"，比看实现更清楚

---

## 第二阶段：从边缘模块切入（2-4 周）

**目标：提交你的前几个 PR**

按难度从低到高，建议从这些模块入手：

| 优先级 | 模块 | 原因 |
|---|---|---|
| ⭐ 先做 | `tools/file/glob.py`, `tools/file/read.py` | 逻辑直观，就是文件操作 |
| ⭐ 先做 | `tools/think/` | 最简单的 tool，大概率就是透传文本 |
| 然后 | `tools/web/fetch.py`, `tools/web/search.py` | HTTP 请求，逻辑独立 |
| 再然后 | `ui/` 里的小改动 | 显示层，改了能直接看到效果 |
| **暂时别碰** | `llm.py`, `session.py`, `agents/`, `wire/` | 核心调度层，牵一发动全身 |

### 找什么样的 issue 做

- 标有 `good first issue` 的
- 文档改进、错误信息优化
- 小 bug fix（比如边界条件处理）
- 给现有功能补测试

---

## 第三阶段：理解异步和核心架构（1-2 月）

到这时候你已经熟悉了项目风格，再学这些：

### 1. `async/await`

这是这个项目的基础范式。

- 先理解概念：不是多线程，是"协作式让出控制权"
- 然后看 `background/worker.py` 这种实际用法

### 2. 理解 Tool 系统的注册机制

- 看 `tools/__init__.py`，搞懂一个 tool 是怎么被定义、注册、被 LLM 调用的
- 这是你未来写新 tool 的基础

### 3. 理解 kosong 这个包

它是 LLM 对话的核心抽象层。

- `packages/kosong/` 里的 `chat_provider/`, `tooling/`, `message.py` 是关键

---

## 一些"反常识"的建议

1. **不要系统学 Python 教程**。你会花 2 周学 `class 继承` 之类的，然后发现这个项目里 `pydantic.BaseModel` 把传统 OOP 都替代了。**遇到什么查什么**。

2. **`git log` 是你最好的老师**。看最近的 commit 和 PR，你能学到：
   - 这个团队的代码风格
   - 改一个功能通常要动哪些文件
   - PR 的粒度和 review 标准

3. **`make check` 会教你写规范代码**。`ruff` + `pyright` 的严格检查意味着你提交的代码会被自动纠正风格，利用这一点。

4. **先读 `AGENTS.md`**。这个文件 7000+ 字，是给 AI Agent（也是给开发者）看的项目导航，里面有架构说明和开发约定。

5. **不要怕问维护者"蠢问题"**。你被邀请做核心贡献者，说明他们看中的不只是你现在的技术水平。尽早建立沟通习惯，比闷头看代码有效 10 倍。

---

## 推荐的学习资源（精简版）

| 内容 | 资源 |
|---|---|
| Python 速查 | 遇到语法不懂时查 [Python 官方文档](https://docs.python.org/3/) |
| async 入门 | 搜 "Python asyncio for beginners"，看一篇就够 |
| pydantic | [pydantic 官方文档](https://docs.pydantic.dev/)，重点看 Model 定义 |
| uv 工具链 | [uv 文档](https://docs.astral.sh/uv/)，理解依赖管理 |
| Git 协作 | 会 `branch → commit → push → PR` 流程即可 |

---

## 总结

**不要试图"准备好了再开始"，直接从跑测试、读测试、改小 bug 开始。3 个月后回头看，你会发现自己已经懂了大部分。**

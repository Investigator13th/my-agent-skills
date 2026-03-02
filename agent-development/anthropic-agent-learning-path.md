# Anthropic Agent 学习路线：15 篇必读博客全解析

**场景说明**：本文档整理自 Anthropic 工程博客，面向正在构建 AI Agent 的开发者，提供一条从基础架构到生产部署的系统学习路径。

**关键词**：Agent、Anthropic、ReAct、Tool Use、Planning、RAG、Context Engineering、Multi-Agent、Evals、Claude

---

> **核心理念**：在这个日新月异的领域，盲目追逐新框架不如深挖底层逻辑，因为工具会变，但工程原理永存。

按照 **架构 → 工具 → 上下文 → 协作 → 评测** 的顺序研读这 15 篇文章，构建从入门到生产的完整能力体系。

---

## Agent 能力金字塔

```
         ┌─────────────────────────────┐
         │          生产部署            │  ← 安全 | 评测 | 监控
         └─────────────────────────────┘
                        ↑
         ┌─────────────────────────────┐
         │        长任务与协作          │  ← 状态管理 | 多 Agent | 恢复机制
         └─────────────────────────────┘
                        ↑
         ┌─────────────────────────────┐
         │         上下文工程          │  ← 记忆管理 | RAG | Contextual Retrieval
         └─────────────────────────────┘
                        ↑
         ┌─────────────────────────────┐
         │          工具能力           │  ← Tool Use | Think Tool | Agent Skills
         └─────────────────────────────┘
                        ↑
         ┌─────────────────────────────┐
         │          基础架构           │  ← Workflow vs Agent | ReAct | Planning
         └─────────────────────────────┘
```

---

## 模块一：Agent 基础架构（入门篇）

**核心建议**：Start Simple。很多时候你只需要一个定义良好的工作流，而非不可控的 Agent。

必读：《Building effective agents》厘清了 Workflow 与 Agent 的区别，并提供基础构建模式（ReAct、Tool Use、Planning）。

| # | 参考文章 | 内容 | 核心价值 |
|---|---------|------|---------|
| 1 | **Building effective agents** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Agent 架构入门：从单轮对话到自主代理 | 理解 Agent 的基本模式：ReAct、Tool Use、Planning |
| 2 | **Building agents with the Claude Agent SDK** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | 用 Agent SDK 构建你的第一个 Agent | 实战入门，快速上手 |

---

## 模块二：工具与能力扩展（进阶篇）

**核心建议**：工具描述（Description）的设计至关重要，直接影响 Agent 使用效果。

**"思考工具"**（The "think" tool）是神技：让 Agent 在行动前先进行链式思考，显著提升复杂推理任务的表现。

| # | 参考文章 | 内容 | 核心价值 |
|---|---------|------|---------|
| 3 | **Introducing advanced tool use** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Agent 高级工具调用：并行、嵌套与错误处理 | 工具调用的进阶技巧 |
| 4 | **Writing effective tools for agents — with agents** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | 如何为 Agent 设计好用的工具 | 工具设计原则和最佳实践 |
| 5 | **The "think" tool** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Think Tool：让 Agent 学会"停下来想一想" | 复杂推理场景的关键技巧 |
| 6 | **Equipping agents for the real world with Agent Skills** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Agent Skills：让 Agent 具备真实世界能力 | 技能封装与复用 |

---

## 模块三：上下文与记忆管理（核心篇）

**核心建议**：注意力是稀缺资源。即使上下文窗口扩大，RAG 依然关键。

创新实践 **"上下文检索"**（Contextual Retrieval）：让模型在检索前先生成解释性上下文，解决传统 RAG 片段信息缺失问题，极大提升准确率。

| # | 参考文章 | 内容 | 核心价值 |
|---|---------|------|---------|
| 7 | **Effective context engineering for AI agents** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | 上下文工程：Agent 的"记忆"与"注意力"管理 | 长对话、多轮任务的关键 |
| 8 | **Introducing Contextual Retrieval** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Contextual Retrieval：让 RAG 更懂上下文 | 检索增强的新范式 |

---

## 模块四：长任务与多 Agent 协作（高级篇）

**核心建议**：长时间运行 Agent 需设计 "Harness"：处理中断、保存状态、防止死循环。

**多 Agent 系统架构参考**：Orchestrator-Workers 模式适合复杂项目。

| # | 参考文章 | 内容 | 核心价值 |
|---|---------|------|---------|
| 9 | **Effective harnesses for long-running agents** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | 长时间运行的 Agent：如何设计可靠的执行框架 | 任务中断恢复、复状态持久化 |
| 10 | **How we built our multi-agent research system** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | 多 Agent 协作系统：Anthropic 的实战经验 | 多 Agent 架构设计 |
| 11 | **Code execution with MCP** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | MCP 代码执行：构建更高效的 Agent | Agent 执行环境设计 |

---

## 模块五：安全、评测与工程化（生产篇）

**核心建议**：
- **没有评测（Evals），就不要上线**。必须建立自动化测试体系，告别"凭感觉"。
- **安全沙箱是必须**：当 Agent 能执行代码时，需通过沙箱与协议控制风险。
- **学习真实失败案例**：了解 Agent 在现实中如何"花式犯错"是宝贵的实战经验。

| # | 参考文章 | 内容 | 核心价值 |
|---|---------|------|---------|
| 12 | **Demystifying evals for AI agents** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Agent 评测怎么做 | 评测体系设计 |
| 13 | **Beyond permission prompts: Claude Code sandboxing** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Agent 安全：从权限提示到沙箱隔离 | 安全与自主性的平衡 |
| 14 | **Claude Code: Best practices for agentic coding** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Coding Agent 最佳实践 | Coding Agent 的工程经验 |
| 15 | **A postmortem of three recent issues** ([anthropic.com/engineering](https://www.anthropic.com/engineering)) | Agent 故障复盘：三个真实案例分析 | 从失败中学习 |

---

## 推荐研读顺序

```
基础架构 (1→2) → 工具能力 (3→4→5→6) → 上下文 (7→8) → 多Agent (9→10→11) → 生产评测 (12→13→14→15)
```

## 参考来源

- Anthropic Engineering Blog: [anthropic.com/engineering](https://www.anthropic.com/engineering)
- 整理时间：2026-03

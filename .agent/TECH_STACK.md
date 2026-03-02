# 全栈项目技术栈配置

本文档记录了你的技术栈偏好，确保所有项目开发遵循统一的技术选型。

---

## 技术栈总览

### 前端

- **框架**: Next.js 14+ (React, App Router)
- **UI 组件库**: Shadcn UI
- **样式**: TailwindCSS
- **包管理器**: **pnpm** (必须)
- **类型系统**: TypeScript

### 后端

- **框架**: FastAPI (Python)
- **ORM**: SQLAlchemy
- **数据验证**: Pydantic
- **AI 框架**: LangChain + LangGraph (如涉及 Agent)

### 数据库

- **关系型数据库**: PostgreSQL 15+
- **迁移工具**: Alembic

### 开发环境

- **Python 环境管理**: Conda
- **运行模式**: 本地开发（暂不涉及部署）

---

## 工作流程

请严格遵循以下开发流程（详见 `/fullstack-project-builder` workflow）：

1. **需求阶段**：编写完整版 PRD（使用 `prd-writing` skill）
2. **设计阶段**：编写技术设计文档，包含系统架构、数据库设计、API设计、UI/UX设计（使用 `design-doc` skill）
3. **前端开发**：先于后端，使用 Mock 数据（使用 `nextjs-dev` skill）
4. **后端开发**：实现 API，替换 Mock 数据（使用 `fastapi-dev` skill）
5. **联调测试**：前后端联调，验收

---

## 关键原则

- **包管理器**：永远使用 `pnpm`，不使用 `npm` 或 `yarn`
- **语言**：所有回复、文档使用简体中文
- **简洁优先**：遵循 KISS 原则
- **测试要求**：暂不需要单元测试，快速迭代优先

---

## Skills 索引

以下是你可以使用的部分 Skills：

| Skill | 使用场景 |
|-------|---------|
| `prd-writing` | 编写产品需求文档 |
| `design-doc` | 编写技术设计文档 |
| `nextjs-dev` | Next.js 前端开发 |
| `fastapi-dev` | FastAPI 后端开发 |
| `database-design` | PostgreSQL 数据库设计 |
| `api-design` | RESTful API 设计 |
| `prompt-engineering` | AI Agent Prompt 设计 |
| `ui-ux-pro-max` | UI/UX 设计 |

---

## Workflows 索引

| Workflow | 说明 |
|----------|------|
| `/fullstack-project-builder` | 全栈项目开发完整流程 |

---

**最后更新**: 2026-02-11

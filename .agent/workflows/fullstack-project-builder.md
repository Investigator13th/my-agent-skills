---
description: 全栈项目开发完整流程（AI Agent + 数据看板）
---

# 全栈项目开发工作流

本工作流适用于 **AI Agent 应用** 和 **数据看板** 类项目的完整开发周期。

## 技术栈

- **前端**: Next.js (React) + TailwindCSS + Shadcn UI + pnpm
- **后端**: Python (FastAPI) + LangGraph（如涉及 Agent）
- **数据库**: PostgreSQL
- **运行环境**: 本地开发

---

## 阶段 1：需求分析与文档编写

### 1.1 编写产品需求文档（PRD）

**目标**: 明确项目的业务目标、用户需求、功能列表和验收标准。

**操作步骤**:
1. 阅读 `prd-writing` skill，了解需求文档的完整格式。
2. 与用户确认以下核心信息：
   - 项目背景与目标
   - 目标用户画像
   - 核心功能列表（按优先级排序）
   - 非功能性需求（性能、安全性等）
3. 创建 `docs/PRD.md` 文档，按照模板填写。
4. 与用户确认需求文档，获得批准后进入下一阶段。

**输出物**: `docs/PRD.md`

---

## 阶段 2：技术设计

### 2.1 编写技术设计文档

**目标**: 将需求转化为可执行的技术方案，包含架构设计、数据库设计、API 设计、UI/UX 设计。

**操作步骤**:
1. 阅读 `design-doc` skill，了解设计文档的结构要求。
2. **系统架构设计**:
   - 画出前后端分层架构图
   - 明确数据流向（用户 → 前端 → API → 后端 → 数据库）
3. **数据库设计**:
   - 阅读 `database-design` skill
   - 设计 ER 图（可使用占位符，不贴图）
   - 定义表结构、字段类型、索引、外键关系
4. **API 设计**:
   - 阅读 `api-design` skill
   - 列出所有 API 端点（RESTful 风格）
   - 定义请求/响应的数据结构（JSON Schema）
5. **UI/UX 设计**:
   - 如果已安装 `ui-ux-pro-max` skill，阅读该 skill
   - 定义页面结构、组件树、交互流程
   - 确定设计系统（颜色、字体、间距）
6. **如果涉及 AI Agent**:
   - 阅读 `prompt-engineering` skill
   - 设计 Agent 的角色、工具链、思维链流程
7. 创建 `docs/DESIGN.md` 文档。
8. 与用户确认设计文档，获得批准后进入开发阶段。

**输出物**: `docs/DESIGN.md`

---

## 阶段 3：前端开发

**📌 注意**: 按照"先前端"的策略，前端开发会先于后端。此时使用 **Mock 数据**，但数据结构必须与设计文档中的 API 响应格式一致。

### 3.1 初始化前端项目

// turbo
```bash
cd c:\Users\Admin\Documents\demo\Recap
mkdir frontend
cd frontend
npx -y create-next-app@latest ./ --typescript --tailwind --app --no-src-dir --import-alias "@/*"
```

### 3.2 安装必要依赖

// turbo
```bash
cd c:\Users\Admin\Documents\demo\Recap\frontend
pnpm install
pnpm add @radix-ui/react-* class-variance-authority clsx tailwind-merge lucide-react
pnpm add -D @types/node
```

### 3.3 配置 Shadcn UI

// turbo
```bash
cd c:\Users\Admin\Documents\demo\Recap\frontend
npx shadcn@latest init -y
```

### 3.4 前端开发流程

1. **阅读规范**: 查看 `nextjs-dev` skill，了解组件组织、样式规范。
2. **创建 Mock 数据**: 在 `frontend/lib/mock-data.ts` 中创建符合 API 设计的假数据。
3. **搭建路由结构**: 根据设计文档在 `app/` 目录下创建页面。
4. **开发组件**: 
   - 在 `components/` 下按功能模块组织组件
   - 优先使用 Shadcn UI 组件，需要时进行二次封装
5. **实现页面**: 将组件组装成完整页面，使用 Mock 数据进行渲染。
6. **样式调优**: 确保符合设计文档中的 UI/UX 规范。

### 3.5 本地运行前端

// turbo
```bash
cd c:\Users\Admin\Documents\demo\Recap\frontend
pnpm dev
```

访问 `http://localhost:3000` 验证前端功能。

**输出物**: `frontend/` 目录（完整的 Next.js 项目）

---

## 阶段 4：后端开发

### 4.1 初始化后端项目

// turbo
```bash
cd c:\Users\Admin\Documents\demo\Recap
mkdir backend
cd backend
```

### 4.2 创建虚拟环境并安装依赖

**⚠️ 需要用户指定虚拟环境名称（如之前的 `demo`）**

```bash
# 激活已有的虚拟环境（假设名为 demo）
conda activate demo

# 安装核心依赖
pip install fastapi uvicorn sqlalchemy psycopg2-binary pydantic python-dotenv

# 如果涉及 AI Agent
pip install langchain langchain-openai langgraph
```

### 4.3 后端开发流程

1. **阅读规范**: 查看 `fastapi-dev` skill，了解项目结构、命名规范。
2. **创建项目结构**:
   ```
   backend/
   ├── app/
   │   ├── main.py          # FastAPI 应用入口
   │   ├── models/          # SQLAlchemy 数据库模型
   │   ├── schemas/         # Pydantic 数据验证模型
   │   ├── api/             # API 路由
   │   ├── services/        # 业务逻辑层
   │   ├── db/              # 数据库连接配置
   │   └── utils/           # 工具函数
   ├── .env                 # 环境变量
   └── requirements.txt     # 依赖列表
   ```
3. **配置数据库**:
   - 阅读 `database-design` skill
   - 在本地启动 PostgreSQL
   - 创建数据库和表（使用 SQLAlchemy 的 ORM）
4. **实现 API**:
   - 按照设计文档中的 API 设计，逐个实现端点
   - 确保请求/响应格式与前端 Mock 数据一致
5. **如果涉及 AI Agent**:
   - 阅读 `prompt-engineering` skill
   - 在 `app/agents/` 目录下实现 LangGraph 逻辑

### 4.4 本地运行后端

// turbo
```bash
cd c:\Users\Admin\Documents\demo\Recap\backend
uvicorn app.main:app --reload --port 8000
```

访问 `http://localhost:8000/docs` 查看自动生成的 API 文档。

**输出物**: `backend/` 目录（完整的 FastAPI 项目）

---

## 阶段 5：前后端联调

### 5.1 连接真实 API

1. 在前端删除 Mock 数据，改为调用后端 API。
2. 处理 CORS（在 FastAPI 中添加 CORS 中间件）。
3. 测试所有功能点，确保前后端数据流通畅。

### 5.2 验收测试

1. 对照 PRD 中的验收标准，逐项检查。
2. 修复 bug，优化用户体验。

---

## 阶段 6：文档交付

### 6.1 更新 README.md

创建项目根目录的 `README.md`，包含：
- 项目简介
- 技术栈
- 本地运行步骤（前后端启动命令）
- 环境变量配置说明

### 6.2 代码注释与文档

- 关键业务逻辑添加注释
- 复杂的 Agent 流程添加流程图说明

---

## 完成标志

- ✅ PRD 和设计文档已完成并获批
- ✅ 前端可在浏览器正常访问，UI 符合设计
- ✅ 后端 API 全部实现，可通过 `/docs` 查看
- ✅ 数据库表结构创建完成，数据可正常增删改查
- ✅ 前后端联调成功，所有功能点通过验收
- ✅ README.md 完善，他人可根据文档快速启动项目

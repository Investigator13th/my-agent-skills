---
name: 技术设计文档规范
description: 编写技术设计文档(Technical Design Document)的完整指南。将PRD需求转化为可执行的技术方案。包含系统架构、数据库设计、API设计、UI/UX设计。当完成PRD后需要进行技术设计时使用。适用于全栈项目开发流程的第二阶段。
---

# 技术设计文档规范

技术设计文档（TDD）是连接需求与实现的桥梁。它将 PRD 中的业务需求转化为具体的技术方案。

---

## 文档结构模板

```markdown
# 技术设计文档（TDD）

**项目名称**: [项目名称]  
**对应 PRD**: PRD.md v1.0  
**文档版本**: v1.0  
**创建日期**: YYYY-MM-DD  
**负责人**: [姓名]

---

## 1. 系统架构

### 1.1 技术栈

| 层级 | 技术选型 | 版本 | 说明 |
|-----|---------|------|------|
| 前端 | Next.js | 14+ | React 框架，支持 SSR |
| 前端 UI | Shadcn UI + TailwindCSS | - | 组件库与样式 |
| 后端 | FastAPI | 0.100+ | Python Web 框架 |
| 数据库 | PostgreSQL | 15+ | 关系型数据库 |
| AI 框架 | LangGraph (可选) | - | 如涉及 Agent |

### 1.2 系统分层架构

[占位符：三层架构图]

```
用户浏览器
    ↓
Next.js 前端 (Port 3000)
    ↓ HTTP/WebSocket
FastAPI 后端 (Port 8000)
    ↓ SQL/ORM
PostgreSQL 数据库 (Port 5432)
```

### 1.3 数据流向

**示例：用户提交表单**
1. 用户在前端填写表单
2. 前端进行客户端验证（Zod/React Hook Form）
3. 通过 `fetch()` 调用后端 API `/api/submit`
4. 后端验证数据（Pydantic）
5. 后端通过 SQLAlchemy ORM 存入数据库
6. 返回 JSON 响应给前端
7. 前端展示成功/失败提示

---

## 2. 数据库设计

> **提示**: 请参考本文档末尾的《PostgreSQL 数据库设计规范》。

### 2.1 ER 图

[占位符：实体关系图]

### 2.2 数据表设计

#### 表1: users (用户表)

| 字段名 | 类型 | 约束 | 说明 |
|-------|------|------|------|
| id | UUID | PRIMARY KEY | 用户唯一标识 |
| username | VARCHAR(50) | UNIQUE, NOT NULL | 用户名 |
| email | VARCHAR(100) | UNIQUE, NOT NULL | 邮箱 |
| hashed_password | VARCHAR(255) | NOT NULL | 密码哈希值 |
| created_at | TIMESTAMP | DEFAULT NOW() | 创建时间 |
| updated_at | TIMESTAMP | ON UPDATE NOW() | 更新时间 |

**索引设计**:
- `idx_email` on (`email`)
- `idx_created_at` on (`created_at`)

**外键关系**:
无

---

#### 表2: [表名]
[重复上述结构]

---

## 3. API 设计

> **提示**: 请参考本文档末尾的《RESTful API 设计规范》。

### 3.1 API 端点列表

| 端点 | 方法 | 描述 | 鉴权 |
|-----|------|------|------|
| `/api/auth/register` | POST | 用户注册 | 否 |
| `/api/auth/login` | POST | 用户登录 | 否 |
| `/api/users/me` | GET | 获取当前用户信息 | 是 |
| `/api/dashboard/stats` | GET | 获取看板统计数据 | 是 |

### 3.2 API 详细设计

#### POST /api/auth/register

**请求体** (JSON):
```json
{
  "username": "string (3-50 chars)",
  "email": "string (valid email)",
  "password": "string (8-50 chars)"
}
```

**成功响应** (201 Created):
```json
{
  "id": "uuid",
  "username": "string",
  "email": "string",
  "created_at": "ISO 8601 timestamp"
}
```

**失败响应** (400 Bad Request):
```json
{
  "detail": "Email already registered"
}
```

---

#### GET /api/dashboard/stats

**请求参数** (Query String):
- `start_date`: ISO 8601 日期，可选
- `end_date`: ISO 8601 日期，可选

**请求头**:
```
Authorization: Bearer <JWT_TOKEN>
```

**成功响应** (200 OK):
```json
{
  "total_users": 1234,
  "active_users_today": 567,
  "revenue": {
    "total": 98765.43,
    "currency": "USD"
  },
  "charts": {
    "daily_active_users": [
      {"date": "2026-02-01", "count": 100},
      {"date": "2026-02-02", "count": 120}
    ]
  }
}
```

---

## 4. UI/UX 设计

> **提示**: 如已安装 `ui-ux-pro-max` skill，可查看详细的 UI/UX 设计规范。

### 4.1 页面结构

| 路由 | 页面名称 | 说明 |
|-----|---------|------|
| `/` | 首页 | 登录后跳转到 `/dashboard` |
| `/login` | 登录页 | 用户登录 |
| `/register` | 注册页 | 用户注册 |
| `/dashboard` | 数据看板 | 主要数据展示 |

### 4.2 组件树

**Dashboard 页面组件结构**:
```
<DashboardLayout>
  ├── <Header>
  │   ├── <Logo>
  │   ├── <UserMenu>
  ├── <Sidebar>
  │   ├── <NavItem>
  │   ├── <NavItem>
  ├── <MainContent>
  │   ├── <StatsCards>
  │   │   ├── <StatCard title="总用户数">
  │   │   ├── <StatCard title="今日活跃">
  │   ├── <ChartSection>
  │   │   ├── <LineChart data={dailyActiveUsers}>
```

### 4.3 设计系统

**颜色规范**:
- Primary: `hsl(221, 83%, 53%)` (蓝色)
- Secondary: `hsl(212, 12%, 15%)` (深灰)
- Success: `hsl(142, 76%, 36%)` (绿色)
- Danger: `hsl(0, 84%, 60%)` (红色)
- Background: `hsl(0, 0%, 100%)` (白色)
- Foreground: `hsl(222, 47%, 11%)` (深灰)

**字体**:
- Primary: `Inter`, sans-serif
- Monospace: `Fira Code`, monospace

**间距系统**:
- xs: 4px
- sm: 8px
- md: 16px
- lg: 24px
- xl: 32px

### 4.4 交互流程

**用户登录流程**:
1. 用户访问 `/login` 页面
2. 输入邮箱和密码
3. 点击"登录"按钮
4. 前端验证输入格式
5. 调用 API `/api/auth/login`
6. 成功：存储 JWT，跳转到 `/dashboard`
7. 失败：显示错误提示（如"邮箱或密码错误"）

---

## 5. AI Agent 设计（如适用）

> **提示**: 可使用 `prompt-engineering` skill 查看详细的 Agent 设计规范。

### 5.1 Agent 架构

**Agent 类型**: [单 Agent / 多 Agent 协作]

**工具链**:
- Tool 1: `search_knowledge_base` - 从知识库检索信息
- Tool 2: `query_database` - 查询业务数据
- Tool 3: `generate_report` - 生成分析报告

### 5.2 思维链流程

[占位符：LangGraph 流程图]

```
用户输入
  ↓
Router (路由 Agent)
  ↓ (问题分类)
  ├→ FAQ Agent (常见问题)
  ├→ Data Analyst Agent (数据分析)
  └→ General Agent (通用对话)
  ↓
生成回复
```

### 5.3 Prompt 设计要点

- **角色定位**: [例如："你是一个专业的客服助手"]
- **知识边界**: [明确 Agent 能回答和不能回答的内容]
- **输出格式**: [JSON / Markdown / 自然语言]

---

## 6. 安全设计

### 6.1 鉴权方案

**方案**: JWT (JSON Web Token)

**流程**:
1. 用户登录成功后，后端生成 JWT (有效期 24h)
2. 前端存储在 `localStorage`
3. 每次请求通过 `Authorization: Bearer <token>` 发送
4. 后端中间件验证 JWT 有效性
5. 过期时需重新登录

### 6.2 数据加密

- 密码存储：使用 `bcrypt` 哈希
- 敏感数据：AES-256 加密
- 数据传输：HTTPS

---

## 7. 性能优化

### 7.1 前端优化

- 使用 Next.js 的 Image 组件自动优化图片
- 代码分割（Dynamic Import）减少首屏加载时间
- 使用 React Server Components 减少客户端 JS

### 7.2 后端优化

- 数据库查询优化（建立索引）
- 使用 Redis 缓存热点数据
- API 响应使用 `response_model` 避免返回多余字段

---

## 8. 部署方案（本地开发）

### 8.1 前端

```bash
cd frontend
pnpm install
pnpm dev  # 运行在 http://localhost:3000
```

### 8.2 后端

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload  # 运行在 http://localhost:8000
```

### 8.3 数据库

```bash
# 启动 PostgreSQL (假设已安装)
pg_ctl -D /path/to/data start

# 创建数据库
createdb myproject_db

# 运行迁移（使用 Alembic）
alembic upgrade head
```

---

## 9. 测试策略

暂不需要测试（根据用户需求）。

---

## 10. 里程碑与交付物

| 里程碑 | 交付物 | 截止日期 |
|-------|-------|---------|
| M1 | 完成数据库设计与 API 设计 | [日期] |
| M2 | 完成前端 UI 原型 | [日期] |
| M3 | 完成后端 API 实现 | [日期] |
| M4 | 完成前后端联调 | [日期] |

---

## 11. 风险与技术挑战

| 风险 | 影响 | 缓解措施 |
|-----|------|---------|
| PostgreSQL 性能瓶颈 | 中 | 提前设计索引，使用连接池 |
| LLM API 延迟高 | 高 | 增加超时重试，考虑流式输出 |
| 前端状态管理复杂 | 低 | 优先使用 React Server Actions |

---

## 12. 附录：技术选型理由

- **Next.js**：一站式解决 SSR、路由、优化问题，开发效率高
- **FastAPI**：异步高性能，自动生成 API 文档，与 LangChain 生态兼容好
- **PostgreSQL**：成熟稳定，支持复杂查询，开源免费
- **LangGraph**：比 LangChain 更灵活的 Agent 编排，适合复杂 Agent 场景

```

---

## 设计文档编写要点

1. **从 PRD 出发**：每个技术决策都应对应 PRD 中的需求。
2. **API 设计优先**：前后端通过 API 契约解耦，可并行开发。
3. **数据库设计先行**：数据结构确定后，API 和前端都围绕它展开。
4. **预留扩展空间**：设计时考虑未来 1-2 个版本的可能需求。

---

## 常见陷阱

- ❌ **过早优化**：在 MVP 阶段就考虑"支持百万并发"。
  - ✅ **正确做法**：先保证功能可用，后续根据实际瓶颈优化。

- ❌ **忽视数据结构**：直接开始写代码，后续频繁修改数据库。
  - ✅ **正确做法**：花时间设计好 ER 图，避免返工。

- ❌ **API 设计不一致**：有的用 RESTful，有的用 GraphQL，有的自定义。
  - ✅ **正确做法**：统一风格，参考本文档附录的 API 设计规范。


# PostgreSQL 数据库设计规范

---

## 设计流程

1. **需求分析** → 从 PRD 提取数据实体
2. **ER 图设计** → 绘制实体关系图
3. **规范化** → 确保至少满足第三范式（3NF）
4. **表结构设计** → 定义字段类型、约束
5. **索引优化** → 为常用查询字段添加索引

---

## ER 图设计

### 实体识别

从 PRD 中识别**名词**作为候选实体：

**示例 PRD 描述**:
> "用户可以创建直播活动，每个活动包含多个商品。用户可以查看活动的数据统计。"

**识别出的实体**:
- User (用户)
- LiveEvent (直播活动)
- Product (商品)
- EventStats (活动统计)

### 关系识别

- User ↔ LiveEvent：一对多（一个用户可创建多个活动）
- LiveEvent ↔ Product：多对多（一个活动包含多个商品，一个商品可出现在多个活动）
- LiveEvent ↔ EventStats：一对一（一个活动对应一份统计数据）

### ER 图示例

```
[User] 1--* [LiveEvent] *--* [Product]
              |
              1
              |
         [EventStats]
```

---

## 表结构设计

### 命名规范

- **表名**: 小写 + 下划线，复数形式，如 `users`, `live_events`
- **字段名**: 小写 + 下划线，如 `user_id`, `created_at`
- **主键**: 统一命名为 `id`
- **外键**: `<关联表单数>_id`，如 `user_id`, `event_id`

---

### 数据类型选择

| 数据类型 | PostgreSQL 类型 | 适用场景 |
|---------|----------------|---------|
| 主键 | `UUID` 或 `SERIAL` | UUID 适合分布式，SERIAL 适合单体 |
| 短文本 | `VARCHAR(n)` | 用户名、邮箱（限定长度） |
| 长文本 | `TEXT` | 文章内容、描述 |
| 整数 | `INTEGER` | 年龄、数量 |
| 大整数 | `BIGINT` | 累计金额（分） |
| 浮点数 | `NUMERIC(10,2)` | 金额（避免精度丢失） |
| 布尔值 | `BOOLEAN` | 是否激活 |
| 时间戳 | `TIMESTAMP` | 创建时间、更新时间 |
| JSON | `JSONB` | 灵活的元数据存储 |

---

### 表设计示例

#### 1. users (用户表)

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

---

#### 2. live_events (直播活动表)

```sql
CREATE TABLE live_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    status VARCHAR(20) DEFAULT 'draft',  -- draft, live, ended
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_live_events_user_id ON live_events(user_id);
CREATE INDEX idx_live_events_start_time ON live_events(start_time);
CREATE INDEX idx_live_events_status ON live_events(status);
```

---

#### 3. products (商品表)

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,  -- 价格，保留两位小数
    stock INTEGER DEFAULT 0,
    image_url TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_products_name ON products(name);
```

---

#### 4. event_products (活动-商品关联表，多对多)

```sql
CREATE TABLE event_products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id UUID NOT NULL REFERENCES live_events(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    display_order INTEGER DEFAULT 0,  -- 商品在活动中的展示顺序
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(event_id, product_id)  -- 防止重复添加同一商品
);

-- 索引
CREATE INDEX idx_event_products_event_id ON event_products(event_id);
CREATE INDEX idx_event_products_product_id ON event_products(product_id);
```

---

#### 5. event_stats (活动统计表，一对一)

```sql
CREATE TABLE event_stats (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id UUID UNIQUE NOT NULL REFERENCES live_events(id) ON DELETE CASCADE,
    total_views INTEGER DEFAULT 0,
    peak_viewers INTEGER DEFAULT 0,
    total_revenue NUMERIC(12, 2) DEFAULT 0,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_event_stats_event_id ON event_stats(event_id);
```

---

## 索引优化

### 何时添加索引？

- ✅ **主键/外键**: 自动创建索引
- ✅ **WHERE 条件**: 经常用于查询条件的字段（如 `email`, `status`）
- ✅ **JOIN 关联**: 外键字段
- ✅ **ORDER BY / GROUP BY**: 排序/分组字段

### 常见索引类型

```sql
-- 单列索引
CREATE INDEX idx_users_email ON users(email);

-- 复合索引（多列）
CREATE INDEX idx_events_user_status ON live_events(user_id, status);

-- 唯一索引
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- 部分索引（条件索引）
CREATE INDEX idx_active_users ON users(email) WHERE is_active = TRUE;
```

---

## 外键约束

### 级联删除

```sql
-- 当用户被删除时，自动删除其所有活动
user_id UUID REFERENCES users(id) ON DELETE CASCADE
```

### 级联更新

```sql
-- 当用户 ID 更新时，自动更新关联表
user_id UUID REFERENCES users(id) ON UPDATE CASCADE
```

### NULL 处理

```sql
-- 允许外键为空（可选关联）
user_id UUID REFERENCES users(id) ON DELETE SET NULL
```

---

## 规范化（Normalization）

### 第一范式 (1NF)：原子性

❌ **错误**：一个字段存储多个值
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    tags TEXT  -- "tag1,tag2,tag3" ❌ 违反 1NF
);
```

✅ **正确**：拆分为独立表
```sql
CREATE TABLE user_tags (
    user_id UUID REFERENCES users(id),
    tag VARCHAR(50)
);
```

---

### 第二范式 (2NF)：消除部分依赖

❌ **错误**：非主键字段依赖于复合主键的一部分
```sql
CREATE TABLE order_items (
    order_id UUID,
    product_id UUID,
    product_name VARCHAR(200),  -- ❌ 仅依赖 product_id
    PRIMARY KEY (order_id, product_id)
);
```

✅ **正确**：将 `product_name` 移至 `products` 表

---

### 第三范式 (3NF)：消除传递依赖

❌ **错误**：非主键字段依赖于另一个非主键字段
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    city VARCHAR(50),
    country VARCHAR(50)  -- ❌ country 依赖于 city
);
```

✅ **正确**：拆分为 `cities` 表

---

## 常用查询优化

### 使用 EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

**关注指标**:
- `Seq Scan` (全表扫描) → ❌ 慢
- `Index Scan` (索引扫描) → ✅ 快

---

### 避免 SELECT *

```sql
-- ❌ 慢：返回所有字段
SELECT * FROM users WHERE id = '...';

-- ✅ 快：只返回需要的字段
SELECT id, username, email FROM users WHERE id = '...';
```

---

## 数据迁移（使用 Alembic）

### 初始化

```bash
alembic init alembic
```

### 生成迁移

```bash
alembic revision --autogenerate -m "create users table"
```

### 执行迁移

```bash
alembic upgrade head
```

---

## 最佳实践

1. **UUID vs SERIAL**：分布式系统用 UUID，单体系统用 SERIAL（性能更好）
2. **时间戳**：统一使用 `TIMESTAMP` 而非 `DATE`，需要时区时用 `TIMESTAMPTZ`
3. **布尔字段**：避免用 `0/1`，使用 `BOOLEAN`
4. **JSON 字段**：使用 `JSONB`（支持索引）而非 `JSON`
5. **软删除**：添加 `deleted_at` 字段，而非真删除

---

## 常见错误

- ❌ **过度规范化**：导致查询需要大量 JOIN
  - ✅ 适度冗余，提升查询性能

- ❌ **缺少索引**：大表全表扫描
  - ✅ 为常用查询字段添加索引

- ❌ **外键命名混乱**：`fk_1`, `fk_2`
  - ✅ 语义化命名：`user_id`, `event_id`

# RESTful API 设计规范

---

## 核心原则

1. **资源导向**：URL 代表资源，动词通过 HTTP 方法表达
2. **无状态**：每个请求包含所有必要信息（通过 JWT 传递认证）
3. **统一接口**：遵循标准 HTTP 方法和状态码
4. **分层系统**：客户端无需知道是否直接连接到后端

---

## URL 设计

### 命名规范

- **小写字母 + 连字符**：`/api/user-profiles` 而非 `/api/UserProfiles`
- **复数形式**：`/api/users` 而非 `/api/user`
- **资源嵌套**：`/api/users/{id}/orders` 表示"用户的订单"

```
✅ 正确
GET /api/users
GET /api/users/123
GET /api/users/123/orders
POST /api/orders

❌ 错误
GET /api/getUsers          # 动词应通过 HTTP 方法体现
GET /api/user/123          # 应使用复数
GET /api/users/getAllOrders  # URL 不应包含动词
```

---

## HTTP 方法

| 方法 | 语义 | 幂等性 | 示例 |
|-----|------|-------|------|
| GET | 获取资源 | ✅ | `GET /api/users` - 获取用户列表 |
| POST | 创建资源 | ❌ | `POST /api/users` - 创建用户 |
| PUT | 完整更新资源 | ✅ | `PUT /api/users/123` - 更新用户全部字段 |
| PATCH | 部分更新资源 | ❌ | `PATCH /api/users/123` - 更新用户部分字段 |
| DELETE | 删除资源 | ✅ | `DELETE /api/users/123` - 删除用户 |

**幂等性说明**：多次执行相同请求，结果一致（如 GET、PUT、DELETE），POST 不是幂等的（多次创建会生成多个资源）。

---

## HTTP 状态码

### 成功响应（2xx）

| 状态码 | 含义 | 使用场景 |
|-------|------|---------|
| 200 OK | 成功 | GET、PUT、PATCH 成功 |
| 201 Created | 创建成功 | POST 创建资源成功 |
| 204 No Content | 成功但无返回内容 | DELETE 成功 |

### 客户端错误（4xx）

| 状态码 | 含义 | 使用场景 |
|-------|------|---------|
| 400 Bad Request | 请求格式错误 | 参数缺失、类型错误 |
| 401 Unauthorized | 未认证 | 缺少或无效的 Token |
| 403 Forbidden | 无权限 | 已登录但无访问权限 |
| 404 Not Found | 资源不存在 | 请求的 ID 不存在 |
| 409 Conflict | 资源冲突 | 邮箱已注册 |
| 422 Unprocessable Entity | 语义错误 | 数据验证失败 |

### 服务器错误（5xx）

| 状态码 | 含义 | 使用场景 |
|-------|------|---------|
| 500 Internal Server Error | 服务器错误 | 未预料的异常 |
| 503 Service Unavailable | 服务不可用 | 数据库连接失败 |

---

## 请求/响应格式

### 统一使用 JSON

**请求示例**:
```http
POST /api/users
Content-Type: application/json

{
  "username": "alice",
  "email": "alice@example.com",
  "password": "SecurePassword123"
}
```

**成功响应示例** (201 Created):
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "alice",
  "email": "alice@example.com",
  "created_at": "2026-02-11T10:30:00Z"
}
```

**失败响应示例** (400 Bad Request):
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "detail": "Email already registered"
}
```

---

## API 端点设计示例

### 1. 用户管理

| 端点 | 方法 | 描述 | 请求体 | 响应体 |
|-----|------|------|-------|-------|
| `/api/users` | GET | 获取用户列表 | 无 | `User[]` |
| `/api/users` | POST | 创建用户 | `UserCreate` | `User` |
| `/api/users/{id}` | GET | 获取单个用户 | 无 | `User` |
| `/api/users/{id}` | PUT | 完整更新用户 | `UserUpdate` | `User` |
| `/api/users/{id}` | PATCH | 部分更新用户 | `UserPartial` | `User` |
| `/api/users/{id}` | DELETE | 删除用户 | 无 | 无 (204) |

---

### 2. 认证

| 端点 | 方法 | 描述 | 请求体 | 响应体 |
|-----|------|------|-------|-------|
| `/api/auth/register` | POST | 注册 | `UserCreate` | `User` |
| `/api/auth/login` | POST | 登录 | `UserLogin` | `{ access_token, token_type }` |
| `/api/auth/me` | GET | 获取当前用户 | 无 (需JWT) | `User` |

---

### 3. 资源嵌套

**场景**：获取某个用户的所有订单

```
GET /api/users/123/orders
```

**响应**:
```json
[
  {
    "id": "order-1",
    "user_id": "123",
    "total": 99.99,
    "created_at": "2026-02-10T10:00:00Z"
  }
]
```

---

## 分页、排序、筛选

### 分页（Pagination）

**请求**:
```
GET /api/users?page=2&per_page=20
```

**响应**:
```json
{
  "data": [ /* User[] */ ],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

---

### 排序（Sorting）

**请求**:
```
GET /api/users?sort=created_at&order=desc
```

---

### 筛选（Filtering）

**请求**:
```
GET /api/users?status=active&role=admin
```

---

## 认证与授权

### 使用 JWT

**登录流程**:
1. 用户 POST `/api/auth/login` 提交邮箱+密码
2. 后端验证成功，返回 JWT：
   ```json
   {
     "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
     "token_type": "bearer"
   }
   ```
3. 前端存储 Token（如 `localStorage`）
4. 后续请求携带 Token：
   ```http
   GET /api/users/me
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   ```

---

## 错误响应格式

### 统一错误结构

```json
{
  "detail": "错误描述",
  "code": "ERROR_CODE",  // 可选：自定义错误码
  "field": "email"        // 可选：哪个字段出错
}
```

**示例**:
```json
{
  "detail": "Email already registered",
  "code": "EMAIL_EXISTS",
  "field": "email"
}
```

---

## 版本控制

### URL 版本控制（推荐）

```
GET /api/v1/users
GET /api/v2/users
```

### Header 版本控制

```http
GET /api/users
Accept: application/vnd.myapi.v1+json
```

---

## CORS 配置

在 FastAPI 中允许前端跨域访问：

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # 前端地址
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 自动文档

FastAPI 自动生成 API 文档：

- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

---

## 最佳实践

1. **使用名词而非动词**：`/api/users` 而非 `/api/getUsers`
2. **资源复数**：`/api/users` 而非 `/api/user`
3. **状态码语义化**：201 表示创建，204 表示删除
4. **分页默认限制**：避免一次返回数万条数据
5. **版本控制**：预留升级空间

---

## 常见错误

- ❌ **URL 包含动词**：`/api/deleteUser/123`
  - ✅ 使用 `DELETE /api/users/123`

- ❌ **错误状态码**：删除成功返回 200 并附带 `{message: "deleted"}`
  - ✅ 返回 204 No Content，无响应体

- ❌ **嵌套过深**：`/api/users/123/orders/456/items/789`
  - ✅ 最多嵌套 2 层：`/api/orders/456/items`

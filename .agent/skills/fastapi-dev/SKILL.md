---
name: FastAPI 后端开发规范
description: FastAPI (Python) 后端项目的组织规范、API设计、数据验证、AI Agent集成。使用SQLAlchemy ORM + Pydantic。当需要开发后端API、实现业务逻辑、集成LangGraph时使用。适用于全栈项目的后端开发阶段。
---

# FastAPI 后端开发规范

本规范适用于基于 **FastAPI** 的 Python 后端项目。

---

## 项目结构

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI 应用入口
│   ├── config.py            # 配置管理
│   ├── models/              # SQLAlchemy 数据库模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── dashboard.py
│   ├── schemas/             # Pydantic 请求/响应模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── dashboard.py
│   ├── api/                 # API 路由
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── dashboard.py
│   ├── services/            # 业务逻辑层
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── agent_service.py  # 如涉及 Agent
│   ├── db/                  # 数据库配置
│   │   ├── __init__.py
│   │   ├── session.py       # 数据库会话
│   │   └── base.py          # Base 类
│   ├── utils/               # 工具函数
│   │   ├── __init__.py
│   │   ├── auth.py          # JWT 生成/验证
│   │   └── helpers.py
│   └── agents/              # AI Agent 逻辑 (可选)
│       ├── __init__.py
│       └── main_agent.py
├── alembic/                 # 数据库迁移 (可选)
│   ├── versions/
│   └── env.py
├── tests/                   # 测试 (暂不需要)
├── .env                     # 环境变量
├── .env.example             # 环境变量示例
├── requirements.txt         # 依赖列表
├── alembic.ini              # Alembic 配置
└── README.md
```

---

## 核心文件说明

### 1. `main.py` - 应用入口

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api import auth, dashboard
from app.config import settings

app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS 配置（允许前端访问）
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # 前端地址
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 注册路由
app.include_router(auth.router, prefix="/api/auth", tags=["认证"])
app.include_router(dashboard.router, prefix="/api/dashboard", tags=["看板"])

@app.get("/")
def read_root():
    return {"message": "Welcome to the API"}
```

---

### 2. `config.py` - 配置管理

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    PROJECT_NAME: str = "My Project"
    VERSION: str = "1.0.0"
    
    # 数据库配置
    DATABASE_URL: str
    
    # JWT 配置
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 1440  # 24 小时
    
    # AI 配置 (如适用)
    OPENAI_API_KEY: str | None = None
    
    class Config:
        env_file = ".env"

settings = Settings()
```

`.env` 文件示例：

```env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
SECRET_KEY=your-secret-key-change-this
OPENAI_API_KEY=sk-...
```

---

### 3. 数据库模型 (`models/user.py`)

```python
from sqlalchemy import Column, String, DateTime
from sqlalchemy.dialects.postgresql import UUID
from app.db.base import Base
import uuid
from datetime import datetime

class User(Base):
    __tablename__ = "users"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(100), unique=True, nullable=False, index=True)
    hashed_password = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

---

### 4. Pydantic Schema (`schemas/user.py`)

```python
from pydantic import BaseModel, EmailStr
from datetime import datetime
import uuid

# 请求模型
class UserCreate(BaseModel):
    username: str
    email: EmailStr
    password: str

class UserLogin(BaseModel):
    email: EmailStr
    password: str

# 响应模型
class UserResponse(BaseModel):
    id: uuid.UUID
    username: str
    email: str
    created_at: datetime
    
    class Config:
        from_attributes = True  # 允许从 ORM 模型转换
```

---

### 5. API 路由 (`api/auth.py`)

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from app.schemas.user import UserCreate, UserLogin, UserResponse
from app.services.user_service import create_user, authenticate_user
from app.db.session import get_db
from app.utils.auth import create_access_token

router = APIRouter()

@router.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
def register(user: UserCreate, db: Session = Depends(get_db)):
    """用户注册"""
    db_user = create_user(db, user)
    if not db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return db_user

@router.post("/login")
def login(user: UserLogin, db: Session = Depends(get_db)):
    """用户登录"""
    db_user = authenticate_user(db, user.email, user.password)
    if not db_user:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    
    access_token = create_access_token(data={"sub": db_user.email})
    return {"access_token": access_token, "token_type": "bearer"}
```

---

### 6. 业务逻辑 (`services/user_service.py`)

```python
from sqlalchemy.orm import Session
from app.models.user import User
from app.schemas.user import UserCreate
from app.utils.auth import get_password_hash, verify_password

def create_user(db: Session, user: UserCreate) -> User | None:
    """创建用户"""
    # 检查邮箱是否已存在
    if db.query(User).filter(User.email == user.email).first():
        return None
    
    db_user = User(
        username=user.username,
        email=user.email,
        hashed_password=get_password_hash(user.password)
    )
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user

def authenticate_user(db: Session, email: str, password: str) -> User | None:
    """验证用户"""
    user = db.query(User).filter(User.email == email).first()
    if not user or not verify_password(password, user.hashed_password):
        return None
    return user
```

---

### 7. 数据库连接 (`db/session.py`)

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.config import settings

engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    """依赖注入：获取数据库会话"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

`db/base.py`:

```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

---

### 8. 工具函数 (`utils/auth.py`)

```python
from passlib.context import CryptContext
from jose import jwt
from datetime import datetime, timedelta
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def get_password_hash(password: str) -> str:
    """密码哈希"""
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码"""
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict) -> str:
    """生成 JWT"""
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
```

---

## AI Agent 集成 (可选)

如果项目涉及 AI Agent，参考 `prompt-engineering` skill。

### 示例：`agents/main_agent.py`

```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from app.config import settings

llm = ChatOpenAI(api_key=settings.OPENAI_API_KEY, model="gpt-4")

def build_agent():
    workflow = StateGraph()
    
    # 定义节点和边
    workflow.add_node("router", router_node)
    workflow.add_node("faq", faq_node)
    workflow.add_edge("router", "faq")
    workflow.add_edge("faq", END)
    
    return workflow.compile()

def router_node(state):
    # 路由逻辑
    return {"next": "faq"}

def faq_node(state):
    # FAQ 逻辑
    response = llm.invoke("Answer the question")
    return {"response": response}
```

---

## 依赖管理

`requirements.txt`:

```txt
fastapi==0.100.0
uvicorn[standard]==0.23.0
sqlalchemy==2.0.0
psycopg2-binary==2.9.6
pydantic==2.0.0
pydantic-settings==2.0.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-dotenv==1.0.0

# 如涉及 AI
langchain==0.1.0
langchain-openai==0.0.5
langgraph==0.0.20
```

---

## 运行与测试

### 启动服务器

```bash
uvicorn app.main:app --reload --port 8000
```

### 访问文档

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

---

## 最佳实践

1. **分层清晰**：路由 → 服务层 → 数据层，避免直接在路由写业务逻辑。
2. **类型安全**：使用 Pydantic 严格定义请求/响应模型。
3. **错误处理**：使用 `HTTPException` 明确返回错误。
4. **依赖注入**：使用 `Depends()` 管理数据库会话、认证等。
5. **环境变量**：敏感信息（如密钥）通过 `.env` 管理。

---

## 常见错误

- ❌ **在路由中写 SQL**：`db.execute("SELECT * FROM users")`
  - ✅ 使用 SQLAlchemy ORM

- ❌ **忽视数据验证**：直接使用 `request.json()`
  - ✅ 使用 Pydantic Schema

- ❌ **数据库会话泄漏**：未关闭 `db.close()`
  - ✅ 使用 `Depends(get_db)` 自动管理

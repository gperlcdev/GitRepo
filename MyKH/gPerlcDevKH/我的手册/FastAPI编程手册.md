# FastAPI 编程手册

> 面向软件工程师的 FastAPI 自学指南 — 从基础到上线
> 适用版本：FastAPI 0.109+ | Python 3.10+

---

## 目录

1. [FastAPI 是什么](#1-fastapi-是什么)
2. [环境搭建 & 第一个应用](#2-环境搭建--第一个应用)
3. [路由与请求处理](#3-路由与请求处理)
4. [请求数据 —— Pydantic 模型](#4-请求数据--pydantic-模型)
5. [响应模型与序列化](#5-响应模型与序列化)
6. [依赖注入](#6-依赖注入)
7. [错误处理](#7-错误处理)
8. [中间件与 CORS](#8-中间件与-cors)
9. [数据库集成（SQLAlchemy + Alembic）](#9-数据库集成sqlalchemy--alembic)
10. [异步与 Background Tasks](#10-异步与-background-tasks)
11. [文件上传与静态文件](#11-文件上传与静态文件)
12. [WebSocket](#12-websocket)
13. [安全与认证 (JWT / OAuth2)](#13-安全与认证-jwt--oauth2)
14. [测试](#14-测试)
15. [部署](#15-部署)
16. [常见模式与最佳实践](#16-常见模式与最佳实践)

---

## 1. FastAPI 是什么

**FastAPI** 是一个现代、高性能的 Python Web 框架，专为构建 API 而设计。

### 核心特性

| 特性 | 说明 |
|---|---|
| **高性能** | 基于 Starlette + Pydantic，性能接近 Node.js / Go |
| **自动生成文档** | 原生支持 OpenAPI / Swagger UI / ReDoc |
| **类型安全** | 基于 Python 类型提示，自动校验 + IDE 智能提示 |
| **异步原生** | 原生 `async/await` 支持，高并发 I/O 场景优秀 |
| **Pydantic 模型** | 声明式数据校验，JSON Schema 自动生成 |

### 与其他框架对比

| 框架 | 性能 | 类型提示 | 异步 | 自动文档 | 学习曲线 |
|---|---|---|---|---|---|
| FastAPI | ⭐⭐⭐⭐⭐ | ✅ 原生 | ✅ | ✅ | 低 |
| Flask | ⭐⭐⭐ | ❌ 手动 | ⚠️ 需扩展 | ❌ 需扩展 | 低 |
| Django | ⭐⭐⭐ | ✅ Django 5.0+ | ✅ | ❌ | 中 |
| Tornado | ⭐⭐⭐⭐ | ❌ | ✅ | ❌ | 中 |

---

## 2. 环境搭建 & 第一个应用

### 2.1 安装

```bash
# 核心安装
pip install fastapi

# ASGI 服务器（选一）
pip install uvicorn[standard]

# 推荐额外安装
pip install httpx          # 测试用 HTTP 客户端
pip install python-multipart  # 文件上传
pip install jinja2         # 模板渲染
pip install "pydantic[email]"  # 邮箱校验等
```

### 2.2 Hello World

```python
# main.py
from fastapi import FastAPI

app = FastAPI(title="我的第一个 API", version="0.1.0")

@app.get("/")
def read_root():
    return {"message": "Hello, FastAPI!"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

### 2.3 启动

```bash
# 开发模式（热重载）
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# 生产模式
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

### 2.4 自动文档

启动后浏览器访问：

- **Swagger UI**: `http://127.0.0.1:8000/docs`
- **ReDoc**: `http://127.0.0.1:8000/redoc`
- **OpenAPI JSON**: `http://127.0.0.1:8000/openapi.json`

---

## 3. 路由与请求处理

### 3.1 路径参数

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):          # 自动类型转换 + 校验
    return {"user_id": user_id}

@app.get("/files/{file_path:path}")  # path：匹配完整路径
def get_file(file_path: str):
    return {"file_path": file_path}
```

### 3.2 查询参数

```python
@app.get("/items/")
def list_items(
    skip: int = 0,                   # 有默认值 → 可选
    limit: int = 10,
    category: str | None = None,     # None → 可选
    published: bool = True,          # 自动类型转换
):
    return {"skip": skip, "limit": limit, "category": category}
```

### 3.3 请求体 —— POST / PUT / PATCH

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool | None = None

@app.post("/items/")
def create_item(item: Item):          # FastAPI 自动从 JSON 请求体解析
    return {"name": item.name, "price": item.price}
```

### 3.4 路径装饰器与 HTTP 方法

```python
@app.get("/resource")       # GET
@app.post("/resource")      # POST
@app.put("/resource/{id}")  # PUT（全量替换）
@app.patch("/resource/{id}")# PATCH（部分更新）
@app.delete("/resource/{id}")# DELETE
@app.options("/resource")   # OPTIONS
@app.head("/resource")      # HEAD
```

### 3.5 路径操作配置

```python
@app.post(
    "/items/",
    status_code=201,                 # 默认响应状态码
    tags=["items"],                  # OpenAPI 标签分组
    summary="创建商品",               # 文档摘要
    description="创建一个新的商品条目", # 文档描述
    deprecated=False,                # 标记为弃用
    response_description="商品创建成功",
)
def create_item(item: Item):
    return item
```

### 3.6 请求头 & Cookie

```python
from fastapi import Header, Cookie

@app.get("/headers/")
def read_headers(
    user_agent: str | None = Header(None),    # 获取请求头（自动转蛇形命名）
    x_token: list[str] | None = Header(None), # 多次出现的头
    session_id: str | None = Cookie(None),    # 获取 Cookie
):
    return {"User-Agent": user_agent, "X-Token": x_token}
```

### 3.7 子路由 —— APIRouter

```python
# app/routers/items.py
from fastapi import APIRouter

router = APIRouter(prefix="/items", tags=["items"])

@router.get("/")
def list_items():
    return [{"name": "Foo"}]

@router.get("/{item_id}")
def get_item(item_id: int):
    return {"item_id": item_id}
```

```python
# app/main.py
from fastapi import FastAPI
from app.routers import items, users

app = FastAPI()
app.include_router(items.router)
app.include_router(users.router, prefix="/api/v1")  # 可叠加前缀
```

### 3.8 路由顺序

```python
@app.get("/users/me")           # 必须放在 /users/{user_id} 之前！
def get_current_user():
    return {"user": "current"}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

---

## 4. 请求数据 —— Pydantic 模型

### 4.1 基础模型

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0, le=10000)
    description: str | None = Field(None, max_length=500)
    tax: float | None = None
    tags: list[str] = []
```

### 4.2 字段校验

```python
from pydantic import BaseModel, Field, EmailStr, HttpUrl
from typing import Annotated
from pydantic.functional_validators import AfterValidator

def validate_phone(v: str) -> str:
    if not v.startswith("+") or not v[1:].isdigit():
        raise ValueError("手机号格式错误")
    return v

class UserCreate(BaseModel):
    email: EmailStr                               # 邮箱格式
    homepage: HttpUrl | None = None               # URL 格式
    age: int = Field(18, ge=0, le=150)            # 范围
    phone: Annotated[str, AfterValidator(validate_phone)]  # 自定义校验
    score: float = Field(..., multiple_of=0.5)    # 0.5 的倍数
```

### 4.3 模型配置

```python
from pydantic import BaseModel, ConfigDict

class Item(BaseModel):
    model_config = ConfigDict(
        extra="forbid",                  # 拒绝未定义字段
        frozen=True,                     # 不可变
        str_strip_whitespace=True,       # 自动去空格
        validate_default=True,
        json_schema_extra={
            "example": {"name": "Foo", "price": 29.99}
        }
    )
    name: str
    price: float

# ORM 兼容（从数据库对象创建 Pydantic 模型）
class ItemOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    name: str
```

### 4.4 嵌套模型

```python
class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    images: list[Image] | None = None   # 嵌套
    metadata: dict[str, str] = {}       # 字典校验
```

### 4.5 字段验证器

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    name: str
    password1: str
    password2: str

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("用户名不能为空")
        return v.strip()

    @field_validator("password2")
    @classmethod
    def passwords_match(cls, v: str, info) -> str:
        if "password1" in info.data and v != info.data["password1"]:
            raise ValueError("两次密码不一致")
        return v

# 模型级别验证
from pydantic import model_validator

class User(BaseModel):
    name: str
    password1: str
    password2: str

    @model_validator(mode="after")
    def check_passwords_match(self):
        if self.password1 != self.password2:
            raise ValueError("两次密码不一致")
        return self
```

### 4.6 路径和查询参数校验

```python
from fastapi import FastAPI, Query, Path, Body
from typing import Annotated

app = FastAPI()

@app.get("/items/")
def read_items(
    q: Annotated[
        str | None,
        Query(
            min_length=3,
            max_length=50,
            pattern=r"^[a-zA-Z]+$",
            title="搜索关键词",
            description="按名称搜索商品",
        )
    ] = None,
    page: int = Query(1, ge=1, description="页码"),
    size: int = Query(10, ge=1, le=100),
):
    return {"q": q, "page": page, "size": size}

@app.get("/items/{item_id}")
def get_item(
    item_id: Annotated[int, Path(ge=1, title="商品 ID")],
):
    return {"item_id": item_id}
```

---

## 5. 响应模型与序列化

### 5.1 指定响应模型

```python
@app.post("/items/", response_model=ItemOut)
def create_item(item: ItemIn) -> ItemOut:
    db_item = create_in_db(item)
    return db_item  # ORM 对象自动序列化
```

### 5.2 响应模型参数

```python
@app.post(
    "/items/",
    response_model=ItemOut,
    response_model_exclude_unset=True,  # 只返回有值的字段
    response_model_exclude_none=True,   # 排除 None 字段
    response_model_include={"id", "name"},  # 只包含
    response_model_exclude={"tax"},         # 排除
)
```

### 5.3 多个响应模型

```python
from typing import Union

@app.get("/items/{item_id}", response_model=Union[ItemOut, Message])
def get_item(item_id: int):
    item = get_from_db(item_id)
    if item is None:
        return Message(message="Item not found")
    return item
```

### 5.4 响应状态码

```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: Item):
    ...

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int):
    ...
```

### 5.5 定制响应

```python
from fastapi import Response
from fastapi.responses import (
    JSONResponse, HTMLResponse, PlainTextResponse,
    RedirectResponse, StreamingResponse, FileResponse,
)

@app.get("/html")
def get_html():
    return HTMLResponse("<h1>Hello</h1>")

@app.get("/redirect")
def redirect():
    return RedirectResponse(url="/new-path")

@app.get("/stream")
async def stream():
    async def generate():
        for i in range(100):
            yield f"data: {i}\n\n"
            await asyncio.sleep(0.1)
    return StreamingResponse(generate(), media_type="text/event-stream")

@app.get("/download")
def download():
    return FileResponse("path/to/file.pdf", filename="report.pdf")
```

### 5.6 ORM 模式（from_attributes）

```python
from pydantic import BaseModel, ConfigDict

class ItemOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    name: str
    price: float

# 从 ORM 对象创建
db_item = db.query(ItemORM).first()
return ItemOut.model_validate(db_item)
```

---

## 6. 依赖注入

### 6.1 基本依赖

```python
from fastapi import Depends, FastAPI

app = FastAPI()

def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return commons

@app.get("/users/")
def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

### 6.2 类作为依赖

```python
class CommonQueryParams(BaseModel):
    q: str | None = None
    skip: int = 0
    limit: int = 100

@app.get("/items/")
def read_items(commons: CommonQueryParams = Depends()):
    # Depends() 不传函数会使用类型注解的类
    return commons
```

### 6.3 依赖链

```python
def query_extractor(q: str | None = None):
    return q

def query_or_cookie_extractor(
    q: str = Depends(query_extractor),
    last_query: str | None = Cookie(None),
):
    if not q:
        return last_query
    return q

@app.get("/items/")
def read_items(query_or_default: str = Depends(query_or_cookie_extractor)):
    return {"q": query_or_default}
```

### 6.4 带 yield 的依赖（资源管理）

用于数据库连接等需要清理的资源：

```python
async def get_db():
    db = DatabaseSession()
    try:
        yield db
    finally:
        db.close()  # 请求结束后自动清理

@app.get("/items/{item_id}")
def get_item(item_id: int, db: DatabaseSession = Depends(get_db)):
    return db.query(item_id)
```

### 6.5 全局依赖

```python
app = FastAPI(dependencies=[Depends(verify_token)])

# 或对特定路由组
router = APIRouter(dependencies=[Depends(verify_token)])
```

### 6.6 依赖缓存

同一请求中多个 `Depends` 引用同一依赖只调用一次（共享同一个对象）：

```python
@app.get("/items/")
def read_items(
    dep1: dict = Depends(common_parameters),
    dep2: dict = Depends(common_parameters),
):
    assert dep1 is dep2  # True
```

### 6.7 实战：数据库会话依赖

```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

engine = create_engine("sqlite:///./test.db")
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# app/dependencies.py
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# app/main.py
@app.get("/items/{item_id}")
def read_item(item_id: int, db: Session = Depends(get_db)):
    return db.query(Item).filter(Item.id == item_id).first()
```

---

## 7. 错误处理

### 7.1 HTTPException

```python
from fastapi import HTTPException, status

@app.get("/items/{item_id}")
def get_item(item_id: int):
    item = find_item(item_id)
    if item is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item {item_id} not found",
            headers={"X-Error": "Not Found"},
        )
    return item
```

### 7.2 自定义异常处理

```python
class ItemNotFoundException(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

@app.exception_handler(ItemNotFoundException)
async def item_not_found_handler(request: Request, exc: ItemNotFoundException):
    return JSONResponse(
        status_code=404,
        content={"message": f"Item {exc.item_id} not found"},
    )

@app.get("/items/{item_id}")
def get_item(item_id: int):
    raise ItemNotFoundException(item_id)
```

### 7.3 全局异常覆盖

```python
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail, "error_code": "HTTP_ERROR"},
    )

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(
        status_code=422,
        content={
            "detail": exc.errors(),
            "body": exc.body,
            "message": "数据校验失败",
        },
    )
```

### 7.4 OpenAPI 错误标注

```python
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    error_code: str
    message: str
    details: dict | None = None

@app.get("/items/{item_id}", responses={
    404: {"model": ErrorResponse, "description": "Item not found"},
})
def get_item(item_id: int):
    ...
```

---

## 8. 中间件与 CORS

### 8.1 自定义中间件

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### 8.2 CORS（跨域）

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "https://myapp.com",
    ],
    allow_origin_regex=r"https://.*\.example\.com",
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    expose_headers=["X-Process-Time"],
    max_age=600,
)
```

> 开发阶段可用 `allow_origins=["*"]`，生产环境请明确指定。

### 8.3 其他常用中间件

```python
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.middleware.gzip import GZipMiddleware

app.add_middleware(TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"])
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

---

## 9. 数据库集成（SQLAlchemy + Alembic）

### 9.1 项目结构

```
project/
├── app/
│   ├── main.py
│   ├── database.py          # 数据库引擎和会话
│   ├── models.py            # SQLAlchemy 模型
│   ├── schemas.py           # Pydantic 模型
│   ├── crud.py              # 数据库操作
│   └── routers/
│       ├── __init__.py
│       └── items.py
├── alembic/
├── alembic.ini
└── requirements.txt
```

### 9.2 数据库配置

```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker, Session

# SQLite（开发）
SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
# PostgreSQL（生产）
# SQLALCHEMY_DATABASE_URL = "postgresql://user:password@host:5432/dbname"
# MySQL
# SQLALCHEMY_DATABASE_URL = "mysql+pymysql://user:password@host:3306/dbname"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False} if "sqlite" in SQLALCHEMY_DATABASE_URL else {},
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

class Base(DeclarativeBase):
    pass

def get_db() -> Session:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 9.3 SQLAlchemy 模型

```python
# app/models.py
from sqlalchemy import Column, Integer, String, Float, Boolean, ForeignKey, DateTime, Text
from sqlalchemy.orm import relationship
from datetime import datetime, timezone
from app.database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(100), nullable=False)
    description = Column(Text, nullable=True)
    price = Column(Float, nullable=False)
    is_active = Column(Boolean, default=True)
    owner_id = Column(Integer, ForeignKey("users.id"))
    created_at = Column(DateTime, default=lambda: datetime.now(timezone.utc))
    updated_at = Column(
        DateTime,
        default=lambda: datetime.now(timezone.utc),
        onupdate=lambda: datetime.now(timezone.utc),
    )

    owner = relationship("User", back_populates="items")

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(100), unique=True, nullable=False, index=True)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)

    items = relationship("Item", back_populates="owner")
```

### 9.4 Pydantic Schema

```python
# app/schemas.py
from pydantic import BaseModel, ConfigDict, EmailStr
from datetime import datetime

class ItemBase(BaseModel):
    title: str
    description: str | None = None
    price: float

class ItemCreate(ItemBase):
    pass

class ItemUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    price: float | None = None

class ItemResponse(ItemBase):
    model_config = ConfigDict(from_attributes=True)
    id: int
    owner_id: int
```

### 9.5 CRUD 操作

```python
# app/crud.py
from sqlalchemy.orm import Session
from app import models, schemas

def create_item(db: Session, item: schemas.ItemCreate, user_id: int) -> models.Item:
    db_item = models.Item(**item.model_dump(), owner_id=user_id)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

def get_item(db: Session, item_id: int) -> models.Item | None:
    return db.query(models.Item).filter(models.Item.id == item_id).first()

def get_items(db: Session, skip: int = 0, limit: int = 100) -> list[models.Item]:
    return db.query(models.Item).offset(skip).limit(limit).all()

def update_item(db: Session, item_id: int, item: schemas.ItemUpdate) -> models.Item | None:
    db_item = get_item(db, item_id)
    if db_item is None:
        return None
    update_data = item.model_dump(exclude_unset=True)
    for key, value in update_data.items():
        setattr(db_item, key, value)
    db.commit()
    db.refresh(db_item)
    return db_item

def delete_item(db: Session, item_id: int) -> bool:
    db_item = get_item(db, item_id)
    if db_item is None:
        return False
    db.delete(db_item)
    db.commit()
    return True
```

### 9.6 路由整合

```python
# app/routers/items.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import List
from app import crud, schemas
from app.database import get_db

router = APIRouter(prefix="/items", tags=["items"])

@router.get("/", response_model=List[schemas.ItemResponse])
def read_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    return crud.get_items(db, skip=skip, limit=limit)

@router.get("/{item_id}", response_model=schemas.ItemResponse)
def read_item(item_id: int, db: Session = Depends(get_db)):
    item = crud.get_item(db, item_id=item_id)
    if item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

@router.post("/", response_model=schemas.ItemResponse, status_code=status.HTTP_201_CREATED)
def create_item(item: schemas.ItemCreate, db: Session = Depends(get_db)):
    return crud.create_item(db=db, item=item, user_id=1)

@router.put("/{item_id}", response_model=schemas.ItemResponse)
def update_item(item_id: int, item: schemas.ItemUpdate, db: Session = Depends(get_db)):
    updated = crud.update_item(db=db, item_id=item_id, item=item)
    if updated is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return updated

@router.delete("/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int, db: Session = Depends(get_db)):
    if not crud.delete_item(db=db, item_id=item_id):
        raise HTTPException(status_code=404, detail="Item not found")
    return None
```

### 9.7 Alembic 迁移

```bash
pip install alembic

alembic init alembic
# 编辑 alembic.ini 中的 sqlalchemy.url
# 编辑 alembic/env.py: 设置 target_metadata = Base.metadata

# 生成迁移
alembic revision --autogenerate -m "create items table"

# 查看 SQL
alembic upgrade --sql

# 执行迁移
alembic upgrade head

# 回滚一步
alembic downgrade -1
```

### 9.8 Async SQLAlchemy

```python
# app/database.py (async version)
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase

ASYNC_DATABASE_URL = "sqlite+aiosqlite:///./async_app.db"
# ASYNC_DATABASE_URL = "postgresql+asyncpg://user:password@host:5432/dbname"

engine = create_async_engine(ASYNC_DATABASE_URL, echo=True)
AsyncSessionLocal = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

---

## 10. 异步与 Background Tasks

### 10.1 async/await 基础

```python
import asyncio

@app.get("/async")
async def read_async():
    await asyncio.sleep(1)
    return {"message": "Async!"}

@app.get("/sync")
def read_sync():
    import time
    time.sleep(1)  # 阻塞！
    return {"message": "Sync!"}
```

> 在异步函数中不要调用阻塞 I/O（如 `time.sleep()`、`requests`），
> 必须用 `asyncio.sleep()`、`httpx.AsyncClient` 或 `run_in_executor`。

### 10.2 避免异步陷阱

```python
import time
from concurrent.futures import ThreadPoolExecutor
import asyncio

# ❌ 错误：在 async 函数中调用阻塞操作
@app.get("/bad")
async def bad():
    time.sleep(5)  # 阻塞事件循环！所有请求都会卡住
    return {"message": "slow"}

# ✅ 正确：使用线程池
@app.get("/good")
async def good():
    loop = asyncio.get_event_loop()
    await loop.run_in_executor(None, time.sleep, 5)
    return {"message": "slow but non-blocking"}

# ✅ 使用 httpx.AsyncClient
import httpx
@app.get("/fetch")
async def fetch_data():
    async with httpx.AsyncClient() as client:
        resp = await client.get("https://api.example.com/data")
    return resp.json()
```

### 10.3 BackgroundTasks

适合发送邮件、处理图片、日志等不阻塞响应的操作：

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def write_log(message: str):
    with open("log.txt", "a") as f:
        f.write(f"{message}\n")

def send_email(email: str, subject: str):
    # 模拟发邮件
    pass

@app.post("/send-notification/{email}")
async def send_notification(
    email: str,
    background_tasks: BackgroundTasks,
):
    background_tasks.add_task(write_log, f"notification sent to {email}")
    background_tasks.add_task(send_email, email, "Hello!")
    return {"message": "Notification sent in background"}
```

> `BackgroundTasks` 不支持 async 函数。如需异步后台任务，用下文的 asyncio 方案。

### 10.4 使用 Asyncio 任务

```python
import asyncio

app = FastAPI()
background_tasks = set()

async def long_running_task(task_id: str):
    await asyncio.sleep(10)
    background_tasks.discard(task_id)

@app.post("/start-task/")
async def start_task():
    task_id = f"task-{id({})}"
    task = asyncio.create_task(long_running_task(task_id))
    background_tasks.add(task)
    task.add_done_callback(background_tasks.discard)
    return {"task_id": task_id, "status": "started"}
```

---

## 11. 文件上传与静态文件

### 11.1 文件上传

```python
from fastapi import FastAPI, File, UploadFile
import shutil

app = FastAPI()

@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    with open(f"uploads/{file.filename}", "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    return {"filename": file.filename, "content_type": file.content_type}

@app.post("/upload-multiple/")
async def upload_multiple(files: list[UploadFile] = File(...)):
    for file in files:
        content = await file.read()
        with open(f"uploads/{file.filename}", "wb") as f:
            f.write(content)
    return {"filenames": [f.filename for f in files]}
```

### 11.2 表单与文件混合

```python
from fastapi import Form

@app.post("/create-item/")
async def create_item(
    name: str = Form(...),
    price: float = Form(...),
    image: UploadFile = File(...),
):
    content = await image.read()
    return {"name": name, "price": price, "image_size": len(content)}
```

### 11.3 静态文件服务

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
# 访问: http://localhost:8000/static/style.css
```

### 11.4 模板渲染（Jinja2）

```python
from fastapi.templating import Jinja2Templates
from fastapi import Request

templates = Jinja2Templates(directory="templates")

@app.get("/")
async def home(request: Request):
    return templates.TemplateResponse(
        "index.html",
        {"request": request, "title": "FastAPI Demo"}
    )
```

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head><title>{{ title }}</title></head>
<body>
    <h1>Welcome to {{ title }}</h1>
</body>
</html>
```

---

## 12. WebSocket

### 12.1 基本 WebSocket

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Echo: {data}")
    except WebSocketDisconnect:
        print("Client disconnected")
```

### 12.2 WebSocket 连接管理

```python
class ConnectionManager:
    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f"Client #{client_id}: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left")
```

---

## 13. 安全与认证 (JWT / OAuth2)

### 13.1 密码哈希

```bash
pip install passlib[bcrypt] python-jose[cryptography]
```

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

### 13.2 JWT Token

```python
from datetime import datetime, timedelta, timezone
from jose import JWTError, jwt

SECRET_KEY = "your-secret-key-keep-it-safe"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

def create_access_token(
    data: dict,
    expires_delta: timedelta | None = None,
) -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + (
        expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict | None:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None
```

### 13.3 OAuth2 密码流

```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token = create_access_token(data={"sub": user.email})
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(token: str = Depends(oauth2_scheme)):
    payload = decode_token(token)
    if payload is None:
        raise HTTPException(status_code=401, detail="Invalid token")
    user = get_user_by_email(payload.get("sub"))
    if user is None:
        raise HTTPException(status_code=401, detail="User not found")
    return user
```

### 13.4 完整的认证依赖

```python
# app/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from sqlalchemy.orm import Session
from app import crud, models
from app.database import get_db
from app.auth import SECRET_KEY, ALGORITHM

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/login")

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db),
) -> models.User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        email: str = payload.get("sub")
        if email is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = crud.get_user_by_email(db, email=email)
    if user is None:
        raise credentials_exception
    return user

@app.get("/me")
def read_current_user(current_user: models.User = Depends(get_current_user)):
    return current_user
```

### 13.5 权限角色

```python
from enum import Enum
from fastapi import Depends, HTTPException, status

class UserRole(str, Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"

def require_role(required_role: UserRole):
    def role_checker(current_user: models.User = Depends(get_current_user)):
        if current_user.role != required_role.value:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Insufficient permissions",
            )
        return current_user
    return role_checker

@app.get("/admin")
def admin_only(user: models.User = Depends(require_role(UserRole.ADMIN))):
    return {"message": "Welcome admin!"}
```

---

## 14. 测试

### 14.1 使用 TestClient

```python
# pip install httpx
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello, FastAPI!"}

def test_read_item():
    response = client.get("/items/42?q=test")
    assert response.status_code == 200
    data = response.json()
    assert data["item_id"] == 42
    assert data["q"] == "test"

def test_create_item():
    response = client.post(
        "/items/",
        json={"name": "Foo", "price": 29.99},
    )
    assert response.status_code == 201
    assert response.json()["name"] == "Foo"

def test_unauthorized():
    response = client.get("/users/me")
    assert response.status_code == 401
```

### 14.2 覆盖依赖

```python
from app import dependencies
from app.main import app

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[dependencies.get_db] = override_get_db

def test_with_db_override():
    response = client.get("/items/1")
    ...
```

### 14.3 Async 测试

```python
import pytest
from httpx import AsyncClient, ASGITransport

@pytest.mark.anyio
async def test_async_endpoint():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        response = await ac.get("/async-endpoint")
    assert response.status_code == 200
```

### 14.4 pytest 配置

```python
# conftest.py
import pytest
from fastapi.testclient import TestClient
from app.main import app
from app.database import Base, engine, SessionLocal
from app.dependencies import get_db

@pytest.fixture
def db_session():
    Base.metadata.create_all(bind=engine)
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
    Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client(db_session):
    def override_get_db():
        yield db_session
    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    del app.dependency_overrides[get_db]

def test_create_item(client):
    response = client.post("/items/", json={"title": "Test", "price": 9.99})
    assert response.status_code == 201
```

---

## 15. 部署

### 15.1 Docker 部署

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./app /app/app

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/app
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 15.2 环境变量管理

```python
# app/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    app_name: str = "FastAPI App"
    debug: bool = False
    database_url: str = "sqlite:///./app.db"
    secret_key: str = "change-me"
    access_token_expire_minutes: int = 30
    allowed_hosts: list[str] = ["*"]

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
    )

settings = Settings()
```

```bash
# .env
APP_NAME=MyApp
DEBUG=true
DATABASE_URL=postgresql://user:pass@localhost:5432/app
SECRET_KEY=your-secret-key-here
```

### 15.3 Gunicorn + Uvicorn

```bash
pip install gunicorn

gunicorn app.main:app \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000 \
    --workers 4 \
    --timeout 120 \
    --keep-alive 5 \
    --max-requests 1000 \
    --max-requests-jitter 50
```

### 15.4 Nginx 反向代理

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 16. 常见模式与最佳实践

### 16.1 推荐项目结构

```
project/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI 应用入口
│   ├── config.py            # 配置管理
│   ├── database.py          # 数据库引擎/会话
│   ├── models.py            # SQLAlchemy 模型
│   ├── schemas.py           # Pydantic 模型
│   ├── crud.py              # 数据库操作
│   ├── auth.py              # 认证逻辑
│   ├── dependencies.py      # 共享依赖
│   └── routers/
│       ├── __init__.py
│       ├── items.py
│       ├── users.py
│       └── auth.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_items.py
├── static/
├── templates/
├── uploads/
├── alembic/
├── alembic.ini
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── pyproject.toml
└── .env
```

### 16.2 生命周期事件

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动时执行
    print("Starting up...")
    app.state.db = await create_connection_pool()
    yield
    # 关闭时执行
    print("Shutting down...")
    await app.state.db.close()

app = FastAPI(lifespan=lifespan)
```

### 16.3 分页模式

```python
class PaginationParams(BaseModel):
    page: int = 1
    size: int = 20

class PaginatedResponse(BaseModel):
    items: list
    total: int
    page: int
    size: int
    pages: int

@app.get("/items/", response_model=PaginatedResponse)
def list_items(
    pagination: PaginationParams = Depends(),
    db: Session = Depends(get_db),
):
    query = db.query(models.Item)
    total = query.count()
    items = query.offset(
        (pagination.page - 1) * pagination.size
    ).limit(pagination.size).all()
    return PaginatedResponse(
        items=items,
        total=total,
        page=pagination.page,
        size=pagination.size,
        pages=(total + pagination.size - 1) // pagination.size,
    )
```

### 16.4 健康检查

```python
@app.get("/health", tags=["system"])
def health_check(db: Session = Depends(get_db)):
    try:
        db.execute(text("SELECT 1"))
        db_status = "healthy"
    except Exception:
        db_status = "unhealthy"

    return {
        "status": "ok",
        "version": app.version,
        "database": db_status,
    }
```

### 16.5 日志配置

```python
import logging
from fastapi import FastAPI
from logging.handlers import RotatingFileHandler

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

handler = RotatingFileHandler("app.log", maxBytes=10_000_000, backupCount=5)
handler.setFormatter(logging.Formatter(
    "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
))
logger.addHandler(handler)

@app.get("/")
def root():
    logger.info("Root endpoint called")
    return {"message": "Hello"}
```

### 16.6 OpenAPI 高级配置

```python
from fastapi.openapi.utils import get_openapi

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    openapi_schema = get_openapi(
        title="My API",
        version="1.0.0",
        description="生产级 Web API",
        routes=app.routes,
    )
    openapi_schema["servers"] = [
        {"url": "https://api.example.com", "description": "生产环境"},
        {"url": "http://localhost:8000", "description": "本地开发"},
    ]
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

### 16.7 CRUD 开发清单

```
□ 1. 环境搭建（uvicorn + fastapi）
□ 2. 定义 Pydantic schema（Create / Update / Response）
□ 3. 配置数据库（SQLAlchemy + Alembic）
□ 4. 实现 CRUD 函数（create / read / update / delete）
□ 5. 创建 Router（GET / POST / PUT / DELETE）
□ 6. 添加依赖注入（get_db）
□ 7. 错误处理（HTTPException + 自定义异常）
□ 8. 认证（JWT + OAuth2）
□ 9. 测试（TestClient + pytest）
□ 10. 部署（Docker + 反向代理）
```

---

## 附录

### A. 速查命令

```bash
# 开发启动
uvicorn app.main:app --reload --port 8000

# 生产启动
gunicorn app.main:app -k uvicorn.workers.UvicornWorker -w 4 -b 0.0.0.0:8000

# 数据库迁移
alembic revision --autogenerate -m "description"
alembic upgrade head
alembic downgrade -1

# 运行测试
pytest -v --cov=app tests/

# 依赖导出
pip freeze > requirements.txt
```

### B. 常用包速查

| 目的 | 推荐包 | 说明 |
|---|---|---|
| Web 框架 | `fastapi` | 核心 |
| ASGI 服务器 | `uvicorn[standard]` | 开发/生产 |
| 生产服务器 | `gunicorn` | 配合 uvicorn worker |
| 数据校验 | `pydantic[email]` | 已随 FastAPI 安装 |
| ORM | `sqlalchemy>=2.0` | 数据库操作 |
| 异步 ORM | `sqlalchemy[asyncio]` + `asyncpg` | 异步数据库 |
| 迁移 | `alembic` | 数据库版本管理 |
| JWT | `python-jose[cryptography]` | 认证 |
| 密码 | `passlib[bcrypt]` | 密码哈希 |
| HTTP | `httpx` | 测试 + 异步请求 |
| 测试 | `pytest` + `httpx` | |
| 配置 | `pydantic-settings` | 环境变量管理 |
| 限流 | `slowapi` | 速率限制 |
| ORM 备选 | `sqlmodel` | Pydantic + SQLAlchemy 融合 |

### C. 7 天学习路线

```
第1天  →  环境搭建 + Hello World + 路由基础
第2天  →  Pydantic 模型 + 请求/响应校验
第3天  →  依赖注入 + 错误处理 + 中间件
第4天  →  数据库 + CRUD 完整实现
第5天  →  认证 + WebSocket
第6天  →  测试 + 部署
第7天  →  独立完成一个小项目
```

### D. 推荐练习项目

1. **TODO List API** —— 入门级 CRUD，无需认证
2. **博客系统 API** —— 用户认证、文章 CRUD、标签分类
3. **电商后台 API** —— 商品管理、订单、用户权限
4. **即时聊天后端** —— WebSocket、消息队列、多房间
5. **文件分享服务** —— 大文件上传、缩略图、分享链接

---

> **手册版本**: v1.0 | **最后更新**: 2026-05-04
> 建议配合官方文档使用: https://fastapi.tiangolo.com/
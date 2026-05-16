# Python 3.12 + FastAPI - Skill Completo para Proyectos Nuevos

> Guía completa para desarrollo backend moderno con Python 3.12 y FastAPI.
> Transversal, sin dependencias de proyectos específicos.

> **⚠️ Importante:** Usar **Poetry** en lugar de pip/requirements.txt. Poetry maneja dependencias de forma segura y reproducible.

---

## Tabla de Contenidos

1. [Setup y Configuración](#1-setup-y-configuración)
2. [Estructura del Proyecto](#2-estructura-del-proyecto)
3. [Modelos de Datos](#3-modelos-de-datos)
4. [DTOs con Pydantic](#4-dtos-con-pydantic)
5. [Repository Pattern](#5-repository-pattern)
6. [Service Layer](#6-service-layer)
7. [Routes FastAPI](#7-routes-fastapi)
8. [Autenticación JWT](#8-autenticación-jwt)
9. [WebSockets Reactivos](#9-websockets-reactivos)
10. [Paginación](#10-paginación)
11. [Manejo de Errores](#11-manejo-de-errores)
12. [Testing](#12-testing)
13. [Deployment](#13-deployment)
14. [Checklist de Código](#14-checklist-de-código)

---

## 1. Setup y Configuración

### 1.1 Entorno con Poetry

```bash
# Instalar Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Crear proyecto
poetry new backend-api
cd backend-api

# Agregar dependencias
poetry add fastapi uvicorn sqlalchemy pydantic pydantic-settings python-jose
poetry add python-multipart aiofiles

# Dev dependencies
poetry add --group dev pytest pytest-asyncio httpx black ruff mypy

# Instalar
poetry install

# Comandos
poetry run fastapi dev      # Desarrollo
poetry run pytest          # Tests
poetry build              # Build
```

### 1.2 Dependencias Modernas

```toml
# pyproject.toml
[tool.poetry]
name = "backend-api"
version = "0.1.0"
description = "Modern FastAPI backend"
python = "^3.12"

[tool.poetry.dependencies]
python = "^3.12"
fastapi = "^0.115"
uvicorn = {extras = ["standard"], version = "^0.32"}
sqlalchemy = "^2.0"
pydantic = "^2.10"
pydantic-settings = "^2.6"
python-jose = {extras = ["cryptography"], version = "^3.3"}
python-multipart = "^0.0"
alembic = "^1.14"
psycopg2-binary = "^2.9"
redis = "^5.0"

[tool.poetry.group.dev.dependencies]
pytest = "^8.3"
pytest-asyncio = "^0.24"
httpx = "^0.28"
```

---

## 2. Estructura del Proyecto

```
backend-api/
├── src/
│   ├── api/                    # Rutas por dominio
│   │   ├── books/
│   │   │   ├── __init__.py
│   │   │   ├── models.py       # SQLAlchemy
│   │   │   ├── schemas.py      # Pydantic DTOs
│   │   │   ├── repository.py   # Acceso datos
│   │   │   ├── service.py      # Lógica negocio
│   │   │   └── routes.py       # Endpoints
│   │   ├── users/
│   │   └── __init__.py
│   ├── core/                   # Config central
│   │   ├── config.py          # Settings
│   │   ├── database.py        # SQLAlchemy setup
│   │   ├── security.py       # JWT, password
│   │   └── exceptions.py     # Custom exceptions
│   ├── shared/                 # Componentes compartidos
│   │   ├── schemas/          # DTOs comunes
│   │   ├── utils/            # Helpers
│   │   └── deps.py           # Dependencies FastAPI
│   └── main.py                # App entry
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── api/
├── alembic/                   # Migraciones
├── pyproject.toml
└── README.md
```

### 2.1 Main App (CSR/SSR Ready)

```python
# src/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from src.api import books, users
from src.core.config import settings
from src.core.database import engine
from src.shared.schemas import api_response

app = FastAPI(
    title=settings.PROJECT_NAME,
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Routes
app.include_router(books.router, prefix="/api/books", tags=["books"])
app.include_router(users.router, prefix="/api/users", tags=["users"])


@app.get("/health")
def health_check():
    return {"status": "healthy", "version": "1.0.0"}
```

---

## 3. Modelos de Datos

### 3.1 Base y Timestamps

```python
# src/core/database.py
from datetime import datetime, timezone
from sqlalchemy import DateTime
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy.sql import func


class Base(DeclarativeBase):
    pass


class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now()
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now()
    )
```

### 3.2 Modelos SQLAlchemy 2.0

```python
# src/api/books/models.py
from sqlalchemy import String, Integer, ForeignKey, Index
from sqlalchemy.orm import Mapped, mapped_column, relationship

from src.core.database import Base, TimestampMixin


class Book(Base, TimestampMixin):
    __tablename__ = "books"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    summary: Mapped[str | None] = mapped_column(String(1000))

    # Foreign Keys
    genre_id: Mapped[int | None] = mapped_column(ForeignKey("genres.id"))
    publisher_id: Mapped[int | None] = mapped_column(ForeignKey("publishers.id"))

    # Relationships
    genre: Mapped["Genre"] = relationship(back_populates="books", lazy="selectin")
    authors: Mapped[list["BookAuthor"]] = relationship(
        back_populates="book", lazy="selectin"
    )
    editions: Mapped[list["Edition"]] = relationship(
        back_populates="book", lazy="selectin"
    )

    __table_args__ = (
        Index("ix_book_title", "title"),
        Index("ix_book_genre", "genre_id"),
    )


class Genre(Base):
    __tablename__ = "genres"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50), unique=True)

    books: Mapped[list["Book"]] = relationship(back_populates="genre")


class BookAuthor(Base):
    __tablename__ = "book_authors"

    book_id: Mapped[int] = mapped_column(ForeignKey("books.id"), primary_key=True)
    author_id: Mapped[int] = mapped_column(ForeignKey("authors.id"), primary_key=True)

    book: Mapped["Book"] = relationship(back_populates="authors")
    author: Mapped["Author"] = relationship(back_populates="books")


class Author(Base):
    __tablename__ = "authors"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))

    books: Mapped[list["BookAuthor"]] = relationship(back_populates="author")
```

### 3.3 Modelos con Properties

```python
class User(Base, TimestampMixin):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    full_name: Mapped[str | None] = mapped_column(String(100))
    is_active: Mapped[bool] = mapped_column(default=True)
    role: Mapped[str] = mapped_column(String(20), default="user")

    @property
    def display_name(self) -> str:
        return self.full_name or self.email.split("@")[0]
```

---

## 4. DTOs con Pydantic

### 4.1 Schemas Base

```python
# src/shared/schemas/base.py
from datetime import datetime
from uuid import UUID
from pydantic import BaseModel, ConfigDict, Field


class BaseSchema(BaseModel):
    model_config = ConfigDict(from_attributes=True)


class TimestampSchema(BaseSchema):
    created_at: datetime = Field(default_factory=lambda: datetime.now())
    updated_at: datetime = Field(default_factory=lambda: datetime.now())
```

### 4.2 DTOs de Entidad

```python
# src/api/books/schemas.py
from datetime import date
from uuid import UUID
from pydantic import BaseModel, ConfigDict, Field, EmailStr
from typing import Optional


# ============ USER DTOs ============
class CreateUserDTO(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8)
    full_name: str | None = None
    role: str = "user"

    model_config = ConfigDict(from_attributes=True)


class UpdateUserDTO(BaseModel):
    email: EmailStr | None = None
    full_name: str | None = None
    is_active: bool | None = None

    model_config = ConfigDict(from_attributes=True)


class UserDTO(BaseSchema):
    id: int
    email: EmailStr
    full_name: str | None
    is_active: bool
    role: str
    created_at: datetime


class UserDetailDTO(UserDTO):
    pass


# ============ BOOK DTOs ============
class CreateBookDTO(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    summary: str | None = None
    genre_id: int | None = None
    author_ids: list[int] = []

    model_config = ConfigDict(from_attributes=True)


class UpdateBookDTO(BaseModel):
    title: str | None = None
    summary: str | None = None
    genre_id: int | None = None

    model_config = ConfigDict(from_attributes=True)


class BookDTO(BaseSchema):
    id: int
    title: str
    summary: str | None
    genre_id: int | None


class BookDetailDTO(BookDTO):
    genre_name: str | None = None
    author_names: list[str] = []
    editions_count: int = 0


class BookFilterDTO(BaseModel):
    genre_id: int | None = None
    author_id: int | None = None
    available_only: bool = False
```

### 4.3 DTOs de Respuesta API

```python
# src/shared/schemas/responses.py
from typing import Generic, TypeVar, Optional
from pydantic import BaseModel

T = TypeVar("T")


class ApiResponse(BaseModel, Generic[T]):
    is_success: bool = True
    data: Optional[T] = None
    message: str | None = None
    error_code: str | None = None

    @classmethod
    def success(cls, data: T, message: str | None = None):
        return cls(is_success=True, data=data, message=message)

    @classmethod
    def error(cls, message: str, error_code: str | None = None):
        return cls(is_success=False, data=None, message=message, error_code=error_code)

    @classmethod
    def created(cls, data: T, message: str = "Recurso creado"):
        return cls(is_success=True, data=data, message=message)


class PaginationResponseDTO(BaseModel, Generic[T]):
    page: int
    pages: int
    items: int
    limit: int
    next: str | None = None
    prev: str | None = None
    data: list[T]
```

### 4.4 Conversión ORM → DTO

```python
# ✅ CORRECTO - model_validate
book_dto = BookDTO.model_validate(book)
detail_dto = BookDetailDTO.model_validate(book, from_attributes=True)

# ❌ INCORRECTO - constructor directo
book_dto = BookDTO(**book)  # Puede fallar con campos extras
```

---

## 5. Repository Pattern

### 5.1 Repository Base

```python
# src/shared/repository.py
from typing import Generic, TypeVar, Type
from sqlalchemy import select, func, and_
from sqlalchemy.orm import Session, joinedload

from src.core.database import Base

ModelType = TypeVar("ModelType", bound=Base)


class BaseRepository(Generic[ModelType]):
    def __init__(self, model: Type[ModelType]):
        self.model = model

    def get(self, db: Session, id: int) -> ModelType | None:
        return db.get(self.model, id)

    def get_all(self, db: Session, skip: int = 0, limit: int = 100) -> list[ModelType]:
        return db.query(self.model).offset(skip).limit(limit).all()

    def create(self, db: Session, obj: ModelType) -> ModelType:
        db.add(obj)
        db.commit()
        db.refresh(obj)
        return obj

    def update(self, db: Session, obj: ModelType, data: dict) -> ModelType:
        for key, value in data.items():
            if value is not None:
                setattr(obj, key, value)
        db.commit()
        db.refresh(obj)
        return obj

    def delete(self, db: Session, obj: ModelType) -> bool:
        db.delete(obj)
        db.commit()
        return True
```

### 5.2 Repository con Joins

```python
# src/api/books/repository.py
from sqlalchemy import select, and_, func
from sqlalchemy.orm import Session, joinedload

from src.core.database import Base
from src.api.books.models import Book, BookAuthor


class BookRepository:
    def get_by_id(self, db: Session, id: int) -> Book | None:
        return (
            db.query(Book)
            .options(
                joinedload(Book.genre),
                joinedload(Book.authors).joinedload(BookAuthor.author),
                joinedload(Book.editions),
            )
            .filter(Book.id == id)
            .first()
        )

    def get_all_paginated(
        self, db: Session, page: int, limit: int, search: str | None = None
    ):
        query = db.query(Book).options(joinedload(Book.genre))

        if search:
            query = query.filter(Book.title.ilike(f"%{search}%"))

        # Count
        total = query.count()

        # Paginate
        offset = (page - 1) * limit
        items = query.order_by(Book.created_at.desc()).offset(offset).limit(limit).all()

        pages = (total + limit - 1) // limit

        return {
            "page": page,
            "pages": pages,
            "items": total,
            "limit": limit,
            "data": items,
        }

    def create(self, db: Session, data: dict) -> Book:
        book = Book(**data)
        db.add(book)
        db.commit()
        db.refresh(book)
        return book

    def delete(self, db: Session, book: Book) -> bool:
        db.delete(book)
        db.commit()
        return True
```

### 5.3 Repository sin Try/Catch

**Regla:** Repository es thin layer. No hace try/catch. Las excepciones propagan al Service.

```python
# ✅ CORRECTO
def create(self, db: Session, data: dict) -> Book:
    book = Book(**data)
    db.add(book)
    db.commit()
    db.refresh(book)
    return book

# ❌ INCORRECTO
def create(self, db: Session, data: dict) -> Book:
    try:
        book = Book(**data)
        db.add(book)
        db.commit()
        return book
    except Exception as e:
        db.rollback()
        raise e  # No aporta nada
```

---

## 6. Service Layer

### 6.1 Service Base

```python
# src/api/books/service.py
from sqlalchemy.orm import Session
from typing import Optional

from src.api.books.repository import BookRepository
from src.api.books.schemas import (
    CreateBookDTO,
    UpdateBookDTO,
    BookDTO,
    BookDetailDTO,
    BookFilterDTO,
)
from src.shared.schemas.responses import PaginationResponseDTO


class BookService:
    def __init__(self):
        self.repository = BookRepository()

    def get_by_id(self, db: Session, id: int) -> BookDetailDTO | None:
        book = self.repository.get_by_id(db, id)
        if not book:
            return None

        return BookDetailDTO(
            id=book.id,
            title=book.title,
            summary=book.summary,
            genre_id=book.genre_id,
            genre_name=book.genre.name if book.genre else None,
            author_names=[ba.author.name for ba in book.authors],
            editions_count=len(book.editions),
        )

    def get_all_paginated(
        self,
        db: Session,
        page: int = 1,
        limit: int = 10,
        search: str | None = None,
    ) -> PaginationResponseDTO[BookDTO]:
        result = self.repository.get_all_paginated(db, page, limit, search)

        books = [
            BookDTO(
                id=b.id,
                title=b.title,
                summary=b.summary,
                genre_id=b.genre_id,
            )
            for b in result["data"]
        ]

        return PaginationResponseDTO(
            page=result["page"],
            pages=result["pages"],
            items=result["items"],
            limit=result["limit"],
            data=books,
        )

    def create(self, db: Session, dto: CreateBookDTO) -> BookDTO:
        data = dto.model_dump()

        # Manejar authors si existen
        author_ids = data.pop("author_ids", [])

        book = self.repository.create(db, data)

        # Crear relaciones
        if author_ids:
            for author_id in author_ids:
                book_author = BookAuthor(book_id=book.id, author_id=author_id)
                db.add(book_author)
            db.commit()

        return BookDTO.model_validate(book)

    def update(self, db: Session, id: int, dto: UpdateBookDTO) -> BookDTO | None:
        book = self.repository.get_by_id(db, id)
        if not book:
            return None

        data = {k: v for k, v in dto.model_dump().items() if v is not None}
        updated = self.repository.update(db, book, data)

        return BookDTO.model_validate(updated)

    def delete(self, db: Session, id: int) -> bool:
        book = self.repository.get_by_id(db, id)
        if not book:
            return False
        return self.repository.delete(db, book)
```

### 6.2 Service sin Try/Catch

**Regla:** Service maneja lógica de negocio. Las excepciones de validación deben subir a Routes.

```python
# ✅ CORRECTO
def create(self, db: Session, dto: CreateBookDTO) -> BookDTO:
    # Validaciones de negocio
    if dto.genre_id:
        genre = db.get(Genre, dto.genre_id)
        if not genre:
            raise ValueError(f"Genre with id {dto.genre_id} not found")

    return self.repository.create(db, dto.model_dump())

# ❌ INCORRECTO
def create(self, db: Session, dto: CreateBookDTO):
    try:
        return self.repository.create(db, dto.model_dump())
    except ValueError as e:
        raise e  # No hace nada
```

---

## 7. Routes FastAPI

### 7.1 Routes con Dependencias

```python
# src/api/books/routes.py
from fastapi import APIRouter, Depends, Query, status
from sqlalchemy.orm import Session

from src.core.database import get_db
from src.core.security import get_current_user
from src.shared.schemas import ApiResponse, PaginationRequestDTO
from src.shared.schemas.responses import PaginationResponseDTO
from src.api.books.schemas import (
    CreateBookDTO,
    UpdateBookDTO,
    BookDTO,
    BookDetailDTO,
    BookFilterDTO,
)
from src.api.books.service import BookService

router = APIRouter(prefix="/books", tags=["books"])


def get_book_service() -> BookService:
    return BookService()


@router.get(
    "/pagination",
    response_model=ApiResponse[PaginationResponseDTO[BookDTO]],
    summary="Listar libros con paginación",
    description="Retorna lista paginada de libros",
)
def get_books_paginated(
    page: int = Query(default=1, ge=1),
    limit: int = Query(default=10, ge=1, le=100),
    search: str = Query(default=""),
    db: Session = Depends(get_db),
    service: BookService = Depends(get_book_service),
):
    result = service.get_all_paginated(db, page, limit, search or None)
    return ApiResponse.success(result)


@router.get(
    "/{id}",
    response_model=ApiResponse[BookDetailDTO],
    summary="Obtener libro por ID",
)
def get_book(
    id: int,
    db: Session = Depends(get_db),
    service: BookService = Depends(get_book_service),
):
    book = service.get_by_id(db, id)
    if not book:
        return ApiResponse.error("Libro no encontrado", "NOT_FOUND")
    return ApiResponse.success(book)


@router.post(
    "/",
    response_model=ApiResponse[BookDTO],
    status_code=status.HTTP_201_CREATED,
    summary="Crear libro",
)
def create_book(
    dto: CreateBookDTO,
    db: Session = Depends(get_db),
    service: BookService = Depends(get_book_service),
    current_user: dict = Depends(get_current_user),
):
    try:
        book = service.create(db, dto)
        return ApiResponse.created(book, "Libro creado exitosamente")
    except ValueError as e:
        return ApiResponse.error(str(e), "VALIDATION_ERROR")


@router.put(
    "/{id}",
    response_model=ApiResponse[BookDTO],
    summary="Actualizar libro",
)
def update_book(
    id: int,
    dto: UpdateBookDTO,
    db: Session = Depends(get_db),
    service: BookService = Depends(get_book_service),
    current_user: dict = Depends(get_current_user),
):
    book = service.update(db, id, dto)
    if not book:
        return ApiResponse.error("Libro no encontrado", "NOT_FOUND")
    return ApiResponse.success(book)


@router.delete(
    "/{id}",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Eliminar libro",
)
def delete_book(
    id: int,
    db: Session = Depends(get_db),
    service: BookService = Depends(get_book_service),
    current_user: dict = Depends(get_current_user),
):
    success = service.delete(db, id)
    if not success:
        return ApiResponse.error("Libro no encontrado", "NOT_FOUND")
    return None
```

### 7.2 Orden de Rutas

```python
# ✅ Correcto - específico primero
@router.get("/overdue", ...)     # Se ejecuta primero
@router.get("/pagination", ...)  # Segundo
@router.get("/{id}", ...)         # Path param al final

# ❌ Incorrecto
@router.get("/{id}", ...)         # Intercepta todo
@router.get("/overdue", ...)     # Nunca se ejecuta
```

---

## 8. Autenticación JWT

### 8.1 Security Module

```python
# src/core/security.py
from datetime import datetime, timedelta, timezone
from typing import Optional

from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import jwt, JWTError
from passlib.context import CryptContext

from src.core.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/auth/login")


def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)


def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + (
        expires_delta or timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    to_encode.update({"exp": expire, "iat": datetime.now(timezone.utc)})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)


def get_current_user(required_roles: list[str] | None = None):
    def _get_user(token: str = Depends(oauth2_scheme)):
        try:
            payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
            user_id: str = payload.get("sub")
            role: str = payload.get("role", "user")

            if not user_id:
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Token inválido",
                )

            if required_roles and role not in required_roles:
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="No tienes permisos suficientes",
                )

            return {"user_id": user_id, "role": role}

        except JWTError:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Token inválido",
            )

    return _get_user
```

### 8.2 Routes de Auth

```python
# src/api/auth/routes.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm

from src.core.security import create_access_token, verify_password
from src.shared.schemas import ApiResponse

router = APIRouter(prefix="/auth", tags=["auth"])


@router.post("/login")
def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: Session = Depends(get_db),
):
    user = user_service.get_by_email(db, form_data.username)

    if not user or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciales incorrectas",
        )

    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Usuario inactivo",
        )

    token = create_access_token({"sub": str(user.id), "role": user.role})

    return {"access_token": token, "token_type": "bearer"}
```

### 8.3 Roles como Enum

```python
# src/core/enums.py
from enum import Enum


class UserRole(str, Enum):
    ADMIN = "admin"
    EDITOR = "editor"
    USER = "user"


# Uso en routes
admin_required = Depends(get_current_user(required_roles=[UserRole.ADMIN]))
```

---

## 9. WebSockets Reactivos

### 9.1 Connection Manager

```python
# src/core/websocket.py
import json
from typing import Dict
from fastapi import WebSocket


class ConnectionManager:
    def __init__(self):
        self.active_connections: Dict[str, WebSocket] = {}

    async def connect(self, websocket: WebSocket, user_id: str):
        await websocket.accept()
        self.active_connections[user_id] = websocket

    def disconnect(self, user_id: str):
        if user_id in self.active_connections:
            del self.active_connections[user_id]

    async def send_message(self, user_id: str, message: dict):
        if user_id in self.active_connections:
            await self.active_connections[user_id].send_json(message)

    async def broadcast(self, message: dict):
        for connection in self.active_connections.values():
            await connection.send_json(message)


manager = ConnectionManager()
```

### 9.2 WebSocket Endpoint

```python
# src/api/websocket/routes.py
from fastapi import APIRouter, WebSocket, WebSocketDisconnect, Depends
from src.core.websocket import manager
from src.core.security import get_current_user

router = APIRouter()


@router.websocket("/ws")
async def websocket_endpoint(
    websocket: WebSocket,
    token: str = None  # Query param
):
    # Extraer user_id del token
    if not token:
        await websocket.close(code=4001)
        return

    user_id = await get_user_from_token(token)  # Tu lógica

    await manager.connect(websocket, user_id)
    try:
        while True:
            data = await websocket.receive_text()
            # Process message
    except WebSocketDisconnect:
        manager.disconnect(user_id)
```

### 9.3 Notificaciones Reactivas (PostgreSQL LISTEN/NOTIFY)

```python
# src/core/realtime.py
import asyncio
import redis.asyncio as redis
from src.core.config import settings


class RealTimeManager:
    def __init__(self):
        self.redis: redis.Redis | None = None
        self._pubsub_task: asyncio.Task | None = None

    async def connect(self):
        self.redis = redis.from_url(settings.REDIS_URL)
        self._pubsub_task = asyncio.create_task(self._listen_messages())

    async def _listen_messages(self):
        pubsub = self.redis.pubsub()
        await pubsub.subscribe("notifications")

        async for message in pubsub.listen():
            if message["type"] == "message":
                data = json.loads(message["data"])
                user_id = data.get("user_id")
                if user_id:
                    from src.core.websocket import manager
                    await manager.send_message(user_id, data)

    async def publish(self, channel: str, message: dict):
        if self.redis:
            await self.redis.publish(channel, json.dumps(message))

    async def disconnect(self):
        if self._pubsub_task:
            self._pubsub_task.cancel()
        if self.redis:
            await self.redis.close()


realtime_manager = RealTimeManager()
```

---

## 10. Paginación

### 10.1 Pagination Request DTO

```python
# src/shared/schemas/pagination.py
from typing import Generic, TypeVar, Optional
from pydantic import BaseModel, Field

T = TypeVar("T")


class PaginationRequestDTO(BaseModel, Generic[T]):
    page: int = Field(default=1, ge=1)
    limit: int = Field(default=10, ge=1, le=100)
    search: str = Field(default="")
    filter: Optional[T] = Field(default=None)


# Uso
pagination = PaginationRequestDTO(page=1, limit=10)
pagination_with_filter = PaginationRequestDTO[BookFilterDTO](
    page=1, limit=10, filter=BookFilterDTO(genre_id=1)
)
```

### 10.2 Repository con Paginación

```python
def get_all_paginated(
    self,
    db: Session,
    page: int,
    limit: int,
    search: str | None = None,
    filter: dict | None = None,
) -> dict:
    query = db.query(Book).options(joinedload(Book.genre))

    # Search
    if search:
        query = query.filter(Book.title.ilike(f"%{search}%"))

    # Filters
    if filter and filter.get("genre_id"):
        query = query.filter(Book.genre_id == filter["genre_id"])

    # Count
    total = query.count()

    # Paginate
    offset = (page - 1) * limit
    items = query.order_by(Book.created_at.desc()).offset(offset).limit(limit).all()

    pages = (total + limit - 1) // limit if total > 0 else 0

    return {
        "page": page,
        "pages": pages,
        "items": total,
        "limit": limit,
        "data": items,
    }
```

---

## 11. Manejo de Errores

### 11.1 Custom Exceptions

```python
# src/core/exceptions.py
from fastapi import HTTPException, status


class ApiException(HTTPException):
    def __init__(self, status_code: int, detail: str, error_code: str | None = None):
        super().__init__(status_code=status_code, detail=detail)
        self.error_code = error_code


class NotFoundError(ApiException):
    def __init__(self, resource: str):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"{resource} no encontrado",
            error_code="NOT_FOUND",
        )


class ValidationError(ApiException):
    def __init__(self, detail: str):
        super().__init__(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            detail=detail,
            error_code="VALIDATION_ERROR",
        )


class UnauthorizedError(ApiException):
    def __init__(self, detail: str = "No autorizado"):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=detail,
            error_code="UNAUTHORIZED",
        )


class ForbiddenError(ApiException):
    def __init__(self, detail: str = "Acceso denegado"):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            detail=detail,
            error_code="FORBIDDEN",
        )
```

### 11.2 Error Handler Global

```python
# src/main.py
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={
            "is_success": False,
            "message": "Error interno del servidor",
            "error_code": "INTERNAL_ERROR",
        },
    )
```

---

## 12. Testing

### 12.1 Fixtures

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from src.main import app
from src.core.database import Base, get_db

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


@pytest.fixture(scope="function")
def db_session():
    Base.metadata.create_all(bind=engine)
    session = TestingSessionLocal()
    try:
        yield session
    finally:
        session.close()
        Base.metadata.drop_all(bind=engine)


@pytest.fixture(scope="function")
def client(db_session):
    def override_get_db():
        try:
            yield db_session
        finally:
            pass

    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    app.dependency_overrides.clear()
```

### 12.2 Tests de API

```python
# tests/api/test_books.py
import pytest
from fastapi.testclient import TestClient


def test_create_book(client: TestClient):
    response = client.post(
        "/api/books/",
        json={
            "title": "Test Book",
            "summary": "Test summary",
            "genre_id": 1,
        },
    )
    assert response.status_code == 201
    data = response.json()
    assert data["is_success"] is True
    assert data["data"]["title"] == "Test Book"


def test_get_book_not_found(client: TestClient):
    response = client.get("/api/books/999")
    assert response.status_code == 200
    data = response.json()
    assert data["is_success"] is False
    assert data["error_code"] == "NOT_FOUND"
```

---

## 13. Deployment

### 13.1 Docker

```dockerfile
# Dockerfile
FROM python:3.12-slim

WORKDIR /app

# Install Poetry
RUN pip install poetry

# Copy only dependency files
COPY pyproject.toml poetry.lock ./

# Install dependencies
RUN poetry config virtualenvs.create false \
    && poetry install --no-dev

# Copy source
COPY . .

# Expose port
EXPOSE 8000

# Run
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 13.2 Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/app
      - REDIS_URL=redis://redis:6379
      - SECRET_KEY=your-secret-key
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 13.3 Environment Variables

```bash
# .env.example
DATABASE_URL=postgresql://user:pass@localhost:5432/app
REDIS_URL=redis://localhost:6379

SECRET_KEY=your-super-secret-key-change-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

CORS_ORIGINS=["http://localhost:4200","http://localhost:3000"]

DEBUG=false
LOG_LEVEL=INFO
```

### 13.4 Config Production

```python
# src/core/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str
    REDIS_URL: str
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60
    CORS_ORIGINS: list[str] = ["*"]
    DEBUG: bool = False
    LOG_LEVEL: str = "INFO"

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )


settings = Settings()
```

---

## 14. Checklist de Código

### Models
- [ ] Timestamps con `func.now()` (created_at, updated_at)
- [ ] Foreign Keys con nombres completos
- [ ] Relationships con `back_populates`
- [ ] Índices en campos de búsqueda
- [ ] Tipo correcto en columnas (String vs Text)

### DTOs
- [ ] `model_config = ConfigDict(from_attributes=True)` en todos
- [ ] Usar `model_validate()` para conversión ORM → DTO
- [ ] `Field(...)` con descripciones
- [ ] Naming consistente (Create/Update/Detail/Filter)
- [ ] Tipado completo (Optional, UUID, date)

### Repository
- [ ] `joinedload` para evitar N+1
- [ ] SIN try/catch (propaga errores)
- [ ] No retorna DTOs, retorna entidades

### Service
- [ ] SIN try/catch (propaga errores)
- [ ] Lógica de negocio aquí
- [ ] Validaciones de dependencias
- [ ] Crear entidades con relaciones

### Routes
- [ ] Docstrings (summary, description)
- [ ] Dependencias de autenticación
- [ ] ApiResponse en todas las respuestas
- [ ] Rutas específicas antes de path params

### Paginación
- [ ] PaginationRequestDTO genérico
- [ ] Total pages y items calculados
- [ ] Offset calculation correcto

### General
- [ ] Poetry para dependencias (NO pip/requirements.txt)
- [ ] Environment variables en config
- [ ] No strings hardcodeados
- [ ] No queries en loops
- [ ] Logging estructurado en producción

---

## Recursos

- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [SQLAlchemy 2.0 Docs](https://docs.sqlalchemy.org/)
- [Pydantic V2 Docs](https://docs.pydantic.dev/)
- [Poetry Docs](https://python-poetry.org/docs/)
- [Python-Jose Docs](https://python-jose.readthedocs.io/)

---

*Python 3.12 + FastAPI Modern Development Skill*
*Versión: 1.0*
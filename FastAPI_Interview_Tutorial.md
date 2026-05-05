# FastAPI Interview Tutorial - Quick Recap Guide


## Table of Contents
1. [FastAPI Basics](#fastapi-basics)
2. [Path Operations & Routing](#path-operations--routing)
3. [Request Body & Pydantic Models](#request-body--pydantic-models)
4. [Path Parameters & Query Parameters](#path-parameters--query-parameters)
5. [Response Models](#response-models)
6. [Dependency Injection](#dependency-injection)
7. [Authentication & Security](#authentication--security)
8. [Database Integration](#database-integration)
9. [Middleware](#middleware)
10. [Background Tasks](#background-tasks)
11. [WebSockets](#websockets)
12. [Testing](#testing)
13. [Common Interview Questions](#common-interview-questions)


---

## FastAPI Basics

### What is FastAPI?
FastAPI is a modern, fast (high-performance) web framework for building APIs with Python 3.7+ based on standard Python type hints.

### Key Features
- **Fast**: Very high performance, on par with NodeJS and Go
- **Fast to code**: Increases development speed
- **Automatic documentation**: Swagger UI and ReDoc
- **Type hints**: Built on Python type hints
- **Async support**: Native async/await support
- **Data validation**: Automatic validation using Pydantic

### Installation
```bash
pip install fastapi
pip install "uvicorn[standard]"
```

### Basic Hello World
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}

# Run with: uvicorn main:app --reload
```


### Interview Question 1: What makes FastAPI fast?

**Answer:**
FastAPI's speed comes from several factors:

1. **Starlette**: Built on Starlette for web routing (async framework)
2. **Pydantic**: Uses Pydantic for data validation (written in Cython)
3. **Async/Await**: Native async support for concurrent operations
4. **Type hints**: Compile-time optimizations
5. **ASGI**: Uses ASGI instead of WSGI for async support

**Performance comparison:**
- FastAPI: ~20,000-25,000 requests/second
- Django: ~3,000-5,000 requests/second
- Flask: ~2,000-3,000 requests/second

```python
# Async example - handles multiple requests concurrently
import asyncio
from fastapi import FastAPI

app = FastAPI()

@app.get("/slow-operation")
async def slow_operation():
    await asyncio.sleep(1)  # Simulates I/O operation
    return {"status": "completed"}

# Multiple requests can be processed during the sleep time
```

---

## Path Operations & Routing

### HTTP Methods
```python
from fastapi import FastAPI

app = FastAPI()

# GET - Retrieve data
@app.get("/items/")
async def read_items():
    return {"items": []}

# POST - Create data
@app.post("/items/")
async def create_item(item: dict):
    return {"item": item}

# PUT - Update data (full update)
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: dict):
    return {"item_id": item_id, "item": item}

# PATCH - Partial update
@app.patch("/items/{item_id}")
async def patch_item(item_id: int, item: dict):
    return {"item_id": item_id, "updated": item}

# DELETE - Delete data
@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    return {"deleted": item_id}
```

### Path Operation Decorators
```python
from fastapi import FastAPI, status

app = FastAPI()

@app.get(
    "/items/",
    tags=["items"],  # Groups in documentation
    summary="Get all items",
    description="Retrieve a list of all items",
    response_description="List of items",
    status_code=status.HTTP_200_OK,
    deprecated=False  # Mark as deprecated
)
async def read_items():
    return {"items": []}
```

### Router (APIRouter)
```python
# users.py
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["users"],
    responses={404: {"description": "Not found"}}
)

@router.get("/")
async def read_users():
    return [{"username": "john"}, {"username": "jane"}]

@router.get("/{user_id}")
async def read_user(user_id: int):
    return {"user_id": user_id}

# main.py
from fastapi import FastAPI
from users import router as users_router

app = FastAPI()
app.include_router(users_router)
```


### Interview Question 2: What's the difference between async def and def in FastAPI?

**Answer:**

**async def (Asynchronous):**
- Use when performing I/O operations (database, API calls, file operations)
- Allows other requests to be processed during waiting
- Must use `await` for async operations
- Better for I/O-bound tasks

**def (Synchronous):**
- Use for CPU-intensive operations
- Runs in a thread pool
- Blocks during execution
- Better for CPU-bound tasks

```python
import asyncio
import time
from fastapi import FastAPI

app = FastAPI()

# Async - Good for I/O operations
@app.get("/async-endpoint")
async def async_endpoint():
    await asyncio.sleep(1)  # Non-blocking
    return {"message": "async"}

# Sync - Good for CPU operations
@app.get("/sync-endpoint")
def sync_endpoint():
    time.sleep(1)  # Blocking
    return {"message": "sync"}

# Wrong - Don't use blocking code in async
@app.get("/wrong")
async def wrong_endpoint():
    time.sleep(1)  # BAD! Blocks the event loop
    return {"message": "wrong"}

# Correct - Use async libraries
import httpx

@app.get("/correct")
async def correct_endpoint():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com")
    return response.json()
```

---

## Request Body & Pydantic Models

### Pydantic BaseModel
```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, EmailStr, validator
from typing import Optional, List
from datetime import datetime

app = FastAPI()

class User(BaseModel):
    id: int
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    full_name: Optional[str] = None
    age: int = Field(..., gt=0, le=120)
    is_active: bool = True
    created_at: datetime = Field(default_factory=datetime.now)
    tags: List[str] = []
    
    # Custom validator
    @validator('username')
    def username_alphanumeric(cls, v):
        assert v.isalnum(), 'must be alphanumeric'
        return v
    
    class Config:
        schema_extra = {
            "example": {
                "id": 1,
                "username": "johndoe",
                "email": "john@example.com",
                "full_name": "John Doe",
                "age": 30,
                "is_active": True,
                "tags": ["developer", "python"]
            }
        }

@app.post("/users/")
async def create_user(user: User):
    return user

# Automatic validation happens before function is called
```

### Nested Models
```python
from pydantic import BaseModel
from typing import List, Optional

class Address(BaseModel):
    street: str
    city: str
    country: str
    zipcode: str

class User(BaseModel):
    username: str
    email: str
    address: Address
    
class Company(BaseModel):
    name: str
    employees: List[User]
    headquarters: Address

@app.post("/companies/")
async def create_company(company: Company):
    return company
```

### Request Body + Path + Query Parameters
```python
from fastapi import FastAPI, Query, Path, Body

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(
    item_id: int = Path(..., title="Item ID", ge=1),
    q: Optional[str] = Query(None, max_length=50),
    item: Item = Body(...),
    user: User = Body(...),
    importance: int = Body(...)
):
    results = {
        "item_id": item_id,
        "item": item,
        "user": user,
        "importance": importance
    }
    if q:
        results.update({"q": q})
    return results
```


### Interview Question 3: How does FastAPI perform automatic data validation?

**Answer:**
FastAPI uses **Pydantic** for automatic data validation:

1. **Type hints**: Python type annotations define expected types
2. **Pydantic models**: Validate complex data structures
3. **Automatic validation**: Happens before your function runs
4. **Error responses**: Returns 422 Unprocessable Entity with details

```python
from pydantic import BaseModel, Field, validator
from fastapi import FastAPI, HTTPException

app = FastAPI()

class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0)
    quantity: int = Field(..., ge=0)
    
    @validator('price')
    def price_must_be_reasonable(cls, v):
        if v > 1000000:
            raise ValueError('price too high')
        return v

@app.post("/items/")
async def create_item(item: Item):
    return item

# Request: {"name": "", "price": -10, "quantity": 5}
# Response: 422 with detailed error:
# {
#   "detail": [
#     {
#       "loc": ["body", "name"],
#       "msg": "ensure this value has at least 1 characters",
#       "type": "value_error.any_str.min_length"
#     },
#     {
#       "loc": ["body", "price"],
#       "msg": "ensure this value is greater than 0",
#       "type": "value_error.number.not_gt"
#     }
#   ]
# }
```

---

## Path Parameters & Query Parameters

### Path Parameters
```python
from fastapi import FastAPI, Path
from enum import Enum

app = FastAPI()

# Basic path parameter
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}

# Path parameter with validation
@app.get("/items/{item_id}")
async def read_item(
    item_id: int = Path(..., title="Item ID", ge=1, le=1000)
):
    return {"item_id": item_id}

# Enum path parameter
class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name == ModelName.alexnet:
        return {"model": "AlexNet", "message": "Deep Learning"}
    return {"model": model_name.value}

# File path parameter
@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

### Query Parameters
```python
from fastapi import FastAPI, Query
from typing import Optional, List

app = FastAPI()

# Optional query parameter
@app.get("/items/")
async def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

# Required query parameter
@app.get("/items/")
async def read_items(q: str):
    return {"q": q}

# Query parameter with validation
@app.get("/items/")
async def read_items(
    q: Optional[str] = Query(
        None,
        min_length=3,
        max_length=50,
        regex="^[a-zA-Z]+$",
        title="Query string",
        description="Search query",
        deprecated=False
    )
):
    return {"q": q}

# List query parameters
@app.get("/items/")
async def read_items(
    tags: List[str] = Query([], description="List of tags")
):
    return {"tags": tags}
# URL: /items/?tags=python&tags=fastapi&tags=api
```


### Interview Question 4: What's the difference between Path, Query, and Body parameters?

**Answer:**

**Path Parameters:**
- Part of the URL path
- Always required
- Defined in the path string with `{}`
- Example: `/users/{user_id}`

**Query Parameters:**
- Come after `?` in URL
- Optional by default (unless no default value)
- Used for filtering, pagination, search
- Example: `/users?skip=0&limit=10`

**Body Parameters:**
- Sent in request body (POST, PUT, PATCH)
- Used for complex data structures
- Defined using Pydantic models
- Example: JSON payload

```python
from fastapi import FastAPI, Path, Query, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.put("/items/{item_id}")
async def update_item(
    # Path parameter - in URL path
    item_id: int = Path(..., ge=1),
    
    # Query parameter - after ?
    q: Optional[str] = Query(None, max_length=50),
    
    # Body parameter - in request body
    item: Item = Body(...)
):
    result = {"item_id": item_id, "item": item}
    if q:
        result["q"] = q
    return result

# Request:
# PUT /items/123?q=search
# Body: {"name": "Laptop", "price": 999.99}
```

---

## Response Models

### Response Model
```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr
from typing import Optional, List

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None
    # No password field!

@app.post("/users/", response_model=UserOut)
async def create_user(user: UserIn):
    # Password is in input but not in output
    return user
```

### Response Model with List
```python
from typing import List

@app.get("/users/", response_model=List[UserOut])
async def read_users():
    return [
        {"username": "john", "email": "john@example.com"},
        {"username": "jane", "email": "jane@example.com"}
    ]
```

### Response Status Code
```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_item(item_id: int):
    return None
```

### Multiple Response Models
```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Union

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class ErrorResponse(BaseModel):
    error: str
    detail: str

@app.get(
    "/items/{item_id}",
    response_model=Union[Item, ErrorResponse],
    responses={
        200: {"model": Item, "description": "Successful response"},
        404: {"model": ErrorResponse, "description": "Item not found"}
    }
)
async def read_item(item_id: int):
    if item_id == 0:
        return {"error": "Not Found", "detail": "Item not found"}
    return {"name": "Item", "price": 99.99}
```


### Interview Question 5: Why use response_model in FastAPI?

**Answer:**
`response_model` provides several benefits:

1. **Data filtering**: Excludes fields not in response model (like passwords)
2. **Validation**: Validates output data
3. **Documentation**: Generates accurate API documentation
4. **Type conversion**: Converts data to specified types
5. **Security**: Prevents accidental data leakage

```python
from pydantic import BaseModel
from typing import Optional

class UserIn(BaseModel):
    username: str
    password: str
    email: str
    credit_card: str

class UserOut(BaseModel):
    username: str
    email: str
    # Sensitive fields excluded

@app.post("/users/", response_model=UserOut)
async def create_user(user: UserIn):
    # Even if we return the full user object,
    # only UserOut fields are sent to client
    return user
    # Response: {"username": "...", "email": "..."}
    # password and credit_card are automatically filtered out

# Additional options:
@app.get("/items/", response_model=List[Item], response_model_exclude_unset=True)
async def read_items():
    # Only returns fields that were explicitly set
    pass

@app.get("/items/", response_model=Item, response_model_exclude={"internal_id"})
async def read_item():
    # Excludes specific fields
    pass
```

---

## Dependency Injection

### Basic Dependency
```python
from fastapi import FastAPI, Depends
from typing import Optional

app = FastAPI()

# Dependency function
async def common_parameters(
    skip: int = 0,
    limit: int = 100,
    q: Optional[str] = None
):
    return {"skip": skip, "limit": limit, "q": q}

@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons

@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

### Class-based Dependency
```python
from fastapi import Depends

class CommonQueryParams:
    def __init__(
        self,
        skip: int = 0,
        limit: int = 100,
        q: Optional[str] = None
    ):
        self.skip = skip
        self.limit = limit
        self.q = q

@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
    return {
        "skip": commons.skip,
        "limit": commons.limit,
        "q": commons.q
    }
```

### Sub-dependencies
```python
from fastapi import Depends, HTTPException, status

def verify_token(token: str):
    if token != "secret-token":
        raise HTTPException(status_code=401, detail="Invalid token")
    return token

def verify_key(key: str):
    if key != "secret-key":
        raise HTTPException(status_code=401, detail="Invalid key")
    return key

def verify_credentials(
    token: str = Depends(verify_token),
    key: str = Depends(verify_key)
):
    return {"token": token, "key": key}

@app.get("/protected/")
async def protected_route(credentials: dict = Depends(verify_credentials)):
    return {"message": "Access granted", "credentials": credentials}
```

### Database Dependency
```python
from sqlalchemy.orm import Session

# Database session dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
async def read_users(db: Session = Depends(get_db)):
    users = db.query(User).all()
    return users
```


### Interview Question 6: Explain Dependency Injection in FastAPI with examples.

**Answer:**
**Dependency Injection** is a design pattern where FastAPI automatically provides required dependencies to path operations.

**Benefits:**
1. **Code reuse**: Share common logic across endpoints
2. **Separation of concerns**: Keep code modular
3. **Testing**: Easy to mock dependencies
4. **Security**: Centralize authentication/authorization
5. **Database connections**: Manage DB sessions

```python
from fastapi import FastAPI, Depends, HTTPException
from typing import Optional

app = FastAPI()

# Example 1: Authentication dependency
def get_current_user(token: str):
    if token != "valid-token":
        raise HTTPException(status_code=401, detail="Invalid token")
    return {"username": "john", "id": 1}

@app.get("/me")
async def read_current_user(user: dict = Depends(get_current_user)):
    return user

# Example 2: Pagination dependency
class Pagination:
    def __init__(self, skip: int = 0, limit: int = 10):
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def get_items(pagination: Pagination = Depends()):
    return {
        "skip": pagination.skip,
        "limit": pagination.limit,
        "items": []
    }

# Example 3: Nested dependencies
def get_db():
    return {"db": "connection"}

def get_current_user(token: str, db: dict = Depends(get_db)):
    # Uses db dependency
    return {"user": "john", "db": db}

@app.get("/profile")
async def get_profile(user: dict = Depends(get_current_user)):
    # Automatically resolves both dependencies
    return user

# Example 4: Global dependencies
app = FastAPI(dependencies=[Depends(verify_token)])
# Now all routes require token verification
```

---

## Authentication & Security

### OAuth2 with Password Flow
```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from typing import Optional
from datetime import datetime, timedelta
import jwt

app = FastAPI()

# OAuth2 scheme
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

class Token(BaseModel):
    access_token: str
    token_type: str

class User(BaseModel):
    username: str
    email: Optional[str] = None
    disabled: Optional[bool] = None

# Fake database
fake_users_db = {
    "john": {
        "username": "john",
        "email": "john@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False
    }
}

def verify_password(plain_password, hashed_password):
    # In production, use proper hashing (bcrypt, passlib)
    return plain_password == "secret" and hashed_password == "fakehashedsecret"

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except jwt.PyJWTError:
        raise credentials_exception
    
    user = fake_users_db.get(username)
    if user is None:
        raise credentials_exception
    return User(**user)

@app.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = fake_users_db.get(form_data.username)
    if not user or not verify_password(form_data.password, user["hashed_password"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user["username"]}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

### API Key Authentication
```python
from fastapi import Security, HTTPException, status
from fastapi.security import APIKeyHeader

API_KEY = "your-api-key"
api_key_header = APIKeyHeader(name="X-API-Key")

def get_api_key(api_key: str = Security(api_key_header)):
    if api_key != API_KEY:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Invalid API Key"
        )
    return api_key

@app.get("/protected")
async def protected_route(api_key: str = Depends(get_api_key)):
    return {"message": "Access granted"}
```


### Interview Question 7: How do you implement JWT authentication in FastAPI?

**Answer:**
JWT (JSON Web Token) authentication in FastAPI involves:

1. **Login endpoint**: Validates credentials and returns JWT token
2. **Token creation**: Encodes user data with expiration
3. **Token validation**: Decodes and verifies token on protected routes
4. **Dependency**: Uses OAuth2PasswordBearer for token extraction

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from pydantic import BaseModel

app = FastAPI()

# Configuration
SECRET_KEY = "your-secret-key-keep-it-secret"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class TokenData(BaseModel):
    username: str

class User(BaseModel):
    username: str
    email: str

# Step 1: Hash password
def get_password_hash(password):
    return pwd_context.hash(password)

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

# Step 2: Create JWT token
def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# Step 3: Verify JWT token
async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    
    # Get user from database
    user = get_user(username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# Step 4: Login endpoint
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token = create_access_token(data={"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}

# Step 5: Protected route
@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

---

## Database Integration

### SQLAlchemy Setup
```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from fastapi import FastAPI, Depends

# Database URL
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
# For PostgreSQL: "postgresql://user:password@localhost/dbname"

# Create engine
engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False}  # Only for SQLite
)

# Session
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base class
Base = declarative_base()

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Models
```python
from sqlalchemy import Column, Integer, String, Boolean, ForeignKey
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    username = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)
    
    items = relationship("Item", back_populates="owner")

class Item(Base):
    __tablename__ = "items"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String)
    owner_id = Column(Integer, ForeignKey("users.id"))
    
    owner = relationship("User", back_populates="items")

# Create tables
Base.metadata.create_all(bind=engine)
```

### Pydantic Schemas
```python
from pydantic import BaseModel
from typing import List, Optional

class ItemBase(BaseModel):
    title: str
    description: Optional[str] = None

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int
    owner_id: int
    
    class Config:
        orm_mode = True  # Allows reading from ORM models

class UserBase(BaseModel):
    email: str
    username: str

class UserCreate(UserBase):
    password: str

class User(UserBase):
    id: int
    is_active: bool
    items: List[Item] = []
    
    class Config:
        orm_mode = True
```

### CRUD Operations
```python
from sqlalchemy.orm import Session
from fastapi import FastAPI, Depends, HTTPException

app = FastAPI()

# Create
@app.post("/users/", response_model=User)
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = User(
        email=user.email,
        username=user.username,
        hashed_password=hash_password(user.password)
    )
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user

# Read
@app.get("/users/{user_id}", response_model=User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Read all
@app.get("/users/", response_model=List[User])
def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = db.query(User).offset(skip).limit(limit).all()
    return users

# Update
@app.put("/users/{user_id}", response_model=User)
def update_user(user_id: int, user: UserCreate, db: Session = Depends(get_db)):
    db_user = db.query(User).filter(User.id == user_id).first()
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    
    db_user.email = user.email
    db_user.username = user.username
    db.commit()
    db.refresh(db_user)
    return db_user

# Delete
@app.delete("/users/{user_id}")
def delete_user(user_id: int, db: Session = Depends(get_db)):
    db_user = db.query(User).filter(User.id == user_id).first()
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    
    db.delete(db_user)
    db.commit()
    return {"message": "User deleted"}
```


### Interview Question 8: How do you integrate a database with FastAPI?

**Answer:**
Database integration in FastAPI typically uses SQLAlchemy ORM:

**Steps:**
1. **Setup**: Configure database connection and engine
2. **Models**: Define SQLAlchemy models
3. **Schemas**: Create Pydantic schemas for validation
4. **Dependency**: Create database session dependency
5. **CRUD**: Implement create, read, update, delete operations

```python
# 1. Database setup (database.py)
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "postgresql://user:password@localhost/dbname"

engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# 2. SQLAlchemy model (models.py)
from sqlalchemy import Column, Integer, String
from database import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    name = Column(String)

# 3. Pydantic schema (schemas.py)
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str
    name: str

class User(BaseModel):
    id: int
    email: str
    name: str
    
    class Config:
        orm_mode = True  # Enables ORM mode

# 4. Main application (main.py)
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from database import get_db, engine
import models, schemas

models.Base.metadata.create_all(bind=engine)

app = FastAPI()

@app.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = models.User(email=user.email, name=user.name)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user

@app.get("/users/{user_id}", response_model=schemas.User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    return db.query(models.User).filter(models.User.id == user_id).first()
```

---

## Middleware

### Custom Middleware
```python
from fastapi import FastAPI, Request
import time

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### CORS Middleware
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

### Trusted Host Middleware
```python
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["example.com", "*.example.com"]
)
```

### GZip Middleware
```python
from fastapi.middleware.gzip import GZipMiddleware

app.add_middleware(GZipMiddleware, minimum_size=1000)
```

### Interview Question 9: What is middleware in FastAPI and when would you use it?

**Answer:**
**Middleware** is a function that processes requests before they reach path operations and responses before they're returned to the client.

**Use cases:**
1. **Logging**: Log all requests and responses
2. **Authentication**: Verify tokens globally
3. **CORS**: Handle cross-origin requests
4. **Compression**: Compress responses
5. **Rate limiting**: Limit request frequency
6. **Request timing**: Measure processing time

```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
import time
import logging

app = FastAPI()

# Example 1: Logging middleware
@app.middleware("http")
async def log_requests(request: Request, call_next):
    logging.info(f"Request: {request.method} {request.url}")
    response = await call_next(request)
    logging.info(f"Response status: {response.status_code}")
    return response

# Example 2: Timing middleware
@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response

# Example 3: CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Example 4: Custom authentication middleware
@app.middleware("http")
async def authenticate(request: Request, call_next):
    if request.url.path.startswith("/public"):
        return await call_next(request)
    
    token = request.headers.get("Authorization")
    if not token or not verify_token(token):
        return JSONResponse(
            status_code=401,
            content={"detail": "Unauthorized"}
        )
    
    response = await call_next(request)
    return response
```

---

## Background Tasks

### Basic Background Task
```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

def write_log(message: str):
    time.sleep(5)  # Simulate slow operation
    with open("log.txt", "a") as f:
        f.write(f"{message}\n")

@app.post("/send-notification/")
async def send_notification(
    email: str,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    return {"message": "Notification sent in the background"}
```

### Multiple Background Tasks
```python
def send_email(email: str, message: str):
    print(f"Sending email to {email}: {message}")
    time.sleep(2)

def write_to_database(data: dict):
    print(f"Writing to database: {data}")
    time.sleep(1)

@app.post("/process/")
async def process_data(
    email: str,
    data: dict,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(send_email, email, "Processing started")
    background_tasks.add_task(write_to_database, data)
    background_tasks.add_task(send_email, email, "Processing completed")
    
    return {"message": "Processing started"}
```

### Background Task with Dependency
```python
from sqlalchemy.orm import Session

def update_user_stats(user_id: int, db: Session):
    # Update user statistics in background
    user = db.query(User).filter(User.id == user_id).first()
    user.last_activity = datetime.now()
    db.commit()

@app.post("/items/")
async def create_item(
    item: ItemCreate,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    # Create item
    db_item = Item(**item.dict(), owner_id=current_user.id)
    db.add(db_item)
    db.commit()
    
    # Update stats in background
    background_tasks.add_task(update_user_stats, current_user.id, db)
    
    return db_item
```

### Interview Question 10: When should you use background tasks in FastAPI?

**Answer:**
**Background tasks** are used for operations that don't need to complete before returning a response.

**Use cases:**
1. **Email sending**: Send emails without blocking response
2. **File processing**: Process uploaded files asynchronously
3. **Logging**: Write logs without slowing down requests
4. **Notifications**: Send push notifications
5. **Data cleanup**: Clean up temporary data
6. **Analytics**: Track events asynchronously

**Important notes:**
- For simple tasks that take a few seconds
- For long-running tasks, use Celery or similar task queues
- Background tasks run in the same process

```python
from fastapi import FastAPI, BackgroundTasks, UploadFile, File
import smtplib

app = FastAPI()

# Example 1: Email notification
def send_email_notification(email: str, message: str):
    # Simulate email sending
    print(f"Sending email to {email}: {message}")
    time.sleep(3)  # Email sending takes time

@app.post("/register/")
async def register_user(
    email: str,
    password: str,
    background_tasks: BackgroundTasks
):
    # Create user immediately
    user = create_user_in_db(email, password)
    
    # Send welcome email in background
    background_tasks.add_task(
        send_email_notification,
        email,
        "Welcome to our platform!"
    )
    
    # Return immediately without waiting for email
    return {"message": "User created", "user_id": user.id}

# Example 2: File processing
def process_file(filename: str):
    # Process file (resize image, convert video, etc.)
    print(f"Processing {filename}")
    time.sleep(10)  # Long operation

@app.post("/upload/")
async def upload_file(
    file: UploadFile = File(...),
    background_tasks: BackgroundTasks = None
):
    # Save file immediately
    with open(f"uploads/{file.filename}", "wb") as f:
        f.write(await file.read())
    
    # Process in background
    background_tasks.add_task(process_file, file.filename)
    
    return {"filename": file.filename, "status": "processing"}

# Example 3: Analytics tracking
def track_event(user_id: int, event: str, data: dict):
    # Send to analytics service
    print(f"Tracking: User {user_id} - {event}")

@app.get("/items/{item_id}")
async def get_item(
    item_id: int,
    background_tasks: BackgroundTasks,
    current_user: User = Depends(get_current_user)
):
    item = get_item_from_db(item_id)
    
    # Track view in background
    background_tasks.add_task(
        track_event,
        current_user.id,
        "item_viewed",
        {"item_id": item_id}
    )
    
    return item
```

---

## WebSockets

### Basic WebSocket
```python
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>WebSocket Test</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""

@app.get("/")
async def get():
    return HTMLResponse(html)

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message received: {data}")
```

### WebSocket with Connection Manager
```python
from fastapi import WebSocket, WebSocketDisconnect
from typing import List

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []
    
    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)
    
    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)
    
    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)
    
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
            await manager.send_personal_message(f"You wrote: {data}", websocket)
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")
```


---

## Testing

### Basic Testing with TestClient
```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/")
async def read_main():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}

# Test file (test_main.py)
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

def test_read_item():
    response = client.get("/items/42?q=test")
    assert response.status_code == 200
    assert response.json() == {"item_id": 42, "q": "test"}

def test_read_item_bad_request():
    response = client.get("/items/foo")
    assert response.status_code == 422  # Validation error
```

### Testing with Database
```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from fastapi.testclient import TestClient

# Test database
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Override dependency
def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

client = TestClient(app)

@pytest.fixture
def test_db():
    Base.metadata.create_all(bind=engine)
    yield
    Base.metadata.drop_all(bind=engine)

def test_create_user(test_db):
    response = client.post(
        "/users/",
        json={"email": "test@example.com", "password": "password123"}
    )
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "id" in data
```

### Testing Authentication
```python
def test_login():
    response = client.post(
        "/token",
        data={"username": "testuser", "password": "testpass"}
    )
    assert response.status_code == 200
    assert "access_token" in response.json()

def test_protected_route():
    # Login first
    login_response = client.post(
        "/token",
        data={"username": "testuser", "password": "testpass"}
    )
    token = login_response.json()["access_token"]
    
    # Access protected route
    response = client.get(
        "/users/me",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200
    assert response.json()["username"] == "testuser"

def test_protected_route_no_token():
    response = client.get("/users/me")
    assert response.status_code == 401
```

### Async Testing
```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_async_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}
```

### Interview Question 11: How do you test FastAPI applications?

**Answer:**
FastAPI provides **TestClient** for testing, which uses the Starlette test client.

**Testing approaches:**
1. **Unit tests**: Test individual functions
2. **Integration tests**: Test API endpoints
3. **Database tests**: Test with test database
4. **Authentication tests**: Test protected routes
5. **Async tests**: Test async endpoints

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import pytest

app = FastAPI()

# Example endpoint
@app.post("/users/")
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    return create_user_in_db(db, user)

# Test setup
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
TestingSessionLocal = sessionmaker(bind=engine)

def override_get_db():
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

client = TestClient(app)

# Test fixtures
@pytest.fixture
def test_db():
    Base.metadata.create_all(bind=engine)
    yield
    Base.metadata.drop_all(bind=engine)

# Test cases
def test_create_user(test_db):
    response = client.post(
        "/users/",
        json={"email": "test@example.com", "username": "testuser"}
    )
    assert response.status_code == 200
    assert response.json()["email"] == "test@example.com"

def test_create_user_duplicate_email(test_db):
    # Create first user
    client.post("/users/", json={"email": "test@example.com", "username": "user1"})
    
    # Try to create duplicate
    response = client.post("/users/", json={"email": "test@example.com", "username": "user2"})
    assert response.status_code == 400

def test_get_user(test_db):
    # Create user
    create_response = client.post("/users/", json={"email": "test@example.com", "username": "testuser"})
    user_id = create_response.json()["id"]
    
    # Get user
    response = client.get(f"/users/{user_id}")
    assert response.status_code == 200
    assert response.json()["id"] == user_id

def test_authentication():
    # Test login
    response = client.post("/token", data={"username": "testuser", "password": "testpass"})
    assert response.status_code == 200
    token = response.json()["access_token"]
    
    # Test protected endpoint
    response = client.get("/users/me", headers={"Authorization": f"Bearer {token}"})
    assert response.status_code == 200
```

---

## Common Interview Questions

### Question 12: What are the main differences between FastAPI and Flask?

**Answer:**

| Feature | FastAPI | Flask |
|---------|---------|-------|
| **Performance** | Very fast (async) | Slower (sync) |
| **Type hints** | Built-in, required | Optional |
| **Validation** | Automatic (Pydantic) | Manual or extensions |
| **Documentation** | Automatic (Swagger/ReDoc) | Manual or extensions |
| **Async support** | Native | Limited (requires extensions) |
| **Learning curve** | Moderate | Easy |
| **Use case** | Modern APIs | General web apps |

```python
# FastAPI
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
async def create_item(item: Item):
    return item
# Automatic validation, documentation, async support

# Flask
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/items/", methods=["POST"])
def create_item():
    data = request.get_json()
    # Manual validation needed
    if "name" not in data or "price" not in data:
        return jsonify({"error": "Invalid data"}), 400
    return jsonify(data)
# No automatic validation or documentation
```

### Question 13: How does FastAPI handle request validation?

**Answer:**
FastAPI uses **Pydantic** models for automatic request validation:

1. **Type checking**: Validates data types
2. **Field validation**: Checks constraints (min, max, regex)
3. **Custom validators**: Custom validation logic
4. **Automatic errors**: Returns 422 with detailed errors

```python
from pydantic import BaseModel, Field, validator, EmailStr
from typing import Optional
from fastapi import FastAPI

app = FastAPI()

class User(BaseModel):
    username: str = Field(..., min_length=3, max_length=50, regex="^[a-zA-Z0-9_]+$")
    email: EmailStr
    age: int = Field(..., gt=0, lt=150)
    password: str = Field(..., min_length=8)
    
    @validator('password')
    def password_strength(cls, v):
        if not any(char.isdigit() for char in v):
            raise ValueError('Password must contain at least one digit')
        if not any(char.isupper() for char in v):
            raise ValueError('Password must contain at least one uppercase letter')
        return v

@app.post("/users/")
async def create_user(user: User):
    return user

# Invalid request:
# {"username": "ab", "email": "invalid", "age": -5, "password": "weak"}
# Response: 422 with detailed validation errors
```

### Question 14: Explain the difference between Depends() and Security() in FastAPI.

**Answer:**

**Depends():**
- General dependency injection
- Used for any reusable logic
- No special security handling
- Examples: database sessions, pagination, common parameters

**Security():**
- Specialized dependency for security
- Adds security information to OpenAPI docs
- Shows lock icon in Swagger UI
- Examples: API keys, OAuth2, JWT tokens

```python
from fastapi import Depends, Security, FastAPI
from fastapi.security import APIKeyHeader, OAuth2PasswordBearer

app = FastAPI()

# Depends - General dependency
def get_db():
    return {"db": "connection"}

@app.get("/items/")
async def get_items(db: dict = Depends(get_db)):
    return {"items": [], "db": db}

# Security - Security dependency
api_key_header = APIKeyHeader(name="X-API-Key")

def verify_api_key(api_key: str = Security(api_key_header)):
    if api_key != "secret-key":
        raise HTTPException(status_code=403, detail="Invalid API Key")
    return api_key

@app.get("/protected/")
async def protected_route(api_key: str = Security(verify_api_key)):
    return {"message": "Access granted"}

# OAuth2
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def get_current_user(token: str = Depends(oauth2_scheme)):
    # Verify token
    return {"user": "john"}

@app.get("/users/me")
async def read_users_me(current_user: dict = Depends(get_current_user)):
    return current_user
```

### Question 15: How do you handle file uploads in FastAPI?

**Answer:**

```python
from fastapi import FastAPI, File, UploadFile, Form
from typing import List
import shutil

app = FastAPI()

# Single file upload
@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": file.size
    }

# Multiple files upload
@app.post("/uploadfiles/")
async def upload_files(files: List[UploadFile] = File(...)):
    return {
        "filenames": [file.filename for file in files]
    }

# File with form data
@app.post("/upload-with-data/")
async def upload_with_data(
    file: UploadFile = File(...),
    description: str = Form(...),
    tags: List[str] = Form(...)
):
    return {
        "filename": file.filename,
        "description": description,
        "tags": tags
    }

# Save file to disk
@app.post("/save-file/")
async def save_file(file: UploadFile = File(...)):
    with open(f"uploads/{file.filename}", "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    return {"filename": file.filename, "status": "saved"}

# Read file content
@app.post("/process-file/")
async def process_file(file: UploadFile = File(...)):
    contents = await file.read()
    # Process contents
    return {"size": len(contents)}
```

### Question 16: What is CORS and how do you enable it in FastAPI?

**Answer:**
**CORS (Cross-Origin Resource Sharing)** allows web applications from one domain to access resources from another domain.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Configure CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",  # React dev server
        "https://example.com",     # Production frontend
    ],
    allow_credentials=True,  # Allow cookies
    allow_methods=["*"],     # Allow all methods (GET, POST, etc.)
    allow_headers=["*"],     # Allow all headers
)

# Or allow all origins (not recommended for production)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def main():
    return {"message": "Hello World"}

# Now frontend applications can make requests to this API
```

**CORS headers explained:**
- `allow_origins`: Which domains can access the API
- `allow_credentials`: Allow cookies and authentication
- `allow_methods`: Which HTTP methods are allowed
- `allow_headers`: Which headers can be sent

### Question 17: How do you handle errors and exceptions in FastAPI?

**Answer:**

```python
from fastapi import FastAPI, HTTPException, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

app = FastAPI()

# 1. HTTPException - Standard way
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in database:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "Custom header"}
        )
    return {"item_id": item_id}

# 2. Custom exception
class ItemNotFoundException(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

@app.exception_handler(ItemNotFoundException)
async def item_not_found_handler(request: Request, exc: ItemNotFoundException):
    return JSONResponse(
        status_code=404,
        content={"message": f"Item {exc.item_id} not found"}
    )

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in database:
        raise ItemNotFoundException(item_id)
    return {"item_id": item_id}

# 3. Override validation error handler
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={"detail": exc.errors(), "body": exc.body}
    )

# 4. Global exception handler
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={"message": "Internal server error"}
    )

# 5. Custom error response
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    error: str
    message: str
    status_code: int

@app.get("/items/{item_id}", responses={404: {"model": ErrorResponse}})
async def read_item(item_id: int):
    if item_id not in database:
        raise HTTPException(
            status_code=404,
            detail={"error": "NOT_FOUND", "message": "Item not found", "status_code": 404}
        )
    return {"item_id": item_id}
```

### Question 18: What are path operation configuration options in FastAPI?

**Answer:**

```python
from fastapi import FastAPI, status
from pydantic import BaseModel
from typing import List

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post(
    "/items/",
    response_model=Item,                    # Response schema
    status_code=status.HTTP_201_CREATED,    # Status code
    tags=["items"],                         # Group in docs
    summary="Create an item",               # Short description
    description="Create a new item with name and price",  # Long description
    response_description="The created item",  # Response description
    deprecated=False,                       # Mark as deprecated
    name="create_item",                     # Operation ID
    responses={                             # Additional responses
        404: {"description": "Not found"},
        400: {"description": "Bad request"}
    },
    include_in_schema=True                  # Include in OpenAPI schema
)
async def create_item(item: Item):
    """
    Create an item with all the information:
    
    - **name**: item name
    - **price**: item price
    """
    return item

# Alternative: Use docstring for description
@app.get("/items/", tags=["items"])
async def read_items():
    """
    Retrieve all items.
    
    This endpoint returns a list of all items in the database.
    """
    return {"items": []}
```

### Question 19: How do you implement pagination in FastAPI?

**Answer:**

```python
from fastapi import FastAPI, Depends, Query
from typing import List, Optional
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str
    price: float

class PaginatedResponse(BaseModel):
    total: int
    page: int
    page_size: int
    items: List[Item]

# Method 1: Simple pagination with query parameters
@app.get("/items/")
async def get_items(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100)
):
    items = get_items_from_db(skip=skip, limit=limit)
    return items

# Method 2: Page-based pagination
@app.get("/items/", response_model=PaginatedResponse)
async def get_items(
    page: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=100)
):
    skip = (page - 1) * page_size
    items = get_items_from_db(skip=skip, limit=page_size)
    total = get_total_items_count()
    
    return {
        "total": total,
        "page": page,
        "page_size": page_size,
        "items": items
    }

# Method 3: Pagination dependency
class Pagination:
    def __init__(
        self,
        page: int = Query(1, ge=1, description="Page number"),
        page_size: int = Query(10, ge=1, le=100, description="Items per page")
    ):
        self.page = page
        self.page_size = page_size
        self.skip = (page - 1) * page_size

@app.get("/items/")
async def get_items(pagination: Pagination = Depends()):
    items = get_items_from_db(skip=pagination.skip, limit=pagination.page_size)
    return items

# Method 4: Cursor-based pagination (for large datasets)
@app.get("/items/")
async def get_items(
    cursor: Optional[str] = None,
    limit: int = Query(10, ge=1, le=100)
):
    items, next_cursor = get_items_with_cursor(cursor=cursor, limit=limit)
    return {
        "items": items,
        "next_cursor": next_cursor
    }
```

### Question 20: How do you deploy a FastAPI application?

**Answer:**

**Deployment options:**

1. **Uvicorn (Development)**
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

2. **Gunicorn with Uvicorn workers (Production)**
```bash
gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

3. **Docker**
```dockerfile
# Dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
# Build and run
docker build -t fastapi-app .
docker run -p 8000:8000 fastapi-app
```

4. **Docker Compose**
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

5. **Cloud Platforms**
- **Heroku**: Use Procfile
- **AWS**: ECS, Lambda, Elastic Beanstalk
- **Google Cloud**: Cloud Run, App Engine
- **Azure**: App Service, Container Instances

**Production best practices:**
- Use environment variables for configuration
- Enable HTTPS
- Set up logging and monitoring
- Use a reverse proxy (Nginx)
- Implement rate limiting
- Use a process manager (systemd, supervisor)
- Set up health checks
- Use CDN for static files

---

## Quick Tips for FastAPI Interviews

1. **Understand async/await**: Know when to use async vs sync
2. **Pydantic models**: Master data validation and serialization
3. **Dependency injection**: Understand how it works and when to use it
4. **Security**: Know OAuth2, JWT, API keys
5. **Database integration**: Be familiar with SQLAlchemy
6. **Testing**: Know how to use TestClient
7. **Performance**: Understand why FastAPI is fast
8. **Documentation**: Leverage automatic API docs
9. **Error handling**: Proper exception handling
10. **Deployment**: Know production deployment strategies

---

## Practice Exercises

### Exercise 1: Build a Todo API
Create a FastAPI application with:
- CRUD operations for todos
- User authentication with JWT
- Database integration (SQLAlchemy)
- Pagination
- Filtering by status
- Unit tests

### Exercise 2: File Upload Service
Build a service that:
- Accepts file uploads
- Validates file types and sizes
- Stores files with unique names
- Returns download URLs
- Implements rate limiting

### Exercise 3: Real-time Chat
Create a WebSocket-based chat:
- Multiple chat rooms
- User authentication
- Message history
- Online user list
- Typing indicators

---

## Conclusion

This tutorial covers essential FastAPI concepts for interview preparation. FastAPI is a powerful, modern framework that combines speed, ease of use, and automatic documentation.

**Key Takeaways:**
- FastAPI is built on Starlette and Pydantic for high performance
- Type hints enable automatic validation and documentation
- Async/await support for concurrent operations
- Dependency injection for clean, reusable code
- Automatic OpenAPI documentation
- Easy testing with TestClient
- Production-ready with proper deployment strategies

Good luck with your FastAPI interview! 🚀

# Day 4: Pydantic + Validation + Request/Response Schemas + Production FastAPI Patterns

Pydantic is one of the most important parts of FastAPI.

It handles:

* Data validation
* Request parsing
* Type conversion
* Response formatting
* API contracts
* Environment settings

Without Pydantic:

```python
user = {
    "name": "KK",
    "age": "25"
}
```

Problem:

* `age` should be integer
* Missing fields possible
* Invalid emails possible
* Wrong data types possible

You manually write checks:

```python
if not isinstance(age, int):
    raise Exception("Invalid age")
```

That becomes messy.

---

# Basic Pydantic Model

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int


user = User(
    name="KK",
    age=25
)

print(user)
```

Output:

```text
name='KK' age=25
```

---

# Automatic Type Conversion

Pydantic tries to convert types.

```python
from pydantic import BaseModel

class User(BaseModel):
    name:str
    age:int


user=User(
    name="KK",
    age="25"
)

print(user)
```

Output:

```text
name='KK' age=25
```

String became integer automatically.

---

# Validation Error Example

```python
from pydantic import BaseModel

class User(BaseModel):
    age:int


user=User(age="hello")
```

Output:

```text
ValidationError:
Input should be a valid integer
```

---

# Required vs Optional Fields

Required:

```python
class User(BaseModel):
    name:str
```

Optional:

```python
from typing import Optional

class User(BaseModel):
    name:str
    phone: Optional[str]=None
```

Usage:

```python
user=User(name="KK")
```

Works because `phone` is optional.

---

# Field Validation

```python
from pydantic import BaseModel, Field

class User(BaseModel):

    username:str=Field(
        min_length=3,
        max_length=20
    )

    age:int=Field(
        gt=18,
        lt=100
    )
```

Usage:

```python
User(
    username="ab",
    age=10
)
```

Output:

```text
ValidationError
```

---

# Common validations

| Validation      | Meaning               |
| --------------- | --------------------- |
| `gt=10`         | greater than          |
| `lt=50`         | less than             |
| `ge=10`         | greater than or equal |
| `le=50`         | less than or equal    |
| `min_length=5`  | minimum characters    |
| `max_length=20` | maximum characters    |

---

# Email Validation

```python
from pydantic import BaseModel, EmailStr

class User(BaseModel):

    email:EmailStr
```

Usage:

```python
User(email="abc@gmail.com")
```

Valid:

```text
abc@gmail.com
```

Invalid:

```text
ValidationError
```

---

# Nested Models

Real APIs often contain nested data.

```python
from pydantic import BaseModel

class Address(BaseModel):

    city:str
    state:str


class User(BaseModel):

    name:str
    address:Address
```

Usage:

```python
user=User(
    name="KK",
    address={
        "city":"Hyderabad",
        "state":"AP"
    }
)

print(user)
```

---

# List Validation

```python
from typing import List
from pydantic import BaseModel

class User(BaseModel):

    skills:List[str]
```

Usage:

```python
User(
    skills=[
        "Python",
        "FastAPI",
        "Redis"
    ]
)
```

---

# FastAPI Request Schema

Without Pydantic:

```python
@app.post("/users")
async def create_user(data:dict):

    return data
```

Problem:

* No validation
* No API documentation
* Bad error messages

---

Correct way:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app=FastAPI()

class UserRequest(BaseModel):

    name:str
    age:int


@app.post("/users")
async def create_user(
    user:UserRequest
):
    return user
```

Request:

```json
{
    "name":"KK",
    "age":25
}
```

Response:

```json
{
    "name":"KK",
    "age":25
}
```

---

# Response Schema

Production APIs separate:

* Request model
* Response model

```python
class UserRequest(BaseModel):

    username:str
    password:str


class UserResponse(BaseModel):

    id:int
    username:str
```

API:

```python
@app.post(
    "/users",
    response_model=UserResponse
)
async def create_user(
    user:UserRequest
):

    return {
        "id":1,
        "username":user.username,
        "password":"secret"
    }
```

Response:

```json
{
    "id":1,
    "username":"KK"
}
```

Notice:

```text
password removed automatically
```

Very important in production systems.

---

# Production Folder Structure

```text
auth/

├── schemas/
│   ├── request.py
│   └── response.py
│
├── api/
│   └── routes.py
│
├── service/
│   └── auth_service.py
```

Example:

**request.py**

```python
class LoginRequest(BaseModel):
    email:EmailStr
    password:str
```

**response.py**

```python
class LoginResponse(BaseModel):
    token:str
    username:str
```

---

# Practice Exercise

Build these schemas:

### 1. Product schema

Fields:

```text
name
price
quantity
description(optional)
```

Rules:

* price > 0
* quantity >= 1

---

### 2. AI Chat Request

```json
{
 "message":"Hello",
 "temperature":0.7,
 "max_tokens":100
}
```

Rules:

* temperature: 0–1
* max_tokens: 1–1000

---

### 3. User Registration

Fields:

```text
username
email
password
skills
address
```

Add validations.

---

# Interview Focus

### Q1: What is Pydantic?

Pydantic is a Python library for data validation and parsing using type annotations.

---

### Q2: Why is Pydantic used in FastAPI?

* Request validation
* Response validation
* Automatic docs
* Type conversion
* Cleaner code

---

### Q3: Difference between request and response schemas?

Request schema:

```text
Accept incoming data
```

Response schema:

```text
Control outgoing data
```

---

### Q4: Why separate request and response models?

Reasons:

* Security
* Maintainability
* Cleaner API contracts

---

# Mini Project Task

Build:

```python
POST /chat
```

Request:

```json
{
    "message":"Tell me a joke",
    "temperature":0.8
}
```

Validate with Pydantic and return:

```json
{
    "response":"AI reply..."
}
```

Next lesson:

**Day 5 → FastAPI Dependency Injection + Middleware + how production APIs share DB, auth, Redis and services**

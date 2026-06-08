# Pydantic v2 — Beginner to Advanced (Interview Ready)

If AsyncIO is the heart of FastAPI, then **Pydantic is the backbone of FastAPI data validation**.

Almost every FastAPI application uses Pydantic.

It is also heavily used in:

* FastAPI
* LangChain
* OpenAI Structured Outputs
* AI Agents
* MCP Servers
* Tool Calling
* Data Validation
* Configuration Management

---

# 1. What is Pydantic?

Pydantic is a Python library for:

* Data Validation
* Data Parsing
* Data Serialization

using Python Type Hints.

Example:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
```

---

# 2. Why Do We Need Pydantic?

Without Pydantic:

```python
user = {
    "name": "John",
    "age": "abc"
}
```

You must manually validate:

```python
if not isinstance(user["age"], int):
    raise ValueError()
```

Imagine doing this for:

```text
100 APIs
50 fields
Thousands of requests
```

Not practical.

---

# 3. Basic Model

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
```

Usage:

```python
user = User(
    name="John",
    age=25
)

print(user)
```

Output:

```text
name='John' age=25
```

---

# 4. Validation

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

user = User(
    name="John",
    age="abc"
)
```

Output:

```text
ValidationError
```

Pydantic validates automatically.

---

# 5. Automatic Type Conversion

```python
user = User(
    name="John",
    age="25"
)
```

Output:

```python
User(
    name='John',
    age=25
)
```

Notice:

```text
"25" → 25
```

Pydantic parsed it automatically.

---

# 6. model_dump()

Pydantic v2 replacement for:

```python
.dict()
```

Use:

```python
user.model_dump()
```

Example:

```python
user = User(
    name="John",
    age=25
)

print(user.model_dump())
```

Output:

```python
{
    "name": "John",
    "age": 25
}
```

---

# 7. model_dump_json()

Convert to JSON.

```python
user.model_dump_json()
```

Output:

```json
{"name":"John","age":25}
```

---

# 8. model_validate()

Create model from data.

```python
data = {
    "name": "John",
    "age": 25
}

user = User.model_validate(data)
```

Very common in AI applications.

---

# 9. Field()

Adds metadata and validation.

```python
from pydantic import BaseModel, Field

class User(BaseModel):

    name: str = Field(
        min_length=3,
        max_length=50
    )

    age: int = Field(
        ge=18,
        le=100
    )
```

---

# 10. Validation Rules

## Minimum Length

```python
name: str = Field(min_length=3)
```

---

## Maximum Length

```python
name: str = Field(max_length=50)
```

---

## Greater Than

```python
age: int = Field(gt=18)
```

---

## Greater Than Equal

```python
age: int = Field(ge=18)
```

---

## Less Than

```python
age: int = Field(lt=100)
```

---

## Less Than Equal

```python
age: int = Field(le=100)
```

---

# 11. Optional Fields

```python
class User(BaseModel):

    name: str

    email: str | None = None
```

Valid:

```python
User(name="John")
```

Also valid:

```python
User(
    name="John",
    email="john@gmail.com"
)
```

---

# 12. Default Values

```python
class User(BaseModel):

    is_active: bool = True
```

Usage:

```python
User()
```

Output:

```python
is_active=True
```

---

# 13. Nested Models

Real APIs contain nested objects.

Example:

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    state: str

class User(BaseModel):
    name: str
    address: Address
```

Usage:

```python
user = User(
    name="John",
    address={
        "city": "Hyderabad",
        "state": "Telangana"
    }
)
```

Pydantic automatically converts.

---

# 14. List Validation

```python
class User(BaseModel):

    skills: list[str]
```

Valid:

```python
{
    "skills": [
        "Python",
        "FastAPI"
    ]
}
```

Invalid:

```python
{
    "skills": "Python"
}
```

---

# 15. Dictionary Validation

```python
class Config(BaseModel):

    settings: dict[str, str]
```

---

# 16. Email Validation

Install:

```bash
pip install email-validator
```

Use:

```python
from pydantic import EmailStr

class User(BaseModel):

    email: EmailStr
```

Valid:

```text
john@gmail.com
```

Invalid:

```text
abc
```

---

# 17. Custom Validators (v2)

Pydantic v2 uses:

```python
@field_validator
```

Example:

```python
from pydantic import (
    BaseModel,
    field_validator
)

class User(BaseModel):

    name: str

    @field_validator("name")
    @classmethod
    def validate_name(cls, value):

        if len(value) < 3:
            raise ValueError(
                "Name too short"
            )

        return value
```

---

# 18. Multiple Field Validation

```python
from pydantic import model_validator

class User(BaseModel):

    password: str
    confirm_password: str

    @model_validator(mode="after")
    def validate_passwords(self):

        if self.password != self.confirm_password:
            raise ValueError(
                "Passwords mismatch"
            )

        return self
```

---

# 19. FastAPI Integration

Request Model:

```python
from pydantic import BaseModel

class UserRequest(BaseModel):

    name: str
    age: int
```

Endpoint:

```python
@app.post("/users")
async def create_user(
    user: UserRequest
):
    return user
```

FastAPI automatically:

* Validates request
* Converts types
* Generates docs

---

# 20. Response Models

```python
class UserResponse(BaseModel):

    id: int
    name: str
```

Usage:

```python
@app.get(
    "/users/{id}",
    response_model=UserResponse
)
async def get_user():
    ...
```

Ensures response structure.

---

# 21. Pydantic Settings

Used for environment variables.

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):

    db_host: str
    db_port: int
```

Reads:

```env
DB_HOST=localhost
DB_PORT=5432
```

Very common in production.

---

# 22. Generative AI Use Case

Structured LLM Output.

Example:

```python
from pydantic import BaseModel

class ResumeAnalysis(BaseModel):

    skills: list[str]
    score: float
    experience: int
```

LLM response:

```python
result = ResumeAnalysis.model_validate(
    llm_response
)
```

Guarantees structure.

---

# 23. Agentic AI Use Case

Tool Output Validation.

```python
class WeatherResult(BaseModel):

    city: str
    temperature: float
```

Agent tool must return:

```python
{
    "city": "Hyderabad",
    "temperature": 34.5
}
```

Validation prevents bad outputs.

---

# 24. Pydantic v1 vs v2

| Pydantic v1   | Pydantic v2          |
| ------------- | -------------------- |
| `.dict()`     | `.model_dump()`      |
| `.json()`     | `.model_dump_json()` |
| `parse_obj()` | `model_validate()`   |
| `@validator`  | `@field_validator`   |
| Slower        | Faster               |

Interviewers often ask this.

---

# Common Interview Questions

### Q1. What is Pydantic?

**Answer**

Pydantic is a data validation and parsing library that uses Python type annotations to enforce structured data models.

---

### Q2. Why Does FastAPI Use Pydantic?

**Answer**

FastAPI uses Pydantic for request validation, response validation, automatic type conversion, and OpenAPI schema generation.

---

### Q3. Difference Between model_dump() and model_validate()?

| Method           | Purpose                    |
| ---------------- | -------------------------- |
| model_dump()     | Convert model → dictionary |
| model_validate() | Convert dictionary → model |

---

### Q4. What Replaced @validator in v2?

Answer:

```python
@field_validator
```

---

### Q5. What Replaced .dict() in v2?

Answer:

```python
model_dump()
```

---

# FastAPI Interview Example

```python
from pydantic import BaseModel, Field

class CreateUserRequest(BaseModel):

    name: str = Field(
        min_length=3,
        max_length=50
    )

    age: int = Field(
        ge=18,
        le=100
    )

    email: EmailStr
```

An interviewer sees this and immediately knows you understand:

* Type Hints
* Validation
* Constraints
* FastAPI Design
* Pydantic

---

# Interview Summary

```text
Pydantic
    ↓
Data Validation Library

Uses
    ↓
Type Hints

Core Concepts
    ↓
BaseModel
Field
Validation
Nested Models
Custom Validators

Important v2 APIs
    ↓
model_validate()
model_dump()
model_dump_json()
field_validator()
model_validator()

Used In
    ↓
FastAPI
GenAI
Agentic AI
Tool Calling
Configuration Management
```

## Master These Before Moving On

* BaseModel
* Field
* model_dump()
* model_validate()
* Nested Models
* field_validator()
* model_validator()
* Response Models
* BaseSettings

After Pydantic, the next topic should be **Context Managers**, because they connect directly to database connections, file handling, HTTP clients, and resource management patterns you'll use in FastAPI and AI applications.

---

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

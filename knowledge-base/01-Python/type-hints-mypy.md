# Type Hints + mypy — Beginner to Advanced (Interview Ready)

This topic is extremely important because modern Python frameworks depend on it:

* FastAPI
* Pydantic
* LangChain
* OpenAI SDKs
* Agent Frameworks
* Enterprise Python Codebases

Without Type Hints:

```python
def process(data):
    return data
```

Questions:

```text
What is data?
string?
list?
dict?
User object?
```

Nobody knows.

---

# 1. What Are Type Hints?

Type hints specify the expected type of variables, parameters, and return values.

Example:

```python
def add(a: int, b: int) -> int:
    return a + b
```

Meaning:

```text
a must be int
b must be int
returns int
```

---

# 2. Why Type Hints Matter

Without type hints:

```python
def send_email(user):
    ...
```

You don't know:

```text
string?
dict?
User object?
```

---

With type hints:

```python
def send_email(user: str) -> None:
    ...
```

Immediately understandable.

Benefits:

* Better readability
* Better IDE support
* Better refactoring
* Better documentation
* Static error detection

---

# 3. Basic Type Hints

## String

```python
name: str = "John"
```

---

## Integer

```python
age: int = 25
```

---

## Float

```python
salary: float = 1000.5
```

---

## Boolean

```python
is_active: bool = True
```

---

# 4. Function Type Hints

Without:

```python
def multiply(a, b):
    return a * b
```

---

With:

```python
def multiply(a: int, b: int) -> int:
    return a * b
```

---

# 5. Type Hints Don't Enforce Types

Many beginners think:

```python
def add(a: int, b: int) -> int:
    return a + b
```

will stop this:

```python
add("10", "20")
```

It won't.

Python still runs.

Type hints are mainly:

```text
Documentation
IDE Support
Static Analysis
```

---

# 6. List Type

Old style:

```python
from typing import List

users: List[str]
```

---

Modern Python (3.9+):

```python
users: list[str]
```

Example:

```python
users: list[str] = [
    "John",
    "Mike"
]
```

---

# 7. Dictionary Type

```python
user: dict[str, str]
```

Example:

```python
user: dict[str, str] = {
    "name": "John",
    "city": "Hyderabad"
}
```

---

# 8. Tuple Type

```python
coordinates: tuple[int, int]
```

Example:

```python
point = (10, 20)
```

---

# 9. Set Type

```python
skills: set[str]
```

Example:

```python
skills = {"Python", "FastAPI"}
```

---

# 10. Optional Type

Sometimes value may be missing.

```python
name: str | None
```

Equivalent:

```python
from typing import Optional

name: Optional[str]
```

---

Example:

```python
def get_user(user_id: int) -> str | None:
    ...
```

Meaning:

```text
Returns string
OR
Returns None
```

---

# 11. Union Types

Multiple valid types.

Modern:

```python
value: int | str
```

Older:

```python
from typing import Union

value: Union[int, str]
```

---

Example:

```python
def process(value: int | str):
    ...
```

---

# 12. Any Type

```python
from typing import Any

data: Any
```

Means:

```text
Accept anything
```

Avoid excessive use.

Interviewers may ask:

> Why is Any dangerous?

Answer:

```text
Removes type safety.
Defeats purpose of type hints.
```

---

# 13. Type Alias

Without:

```python
dict[str, list[str]]
```

Hard to read.

---

Better:

```python
UserSkills = dict[str, list[str]]
```

Usage:

```python
skills: UserSkills
```

---

# 14. Typed Dictionaries

Normal dictionary:

```python
user = {
    "name": "John",
    "age": 25
}
```

No structure guarantee.

---

Using TypedDict:

```python
from typing import TypedDict

class User(TypedDict):
    name: str
    age: int
```

Usage:

```python
user: User = {
    "name": "John",
    "age": 25
}
```

---

# 15. Type Hints for Classes

```python
class User:

    def __init__(
        self,
        name: str,
        age: int
    ):
        self.name = name
        self.age = age
```

---

# 16. Generic Types

Example:

```python
def get_first(items: list[str]) -> str:
    return items[0]
```

Only works for strings.

---

Generics:

```python
from typing import TypeVar

T = TypeVar("T")

def get_first(items: list[T]) -> T:
    return items[0]
```

Works for:

```python
list[str]
list[int]
list[User]
```

---

# 17. Type Hints in FastAPI

Example:

```python
@app.get("/users/{id}")
async def get_user(
    id: int
) -> dict:
    ...
```

Benefits:

* Validation
* Documentation
* OpenAPI generation

FastAPI heavily relies on type hints.

---

# 18. Type Hints in Pydantic

```python
from pydantic import BaseModel

class User(BaseModel):

    name: str
    age: int
```

Pydantic reads type hints and validates automatically.

---

# 19. Type Hints in AI Systems

Structured LLM outputs:

```python
class Resume(BaseModel):

    skills: list[str]
    experience: int
```

AI response:

```python
response = Resume.model_validate(data)
```

Type hints drive validation.

---

# 20. What is mypy?

Type hints alone don't catch errors.

Example:

```python
def add(a: int, b: int) -> int:
    return a + b

add("10", "20")
```

Python runs.

---

mypy checks before runtime.

Installation:

```bash
pip install mypy
```

Run:

```bash
mypy app.py
```

---

# 21. mypy Example

Code:

```python
def add(a: int, b: int) -> int:
    return a + b

add("10", "20")
```

mypy:

```text
Argument 1 has incompatible type "str"
Argument 2 has incompatible type "str"
```

Found before production.

---

# 22. Common mypy Errors

## Wrong Return Type

```python
def get_age() -> int:
    return "25"
```

mypy:

```text
Expected int
Got str
```

---

## Missing Return

```python
def get_user() -> str:

    if True:
        return "John"
```

mypy warns:

```text
Missing return statement
```

---

# 23. Why Companies Use mypy

Benefits:

```text
Catch bugs early
Safer refactoring
Better maintainability
Enterprise reliability
```

Large companies often enforce:

```bash
mypy .
```

inside CI/CD pipelines.

---

# 24. Best Practices

### Good

```python
def create_user(
    name: str,
    age: int
) -> User:
    ...
```

---

### Avoid

```python
def create_user(name, age):
    ...
```

---

### Avoid Excessive Any

```python
data: Any
```

Only when necessary.

---

### Prefer Modern Syntax

Use:

```python
str | None
```

instead of:

```python
Optional[str]
```

for modern Python.

---

# Common Interview Questions

## Q1. What are Type Hints?

**Answer**

Type hints are annotations that specify expected parameter, variable, and return types, improving readability and enabling static analysis.

---

## Q2. Do Type Hints Enforce Types?

**Answer**

No.

Python does not enforce them at runtime.

Tools like mypy use them for static checking.

---

## Q3. What is mypy?

**Answer**

mypy is a static type checker that analyzes Python code using type annotations and detects type-related issues before runtime.

---

## Q4. Why Are Type Hints Important in FastAPI?

**Answer**

FastAPI uses type hints for:

* Request validation
* Response validation
* OpenAPI documentation
* Dependency injection

---

## Q5. Difference Between Any and Union?

**Answer**

```python
Any
```

Accepts everything.

```python
Union[int, str]
```

Accepts only specified types.

---

# FastAPI + GenAI Examples You Should Remember

### FastAPI Endpoint

```python
@app.get("/users/{id}")
async def get_user(
    id: int
) -> dict:
    ...
```

---

### Pydantic Model

```python
class User(BaseModel):
    name: str
    age: int
```

---

### LLM Structured Output

```python
class JobMatch(BaseModel):
    score: float
    skills: list[str]
```

---

# Interview Summary

```text
Type Hints
    ↓
Describe expected types

Benefits
    ↓
Readability
IDE Support
Static Checking
Documentation

mypy
    ↓
Uses type hints
Finds bugs before runtime

Used Heavily In
    ↓
FastAPI
Pydantic
LangChain
OpenAI SDKs
Agentic AI Systems
```

After mastering Type Hints + mypy, the next topic should be **Pydantic v2**, because FastAPI, GenAI structured outputs, tool calling, agent responses, and production APIs all depend heavily on Pydantic models and validation.

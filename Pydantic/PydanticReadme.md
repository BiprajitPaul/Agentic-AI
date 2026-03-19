# Pydantic Data Validation - Learning Guide

This guide covers the essential concepts of Pydantic data validation learned through practical examples.

## Table of Contents
- [What is Pydantic?](#what-is-pydantic)
- [Pydantic vs Dataclasses](#pydantic-vs-dataclasses)
- [Basic Models](#basic-models)
- [Type Validation](#type-validation)
- [Optional Fields & Defaults](#optional-fields--defaults)
- [Collections & Type Hints](#collections--type-hints)
- [Nested Models](#nested-models)
- [Field Constraints](#field-constraints)
- [JSON Schema Generation](#json-schema-generation)

---

## What is Pydantic?

Pydantic is a Python library that provides data validation and settings management using Python type annotations. It enforces type hints at runtime and provides clear error messages for validation failures.

### Installation
```python
from pydantic import BaseModel, Field
```

---

## Pydantic vs Dataclasses

### Dataclasses (No Validation)
```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    city: str

# Creates instance without validation - even with invalid data!
person = Person(name="Tuhin", age=23, city=12)  # No error
```

### Pydantic Models (With Validation)
```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    city: str

# Validates data and raises ValidationError for invalid types
person = Person(name="Tuhin", age=23, city="Kumarghat")  # Valid ✓
person1 = Person(name="Tuhin", age=23, city=12)  # ValidationError ✗
```

**Key Difference:** Pydantic validates all data when creating instances, while dataclasses do not. With Pydantic, passing `city=12` (an integer) raises:
```
ValidationError: 1 validation error for Person
city
  Input should be a valid string [type=string_type, input_value=12, input_type=int]
```

---

## Basic Models

Create a basic Pydantic model by inheriting from `BaseModel`:

```python
class Person(BaseModel):
    name: str
    age: int
    city: str

person = Person(name="Tuhin", age=23, city="Kumarghat")
print(person)  # name='Tuhin' age=23 city='Kumarghat'
```

---

## Type Validation

Pydantic automatically validates field types when creating model instances.

```python
class Person(BaseModel):
    name: str
    age: int
    city: str

# Valid - all types match
person = Person(name="Tuhin", age=23, city="Kumarghat")  # ✓

# Invalid - city should be string, not int
try:
    person1 = Person(name="Tuhin", age=23, city=12)
except ValidationError as e:
    print(f"Validation error: {e}")
```

**Tip:** Invalid data raises a `ValidationError` with detailed information about what went wrong.

---

## Optional Fields & Defaults

Use `Optional` to allow fields to be `None`, and provide default values:

```python
from typing import Optional

class Employee(BaseModel):
    id: int
    name: str
    department: str
    salary: Optional[float] = None      # Can be None, defaults to None
    is_active: bool = True              # Defaults to True

# Can be created without optional fields
emp = Employee(id=1, name="Alice", department="HR")
# Result: id=1 name='Alice' department='HR' salary=None is_active=True
```

**Key Points:**
- `Optional[float] = None` - Field can be `None` or a float
- `bool = True` - Default value is `True` if not provided
- Required fields must always be provided

---

## Collections & Type Hints

Use type hints like `List` to validate collections of data:

```python
from typing import List

class Classroom(BaseModel):
    name: str
    students: List[str]  # List of strings
    capacity: int

# Valid - list of strings
classroom = Classroom(
    name="Math 101",
    students=["Alice", "Bob", "Charlie"],
    capacity=30
)

# Tuples are accepted and converted to lists
classroom2 = Classroom(
    name="Math 102",
    students=("Tuhin", "Sagar", "Trisit"),  # Tuple
    capacity=30
)
# Result: students=['Tuhin', 'Sagar', 'Trisit']  (converted to list)

# Invalid - list contains an integer
try:
    invalid = Classroom(
        name="Math101",
        students=["Alice", 123],  # 123 is not a string
        capacity=30
    )
except ValidationError as e:
    print(f"Validation error: {e}")
```

**Key Points:**
- `List[str]` ensures all items are strings
- Tuples are automatically converted to lists
- Invalid item types raise `ValidationError`

---

## Nested Models

Create models that contain other models:

```python
class Address(BaseModel):
    street: str
    city: str
    state: str
    zip_code: int

class Customer(BaseModel):
    name: str
    email: str
    address: Address  # Nested model

customer = Customer(
    name="John Doe",
    email="john.doe@example.com",
    address=Address(
        street="123 Main St",
        city="Anytown",
        state="CA",
        zip_code="799290"  # Will be converted to int
    )
)

print(customer)
# name='John Doe' email='john.doe@example.com'
# address=Address(street='123 Main St', city='Anytown', state='CA', zip_code=799290)
```

**Key Points:**
- Nested models provide structured, reusable data organization
- Validation applies to nested models recursively
- Type coercion works in nested models (string "799290" → int 799290)

---

## Field Constraints

Use the `Field` function to add validation constraints to specific fields:

```python
from pydantic import Field

class Item(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    price: float = Field(gt=0, le=1000)      # gt=greater than, le=less than or equal
    quantity: int = Field(ge=1)              # ge=greater than or equal

item = Item(name="Laptop", price=23, quantity=1)
print(item)  # name='Laptop' price=23.0 quantity=1
```

### Common Field Constraints:
- `min_length` / `max_length` - String length constraints
- `gt` (greater than) / `ge` (greater than or equal)
- `lt` (less than) / `le` (less than or equal)

---

## JSON Schema Generation

Generate a JSON schema from your model for API documentation:

```python
class Item(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    price: float = Field(gt=0, le=1000)
    quantity: int = Field(ge=1)

schema = Item.model_json_schema()
print(schema)

# Output:
# {
#   'properties': {
#     'name': {
#       'maxLength': 50,
#       'minLength': 2,
#       'title': 'Name',
#       'type': 'string'
#     },
#     'price': {
#       'exclusiveMinimum': 0,
#       'maximum': 1000,
#       'title': 'Price',
#       'type': 'number'
#     },
#     'quantity': {
#       'minimum': 1,
#       'title': 'Quantity',
#       'type': 'integer'
#     }
#   },
#   'required': ['name', 'price', 'quantity'],
#   'title': 'Item',
#   'type': 'object'
# }
```

**Use Cases:**
- API documentation (OpenAPI/Swagger)
- Validation in other systems
- Client-side form generation

---

## Summary

Pydantic provides:
✓ **Type validation** at runtime
✓ **Clear error messages** for invalid data
✓ **Type coercion** (converts "123" to 123)
✓ **Optional fields** and defaults
✓ **Nested models** for complex structures
✓ **Field constraints** for business logic
✓ **JSON schema** generation

This makes it ideal for:
- API development (with FastAPI)
- Data validation in applications
- Configuration management
- Database model definition

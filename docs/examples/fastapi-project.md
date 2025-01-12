# FastAPI Project Example

This example demonstrates how to use CursorRules in a FastAPI project.

## Project Setup

1. Create project directory structure:
```
/my-fastapi-project
  /.cursorrules           # Directory for rule files
    project.cursorrules   # Core project rules
    backend.cursorrules   # API-specific rules
  /src
    /api
      /v1
        /__init__.py
        /endpoints
        /models
        /schemas
    /core
      /__init__.py
      /config.py
      /security.py
    /db
      /__init__.py
      /models
      /repositories
    /services
      /__init__.py
      /business_logic
  /tests
  /alembic              # Database migrations
  pyproject.toml
  requirements.txt
```

2. Copy relevant rules:
```bash
# Copy core rules
cp rules/core/project.cursorrules .cursorrules/
cp rules/core/code_quality.cursorrules .cursorrules/

# Copy backend rules
cp rules/backend/backend.cursorrules .cursorrules/

# Copy testing rules
cp rules/testing/testing.cursorrules .cursorrules/
```

## Rule Customization

### Project Rules
```
// project.cursorrules
// Customize for FastAPI project

// Technology Stack
- Primary language: Python 3.11+
- Framework: FastAPI
- Database: PostgreSQL + SQLAlchemy
- Authentication: JWT + OAuth2
- Documentation: OpenAPI/Swagger

// Project Structure
/src
  /api                 # API endpoints
    /v1               # API version
      /endpoints     # Route handlers
      /models       # Pydantic models
      /schemas      # Response schemas
  /core              # Core functionality
    /config         # Configuration
    /security      # Auth & security
  /db                # Database
    /models        # SQLAlchemy models
    /repositories  # Data access
  /services          # Business logic
```

### Backend Rules
```
// backend.cursorrules
// FastAPI-specific patterns

// Endpoint Structure
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

router = APIRouter()

@router.get("/{item_id}", response_model=schemas.ItemResponse)
async def read_item(
    item_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    # Authorization
    if not current_user.can_access_item(item_id):
        raise HTTPException(status_code=403, detail="Not authorized")
    
    # Business logic
    item = await service.get_item(db, item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    
    # Return response
    return item

// Error Handling
class AppException(Exception):
    def __init__(
        self,
        status_code: int,
        detail: str,
        internal_code: str | None = None
    ):
        self.status_code = status_code
        self.detail = detail
        self.internal_code = internal_code
```

## Implementation Examples

### Model Example
```python
# src/db/models/user.py
from sqlalchemy import Column, Integer, String, Boolean
from sqlalchemy.orm import relationship

from ..base import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)

    items = relationship("Item", back_populates="owner")

    def verify_password(self, password: str) -> bool:
        return verify_password(self.hashed_password, password)

    def can_access_item(self, item_id: int) -> bool:
        return self.is_superuser or any(item.id == item_id for item in self.items)
```

### Repository Example
```python
# src/db/repositories/base.py
from typing import Generic, TypeVar, Type
from sqlalchemy.orm import Session
from fastapi.encoders import jsonable_encoder

from ..base import Base

ModelType = TypeVar("ModelType", bound=Base)

class BaseRepository(Generic[ModelType]):
    def __init__(self, model: Type[ModelType]):
        self.model = model

    async def get(self, db: Session, id: int) -> ModelType | None:
        return db.query(self.model).filter(self.model.id == id).first()

    async def get_multi(
        self,
        db: Session,
        *,
        skip: int = 0,
        limit: int = 100
    ) -> list[ModelType]:
        return db.query(self.model).offset(skip).limit(limit).all()

    async def create(self, db: Session, *, obj_in: dict) -> ModelType:
        obj_data = jsonable_encoder(obj_in)
        db_obj = self.model(**obj_data)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj
```

### Service Example
```python
# src/services/business_logic/item_service.py
from typing import Optional
from sqlalchemy.orm import Session

from ...db.repositories import ItemRepository
from ...db.models import Item, User

class ItemService:
    def __init__(self):
        self.repository = ItemRepository()

    async def get_item(
        self,
        db: Session,
        item_id: int,
        current_user: Optional[User] = None
    ) -> Optional[Item]:
        item = await self.repository.get(db, item_id)
        if not item:
            return None
        
        if current_user and not current_user.can_access_item(item_id):
            return None
            
        return item

    async def create_item(
        self,
        db: Session,
        *,
        item_data: dict,
        owner_id: int
    ) -> Item:
        item_data["owner_id"] = owner_id
        return await self.repository.create(db, obj_in=item_data)
```

### Testing Example
```python
# tests/api/test_items.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy.orm import Session

from app.main import app
from app.db.models import Item, User

def test_create_item(
    client: TestClient,
    db: Session,
    normal_user_token_headers: dict[str, str]
):
    data = {"title": "Test Item", "description": "test description"}
    response = client.post(
        "/api/v1/items/",
        headers=normal_user_token_headers,
        json=data,
    )
    assert response.status_code == 200
    content = response.json()
    assert content["title"] == data["title"]
    assert content["description"] == data["description"]
    assert "id" in content
    assert "owner_id" in content
```

## Best Practices

1. Follow FastAPI patterns:
   - Use dependency injection
   - Implement proper validation
   - Use async where beneficial
   - Handle errors properly

2. Database practices:
   - Use migrations
   - Implement repositories
   - Handle transactions
   - Optimize queries

3. Security considerations:
   - Validate input data
   - Implement proper auth
   - Use rate limiting
   - Handle sensitive data

4. Testing strategy:
   - Use pytest fixtures
   - Mock external services
   - Test error cases
   - Check edge cases 
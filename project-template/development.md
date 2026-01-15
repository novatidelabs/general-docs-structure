# Development

**Version:** 1.0  
**Last Updated:** [Date]  
**Status:** [Draft | Review | Approved]

---

## Table of Contents
1. [Setup Guide](#1-setup-guide)
2. [Coding Standards](#2-coding-standards)
3. [Testing Guide](#3-testing-guide)

---

## 1. Setup Guide

### 1.1 Prerequisites

**Required:**
- **Python:** 3.11 or higher
- **PostgreSQL:** 15 or higher
- **Redis:** 7 or higher
- **Docker:** 20.10 or higher (recommended)
- **Git:** Latest version

**Recommended:**
- **IDE:** VS Code or PyCharm
- **Terminal:** [iTerm2 (Mac) | Windows Terminal | Terminator (Linux)]
- **API Client:** Postman or HTTPie

### 1.2 Local Development Setup

#### Option 1: Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/your-org/your-project.git
cd your-project

# Copy environment variables
cp .env.example .env

# Edit .env with your local settings
nano .env

# Start all services with Docker Compose
docker-compose up -d

# Verify services are running
docker-compose ps

# View logs
docker-compose logs -f backend

# Run database migrations
docker-compose exec backend alembic upgrade head

# Seed database with test data
docker-compose exec backend python scripts/seed_database.py

# Access the application
# API: http://localhost:8000
# Docs: http://localhost:8000/docs
# Admin: http://localhost:3000
```

#### Option 2: Local Installation

```bash
# Clone the repository
git clone https://github.com/your-org/your-project.git
cd your-project

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Mac/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Install pre-commit hooks
pre-commit install

# Copy environment variables
cp .env.example .env

# Start PostgreSQL (if not running)
brew services start postgresql@15  # Mac
sudo systemctl start postgresql    # Linux

# Start Redis (if not running)
brew services start redis          # Mac
sudo systemctl start redis         # Linux

# Create database
createdb myapp_dev

# Run migrations
alembic upgrade head

# Seed database
python scripts/seed_database.py

# Start development server
uvicorn main:app --reload --port 8000

# In another terminal, start worker (if needed)
celery -A tasks worker --loglevel=info
```

### 1.3 Project Structure

```
project-root/
├── src/                          # Source code
│   ├── api/                      # API routes
│   │   ├── __init__.py
│   │   ├── v1/                   # API version 1
│   │   │   ├── __init__.py
│   │   │   ├── users.py          # User endpoints
│   │   │   └── auth.py           # Auth endpoints
│   │   └── dependencies.py       # Shared dependencies
│   ├── core/                     # Core functionality
│   │   ├── config.py             # Configuration
│   │   ├── security.py           # Security utilities
│   │   └── database.py           # Database connection
│   ├── models/                   # Database models
│   │   ├── __init__.py
│   │   └── user.py               # User model
│   ├── schemas/                  # Pydantic schemas
│   │   ├── __init__.py
│   │   └── user.py               # User schemas
│   ├── services/                 # Business logic
│   │   ├── __init__.py
│   │   └── user_service.py       # User service
│   ├── repositories/             # Data access layer
│   │   ├── __init__.py
│   │   └── user_repository.py    # User repository
│   └── utils/                    # Utility functions
│       ├── __init__.py
│       └── helpers.py
├── tests/                        # Test suite
│   ├── unit/                     # Unit tests
│   ├── integration/              # Integration tests
│   ├── e2e/                      # End-to-end tests
│   └── conftest.py               # Test configuration
├── migrations/                   # Database migrations
│   └── versions/
├── scripts/                      # Utility scripts
│   ├── seed_database.py
│   └── clean_database.py
├── docs/                         # Documentation
├── .env.example                  # Example environment variables
├── .gitignore
├── docker-compose.yml
├── Dockerfile
├── requirements.txt              # Production dependencies
├── requirements-dev.txt          # Development dependencies
├── pyproject.toml                # Project configuration
├── pytest.ini                    # Pytest configuration
└── README.md
```

### 1.4 Configuration

**Environment Variables:**

Create `.env` file in project root:

```bash
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp_dev

# Redis
REDIS_URL=redis://localhost:6379/0

# Security
SECRET_KEY=dev-secret-key-CHANGE-IN-PRODUCTION
JWT_ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=60

# External APIs
EXTERNAL_API_KEY=your-dev-api-key
EXTERNAL_API_URL=https://api.example.com

# Logging
LOG_LEVEL=DEBUG

# Environment
ENVIRONMENT=development

# CORS (for local development)
CORS_ORIGINS=["http://localhost:3000", "http://localhost:8000"]
```

### 1.5 Development Tools

**Recommended VS Code Extensions:**
- Python (Microsoft)
- Pylance
- Python Test Explorer
- Docker
- GitLens
- REST Client
- YAML
- Markdown All in One

**VS Code Settings (`.vscode/settings.json`):**
```json
{
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": false,
  "python.linting.flake8Enabled": true,
  "python.linting.mypyEnabled": true,
  "python.formatting.provider": "black",
  "python.formatting.blackArgs": ["--line-length", "100"],
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false
}
```

### 1.6 Common Development Tasks

**Run Tests:**
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/unit/test_user_service.py

# Run with coverage
pytest --cov=src --cov-report=html

# Run tests in watch mode
pytest-watch
```

**Code Quality Checks:**
```bash
# Format code
black src/ tests/

# Check imports
isort src/ tests/

# Lint code
flake8 src/ tests/

# Type checking
mypy src/

# Run all checks (as in pre-commit)
pre-commit run --all-files
```

**Database Operations:**
```bash
# Create new migration
alembic revision -m "description"

# Apply migrations
alembic upgrade head

# Rollback migration
alembic downgrade -1

# View migration history
alembic history

# Reset database (development only!)
python scripts/clean_database.py
alembic upgrade head
python scripts/seed_database.py
```

**Build and Run Docker:**
```bash
# Build image
docker build -t myapp:latest .

# Run container
docker run -p 8000:8000 --env-file .env myapp:latest

# Build and run with docker-compose
docker-compose up --build

# Stop services
docker-compose down

# Clean up (removes volumes)
docker-compose down -v
```

---

## 2. Coding Standards

### 2.1 Python Code Style

**Style Guide:** PEP 8 + Black formatter

**Key Principles:**
1. **KISS (Keep It Simple, Stupid):** Prefer simple, clear solutions over clever ones
2. **DRY (Don't Repeat Yourself):** Extract common logic into reusable functions
3. **SOLID Principles:** Follow SOLID design principles
4. **Explicit over Implicit:** Code should be self-documenting
5. **Zero Technical Debt:** Production code must be clean, maintainable, and well-tested

### 2.2 Code Formatting

**Formatter:** Black (line length: 100)

```python
# Good: Clear, formatted, readable
def get_user_by_email(email: str) -> Optional[User]:
    """
    Retrieve a user by their email address.
    
    Args:
        email: The user's email address
        
    Returns:
        User object if found, None otherwise
    """
    return db.query(User).filter(User.email == email).first()


# Bad: Poor formatting, no type hints, no docstring
def get_user(e):
    return db.query(User).filter(User.email==e).first()
```

### 2.3 Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| **Variables** | snake_case | `user_id`, `api_key` |
| **Functions** | snake_case | `get_user()`, `create_order()` |
| **Classes** | PascalCase | `UserService`, `OrderRepository` |
| **Constants** | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_VERSION` |
| **Private** | _leading_underscore | `_internal_function()` |
| **Files** | snake_case | `user_service.py`, `api_client.py` |

**Naming Guidelines:**
- Use descriptive names (avoid `x`, `temp`, `data` unless context is clear)
- Boolean variables: `is_active`, `has_permission`, `can_delete`
- Functions: Start with verbs (`get_`, `create_`, `update_`, `delete_`, `validate_`)

```python
# Good: Descriptive names
def calculate_order_total(items: List[OrderItem]) -> Decimal:
    subtotal = sum(item.price * item.quantity for item in items)
    tax = subtotal * Decimal('0.1')
    return subtotal + tax


# Bad: Unclear names
def calc(i: list) -> float:
    s = sum(x.p * x.q for x in i)
    t = s * 0.1
    return s + t
```

### 2.4 Type Hints

**Required:** All function signatures must have type hints

```python
from typing import List, Optional, Dict, Any
from decimal import Decimal

# Good: Complete type hints
def create_user(
    email: str,
    name: str,
    role: str = "user"
) -> User:
    """Create a new user."""
    user = User(email=email, name=name, role=role)
    db.add(user)
    db.commit()
    return user


def get_users(
    page: int = 1,
    page_size: int = 20,
    filters: Optional[Dict[str, Any]] = None
) -> List[User]:
    """Get paginated list of users."""
    query = db.query(User)
    if filters:
        query = apply_filters(query, filters)
    return query.offset((page - 1) * page_size).limit(page_size).all()


# Bad: Missing type hints
def create_user(email, name, role="user"):
    user = User(email=email, name=name, role=role)
    db.add(user)
    db.commit()
    return user
```

### 2.5 Documentation

**Docstrings:** Required for all public functions, classes, and modules

**Format:** Google-style docstrings

```python
def calculate_discount(
    order_total: Decimal,
    discount_code: Optional[str] = None,
    user_tier: str = "standard"
) -> Decimal:
    """
    Calculate the discount amount for an order.
    
    This function applies tiered discounts based on user membership
    level and optional discount codes. Discount codes take precedence
    over tier-based discounts.
    
    Args:
        order_total: The total order amount before discounts
        discount_code: Optional promotional code for additional discount
        user_tier: User membership tier (standard, premium, vip)
        
    Returns:
        The discount amount to be applied to the order
        
    Raises:
        ValueError: If discount_code is invalid
        
    Examples:
        >>> calculate_discount(Decimal('100.00'), user_tier='premium')
        Decimal('10.00')
        
        >>> calculate_discount(Decimal('100.00'), discount_code='SAVE20')
        Decimal('20.00')
    """
    if discount_code:
        return apply_discount_code(order_total, discount_code)
    
    tier_discounts = {
        "standard": Decimal('0.05'),
        "premium": Decimal('0.10'),
        "vip": Decimal('0.15')
    }
    
    discount_rate = tier_discounts.get(user_tier, Decimal('0'))
    return order_total * discount_rate
```

### 2.6 Error Handling

**Principle:** Fail fast, fail explicitly

```python
from fastapi import HTTPException
from pydantic import ValidationError

# Good: Explicit error handling
def get_user_by_id(user_id: str) -> User:
    """
    Retrieve user by ID.
    
    Raises:
        HTTPException: 404 if user not found
    """
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(
            status_code=404,
            detail=f"User with ID '{user_id}' not found"
        )
    
    return user


# Good: Handle expected exceptions
def create_user(user_data: UserCreate) -> User:
    """
    Create a new user.
    
    Raises:
        HTTPException: 400 if email already exists
    """
    try:
        user = User(**user_data.dict())
        db.add(user)
        db.commit()
        return user
    except IntegrityError as e:
        db.rollback()
        if "users_email_key" in str(e):
            raise HTTPException(
                status_code=400,
                detail="Email already exists"
            )
        raise


# Bad: Silent failures
def get_user_by_id(user_id: str) -> Optional[User]:
    try:
        return db.query(User).filter(User.id == user_id).first()
    except:
        return None  # What went wrong? No way to know.
```

### 2.7 Dependency Injection

**Use FastAPI's dependency injection system:**

```python
from fastapi import Depends
from sqlalchemy.orm import Session

# Dependency
def get_db() -> Session:
    """Get database session."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


# Usage in routes
@router.get("/users/{user_id}")
async def get_user(
    user_id: str,
    db: Session = Depends(get_db)
) -> User:
    """Get user by ID."""
    return UserService(db).get_by_id(user_id)


# Service layer
class UserService:
    def __init__(self, db: Session):
        self.db = db
        self.repository = UserRepository(db)
    
    def get_by_id(self, user_id: str) -> User:
        """Get user by ID."""
        user = self.repository.get_by_id(user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")
        return user
```

### 2.8 Layered Architecture

**Enforce clean separation of concerns:**

```python
# ❌ BAD: Business logic in API route
@router.post("/users")
async def create_user(user: UserCreate, db: Session = Depends(get_db)):
    # Validation
    if db.query(User).filter(User.email == user.email).first():
        raise HTTPException(400, "Email exists")
    
    # Business logic
    new_user = User(**user.dict())
    new_user.password_hash = hash_password(user.password)
    
    # Persistence
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    
    # External calls
    send_welcome_email(new_user.email)
    
    return new_user


# ✅ GOOD: Layered architecture
# Route layer (API)
@router.post("/users")
async def create_user(
    user: UserCreate,
    service: UserService = Depends(get_user_service)
) -> User:
    """Create a new user."""
    return service.create_user(user)


# Service layer (Business Logic)
class UserService:
    def __init__(self, db: Session, email_service: EmailService):
        self.repository = UserRepository(db)
        self.email_service = email_service
    
    def create_user(self, user_data: UserCreate) -> User:
        """Create a new user with validation and notifications."""
        # Validation
        if self.repository.email_exists(user_data.email):
            raise HTTPException(400, "Email already exists")
        
        # Business logic
        user = User(
            email=user_data.email,
            name=user_data.name,
            password_hash=hash_password(user_data.password)
        )
        
        # Persistence
        created_user = self.repository.create(user)
        
        # Side effects
        self.email_service.send_welcome_email(created_user.email)
        
        return created_user


# Repository layer (Data Access)
class UserRepository:
    def __init__(self, db: Session):
        self.db = db
    
    def create(self, user: User) -> User:
        """Create user in database."""
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        return user
    
    def email_exists(self, email: str) -> bool:
        """Check if email exists."""
        return self.db.query(User).filter(User.email == email).first() is not None
```

### 2.9 Code Review Checklist

**Before Creating PR:**
- [ ] Code follows style guide (Black formatted)
- [ ] All functions have type hints
- [ ] All public functions have docstrings
- [ ] No commented-out code
- [ ] No debug print statements
- [ ] Tests added for new functionality
- [ ] Tests passing locally
- [ ] Pre-commit hooks passing
- [ ] Documentation updated (if needed)

**During Code Review:**
- [ ] Code is readable and maintainable
- [ ] No unnecessary complexity
- [ ] Error handling is appropriate
- [ ] Security considerations addressed
- [ ] Performance considerations addressed
- [ ] No hardcoded values (use config)
- [ ] Database queries are optimized
- [ ] API contracts documented

---

## 3. Testing Guide

### 3.1 Testing Philosophy

**Principles:**
1. **Test Pyramid:** More unit tests, fewer integration tests, even fewer E2E tests
2. **Test Behavior, Not Implementation:** Tests should verify outcomes, not internal details
3. **Fast Feedback:** Tests should run quickly
4. **Deterministic:** Tests should always produce the same result
5. **Isolated:** Tests should not depend on each other

**Coverage Target:** 80% minimum for production code

### 3.2 Test Structure

```
tests/
├── unit/                         # Fast, isolated tests
│   ├── test_user_service.py
│   ├── test_auth_service.py
│   └── test_validators.py
├── integration/                  # Tests with database/external services
│   ├── test_user_repository.py
│   ├── test_api_routes.py
│   └── test_external_api.py
├── e2e/                         # End-to-end user flows
│   └── test_user_registration_flow.py
├── fixtures/                    # Shared test data
│   └── sample_data.py
└── conftest.py                  # Pytest configuration and fixtures
```

### 3.3 Unit Tests

**Example: Testing Service Layer**

```python
# tests/unit/test_user_service.py
import pytest
from unittest.mock import Mock, MagicMock
from src.services.user_service import UserService
from src.models.user import User
from fastapi import HTTPException


class TestUserService:
    """Test suite for UserService."""
    
    def test_get_user_by_id_success(self):
        """Should return user when user exists."""
        # Arrange
        mock_repo = Mock()
        mock_user = User(id="123", email="test@example.com", name="Test User")
        mock_repo.get_by_id.return_value = mock_user
        
        service = UserService(repository=mock_repo)
        
        # Act
        result = service.get_by_id("123")
        
        # Assert
        assert result == mock_user
        mock_repo.get_by_id.assert_called_once_with("123")
    
    def test_get_user_by_id_not_found(self):
        """Should raise 404 when user does not exist."""
        # Arrange
        mock_repo = Mock()
        mock_repo.get_by_id.return_value = None
        
        service = UserService(repository=mock_repo)
        
        # Act & Assert
        with pytest.raises(HTTPException) as exc_info:
            service.get_by_id("nonexistent")
        
        assert exc_info.value.status_code == 404
    
    def test_create_user_success(self):
        """Should create user when email is unique."""
        # Arrange
        mock_repo = Mock()
        mock_repo.email_exists.return_value = False
        mock_repo.create.return_value = User(
            id="123",
            email="new@example.com",
            name="New User"
        )
        
        service = UserService(repository=mock_repo)
        user_data = {"email": "new@example.com", "name": "New User"}
        
        # Act
        result = service.create_user(user_data)
        
        # Assert
        assert result.email == "new@example.com"
        mock_repo.email_exists.assert_called_once_with("new@example.com")
        mock_repo.create.assert_called_once()
    
    def test_create_user_email_exists(self):
        """Should raise 400 when email already exists."""
        # Arrange
        mock_repo = Mock()
        mock_repo.email_exists.return_value = True
        
        service = UserService(repository=mock_repo)
        user_data = {"email": "existing@example.com", "name": "User"}
        
        # Act & Assert
        with pytest.raises(HTTPException) as exc_info:
            service.create_user(user_data)
        
        assert exc_info.value.status_code == 400
        assert "already exists" in str(exc_info.value.detail).lower()
```

### 3.4 Integration Tests

**Example: Testing Repository with Database**

```python
# tests/integration/test_user_repository.py
import pytest
from src.repositories.user_repository import UserRepository
from src.models.user import User


@pytest.fixture
def db_session(test_db):
    """Provide a database session for tests."""
    session = TestSession()
    yield session
    session.rollback()
    session.close()


class TestUserRepository:
    """Test suite for UserRepository."""
    
    def test_create_user(self, db_session):
        """Should create user in database."""
        # Arrange
        repository = UserRepository(db_session)
        user = User(email="test@example.com", name="Test User")
        
        # Act
        created_user = repository.create(user)
        
        # Assert
        assert created_user.id is not None
        assert created_user.email == "test@example.com"
        assert created_user.created_at is not None
        
        # Verify in database
        db_user = db_session.query(User).filter(User.id == created_user.id).first()
        assert db_user is not None
        assert db_user.email == "test@example.com"
    
    def test_get_by_email(self, db_session):
        """Should retrieve user by email."""
        # Arrange
        repository = UserRepository(db_session)
        user = User(email="find@example.com", name="Find Me")
        db_session.add(user)
        db_session.commit()
        
        # Act
        found_user = repository.get_by_email("find@example.com")
        
        # Assert
        assert found_user is not None
        assert found_user.email == "find@example.com"
    
    def test_email_exists(self, db_session):
        """Should return True if email exists."""
        # Arrange
        repository = UserRepository(db_session)
        user = User(email="exists@example.com", name="User")
        db_session.add(user)
        db_session.commit()
        
        # Act & Assert
        assert repository.email_exists("exists@example.com") is True
        assert repository.email_exists("notexists@example.com") is False
```

### 3.5 API Integration Tests

**Example: Testing API Endpoints**

```python
# tests/integration/test_api_users.py
import pytest
from fastapi.testclient import TestClient
from src.main import app


@pytest.fixture
def client():
    """Provide test client."""
    return TestClient(app)


class TestUserAPI:
    """Test suite for User API endpoints."""
    
    def test_create_user_success(self, client):
        """Should create user and return 201."""
        # Arrange
        user_data = {
            "email": "newuser@example.com",
            "name": "New User",
            "password": "securepassword123"
        }
        
        # Act
        response = client.post("/api/v1/users", json=user_data)
        
        # Assert
        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "newuser@example.com"
        assert data["name"] == "New User"
        assert "id" in data
        assert "password" not in data  # Password should not be in response
    
    def test_create_user_duplicate_email(self, client):
        """Should return 400 when email already exists."""
        # Arrange
        user_data = {
            "email": "duplicate@example.com",
            "name": "User",
            "password": "password123"
        }
        client.post("/api/v1/users", json=user_data)
        
        # Act
        response = client.post("/api/v1/users", json=user_data)
        
        # Assert
        assert response.status_code == 400
        assert "already exists" in response.json()["detail"].lower()
    
    def test_get_user_by_id(self, client):
        """Should return user when ID exists."""
        # Arrange: Create a user first
        user_data = {"email": "getme@example.com", "name": "Get Me", "password": "pass"}
        create_response = client.post("/api/v1/users", json=user_data)
        user_id = create_response.json()["id"]
        
        # Act
        response = client.get(f"/api/v1/users/{user_id}")
        
        # Assert
        assert response.status_code == 200
        data = response.json()
        assert data["id"] == user_id
        assert data["email"] == "getme@example.com"
    
    def test_get_user_not_found(self, client):
        """Should return 404 when user does not exist."""
        # Act
        response = client.get("/api/v1/users/nonexistent-id")
        
        # Assert
        assert response.status_code == 404
```

### 3.6 Test Fixtures

**Example: Common Fixtures**

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from src.core.database import Base
from src.models import User


# Database fixtures
@pytest.fixture(scope="session")
def test_engine():
    """Create test database engine."""
    engine = create_engine("postgresql://postgres:postgres@localhost/myapp_test")
    Base.metadata.create_all(engine)
    yield engine
    Base.metadata.drop_all(engine)


@pytest.fixture
def db_session(test_engine):
    """Provide clean database session for each test."""
    Session = sessionmaker(bind=test_engine)
    session = Session()
    
    yield session
    
    session.rollback()
    session.close()


# Sample data fixtures
@pytest.fixture
def sample_user(db_session):
    """Create a sample user for testing."""
    user = User(
        email="sample@example.com",
        name="Sample User",
        role="user"
    )
    db_session.add(user)
    db_session.commit()
    db_session.refresh(user)
    return user


@pytest.fixture
def admin_user(db_session):
    """Create an admin user for testing."""
    user = User(
        email="admin@example.com",
        name="Admin User",
        role="admin"
    )
    db_session.add(user)
    db_session.commit()
    db_session.refresh(user)
    return user
```

### 3.7 Running Tests

**Run All Tests:**
```bash
pytest
```

**Run Specific Test File:**
```bash
pytest tests/unit/test_user_service.py
```

**Run Specific Test:**
```bash
pytest tests/unit/test_user_service.py::TestUserService::test_create_user_success
```

**Run with Coverage:**
```bash
pytest --cov=src --cov-report=html --cov-report=term
```

**Run Tests Matching Pattern:**
```bash
pytest -k "create_user"
```

**Run in Parallel:**
```bash
pytest -n auto  # Uses all CPU cores
```

**Watch Mode (auto-rerun on file changes):**
```bash
pytest-watch
```

### 3.8 Test Coverage

**View Coverage Report:**
```bash
# Generate HTML report
pytest --cov=src --cov-report=html

# Open report
open htmlcov/index.html  # Mac
xdg-open htmlcov/index.html  # Linux
```

**Coverage Requirements:**
- Minimum 80% overall coverage
- Critical paths (auth, payments, etc.) should have 95%+ coverage
- New code must maintain or improve coverage

---

## 4. Git Workflow

### 4.1 Branching Strategy

**Branch Types:**
- `main`: Production code (protected)
- `develop`: Integration branch (protected)
- `feature/*`: New features
- `bugfix/*`: Bug fixes
- `hotfix/*`: Emergency production fixes

**Branch Naming:**
```
feature/user-authentication
feature/TICKET-123-payment-integration
bugfix/fix-login-redirect
hotfix/critical-security-patch
```

### 4.2 Commit Messages

**Format:** Conventional Commits

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Test additions or changes
- `chore`: Build process or tooling changes

**Examples:**
```
feat(auth): add JWT authentication

Implemented JWT-based authentication with access and refresh tokens.
Tokens expire after 1 hour (access) and 7 days (refresh).

Closes #123

---

fix(api): handle null values in user response

Previously, API would crash when user had null address.
Now returns empty string for null values.

Fixes #456

---

docs: update API documentation for user endpoints

Added examples and response schemas for all user endpoints.
```

### 4.3 Pull Request Process

1. **Create Branch:**
   ```bash
   git checkout -b feature/my-new-feature
   ```

2. **Make Changes:**
   - Write code
   - Write tests
   - Update documentation

3. **Commit Changes:**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```

4. **Push Branch:**
   ```bash
   git push origin feature/my-new-feature
   ```

5. **Create Pull Request:**
   - Clear title describing the change
   - Description with context and testing notes
   - Link to related issues
   - Request reviewers

6. **Code Review:**
   - Address review comments
   - Push additional commits
   - Re-request review

7. **Merge:**
   - Squash and merge (for feature branches)
   - Include PR number in commit message
   - Delete branch after merge

**PR Template:**
```markdown
## Description
[Brief description of what this PR does]

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## How to Test
1. [Step 1]
2. [Step 2]
3. [Expected result]

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Code follows style guide
- [ ] All tests passing
- [ ] No new warnings

## Related Issues
Closes #123
Related to #456
```

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | [Date] | [Name] | Initial version |

**Next Review:** [Date]

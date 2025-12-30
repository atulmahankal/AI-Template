# Coding Guidelines

## Code Style

### TypeScript/React (Frontend)

#### Naming Conventions
- **Variables/Functions**: `camelCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Components**: `PascalCase`
- **Interfaces/Types**: `PascalCase` with `I` prefix for interfaces (optional)
- **Files**:
  - Components: `PascalCase.tsx`
  - Hooks: `use-hook-name.ts`
  - Utils: `kebab-case.ts`
  - Types: `kebab-case.types.ts`

#### Component Structure
```tsx
// 1. Imports (grouped: react, external, internal, styles)
import { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Button } from '@/components/common';
import type { ExamProps } from './exam.types';

// 2. Types/Interfaces (if not in separate file)
interface Props {
  examId: string;
  onComplete: () => void;
}

// 3. Component
export function ExamCard({ examId, onComplete }: Props) {
  // 3a. Hooks
  const [isLoading, setIsLoading] = useState(false);

  // 3b. Derived state/memos
  const isValid = useMemo(() => /* ... */, [deps]);

  // 3c. Effects
  useEffect(() => {
    // ...
  }, []);

  // 3d. Handlers
  const handleSubmit = () => {
    // ...
  };

  // 3e. Render
  return (
    <div className="exam-card">
      {/* ... */}
    </div>
  );
}
```

#### TailwindCSS Guidelines
```tsx
// Prefer utility classes
<div className="flex items-center gap-4 p-4 bg-white rounded-lg shadow-md">

// For complex/repeated patterns, use @apply in CSS
// tailwind.css
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700;
  }
}

// Use cn() helper for conditional classes
import { cn } from '@/utils/cn';
<div className={cn(
  "base-class",
  isActive && "active-class",
  variant === 'primary' && "primary-class"
)}>
```

### Python/FastAPI (Backend)

#### Naming Conventions
- **Variables/Functions**: `snake_case`
- **Constants**: `UPPER_SNAKE_CASE`
- **Classes**: `PascalCase`
- **Files/Modules**: `snake_case.py`
- **Private**: `_prefix_with_underscore`

#### API Endpoint Structure
```python
# app/api/v1/exams.py (routes prefixed with /v1/ - nginx handles /api/)
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db, get_current_user
from app.schemas.exam import ExamCreate, ExamResponse
from app.services.exam import ExamService
from app.core.exceptions import ExamNotFoundError
from app.core.logging import get_logger

router = APIRouter(prefix="/exams", tags=["exams"])
logger = get_logger(__name__)

@router.post("/", response_model=ExamResponse, status_code=status.HTTP_201_CREATED)
async def create_exam(
    exam_data: ExamCreate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    """Create a new exam."""
    logger.info(f"Creating exam: {exam_data.title} by user: {current_user.id}")
    try:
        exam = await ExamService(db).create(exam_data, current_user)
        return exam
    except Exception as e:
        logger.error(f"Failed to create exam: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to create exam"
        )
```

#### Pydantic Schema Structure
```python
# app/schemas/exam.py
from datetime import datetime
from pydantic import BaseModel, Field, ConfigDict

class ExamBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: str | None = None
    duration_minutes: int = Field(..., ge=1, le=480)

class ExamCreate(ExamBase):
    pass

class ExamUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    duration_minutes: int | None = Field(None, ge=1, le=480)

class ExamResponse(ExamBase):
    model_config = ConfigDict(from_attributes=True)

    id: int
    created_at: datetime
    created_by_id: int
```

#### SQLAlchemy Model Structure
```python
# app/models/exam.py
from datetime import datetime
from sqlalchemy import String, Integer, ForeignKey, Text
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.models.base import Base

class Exam(Base):
    __tablename__ = "exams"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    description: Mapped[str | None] = mapped_column(Text)
    duration_minutes: Mapped[int] = mapped_column(Integer)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
    created_by_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    # Relationships
    created_by: Mapped["User"] = relationship(back_populates="exams")
    questions: Mapped[list["Question"]] = relationship(back_populates="exam")
```

## Best Practices

### General
- Write self-documenting code with clear variable names
- Keep functions small and focused (< 50 lines ideally)
- DRY (Don't Repeat Yourself) - extract common logic
- KISS (Keep It Simple, Stupid) - avoid over-engineering
- Single Responsibility Principle

### React Specific
- Use functional components with hooks
- Memoize expensive computations with `useMemo`
- Use `useCallback` for event handlers passed to children
- Prefer composition over prop drilling
- Use React Query for server state
- Use Zustand for client state

### FastAPI Specific
- Always use async functions for database operations
- Use dependency injection for database sessions
- Validate all inputs with Pydantic
- Return consistent response shapes
- Use status codes appropriately

## Error Handling

### Frontend
```tsx
// Use error boundaries for component errors
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" className="error-container">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// API error handling
try {
  const data = await api.createExam(examData);
} catch (error) {
  if (error instanceof ApiError) {
    toast.error(error.message);
  } else {
    toast.error('An unexpected error occurred');
    console.error(error);
  }
}
```

### Backend
```python
# app/core/exceptions.py
class AppException(Exception):
    """Base application exception."""
    def __init__(self, message: str, code: str, status_code: int = 400):
        self.message = message
        self.code = code
        self.status_code = status_code
        super().__init__(message)

class ExamNotFoundError(AppException):
    def __init__(self, exam_id: int):
        super().__init__(
            message=f"Exam with ID {exam_id} not found",
            code="EXAM_NOT_FOUND",
            status_code=404
        )

# app/core/error_handler.py
from fastapi import Request
from fastapi.responses import JSONResponse

async def app_exception_handler(request: Request, exc: AppException):
    logger.error(f"AppException: {exc.code} - {exc.message}")
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error": {
                "code": exc.code,
                "message": exc.message
            }
        }
    )
```

## Logging

### Backend Logging
```python
# app/core/logging.py
import logging
from datetime import datetime
from pathlib import Path

def setup_logging():
    """Configure application logging."""
    log_dir = Path("./logs")
    log_dir.mkdir(exist_ok=True)

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    log_file = log_dir / f"{timestamp}.log"

    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
        handlers=[
            logging.FileHandler(log_file),
            logging.StreamHandler()
        ]
    )

def get_logger(name: str) -> logging.Logger:
    """Get a logger instance."""
    return logging.getLogger(name)
```

### Frontend Logging
```tsx
// Use console methods appropriately
console.log('Info message');      // Development info
console.warn('Warning message');  // Potential issues
console.error('Error message');   // Actual errors

// In production, consider a logging service
if (import.meta.env.DEV) {
  console.log('Debug info');
}
```

## Security

### Never Commit
- API keys and secrets
- `.env` files with real values
- Database credentials
- JWT secret keys

### Input Validation
```python
# Backend: Always validate with Pydantic
class QuestionCreate(BaseModel):
    text: str = Field(..., min_length=1, max_length=5000)
    question_type: QuestionType
    options: list[str] = Field(..., min_length=2, max_length=10)
```

```tsx
// Frontend: Validate before submission
import { z } from 'zod';

const questionSchema = z.object({
  text: z.string().min(1).max(5000),
  questionType: z.enum(['mcq', 'short', 'long']),
  options: z.array(z.string()).min(2).max(10),
});
```

### Sanitize Outputs
- Escape HTML in user-generated content
- Use parameterized queries (SQLAlchemy handles this)
- Validate file uploads (type, size, name)

## Comments

### When to Comment
- Complex business logic
- Non-obvious algorithms
- TODO/FIXME notes with ticket references
- Public API documentation (docstrings)

### When NOT to Comment
- Obvious code (`i += 1  # increment i`)
- Code that should be refactored instead
- Commented-out code (delete it)

### Docstring Format (Python)
```python
async def evaluate_mcq(
    submission: Submission,
    answers: dict[int, str]
) -> EvaluationResult:
    """
    Evaluate MCQ submission and calculate score.

    Args:
        submission: The exam submission to evaluate
        answers: Dict mapping question_id to selected answer

    Returns:
        EvaluationResult with score and feedback

    Raises:
        ValidationError: If answers format is invalid
    """
```

---
*Last Updated: 2024-12-28*

# Testing Guidelines

## Test Types

### Unit Tests
- Test individual functions/methods in isolation
- Mock external dependencies
- Fast execution (< 100ms per test)

### Integration Tests
- Test component interactions
- Test API endpoints with database
- Test service layer logic

### End-to-End (E2E) Tests
- Test complete user flows
- Browser-based testing with Playwright
- Critical path coverage

## Frontend Testing (Vitest + React Testing Library)

### Setup
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/tests/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'src/tests/'],
    },
  },
});
```

### Test Structure
```typescript
// src/components/exam/ExamCard.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { ExamCard } from './ExamCard';

describe('ExamCard', () => {
  const mockExam = {
    id: 1,
    title: 'Math Test',
    duration: 60,
    questionCount: 20,
  };

  it('should render exam title', () => {
    render(<ExamCard exam={mockExam} />);
    expect(screen.getByText('Math Test')).toBeInTheDocument();
  });

  it('should display duration in minutes', () => {
    render(<ExamCard exam={mockExam} />);
    expect(screen.getByText('60 minutes')).toBeInTheDocument();
  });

  it('should call onStart when start button clicked', () => {
    const onStart = vi.fn();
    render(<ExamCard exam={mockExam} onStart={onStart} />);

    fireEvent.click(screen.getByRole('button', { name: /start/i }));

    expect(onStart).toHaveBeenCalledWith(mockExam.id);
  });

  it('should show disabled state when exam is locked', () => {
    render(<ExamCard exam={{ ...mockExam, isLocked: true }} />);

    const button = screen.getByRole('button', { name: /start/i });
    expect(button).toBeDisabled();
  });
});
```

### Hook Testing
```typescript
// src/hooks/use-exam-timer.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useExamTimer } from './use-exam-timer';

describe('useExamTimer', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should initialize with given duration', () => {
    const { result } = renderHook(() => useExamTimer(3600));

    expect(result.current.timeRemaining).toBe(3600);
    expect(result.current.isRunning).toBe(false);
  });

  it('should countdown when started', () => {
    const { result } = renderHook(() => useExamTimer(60));

    act(() => {
      result.current.start();
    });

    act(() => {
      vi.advanceTimersByTime(1000);
    });

    expect(result.current.timeRemaining).toBe(59);
  });

  it('should call onTimeUp when timer reaches zero', () => {
    const onTimeUp = vi.fn();
    const { result } = renderHook(() => useExamTimer(2, { onTimeUp }));

    act(() => {
      result.current.start();
      vi.advanceTimersByTime(3000);
    });

    expect(onTimeUp).toHaveBeenCalled();
  });
});
```

### API Service Testing
```typescript
// src/services/exam.service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { examService } from './exam.service';

describe('examService', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should fetch exam by id', async () => {
    const mockExam = { id: 1, title: 'Test Exam' };
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockExam),
    });

    const result = await examService.getById(1);

    expect(fetch).toHaveBeenCalledWith('/v1/exams/1');
    expect(result).toEqual(mockExam);
  });

  it('should throw error on failed request', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 404,
    });

    await expect(examService.getById(999)).rejects.toThrow('Exam not found');
  });
});
```

## Backend Testing (Pytest)

### Setup
```python
# backend/conftest.py
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.models.base import Base
from app.api.deps import get_db

TEST_DATABASE_URL = "sqlite+aiosqlite:///./test.db"

@pytest.fixture(scope="session")
def event_loop():
    import asyncio
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def engine():
    engine = create_async_engine(TEST_DATABASE_URL, echo=False)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.fixture
async def db_session(engine):
    async_session = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
    async with async_session() as session:
        yield session
        await session.rollback()

@pytest.fixture
async def client(db_session):
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    app.dependency_overrides.clear()

@pytest.fixture
async def authenticated_client(client, test_user):
    """Client with authentication token."""
    response = await client.post("/v1/auth/login", json={
        "email": test_user.email,
        "password": "testpass123"
    })
    token = response.json()["access_token"]
    client.headers["Authorization"] = f"Bearer {token}"
    return client
```

### Model Tests
```python
# backend/app/tests/test_models.py
import pytest
from app.models.exam import Exam
from app.models.question import Question, QuestionType

@pytest.mark.asyncio
async def test_exam_creation(db_session):
    exam = Exam(
        title="Python Basics",
        description="Test your Python knowledge",
        duration_minutes=60,
        created_by_id=1
    )
    db_session.add(exam)
    await db_session.commit()
    await db_session.refresh(exam)

    assert exam.id is not None
    assert exam.title == "Python Basics"
    assert exam.duration_minutes == 60

@pytest.mark.asyncio
async def test_question_with_options(db_session, test_exam):
    question = Question(
        exam_id=test_exam.id,
        text="What is 2 + 2?",
        question_type=QuestionType.MCQ,
        options=["3", "4", "5", "6"],
        correct_answer="4",
        points=1
    )
    db_session.add(question)
    await db_session.commit()

    assert question.id is not None
    assert question.options == ["3", "4", "5", "6"]
```

### API Endpoint Tests
```python
# backend/app/tests/test_api/test_exams.py
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_exam(authenticated_client: AsyncClient):
    exam_data = {
        "title": "Math Exam",
        "description": "Basic mathematics",
        "duration_minutes": 60
    }

    response = await authenticated_client.post("/v1/exams/", json=exam_data)

    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Math Exam"
    assert data["id"] is not None

@pytest.mark.asyncio
async def test_create_exam_unauthorized(client: AsyncClient):
    response = await client.post("/v1/exams/", json={"title": "Test"})

    assert response.status_code == 401

@pytest.mark.asyncio
async def test_get_exam(authenticated_client: AsyncClient, test_exam):
    response = await authenticated_client.get(f"/v1/exams/{test_exam.id}")

    assert response.status_code == 200
    data = response.json()
    assert data["id"] == test_exam.id

@pytest.mark.asyncio
async def test_get_nonexistent_exam(authenticated_client: AsyncClient):
    response = await authenticated_client.get("/v1/exams/99999")

    assert response.status_code == 404
    data = response.json()
    assert data["error"]["code"] == "EXAM_NOT_FOUND"

@pytest.mark.asyncio
async def test_list_exams_pagination(authenticated_client: AsyncClient):
    response = await authenticated_client.get("/v1/exams/?page=1&per_page=10")

    assert response.status_code == 200
    data = response.json()
    assert "items" in data
    assert "total" in data
    assert "page" in data
```

### Service Layer Tests
```python
# backend/app/tests/test_services/test_evaluation.py
import pytest
from app.services.evaluation import EvaluationService
from app.schemas.submission import MCQAnswer

@pytest.mark.asyncio
async def test_evaluate_mcq_all_correct(db_session, test_exam_with_questions):
    service = EvaluationService(db_session)

    answers = [
        MCQAnswer(question_id=1, selected="B"),
        MCQAnswer(question_id=2, selected="A"),
    ]

    result = await service.evaluate_mcq(test_exam_with_questions.id, answers)

    assert result.score == 100
    assert result.correct_count == 2
    assert result.total_count == 2

@pytest.mark.asyncio
async def test_evaluate_mcq_partial_correct(db_session, test_exam_with_questions):
    service = EvaluationService(db_session)

    answers = [
        MCQAnswer(question_id=1, selected="B"),  # Correct
        MCQAnswer(question_id=2, selected="C"),  # Wrong
    ]

    result = await service.evaluate_mcq(test_exam_with_questions.id, answers)

    assert result.score == 50
    assert result.correct_count == 1
```

## E2E Testing (Playwright)

### Setup
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Test Example
```typescript
// e2e/exam-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Exam Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('[name="email"]', 'student@test.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
  });

  test('should complete MCQ exam', async ({ page }) => {
    // Navigate to exam
    await page.click('text=Math Exam');
    await page.click('text=Start Exam');

    // Answer questions
    await expect(page.locator('h2')).toContainText('Question 1');
    await page.click('label:has-text("Option B")');
    await page.click('button:text("Next")');

    // Submit exam
    await page.click('button:text("Submit")');
    await page.click('button:text("Confirm")');

    // Verify results
    await expect(page.locator('.score')).toBeVisible();
  });

  test('should auto-save answers', async ({ page }) => {
    await page.goto('/exam/1');
    await page.click('label:has-text("Option A")');

    // Refresh page
    await page.reload();

    // Verify answer persisted
    await expect(page.locator('input[value="A"]')).toBeChecked();
  });

  test('should show timer warning', async ({ page }) => {
    await page.goto('/exam/1');

    // Fast-forward timer (mock)
    await page.evaluate(() => {
      window.dispatchEvent(new CustomEvent('test:setTimer', { detail: 60 }));
    });

    await expect(page.locator('.timer-warning')).toBeVisible();
  });
});
```

## Coverage Requirements

| Layer | Minimum | Target |
|-------|---------|--------|
| Backend Unit | 80% | 90% |
| Backend Integration | 70% | 80% |
| Frontend Components | 70% | 85% |
| Frontend Hooks | 80% | 90% |
| E2E Critical Paths | 100% | 100% |

### Critical Paths (Must be 100%)
- User authentication flow
- Exam submission process
- Answer auto-save functionality
- Results calculation and display
- Timer functionality

## Running Tests

### Frontend (pnpm)
```bash
# Run all tests
pnpm test

# Run with coverage
pnpm test:coverage

# Run specific file
pnpm test ExamCard.test.tsx

# Run in watch mode
pnpm test --watch

# Run E2E tests
pnpm test:e2e
```

### Backend (uv)
```bash
# Run all tests
uv run pytest

# Run with coverage
uv run pytest --cov=app --cov-report=html

# Run specific file
uv run pytest app/tests/test_api/test_exams.py

# Run specific test
uv run pytest -k "test_create_exam"

# Run with verbose output
uv run pytest -v

# Run async tests only
uv run pytest -m asyncio
```

### Docker
```bash
# Run all tests in containers
docker-compose run --rm frontend pnpm test
docker-compose run --rm backend uv run pytest

# Run with coverage reports
docker-compose run --rm backend uv run pytest --cov=app
```

## Mocking Guidelines

### Frontend - API Mocking
```typescript
// Use MSW for API mocking
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const handlers = [
  rest.get('/v1/exams/:id', (req, res, ctx) => {
    return res(ctx.json({ id: req.params.id, title: 'Mock Exam' }));
  }),
];

const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### Backend - Database Mocking
```python
# Use fixtures for test data
@pytest.fixture
async def test_user(db_session):
    user = User(
        email="test@example.com",
        hashed_password=hash_password("testpass123"),
        role=UserRole.STUDENT
    )
    db_session.add(user)
    await db_session.commit()
    return user

@pytest.fixture
async def test_exam(db_session, test_user):
    exam = Exam(
        title="Test Exam",
        duration_minutes=60,
        created_by_id=test_user.id
    )
    db_session.add(exam)
    await db_session.commit()
    return exam
```

---
*Last Updated: 2024-12-28*

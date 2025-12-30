# Operational Guidelines

## Development Workflow

### Branch Strategy
- `main` - Production ready code
- `develop` - Integration branch for features
- `feature/*` - New features (e.g., `feature/mcq-module`)
- `fix/*` - Bug fixes (e.g., `fix/timer-countdown`)
- `release/*` - Release preparation

### Commit Messages
```
type(scope): subject

body (optional)

footer (optional)
```

**Types**: feat, fix, docs, style, refactor, test, chore

**Examples**:
```
feat(exam): add MCQ question component

- Implement option selection UI
- Add keyboard navigation support
- Include accessibility attributes

Closes #123
```

```
fix(auth): resolve token refresh race condition

The refresh token was being sent multiple times when
multiple API calls failed simultaneously.

Fixes #456
```

### Code Review Checklist
- [ ] Tests written and passing
- [ ] Code follows project guidelines
- [ ] No console.log or print statements
- [ ] Error handling implemented
- [ ] Documentation updated if needed

## Environment Setup

### Development
```bash
# Clone and install
git clone <repo-url>
cd your-project

# Start with Docker (recommended)
docker-compose up -d

# Or run locally (see README for full instructions)
# Frontend (pnpm)
cd frontend && pnpm install && pnpm dev

# Backend (uv)
cd backend && uv sync && uv run uvicorn app.main:app --reload
```

### Environment Variables
```bash
# Backend (.env)
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/your_db
REDIS_URL=redis://localhost:6379
JWT_SECRET_KEY=your-secret-key
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=30

# Frontend (.env)
VITE_API_URL=https://api.yourproject.local/v1
```

## Deployment

### Docker Deployment
```bash
# Build images
docker-compose -f docker-compose.prod.yml build

# Deploy
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose -f docker-compose.prod.yml logs -f
```

### Health Checks
- Backend: `GET /health` - Returns 200 if healthy
- Frontend: Check if static files are served
- Database: Connection pool status
- Redis: PING command

## Monitoring

### Logging
- **Location**: `./logs/<timestamp>.log`
- **Format**: `timestamp | level | logger | message`
- **Rotation**: Daily, keep 30 days
- **Levels**: DEBUG (dev), INFO (prod)

### Log Examples
```
2024-12-28 10:30:45 | INFO | app.api.auth | User login: user@example.com
2024-12-28 10:31:00 | INFO | app.services.exam | Exam started: exam_id=123 user_id=456
2024-12-28 10:45:00 | INFO | app.services.evaluation | Exam submitted: submission_id=789 score=85
2024-12-28 10:45:01 | ERROR | app.api.submissions | Failed to save submission: Database timeout
```

### Metrics to Monitor
- API response times
- Database query times
- Active exam sessions
- Error rates
- User authentication attempts

## Incident Response

### Severity Levels
1. **Critical**: System down, data loss risk
2. **High**: Major feature broken, workaround exists
3. **Medium**: Minor feature issue, cosmetic problems
4. **Low**: Enhancement requests, minor bugs

### Response Steps
1. **Identify**: Check logs, reproduce issue
2. **Contain**: Apply temporary fix if needed
3. **Investigate**: Find root cause
4. **Fix**: Implement and test solution
5. **Document**: Update incident log

### Incident Log Template
```markdown
## Incident: [Title]
- **Date**: YYYY-MM-DD HH:MM
- **Severity**: Critical/High/Medium/Low
- **Status**: Investigating/Resolved
- **Impact**: [Description of impact]
- **Root Cause**: [What caused it]
- **Resolution**: [How it was fixed]
- **Prevention**: [Steps to prevent recurrence]
```

## Backup Strategy

### Database
- Full backup: Daily at 00:00 UTC
- Incremental: Every 6 hours
- Retention: 30 days
- Storage: Encrypted, off-site

### User Uploads
- Sync to cloud storage
- Versioning enabled
- Retention: Indefinite

## Security Guidelines

### Authentication
- JWT tokens with short expiry (30 min)
- Refresh tokens stored in httpOnly cookies
- Rate limiting on auth endpoints
- Account lockout after failed attempts

### Data Protection
- Passwords hashed with bcrypt
- Sensitive data encrypted at rest
- HTTPS only in production
- CORS configured for allowed origins

### Exam Security
- Tab switch detection
- Fullscreen enforcement (writing pad)
- Time-limited exam sessions
- Answer auto-save with timestamps

---
*Last Updated: 2024-12-28*

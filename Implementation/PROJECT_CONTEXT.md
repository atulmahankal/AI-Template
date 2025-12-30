# Project Context

## Project Name
<!-- Enter your project name -->

## Description
<!-- Brief description of the project -->

## Tech Stack

### Frontend
- **Framework**: React 18+ with TypeScript (SSR with SWC)
- **Styling**: TailwindCSS v4 (latest)
- **Build Tool**: Vite
- **Init Command**: `pnpm create vite-extra frontend --template ssr-react-swc-streaming-ts`
- **Package Manager**: pnpm (fast, disk-efficient)
- **State Management**: Zustand / React Query
- **Testing**: Vitest + React Testing Library + Playwright (E2E)
- **Writing Pad**: Canvas API / Fabric.js for handwriting support

### Backend
- **Framework**: FastAPI (Python 3.12+)
- **Package Manager**: uv (fast Python package installer)
- **Database**: PostgreSQL with SQLAlchemy ORM
- **Cache**: Redis for session management
- **Authentication**: JWT with OAuth2
- **Testing**: Pytest with pytest-asyncio
- **Logging**: Python logging to `./logs/<timestamp>.log`

### DevOps
- **Containerization**: Docker + Docker Compose
- **Debugging**: Vite dev tools
- **CI/CD**: GitHub Actions (planned)

## Project Goals
1. <!-- Primary goal -->
2. <!-- Secondary goal -->

## Key Features

### Phase 1: [Name]
- <!-- Feature 1 -->
- <!-- Feature 2 -->

### Phase 2: [Name]
- <!-- Feature 1 -->
- <!-- Feature 2 -->

## External Dependencies
- PostgreSQL 15+
- Redis 7+
- Node.js 20+ (LTS)
- pnpm 9+ (frontend package manager)
- Python 3.12+
- uv (Python package manager - https://docs.astral.sh/uv/)
- mkcert (local SSL certificates)
- Docker & Docker Compose

## Repository
<!-- Git repository URL -->

## Team
- <!-- Role: Name -->

---
*Last Updated: <!-- date -->*

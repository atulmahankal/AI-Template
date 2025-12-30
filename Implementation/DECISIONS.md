# Design Decisions

## Decision Template

### Decision: [Title]
- **Date**: <!-- Added on confirmation -->
- **Status**: Pending | Accepted | Deprecated
- **Context**: <!-- Why this decision was needed -->
- **Decision**: <!-- What was decided -->
- **Consequences**:
  - <!-- Positive/negative outcomes -->

---

## Decisions Log

> Primary decisions can be modified if needed. Update Date & Status on confirmation.

### Decision 1: Simple Folder Structure
- **Date**:
- **Status**: Pending
- **Context**: Need to manage frontend and backend in a single repository
- **Decision**: Use simple folder structure with separate `frontend/` and `backend/` directories (no monorepo tooling)
- **Consequences**:
  - Easy to understand and navigate
  - No additional tooling complexity
  - Can be separated into multiple repos later if needed
  - Each folder is independently deployable

### Decision 2: FastAPI over Django/Flask
- **Date**:
- **Status**: Pending
- **Context**: Need a Python backend framework with async support and automatic API documentation
- **Decision**: Use FastAPI for its async capabilities, automatic OpenAPI docs, and Pydantic validation
- **Consequences**:
  - Better performance for IO-bound operations
  - Auto-generated API documentation
  - Type safety with Pydantic models

### Decision 3: pnpm for Frontend Package Management
- **Date**:
- **Status**: Pending
- **Context**: Need fast, disk-efficient package manager for frontend
- **Decision**: Use pnpm instead of npm/yarn
- **Consequences**:
  - Faster installs with hard-linked packages
  - Disk space savings via content-addressable storage
  - Strict dependency resolution (no phantom dependencies)

### Decision 4: uv for Backend Package Management
- **Date**:
- **Status**: Pending
- **Context**: Need fast Python package manager with lockfile support
- **Decision**: Use uv (Astral) instead of pip/poetry
- **Consequences**:
  - 10-100x faster than pip
  - Built-in lockfile support (uv.lock)
  - Drop-in replacement for pip commands
  - Excellent Docker layer caching with `uv sync --locked`

---
*Last Updated: <!-- date -->*

# AI Agent Instructions

> **Local Overrides**: If `.ai/AI_INSTRUCTIONS.local.md` exists, read it first.
> Settings in the local file override this file (e.g., deployment mode, package managers).
> **Note**: Do NOT read `.ai/AI_INSTRUCTIONS.local.md.example` - it's a template for users to copy and rename to `.ai/AI_INSTRUCTIONS.local.md`.

---

## Multi-Agent Concurrency System

This template supports **multiple AI agents working simultaneously**. Follow these rules to avoid conflicts.

### Agent Identification

Each agent MUST identify itself using the `AGENT_NAME` from its config file:

- Claude → `claude`
- Gemini → `gemini`
- Copilot → `copilot`
- Cursor → `cursor`
- Codex → `codex`
- OpenCode → `opencode`
- Windsurf → `windsurf`
- Aider → `aider`

### Memory File Naming (Per-Agent)

Create memory files with agent identifier:

```
.ai/memory/YYYYMMDD_HHMMSS_{agent}.md

Examples:
.ai/memory/20241229_143022_claude.md
.ai/memory/20241229_143105_gemini.md
.ai/memory/20241229_143200_cursor.md
```

### Task Claiming Protocol

#### Step 1: Check Active Sessions

Before starting work, read `Implementation/CURRENT_TASK.md` to see active sessions.

#### Step 2: Claim a Task

When starting a task, update `TODO.md` with your claim:

```markdown
# Before (unclaimed)

- [ ] Create FastAPI structure

# After (claimed/in-progress by claude)

- [•] Create FastAPI structure `@claude` `#20241229-143022`
```

Format: `@{agent}` `#{session-id}`

#### Step 3: Register in CURRENT_TASK.md

Add your session to the active sessions table in `CURRENT_TASK.md`.

#### Step 4: Complete or Release

When done or stopping:

```markdown
# Completed

- [✓] Create FastAPI structure ~~@claude~~ ~~#20241229-143022~~

# Released (if stopping without completing)

- [ ] Create FastAPI structure <!-- Released by agent -->
```

---

### ⚠️ Task Completion Rules (MANDATORY)

**IMMEDIATELY after completing ANY task, you MUST:**

1. **UPDATE TODO.md** - Change `- [•]` to `- [✓]` for the completed task
2. **UPDATE your memory file** - Record what was done
3. **UPDATE CURRENT_TASK.md** - Update your status or remove session if done

```
❌ WRONG: Complete task → Start next task
✅ RIGHT: Complete task → Update TODO.md → Start next task
```

**DO NOT proceed to the next task until TODO.md is updated.**

---

### Conflict Prevention Rules

1. **Never claim a task already claimed by another agent**

   - Check for `@{agent}` tag before claiming
   - If tagged, skip to next available task

2. **Never edit files another agent is working on**

   - Check `CURRENT_TASK.md` for active sessions
   - See which files each agent is modifying

3. **Use git as sync point**

   - Before starting: `git pull`
   - After completing task: commit and push (or notify user)

4. **Communicate via CURRENT_TASK.md**
   - All agents can read this file
   - Update your row when status changes

### Conflict Detection

If you detect a conflict:

```
⚠️ CONFLICT DETECTED

Task "[task name]" is already claimed by @{other-agent}
Session: #{session-id}
Started: [timestamp]

Options:
1) Work on a different unclaimed task
2) Wait for @{other-agent} to complete
3) Ask user to resolve

Choose (1/2/3):
```

---

## Session Start Protocol (MANDATORY)

When a new session starts, follow this decision tree:

### Step 1: Read Project State

1. Read `Implementation/PROJECT_CONTEXT.md` to get **Project Name** (if exists)
2. Read `Implementation/TODO.md` to check task state (any `- [ ]` entries?)
3. Read `Implementation/CURRENT_TASK.md` to check **active sessions**

### Step 2: Determine Response Based on Project State

```
┌─────────────────────────────────────────────────────────────────┐
│                     USER'S FIRST PROMPT                         │
│         "I want to create Online examination application"       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                   ┌──────────────────┐
                   │ Read TODO.md &   │
                   │ PROJECT_CONTEXT  │
                   └──────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
    ┌───────────┐       ┌───────────┐       ┌───────────┐
    │  No Tasks │       │  Project  │       │  Project  │
    │  (Empty)  │       │  Exists & │       │  Exists & │
    │           │       │ Different │       │  Matches  │
    └───────────┘       └───────────┘       └───────────┘
          │                   │                   │
          ▼                   ▼                   ▼
       CASE A              CASE B              CASE C
    Start Phase 1      Ask what to do     Offer options
```

---

#### Case A: Project NOT Initialized (No tasks in TODO.md)

Start **Phase 1: Planning** immediately.

```
Starting Phase 1: Planning for "[Project Idea]"

I'll create project documentation first, then get your confirmation before implementation.
```

- Create memory file: `.ai/memory/YYYYMMDD_HHMMSS_{agent}.md`
- Proceed to **Phase 1: Planning**

---

#### Case B: Project EXISTS but User Request is DIFFERENT Project

```
⚠️ Project exists with pending todos for a different application.

Existing Project: "[Existing Project Name]"
Pending Tasks: [count] tasks remaining

You requested: "[User's different project idea]"

What would you like to do?
```

- **Wait for user response** before proceeding
- User may choose to continue existing or start fresh

---

#### Case C: Project EXISTS and User Request MATCHES (Additional Feature)

```
Project exists with pending todos.

Project: "[Project Name]"
Pending Tasks: [count]
Last Completed: [task name]

You requested: "[User's request - additional feature]"

Options:
1) Add to TODO and implement AFTER pending tasks
2) Implement this FIRST, then proceed with pending tasks

Choose (1/2):
```

- **Wait for user choice** before proceeding
- Option 1: Add new feature to end of TODO.md, continue with pending
- Option 2: Add new feature to top priority, implement first

---

#### Case D: General Prompt (continue/hello/proceed)

**If project NOT initialized:**

```
Project not yet initialized. Please describe your project idea.
```

**If project EXISTS:**

```
Project: "[Project Name]"

Status:
- Phase: [current phase]
- Pending Tasks: [count]
- Next Available: [unclaimed task name]

Continuing from last session...
```

- Claim next available task and proceed

---

## Development Phases

### Phase 1: Planning (ALWAYS FIRST)

**Objective**: Create complete project documentation before any code.

⚠️ **Phase 1 should be done by ONE agent only** to avoid documentation conflicts.

#### 1.1 Gather Requirements

- Clarify project scope and goals
- Define core features (MVP)
- Confirm tech stack (or use defaults)

#### 1.2 Create Documentation

Update these files based on requirements:

| File                                   | Content                                            |
| -------------------------------------- | -------------------------------------------------- |
| `Implementation/PROJECT_CONTEXT.md`    | Project name, description, goals, tech stack       |
| `Implementation/ARCHITECTURE.md`       | System architecture, directory structure, diagrams |
| `Implementation/DECISIONS.md`          | Design decisions with rationale                    |
| `Implementation/PROJECT_MANAGEMENT.md` | Milestones, phases, timeline                       |
| `Implementation/TODO.md`               | Detailed task list organized by phases             |

**ARCHITECTURE.md MUST include:**
- Frontend: Next.js App Router, Turbopack (dev), PWA setup
- Backend: FastAPI structure, logging setup, static files
- Development tools and their configuration
- Directory structure for both services

**DECISIONS.md MUST include:**
- Why Next.js with Turbopack for development
- Why PWA with offline support
- Logging strategy (single rotating file, format)
- Authentication approach (superadmin, RBAC)
- Any deviations from default tech stack

#### 1.3 Update README.md

Replace template README with project-specific documentation:

```markdown
# [Project Name]

[Brief description]

## Features

- Feature 1
- Feature 2

## Tech Stack

### Frontend

- [Technologies]

### Backend

- [Technologies]

## Quick Start

### Prerequisites

[Required tools]

### Installation

[Setup steps]

## Development

### Running Locally

[Commands]

### Running Tests

[Test commands]

## Project Structure

[Directory layout]

## License

[License type]

---

_Built with [DevAI](https://github.com/atulmahankal/AI-Template)_
```

#### 1.4 Review & Confirm

```
📋 Phase 1: Planning Complete

Please review the following files:
- README.md
- Implementation/PROJECT_CONTEXT.md
- Implementation/ARCHITECTURE.md
- Implementation/DECISIONS.md
- Implementation/PROJECT_MANAGEMENT.md
- Implementation/TODO.md

Confirm to commit and proceed to Phase 2? (yes/no)
```

#### 1.5 Commit Planning Phase

After user confirmation:

```bash
git add .
git commit -m "feat: Phase 1 - Project planning and documentation"
```

⚠️ After Phase 1 commit, `Implementation/PROJECT_CONTEXT.md`, `ARCHITECTURE.md`, `DECISIONS.md`, and `PROJECT_MANAGEMENT.md` become **READ-ONLY**.

---

### Phase 2: Base Application

**Objective**: Create working foundation with error handling and logging.

✅ **Phase 2 can use multiple agents** - one for backend, one for frontend.

#### 2.1 Backend Base

Create `services/backend/` with:

- **App Structure**:

  ```
  services/backend/
  ├── app/
  │   ├── main.py           # FastAPI app entry
  │   ├── api/
  │   │   └── v1/
  │   │       └── health.py # Health & version endpoint
  │   ├── core/
  │   │   ├── config.py     # App configuration (reads from .env)
  │   │   ├── logging.py    # Rotating log setup
  │   │   └── exceptions.py # Custom exceptions
  │   └── schemas/
  │       └── response.py   # Response models
  ├── logs/                  # Auto-generated logs
  ├── .env.example           # Environment template (committed)
  ├── .env                   # Local environment (gitignored)
  ├── pyproject.toml
  └── Dockerfile
  ```

- **Required Endpoints**:

  - `GET /health` - Returns `{"status": "healthy", "app": "[AppName]", "version": "1.0.0"}`

- **Error Handling**:

  - Global exception handler
  - Custom exception classes
  - Consistent error response format

- **Logging**:

  - Rotating file logs in `./logs/`
  - Format: `YYYY-MM-DD_HH-MM-SS.log`
  - Log levels: DEBUG (dev), INFO (prod)

- **FastAPI App Configuration** (MANDATORY):

  ```python
  # Disable default docs to customize with favicon
  app = FastAPI(
      docs_url=None,
      redoc_url=None,
      title=settings.PROJECT_NAME,
      description="Project description here",
      version="1.0.0",
  )

  # Mount static files for favicon
  app.mount("/static", StaticFiles(directory="static"), name="static")

  # Custom Swagger UI with favicon
  @app.get("/docs", include_in_schema=False)
  async def custom_swagger_ui_html():
      return get_swagger_ui_html(
          openapi_url=app.openapi_url,
          title=app.title + " - API Docs",
          swagger_favicon_url="/static/favicon.ico",
      )

  # Custom ReDoc with favicon
  @app.get("/redoc", include_in_schema=False)
  async def custom_redoc_html():
      return get_redoc_html(
          openapi_url=app.openapi_url,
          title=app.title + " - ReDoc",
          redoc_favicon_url="/static/favicon.ico",
      )

  # Serve favicon directly
  @app.get("/favicon.ico", include_in_schema=False)
  async def favicon():
      return FileResponse("static/favicon.ico", media_type="image/x-icon")

  # Root redirect to docs
  @app.get("/")
  async def root():
      return RedirectResponse(url="/docs")

  # API redirect to docs
  @app.get("/api")
  async def api_redirect():
      return RedirectResponse(url="/docs")
  ```

- **Static Files Setup**:
  - Create `static/` directory in backend root
  - Add project-specific `favicon.ico` (or `favicon.svg`)
  - Favicon must appear in browser tab for `/docs`, `/redoc`, and all pages

#### 2.2 Frontend Base

Create `services/frontend/` using Next.js:

**Init Command:**
```bash
pnpm create next-app frontend --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

- **App Structure** (Next.js App Router):

  ```
  services/frontend/
  ├── src/
  │   ├── app/
  │   │   ├── layout.tsx       # Root layout
  │   │   ├── page.tsx         # Home/Landing page
  │   │   ├── not-found.tsx    # 404 page
  │   │   ├── error.tsx        # Error boundary
  │   │   ├── offline/
  │   │   │   └── page.tsx     # Offline fallback page
  │   │   └── globals.css      # Global styles
  │   ├── components/
  │   │   └── ErrorBoundary.tsx
  │   └── lib/
  │       └── logger.ts        # Frontend logging
  ├── public/
  │   ├── manifest.json        # PWA manifest
  │   ├── sw.js                # Service worker
  │   ├── favicon.ico
  │   └── icons/               # PWA icons
  ├── .env.example             # Environment template (committed)
  ├── .env.local               # Local environment (gitignored)
  ├── next.config.ts           # Next.js config
  ├── package.json
  └── Dockerfile
  ```

- **Required Pages**:

  - Landing page with project branding

- **Favicon**:

  - Update `favicon.ico` in `public/` folder with project-specific icon
  - Update page title in `index.html` to match project name

- **Error Handling**:

  - React Error Boundary
  - Global error handler
  - Toast notifications for user feedback

- **Logging**:

  - Console logging (dev)
  - Log to file for SSR apps

- **Next.js Configuration**:

  - Configure `next.config.ts` for production optimizations
  - Enable image optimization
  - Configure allowed hosts for local network access

- **Development Mode (Turbopack)**:

  Enable Turbopack for faster development with better debugging:

  ```json
  // package.json scripts
  {
    "scripts": {
      "dev": "next dev --turbo",
      "dev:debug": "NODE_OPTIONS='--inspect' next dev --turbo",
      "build": "next build",
      "start": "next start"
    }
  }
  ```

  **Turbopack Benefits:**
  - Faster hot module replacement (HMR)
  - Better error overlay with stack traces
  - Improved debugging experience
  - Real-time compilation feedback

  **Error Overlay Features:**
  - Shows compilation errors inline
  - Displays runtime errors with component stack
  - Click-to-open source file in editor
  - Clear error messages with suggestions

- **PM2 Configuration**:

  - Create `ecosystem.config.js` for pm2 deployment
  - Configure for production builds

- **PWA Configuration** (MANDATORY):

  Setup Progressive Web App with offline support:

  ```
  public/
  ├── manifest.json          # PWA manifest
  ├── sw.js                  # Service worker (or use next-pwa)
  └── icons/
      ├── icon-192x192.png
      └── icon-512x512.png
  ```

  **Required Offline Pages:**

  - Home page (cached for offline access)
  - "You are offline" fallback page

  **manifest.json:**

  ```json
  {
    "name": "Project Name",
    "short_name": "Project",
    "description": "Project description",
    "start_url": "/",
    "display": "standalone",
    "background_color": "#ffffff",
    "theme_color": "#000000",
    "icons": [
      {
        "src": "/icons/icon-192x192.png",
        "sizes": "192x192",
        "type": "image/png"
      },
      {
        "src": "/icons/icon-512x512.png",
        "sizes": "512x512",
        "type": "image/png"
      }
    ]
  }
  ```

  **Service Worker Strategy:**

  - Use `next-pwa` package for automatic SW generation
  - Cache static assets (JS, CSS, images)
  - Cache API responses where appropriate
  - Show offline page when network unavailable

  **Offline Page Requirements:**

  - Create `/offline` route with friendly "You are offline" message
  - Include retry button to check connection
  - Match app's design system

#### 2.3 Environment Configuration

Create `.env` files at three levels:

```
.
├── .env.example              # Root Docker environment template
├── .env                      # Root Docker environment (gitignored)
├── services/
│   ├── backend/
│   │   ├── .env.example      # Backend environment template
│   │   └── .env              # Backend environment (gitignored)
│   └── frontend/
│       ├── .env.example      # Frontend environment template
│       └── .env              # Frontend environment (gitignored)
```

**Root `.env.example`** (for Docker):

```env
# Docker Compose Environment
COMPOSE_PROJECT_NAME=appname

# Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=changeme
POSTGRES_DB=appname

# Service URLs (for inter-container communication)
BACKEND_URL=http://backend:8000
FRONTEND_URL=http://frontend:3000
```

**Docker injects environment variables** into services via `compose.yml`:

```yaml
services:
  backend:
    env_file:
      - .env
      - ./services/backend/.env
  frontend:
    env_file:
      - .env
      - ./services/frontend/.env
```

#### 2.4 Docker Setup

- Create `compose.yml` and `compose.dev.yml`
- Create service-level Dockerfiles
- Docker reads root `.env` and injects into services
- Verify both services start correctly

#### 2.5 Production Deployment Documentation

Create `./docs/ProductionDeployment.md` with:

- **Prerequisites**: Required tools, dependencies, system requirements
- **Quick Deploy**: One-command deployment steps
- **Manual Deployment**: Step-by-step manual deployment guide
- **Environment Setup**: Production environment variables
- **PM2 Commands**: Process management commands

#### 2.6 Commit Phase 2

```bash
git add .
git commit -m "feat: Phase 2 - Base application with error handling and logging"
```

---

### Phase 3: Core Features

**Objective**: Implement project-specific features.

✅ **Multiple agents can work in parallel** - assign different features to different agents.

#### 3.1 Authentication (if required)

- User registration/login
- JWT token management
- Password hashing (bcrypt)
- Session management

**⚠️ MANDATORY Authentication Rules:**

1. **Superadmin Creation Command**:

   - Add CLI command to create superadmin user (e.g., `python manage.py createsuperadmin`)
   - Document command usage in `./docs/ProductionDeployment.md`

2. **RBAC Seed Data**:

   - Default permissions must be seeded during database migration
   - Default roles must be seeded during database migration
   - Create seed/migration script for initial RBAC setup

3. **Superadmin Validation**:
   - Backend: All auth endpoints must check if superadmin exists
   - Frontend: Auth pages must verify superadmin existence on load
   - If no superadmin exists, throw error:
     ```
     "No Superadmin exists. Read Implementation details to create it."
     ```
   - Block all authentication operations until superadmin is created

#### 3.2 Two-Factor Authentication (if required)

- TOTP setup
- Backup codes
- Device trust

#### 3.3 Additional Features

Implement features as defined in `Implementation/TODO.md`:

- Each feature should be a separate task
- Write tests for each feature
- Update README.md as needed
- Commit after completing each major feature

---

## Phase Completion Protocol

### ⛔ COMMIT GATE (MANDATORY)

**You MUST NOT start the next phase until the current phase is committed.**

```
┌──────────────────────────────────────────────────────────────┐
│  ⛔ STOP: PHASE TRANSITION CHECKPOINT                        │
│                                                              │
│  Before proceeding to Phase [N+1], verify:                   │
│                                                              │
│  □ All Phase [N] tasks marked [✓] in TODO.md                 │
│  □ CHANGELOG.md updated                                      │
│  □ Tests pass (if applicable)                                │
│  □ User has confirmed commit                                 │
│  □ Git commit completed                                      │
│                                                              │
│  DO NOT PROCEED until all boxes are checked.                 │
└──────────────────────────────────────────────────────────────┘
```

### Phase Completion Steps

1. **VERIFY** all phase tasks are marked `[✓]` in `Implementation/TODO.md`
2. **REMOVE** your session from `Implementation/CURRENT_TASK.md`
3. **UPDATE** `CHANGELOG.md` in the `[Unreleased]` section
4. **RUN** tests and verify they pass
5. **PROMPT** user for commit confirmation:

   ```
   📋 Phase [N] Complete - Commit Required

   All tasks completed:
   - [✓] Task 1
   - [✓] Task 2
   ...

   Suggested commit message:
   "feat: complete Phase [N] - [phase description]"

   ⛔ WAITING FOR COMMIT CONFIRMATION

   Please confirm to commit (yes/no):
   ```

6. **WAIT** for user response - DO NOT proceed
7. **COMMIT** only after user confirms
8. **THEN** proceed to next phase

---

## Workflow Commands

| Command    | Action                                              |
| ---------- | --------------------------------------------------- |
| `continue` | Continue from last session (Case D)                 |
| `proceed`  | Same as continue                                    |
| `status`   | Show current project status including active agents |

---

## File Permissions

| Path                                   | Permission     | Description                              |
| -------------------------------------- | -------------- | ---------------------------------------- |
| `.ai/AI_INSTRUCTIONS.md`               | read-only      | Workflow rules (this file)               |
| `.ai/CODING_GUIDELINES.md`             | read-only      | Code style guidelines                    |
| `.ai/TESTING_GUIDELINES.md`            | read-only      | Testing standards                        |
| `.ai/OPERATIONAL_GUIDELINES.md`        | read-only      | DevOps procedures                        |
| `.ai/memory/*.md`                      | session file   | Create new per session (with agent name) |
| `Implementation/TODO.md`               | editable       | Task list with agent claims              |
| `Implementation/CURRENT_TASK.md`       | editable       | Active sessions registry                 |
| `Implementation/PROJECT_CONTEXT.md`    | Phase 1 only   | Read-only after planning                 |
| `Implementation/ARCHITECTURE.md`       | Phase 1 only   | Read-only after planning                 |
| `Implementation/DECISIONS.md`          | Phase 1 only   | Read-only after planning                 |
| `Implementation/PROJECT_MANAGEMENT.md` | Phase 1 only   | Read-only after planning                 |
| `README.md`                            | editable       | Update on feature completion             |
| `CHANGELOG.md`                         | editable       | Update on phase completion               |
| `services/**`                          | editable       | Application code (check claims first!)   |
| `logs/**`                              | auto-generated | Log files                                |

---

## Error Handling Requirements

### Backend

```python
# Response format for all errors
{
    "success": false,
    "error": {
        "code": "ERROR_CODE",
        "message": "Human readable message",
        "details": {}  # Optional additional info
    }
}
```

### Frontend

- Error Boundary for component errors
- Toast notifications for API errors
- Console logging for development

---

## Logging Requirements

### Log Entry Format (MANDATORY for both Frontend & Backend)

```
<timestamp> <LEVEL> : <message>
```

**Example:**
```
2024-12-30 14:30:22 DEBUG : Database connection established
2024-12-30 14:30:23 INFO : User login successful for user_id=123
2024-12-30 14:30:25 WARNING : Rate limit approaching for IP 192.168.1.1
2024-12-30 14:30:30 ERROR : Failed to process payment: timeout
```

### Backend (`./logs/`)

```python
# File: services/backend/app/core/logging.py

# Rotating logs configuration
LOG_DIR = "./logs/"
LOG_FILE = "app.log"           # Single rotating log file
MAX_SIZE = 10 * 1024 * 1024    # 10MB rotation
BACKUP_COUNT = 30              # Keep 30 days of logs

# Log format
LOG_FORMAT = "%(asctime)s %(levelname)s : %(message)s"
DATE_FORMAT = "%Y-%m-%d %H:%M:%S"

# Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
# Development: DEBUG
# Production: INFO
```

**Backend Logging Setup:**
- Use Python's `logging.handlers.RotatingFileHandler`
- Create `./logs/` directory if not exists
- Single `app.log` file with rotation
- All log levels in same file with level prefix

### Frontend (`./logs/`)

```typescript
// File: services/frontend/src/lib/logger.ts

// Rotating logs configuration (for SSR/server-side)
const LOG_DIR = "./logs/";
const LOG_FILE = "app.log";    // Single rotating log file
const MAX_SIZE = 10 * 1024 * 1024;  // 10MB rotation

// Log format
// <timestamp> <LEVEL> : <message>

// Log levels: DEBUG, INFO, WARNING, ERROR
```

**Frontend Logging Setup:**
- Development: Console logging with same format
- Production (SSR): Write to `./logs/app.log` with rotation
- Use libraries like `winston` or `pino` for file logging
- All log levels in same file with level prefix

**Example Logger Implementation:**
```typescript
const logger = {
  debug: (msg: string) => log("DEBUG", msg),
  info: (msg: string) => log("INFO", msg),
  warning: (msg: string) => log("WARNING", msg),
  error: (msg: string) => log("ERROR", msg),
};

function log(level: string, message: string) {
  const timestamp = new Date().toISOString().replace("T", " ").slice(0, 19);
  const entry = `${timestamp} ${level} : ${message}`;
  // Write to console (dev) or file (prod)
}
```

---

## Memory Management

### Memory File Rules

1. **CREATE** new file each session: `.ai/memory/YYYYMMDD_HHMMSS_{agent}.md`
2. **UPDATE** after completing tasks
3. **UPDATE** before context exhaustion
4. Previous sessions are READ-ONLY
5. Include agent name in filename for multi-agent support

### Memory File Structure

```markdown
# Session: YYYY-MM-DD HH:MM:SS

Agent: {agent_name}

## Status

- Phase: [current phase]
- Task Claimed: [task with @tag]
- Last Completed: [task]
- Current: [task]
- Next: [task]

## Active Session Info

- Session ID: YYYYMMDD-HHMMSS
- Files Being Modified:
  - services/backend/app/main.py
  - services/backend/app/core/logging.py

## Completed This Session

- [✓] Task 1 - notes
- [✓] Task 2 - notes

## In Progress

- [•] Current task - state, blockers

## Key Decisions

- Decision: rationale

## Next Steps

- Step 1
- Step 2

## Handoff Notes

<!-- Notes for next session or other agents -->
```

---

## Context Files to Read

When continuing a session, read these files in order:

1. `Implementation/CURRENT_TASK.md` - Check active sessions first
2. `Implementation/TODO.md` - Task state and claims
3. `Implementation/PROJECT_CONTEXT.md` - Project overview
4. Your latest `.ai/memory/*_{agent}.md` - Your previous session context

---

## Package Manager Commands

### Frontend (pnpm)

```bash
pnpm install           # Install dependencies
pnpm add <package>     # Add package
pnpm add -D <package>  # Add dev dependency
pnpm dev               # Start dev server
pnpm build             # Build for production
pnpm test              # Run tests
```

### Backend (uv)

```bash
uv sync                # Sync dependencies
uv add <package>       # Add package
uv add --dev <package> # Add dev dependency
uv run uvicorn app.main:app --reload  # Start dev server
uv run pytest          # Run tests
```

---

## Docker Commands

```bash
# Development
docker compose -f compose.yml -f compose.dev.yml up -d

# Production
docker compose up -d

# Rebuild
docker compose up -d --build

# Logs
docker compose logs -f

# Stop
docker compose down
```

---

_These instructions are read-only. All AI agents must follow this workflow._

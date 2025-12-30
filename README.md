# DevAI

A universal template for AI-assisted software development. Work with **multiple AI coding assistants simultaneously** on the same project with consistent behavior, shared context, and conflict-free collaboration.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![CI](https://github.com/atulmahankal/AI-Template/actions/workflows/ci.yml/badge.svg)](https://github.com/atulmahankal/AI-Template/actions/workflows/ci.yml)
[![Security](https://github.com/atulmahankal/AI-Template/actions/workflows/security.yml/badge.svg)](https://github.com/atulmahankal/AI-Template/actions/workflows/security.yml)

## Why DevAI?

- **Multi-Agent Support**: Use Claude, Gemini, Copilot, Cursor, Codex, OpenCode, Windsurf, and Aider together
- **Concurrent Sessions**: Multiple agents can work simultaneously without conflicts
- **Task Claiming**: Agents claim tasks with `@agent` tags to prevent overlap
- **Session Tracking**: `CURRENT_TASK.md` shows all active agent sessions
- **Memory Per Agent**: Each agent maintains its own session memory
- **Structured Phases**: Planning → Base App → Features workflow
- **Production-Ready**: CI/CD, security scanning, code quality tools pre-configured

## Supported AI Tools

| Tool | Config File | Agent Tag |
|------|-------------|-----------|
| [Claude Code](https://claude.ai/claude-code) | `CLAUDE.md` | `@claude` |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | `GEMINI.md` | `@gemini` |
| [GitHub Copilot](https://github.com/features/copilot) | `.github/copilot-instructions.md` | `@copilot` |
| [Cursor](https://cursor.sh) | `.cursorrules` | `@cursor` |
| [OpenCode](https://github.com/sst/opencode) | `.opencode/config.json` | `@opencode` |
| [Windsurf](https://codeium.com/windsurf) | `.windsurfrules` | `@windsurf` |
| [Aider](https://aider.chat) | `.aider.conf.yml` | `@aider` |
| Other agents | [`AGENTS.md`](https://agents.md/) | `@{agent}` |

> **Note**: `AGENTS.md` follows the [agents.md standard](https://agents.md/) supported by 60,000+ projects. Tools without specific config files (Codex, Devin, Factory, etc.) use this file.

## Quick Start

### 1. Create from Template

```bash
# Option A: Use GitHub template
# Click "Use this template" on GitHub

# Option B: Clone and reinitialize
git clone https://github.com/atulmahankal/AI-Template.git my-project
cd my-project
rm -rf .git && git init
```

### 2. Start Your AI Assistant(s)

```bash
# Terminal 1
claude

# Terminal 2 (optional - parallel work)
cursor .

# Terminal 3 (optional - more agents)
gemini
```

### 3. Describe Your Project

```
> I want to create Online examination application
```

The AI will start **Phase 1: Planning** and create all documentation before implementation.

## Multi-Agent Workflow

### How Concurrent Sessions Work

```
┌──────────────────────────────────────────────────────────────────┐
│                    CURRENT_TASK.md (Active Sessions)              │
├──────────────────────────────────────────────────────────────────┤
│ Agent   │ Session ID      │ Task                    │ Status     │
├─────────┼─────────────────┼─────────────────────────┼────────────┤
│ claude  │ 20241229-143022 │ Create FastAPI structure│ In Progress│
│ cursor  │ 20241229-143200 │ Create React structure  │ In Progress│
│ gemini  │ 20241229-143300 │ Write unit tests        │ In Progress│
└──────────────────────────────────────────────────────────────────┘
```

### Task Claiming in TODO.md

```markdown
## Phase 2: Base Application

### Backend
- [ ] Create FastAPI structure `@claude` `#20241229-143022`
- [ ] Add health endpoint                                    ← Available
- [x] Setup logging ~~@gemini~~ ~~#20241229-140000~~        ← Completed

### Frontend
- [ ] Create React structure `@cursor` `#20241229-143200`
- [ ] Add landing page                                       ← Available
```

### Memory Files Per Agent

```
.ai/memory/
├── 20241229_143022_claude.md   # Claude's session
├── 20241229_143200_cursor.md   # Cursor's session
└── 20241229_143300_gemini.md   # Gemini's session
```

### Conflict Prevention

1. **Check Active Sessions**: Read `CURRENT_TASK.md` before starting
2. **Claim Tasks**: Add `@agent` tag before working
3. **Avoid File Conflicts**: Don't edit files another agent is modifying
4. **Register Session**: Add yourself to active sessions table
5. **Clean Up**: Remove entry when done

## Session Start Protocol

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

### Case Responses

| Case | Scenario | AI Response |
|------|----------|-------------|
| **A** | No project (empty TODO) | Start Phase 1: Planning immediately |
| **B** | Different project exists | "Project exists with pending todos for different app. What to do?" |
| **C** | Same project (add feature) | "Options: 1) Add & implement after pending 2) Implement first" |

## Development Phases

### Phase 1: Planning (Single Agent)

- Prepare `Implementation/*.md` (architecture, decisions, todos)
- Update `README.md` with project-specific documentation
- Get user confirmation before proceeding
- **Commit**: `feat: Phase 1 - Project planning and documentation`

### Phase 2: Base Application (Multi-Agent OK)

| Component | Requirements |
|-----------|--------------|
| **Backend** | App name + version endpoint (`GET /health`), error handling, rotating logs in `./logs` |
| **Frontend** | Landing page, error boundary, logging |
| **Both** | App-level error handling, store rotating logs in `./logs` |

- **Commit**: `feat: Phase 2 - Base application`

### Phase 3: Core Features (Multi-Agent OK)

- Authentication, TFA, and other features
- Each agent claims their task with `@agent` tag
- **Commit**: Per feature completion

## Directory Structure

```
.
├── .ai/                          # AI Configuration
│   ├── AI_INSTRUCTIONS.md        # Master workflow (read-only)
│   ├── CODING_GUIDELINES.md      # Code standards
│   ├── TESTING_GUIDELINES.md     # Testing standards
│   ├── OPERATIONAL_GUIDELINES.md # DevOps procedures
│   ├── SECURITY_GUIDELINES.md    # Security best practices
│   └── memory/                   # Per-agent session files
│
├── .github/                      # GitHub Configuration
│   ├── workflows/
│   │   ├── ci.yml                # CI pipeline (lint, test, build)
│   │   └── security.yml          # Security scanning
│   ├── dependabot.yml            # Dependency updates
│   ├── ISSUE_TEMPLATE/           # Issue templates
│   └── PULL_REQUEST_TEMPLATE.md  # PR template
│
├── Implementation/               # Project Planning & Tracking
│   ├── CURRENT_TASK.md           # Active sessions registry
│   ├── TODO.md                   # Task list with @agent claims
│   ├── PROJECT_CONTEXT.md        # Project overview
│   ├── ARCHITECTURE.md           # System architecture
│   ├── DECISIONS.md              # Design decisions
│   └── PROJECT_MANAGEMENT.md     # Milestones
│
├── services/                     # Application Code
│   ├── backend/                  # FastAPI application
│   │   ├── app/                  # Python source code
│   │   ├── pyproject.toml        # Python dependencies + mypy config
│   │   └── ruff.toml             # Python linting config
│   └── frontend/                 # React application
│       ├── src/                  # TypeScript source code
│       ├── eslint.config.js      # ESLint config
│       ├── .prettierrc           # Prettier config
│       └── package.json          # Node dependencies
│
├── .pre-commit-config.yaml       # Pre-commit hooks
├── .secrets.baseline             # Secret detection baseline
├── Agent Configs                 # 8+ AI agent config files
└── README.md                     # This file
```

## Commands

| Command | Action |
|---------|--------|
| `continue` | Continue from last session |
| `status` | Show active agents and available tasks |

## Best Practices for Multi-Agent

1. **Assign by Specialty**:
   - Claude: Complex logic, backend
   - Cursor: UI/UX, frontend
   - Aider: Refactoring, tests

2. **Communicate via Files**:
   - Use `CURRENT_TASK.md` for status
   - Add handoff notes in memory files

3. **Sync with Git**:
   - Commit after completing tasks
   - Pull before starting new work

4. **Avoid Conflicts**:
   - One agent per file at a time
   - Check active sessions before editing

## CI/CD & Security

### Automated Pipelines

| Workflow | Trigger | What it does |
|----------|---------|--------------|
| **CI** | Push/PR | Lint, test, build frontend & backend |
| **Security** | Push/PR/Weekly | CodeQL, secret scanning, dependency audit, container scan |

### Code Quality Tools

| Tool | Purpose | Config File |
|------|---------|-------------|
| **ESLint** | TypeScript/React linting | `services/frontend/eslint.config.js` |
| **Prettier** | Code formatting | `services/frontend/.prettierrc` |
| **Ruff** | Python linting & formatting | `services/backend/ruff.toml` |
| **mypy** | Python type checking | `services/backend/pyproject.toml` |
| **pre-commit** | Git hooks | `.pre-commit-config.yaml` |

### Security Features

- **Secret Detection**: Gitleaks + detect-secrets in pre-commit hooks
- **SAST**: Semgrep scans for OWASP vulnerabilities
- **Container Scanning**: Trivy scans Docker images
- **Dependency Audit**: Automated vulnerability checks
- **Dependabot**: Weekly dependency updates with grouping

### Setup Pre-commit Hooks

```bash
pip install pre-commit
pre-commit install
```

### Guidelines

| Document | Content |
|----------|---------|
| `.ai/SECURITY_GUIDELINES.md` | OWASP Top 10, auth, input validation, rate limiting |
| `.ai/CODING_GUIDELINES.md` | Code style, patterns, error handling |
| `.ai/TESTING_GUIDELINES.md` | Test types, coverage requirements, mocking |
| `.ai/OPERATIONAL_GUIDELINES.md` | DevOps, deployment, monitoring |

## Customization

1. **Tech Stack**: Modify `Implementation/PROJECT_CONTEXT.md`
2. **Code Standards**: Review `.ai/CODING_GUIDELINES.md`
3. **Security Policies**: Review `.ai/SECURITY_GUIDELINES.md`
4. **Local Setup**: Copy `.ai/AI_INSTRUCTIONS.local.md.example` to `.ai/AI_INSTRUCTIONS.local.md`

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT - See [LICENSE](LICENSE)

---

_Built with AI, for AI-assisted development._

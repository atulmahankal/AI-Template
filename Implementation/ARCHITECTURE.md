# System Architecture

## Overview

<!-- High-level system overview -->

## Architecture Diagram

```
<!-- Add architecture diagram using Mermaid or ASCII -->
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Frontend Application                        │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Reverse Proxy / Gateway                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        API Layer                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Backend Service                       │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Data Layer                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐   │
│  │    Database     │  │      Cache      │  │  File Storage  │   │
│  └─────────────────┘  └─────────────────┘  └────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
project/
├── services/                    # All services
│   ├── backend/                 # Backend Service
│   │   ├── app/
│   │   │   ├── api/             # API route handlers
│   │   │   ├── core/            # Core functionality
│   │   │   ├── models/          # Database models
│   │   │   ├── schemas/         # Data schemas
│   │   │   ├── services/        # Business logic
│   │   │   └── tests/           # Test files
│   │   ├── logs/                # Log files (gitignored)
│   │   ├── .env                 # Service-specific env
│   │   ├── .dockerignore
│   │   ├── Dockerfile
│   │   └── compose.yml
│   │
│   ├── frontend/                # Frontend Application
│   │   ├── src/
│   │   │   ├── components/      # Reusable UI components
│   │   │   ├── pages/           # Route pages
│   │   │   ├── hooks/           # Custom hooks
│   │   │   ├── services/        # API client functions
│   │   │   ├── stores/          # State stores
│   │   │   ├── types/           # Type definitions
│   │   │   ├── utils/           # Utility functions
│   │   │   └── tests/           # Test files
│   │   ├── public/              # Static assets
│   │   ├── .env                 # Service-specific env
│   │   ├── .dockerignore
│   │   ├── Dockerfile
│   │   └── compose.yml
│   │
│   ├── database/                # Database service
│   │   └── compose.yml
│   │
│   └── cache/                   # Cache service
│       └── compose.yml
│
├── nginx/                       # Reverse proxy
│   ├── nginx.conf
│   ├── certs/                   # SSL certs (gitignored)
│   └── .gitignore
│
├── docs/                        # Documentation
├── .ai/                         # AI configuration
├── Implementation/              # Task tracking
├── .env                         # Root environment variables
├── compose.yml                  # Production orchestration
├── compose.dev.yml              # Dev overrides
└── README.md
```

## Data Flow

### Primary Flow

```
User Request → Gateway → API → Service → Database → Response
```

### Authentication Flow

```
Login Request → Validate Credentials → Generate Token → Store Session → Return Token
```

## Integration Points

### External Services

## <!-- APIs, third-party services -->

### Internal Integrations

## <!-- Database, cache, message queues -->

## API Versioning

- API routes use version prefix (e.g., `/v1/`)
- Version incremented for breaking changes only
- Deprecation notices provided in advance

---

_Last Updated: <!-- date -->_

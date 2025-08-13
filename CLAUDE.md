# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DevPocket Server is a production-ready FastAPI backend for a mobile-first cloud IDE. It provides secure, scalable development environments accessible from mobile devices. The system manages user authentication, development environments (containers), and real-time terminal access via WebSockets.

## Architecture

The application follows a layered architecture with clear separation of concerns:

```
app/
├── main.py              # FastAPI application entry point with middleware
├── core/                # Core infrastructure components
│   ├── config.py        # Pydantic settings with environment variables
│   ├── database.py      # MongoDB async connection and indexing
│   ├── security.py      # JWT, password hashing, security headers
│   └── logging.py       # Structured logging with structlog
├── models/              # Pydantic data models
│   ├── user.py          # User, authentication, and subscription models
│   └── environment.py   # Development environment and resource models
├── services/            # Business logic layer
│   ├── auth_service.py  # User authentication, Google OAuth, account lockout
│   └── environment_service.py  # Environment lifecycle, WebSocket sessions
├── api/                 # HTTP/WebSocket route handlers
│   ├── auth.py          # Authentication endpoints (register, login, OAuth)
│   ├── environments.py  # Environment CRUD operations
│   └── websocket.py     # Real-time terminal and log streaming
└── middleware/          # Request/response middleware
    ├── auth.py          # JWT token validation and user context
    └── rate_limiting.py # In-memory rate limiting
```

## Key Architectural Patterns

**Dependency Injection**: FastAPI's `Depends()` system is used throughout for database connections, authentication, and service injection. Services require `set_database()` calls to initialize database connections.

**Async/Await**: All I/O operations are asynchronous using Motor (async MongoDB driver) and httpx for external API calls.

**Service Layer Pattern**: Business logic is separated into service classes (`auth_service`, `environment_service`) that are injected into API routes.

**Settings Management**: Configuration uses Pydantic Settings with environment variable support. Settings are centralized in `app.core.config.Settings`.

**Security Middleware Chain**: Multiple middleware layers handle security (rate limiting, CORS, security headers) before requests reach route handlers.

## Tmux Session Management

The application now uses tmux for persistent terminal sessions in development environments:

**Session Architecture**: Each environment runs a tmux session that persists across WebSocket disconnections and container restarts (via persistent volumes).

**Session Lifecycle**:
- Sessions are created when environments start and stored with format `devpocket_{environment_id}`
- Multiple WebSocket connections can attach to the same session
- Sessions persist in `/home/devpocket/.tmux/` on persistent storage
- Automatic session recovery on reconnection

**ConfigMap-based Initialization**: Startup scripts are now stored in Kubernetes ConfigMaps instead of deployment commands, preventing pod crashes from script errors.

**Template Management**: Templates are now stored as YAML files in `./scripts/templates/` and can be loaded into the database using `scripts/load_templates.py`.

## Development rules

- always update the related docs in `./docs` folder if the code changes affect the docs
- use `source venv/bin/activate` to activate the virtual environment before running any command

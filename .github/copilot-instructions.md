# Copilot Instructions

## Project Overview

Movie showtime website for Colombia called "Pa' Cine".

## Architecture

```
cine_medallo_2/
├── .github/             # Copilot instructions and GitHub config
├── backend/             # Django backend (Python)
│   ├── config/          # Django project settings
│   ├── movies_app/      # Core app
│   │   ├── models/      # Theater, Movie, Showtime, OperationalIssue, etc.
│   │   ├── services/    # TMDBService, MovieLookupService, SupabaseStorageService
│   │   ├── tasks/       # Celery tasks
│   │   ├── management/  # Management commands
│   │   └── utils/       #
│   ├── scripts/         # Setup and utility scripts
│   └── seed_data/       # theaters.json and other seed data
├── frontend/            # Next.js frontend (TypeScript/React)
│   ├── src/app/         # App Router pages and API routes
│   │   ├── admin/       # Admin dashboard pages
│   │   ├── api/         # API routes (Route Handlers)
│   │   ├── movies/      # Movie pages
│   │   ├── cines/       # Theater pages
│   │   └── layout.tsx   # Root layout
│   ├── src/components/
│   └── src/lib/         # Supabase client, type definitions
└── docs/                # Requirements and documentation
```

### Architecture Notes

* The Django code does not expose an API. The Next.js frontend accesses the database directly via Supabase.
- Admin dashboard is in frontend code (not Django admin)

---

# Git and workflow guidelines

- Create feature branches from `main` for new work (e.g., `git checkout -b feature/scrape-cine-colombia-api`).
    - Do not work directly on `main`.
    - Feature branches should start with `feature/` prefix.
- Do not commit until the user tells you to commit code.
- Never push to github unless the user explicitly tells you to push.
- When fixing PR feedback, always create a new commit rather than amending or squashing. This preserves the history of changes and feedback.
- Communicate with the user about design decisions and trade-offs before implementation.
- Communicate with the user about requirements decisions before implementation and after implementation.


## Copilot-instructions updates
Update this file when:
- You discover a solution that future agents would also get stuck on
- You find incorrect information that needs correction


---


# Frontend (Next.js)

## Tech Stack
- **Framework**: Next.js 15 with TypeScript
- **Styling**: Tailwind CSS 3
- **Database**: Supabase (hosted PostgreSQL)
- **UI Components**: Shadcn and Radix UI primitives


## Before Committing


Always run these before committing and fix any issues:
```bash
cd frontend
npm run lint         # ESLint linting
npx tsc --noEmit     # TypeScript type checking
npm run build:local  # Build without env validation (catches all errors)
```

## Key Files
- `src/lib/supabase.ts` - Database client and type definitions


## Conventions
- User-facing text: Spanish
- Admin dashboard: English
- Use slugs for lookups and URLs (e.g., `/movies/el-brutalista`), not names
- Timestamps use Colombia timezone context
- Reusable Tailwind styles: use `@apply` in CSS instead of repeating utility classes

---

# Backend (Django)

## Tech Stack
Django 5.2 · Celery 5.4 · Supabase (hosted PostgreSQL) · Redis · Python 3.13

## Essential Commands
```bash
cd backend
./scripts/setup.sh   # First-time setup (installs uv, deps, Camoufox)
source .venv/bin/activate
ruff check .
pyright
pytest
```

## Hard Rules

* Clear, high quality code is more important than speed of implementation. Take the time to do it right.
* Use OOP—classes, not free functions; composition over inheritance
* No default values in method parameters
* Never swallow exceptions—log errors properly
* **NO inline imports**—all imports at top of file. The only exception is to avoid circular imports.
* Avoid synchronous calls in asynchronous code (use await for all I/O operations)
* Run pre-commit checks (see Testing section) before committing
* Generate migrations before pushing a PR
* If blocked, ask user—don't claim work complete when blocked

## Code Architecture

### Size Limits

These limits apply to production code (not test fixtures or data).

* **Functions: 40 lines max** (excluding docstring). If a function exceeds this, decompose it into smaller functions with clear names.
* **Classes: 400 lines max.** If a class exceeds this, extract a collaborator class that handles a distinct responsibility. Excluded from the class size limit: management commands (`commands/`) and scraper task classes (`tasks/`). These classes are inherently large due to chain-specific logic, but their individual methods must still respect the function size limit.
* **Constructor dependencies: 4 max.** More than 4 injected dependencies signals the class is doing too much.
* **Nesting depth: 3 levels max.** Use guard clauses and early returns to flatten conditional logic.

When adding new logic, check whether the target function or class is already near its limit. If it is, extract first, then add.

### Single Responsibility

Every class and function should have one reason to change. Common violations to avoid:

* **Don't mix parsing and persistence.** A method that parses HTML/API data should return a data structure. A separate method or service should handle saving to the database.
* **Don't mix orchestration and implementation.** An orchestrator method calls other methods in sequence. It should not contain the low-level logic itself.
* **Don't add "one more thing" to an existing method.** If a method does X and you need to also do Y, extract Y into its own method. Do not append Y to the bottom of the X method.

### Dependency Management

* **Inject dependencies through the constructor**, not by importing and instantiating them inside methods. This makes classes testable and their dependencies explicit.
* **Depend on the narrowest interface you need.** If a method only needs to search TMDB, accept `TMDBService`, not the entire `MovieLookupService`.
* **Keep the dependency graph shallow.** If class A depends on B depends on C depends on D, consider whether A should depend on C or D directly instead of reaching through B.

### When Modifying Existing Code

Before adding code to an existing function or class, ask:

1. **Does this belong here?** If the new logic serves a different purpose than the existing code, it belongs in a new function or class.
2. **Will this push the function/class over its size limit?** If yes, extract existing logic first to make room.
3. **Am I increasing this class's number of responsibilities?** If yes, extract a new collaborator class instead.

## Typing Conventions

* Use `from __future__ import annotations` at top of files
* Use `TYPE_CHECKING` block to avoid circular imports:
  ```python
  from typing import TYPE_CHECKING
  if TYPE_CHECKING:
      from movies_app.services.tmdb_service import TMDBService
  ```
* Use `@dataclass` for DTOs and intermediate data structures
* Return type hints should be included in all functions, even if the return type is None

## Comments and Docstrings

* Short methods under 10 lines should not need a docstring
* Medium length methods (10 to 50 lines) should have a docstring describing the overall purpose and any non-obvious behavior
* Long methods (>50 lines) must be decomposed into smaller methods (see Size Limits above). If decomposition is genuinely not possible, they must have a docstring and inline comments explaining the logic
* Write additional comments only for non-obvious behavior or important constraints
* Do not describe what the code does; describe why if unclear
* Logger statements ending in "\n\n" are fine to keep log separation clear
* Python classes with functions should have docstrings of 5 to 20 lines describing their purpose and any important details about usage or behavior
* Return type should be commented


## Async/Sync Best Practices

Scrapers use async code for browser automation with Camoufox. Follow these rules to avoid runtime errors:

### Use `run_async_safely()` Instead of `asyncio.run()`

**NEVER** use `asyncio.run()` directly. It fails when called from an already-running event loop:
```python
# BAD - will fail with "RuntimeError: This event loop is already running"
def download_page(url: str) -> str:
    return asyncio.run(_fetch_page_async(url))

# GOOD - handles both sync and async contexts
from movies_app.tasks.download_utilities import run_async_safely

def download_page(url: str) -> str:
    return run_async_safely(_fetch_page_async(url))
```

### Why This Matters

- Celery tasks may run in contexts with existing event loops
- Async tests use `@pytest.mark.asyncio` which creates an event loop
- `run_async_safely()` uses `asyncio.run()` when no loop is running, or runs the coroutine in a separate thread with its own event loop when one already exists

### Testing Async Code

Add async safety tests to catch `asyncio.run()` usage. Example test pattern:
```python
@pytest.mark.django_db
class TestAsyncSyncSafety:
    async def test_download_can_be_called_from_async_context(self):
        """Catches asyncio.run() bug - would fail with 'event loop already running'."""
        from unittest.mock import patch, AsyncMock

        with patch.object(
            MyScraperClass,
            "_async_method",
            new_callable=AsyncMock,
            return_value="<html></html>",
        ):
            # This should NOT raise RuntimeError
            result = MyScraperClass.download_method("https://example.com")
            assert result == "<html></html>"
```

### Django ORM in Async Context

When calling Django ORM operations from async functions, use `sync_to_async`:
```python
from asgiref.sync import sync_to_async

# Inside an async function:
await sync_to_async(OperationalIssue.objects.create)(
    name="Issue Name",
    task=TASK_NAME,
    error_message=str(e),
)
```

## Services

Located in `movies_app/services/`:
- `tmdb_service.py` — TMDB API integration, returns dataclass DTOs
- `movie_lookup_service.py` — Orchestrates movie matching: check existing → search TMDB → match → create
- `existing_movie_lookup_service.py` — Fuzzy matching against movies already in the database
- `tmdb_persist_movie_service.py` — Searches TMDB and persists results (movie creation, aliases, unfindable URLs)
- `tmdb_movie_lookup_service.py` — TMDB search and candidate retrieval
- `tmdb_movie_matcher.py` — Scoring algorithm for selecting the best TMDB match
- `ai_movie_lookup_service.py` — AI-assisted movie matching for difficult cases
- `ai_providers.py` — AI provider abstraction for movie matching
- `showtime_saver_service.py` — Saves showtimes with deduplication and cleanup
- `supabase_storage_service.py` — Image upload to Supabase storage
- `time_service.py` — Centralized Colombia timezone operations (single source of truth)
- `theater_sync_service.py` — Syncs theater data from chain websites
- `betterstack_service.py` — BetterStack logging integration

## Management Commands

Located in `movies_app/management/commands/`:
- Each scraper has `scrape_{source}.py`
- Standard arguments: `--list` (list theaters), `--theater <slug>` (scrape one theater)
- **Include verbose docstrings** with usage examples at the top of command files

---

## Testing

Pre-commit checks:
```bash
ruff check .
pyright
python manage.py check
python manage.py makemigrations --check --dry-run
pytest
```

* Use pytest fixtures from `conftest.py`—not `Model.objects.create()`
* Run tests with `pytest -v <file> -k <name>` (NOT `python -m pytest`)
* Run the associated management command to manually verify before completing work
* New features require success and failure tests
* Write integration tests in addition to unit tests for complex logic
* Tests use settings_test.py, not the standard settings.py

**Scraper test patterns:**
- Use HTML snapshots in `html_snapshot/` directories for deterministic tests
- Helper function: `load_html_snapshot(filename)` to load test HTML
- Auto-mock fixtures in `conftest.py`: `mock_supabase_storage`, `mock_tmdb_service`
- Use `get_or_create` in fixtures for idempotent test data

## Error Handling

* Fail fast—only catch exceptions if recovery is possible
* Let unexpected exceptions propagate (don't silently handle)
* Only check for None if it's expected; crash on unexpected None
* Always log when catching exceptions
* Log operational issues to `OperationalIssue` model:
  ```python
  OperationalIssue.objects.create(
      name="Descriptive Name",
      task=TASK_NAME,
      error_message=str(e),
      traceback=traceback.format_exc(),
      context={"key": "value"},
      severity=OperationalIssue.Severity.ERROR,  # or WARNING, INFO
  )
  ```

---

# Database

* Use `select_related()` and `prefetch_related()`
* Wrap multi-step operations in `@transaction.atomic`
* All showtimes stored as timezone-aware datetimes (UTC)
* Use underscore_case for table names (e.g., `movies_app_operational_issue`, not `movies_app_operationalissue`)
* Colombia timezone: Use `TimeService` from `movies_app.services.time_service` — never call `datetime.datetime.now()` or `datetime.date.today()` directly for Colombia time. Use `TimeService.get_colombia_date()`, `TimeService.get_colombia_now()`, etc. In tests, mock `TimeService.get_colombia_now()` or `TimeService.get_colombia_date()` instead of patching `datetime`.

### Key Database Tables
- `movies_app_movie` - Movie information
- `movies_app_theater` - Theater/cinema information
- `movies_app_showtime` - Showtime schedules

### Frontend Database Access (Supabase)

The frontend talks to Supabase directly. Query patterns matter because every row crosses the network to the user's browser (or to a Vercel function) on every request.

* **Never fetch unbounded or bulk table data from the frontend.** Do not write queries like `.select("*").limit(10000)` or `.select(...)` without a narrowly-scoped filter. This includes "I just want all movies" or "all theaters" — there is no such legitimate frontend use case.
* **Push aggregation and bulk reads into Postgres.** If a page needs data derived from many rows (sitemaps, counts, summaries, joins across large tables), write a Postgres function (RPC) in a Django migration using `migrations.RunSQL()` and call it from the frontend with `supabase.rpc("function_name")`. The function should return the already-shaped, minimal result.
* **Always filter by a bounded key** (slug, id, date range, city, theater) when using `.from(...).select(...)`. Pagination/limits are a safety net, not a substitute for filters.
* **Example**: the sitemap uses `get_movie_latest_showtime()` RPC (see `backend/movies_app/migrations/0072_add_get_movie_latest_showtime_function.py`) instead of pulling every showtime row into the frontend.

### Migrations

* **Never modify the database directly**—all changes must be in a migration
* This includes adding functions, triggers, RLS policies, or any schema changes
* All migrations live in the Django directory (`backend/movies_app/migrations/`)
* Do not add migrations in frontend code
* Generate model migrations with `python manage.py makemigrations`
* For raw SQL (functions, RLS), use `migrations.RunSQL()` in a migration file

---

# Admin Dashboard (Frontend)

Located at `/admin` in the frontend (not Django admin).

* Focus on functionality, not styling
* Text in English

## Security Model

* Authentication uses Supabase Auth
* Authorization checks the `admin_users` table to verify if a user is an admin
* Row Level Security (RLS) is **disabled** on admin-facing tables (e.g., `movies_app_operational_issue`, `movies_app_scraper_run_report`)
* The frontend uses the Supabase anon key, but admin pages check session + admin status before rendering
* For local development, set `NEXT_PUBLIC_DISABLE_ADMIN_AUTH=true` to bypass auth checks

## Adding New Admin Pages

1. Create the page directory in `frontend/src/app/admin/<page-name>/` with a `page.tsx`
2. The admin `layout.tsx` provides consistent styling and auth protection
3. **Add a link in the sidebar**: Edit `frontend/src/components/AdminSidebar.tsx` and add to `navItems` array
4. Optionally add a card on the dashboard index page (`frontend/src/app/admin/page.tsx`)

## Existing Admin Pages
- `/admin` - Dashboard with health checks and summary
- `/admin/scraper-runs` - Monitor scraper health and statistics
- `/admin/operational-issues` - View scraper errors and warnings
- `/admin/unfindable-urls` - Movies that couldn't be matched to TMDB
- `/admin/external-links` - External link management
- `/admin/movies` - Movie management
- `/admin/theaters` - Theater management
- `/admin/scraper-movie-metadata` - Scraper movie metadata
- `/admin/industry-resources` - Industry resources

---

# MCP Tools

MCP (Model Context Protocol) tools are available for enhanced development assistance.

## Available Tools

* **Supabase MCP** - Database queries, schema inspection, migrations
* **Sentry MCP** - Error tracking, issue analysis
* **Browser/Playwright MCP** - Web scraping assistance, page inspection

## Usage Guidelines

* Use Supabase MCP for exploring database schema and running read queries
* Do not use MCP tools to directly modify production data—use migrations instead
* Sentry MCP is useful for investigating production errors and their context

## Verifying Frontend Work

After frontend changes, verify by visiting the page:
1. Use Chrome DevTools MCP or `open_simple_browser` to view the page
2. Take a screenshot to verify UI renders correctly
3. Check console for JavaScript errors
4. Test key interactions if applicable

---

# Additional information

## Command Line Conventions

Run commands separately (not with `&&`). Combined commands fail auto-allow:
**NOT CORRECT:**
```bash
source .venv/bin/activate && pytest movies_app/tests/test_models.py
```

**CORRECT:**
```bash
source .venv/bin/activate
pytest movies_app/tests/test_models.py
```

## Documentation

Do not add large amounts of code to files under /docs when you can point to a file to emulate.


## Platform

* No external customers. Internal tool—no backwards compatibility concerns. Delete deprecated code immediately.
* Development on **macOS**
* Backend code runs on macOS in production
* Frontend code runs in Vercel serverless environment in production
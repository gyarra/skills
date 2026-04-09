---
name: s-creating-a-theater-sync-command
description: Use when building a new theater sync management command — covers website investigation, the command structure (WebsiteTheater, TheatersJsonManager, ComparisonResult), and the dry-run/apply pattern
---

# Creating a Theater Sync Command

Theater sync commands compare theaters on a chain's website against the JSON seed files and database. They help keep theater data in sync when cinemas open or close.

## Naming Convention

`theater_sync_{chain}.py` (e.g., `theater_sync_cine_colombia.py`, `theater_sync_royal_films.py`)

## Existing Examples

| Command | Data Source |
|---------|-------------|
| `theater_sync_cine_colombia.py` | Embedded JavaScript data (`window.initialData.cinemaGroups`) |
| `theater_sync_royal_films.py` | HTML scraping via browser automation |

## Step 1: Investigate the Website

Use Chrome DevTools MCP or Camoufox MCP to explore the target website:

```
mcp_chrome-devtoo_new_page              # Open a new browser page
mcp_chrome-devtoo_navigate_page         # Navigate to theaters/locations page
mcp_chrome-devtoo_list_network_requests # See all XHR/fetch requests
mcp_chrome-devtoo_get_network_request   # Get full request/response details
mcp_chrome-devtoo_evaluate_script       # Run JS (e.g., Object.keys(window))
mcp_chrome-devtoo_take_screenshot       # Verify page loaded correctly
```

1. Capture network requests and look for API calls returning JSON
2. Check for embedded data in `window.*` variables
3. **Prefer using an API over HTML scraping** -- APIs are more stable and faster

If you find an API, test it directly with fetch/curl to verify it contains all needed theater data (name, ID, city, URL).

## Step 2: Create the Command

Emulate `theater_sync_cine_colombia.py`.

### Required Components

- `WebsiteTheater` dataclass for extracted data
- `{Chain}TheaterExtractor` class with `extract_theaters()` method
- `TheatersJsonManager` class for reading/writing the JSON file
- `ComparisonResult` dataclass for tracking changes
- Support `--apply` and `--dry-run` flags

### Command Structure

```python
from __future__ import annotations

import json
from dataclasses import dataclass
from pathlib import Path
from typing import Any

from camoufox.async_api import AsyncCamoufox
from django.core.management.base import BaseCommand, CommandParser

from movies_app.models import Theater
from movies_app.tasks.download_utilities import (
    get_browser_options,
    get_context_options,
    run_async_safely,
)


@dataclass
class WebsiteTheater:
    """Theater data extracted from the website."""
    name: str
    slug: str
    city: str
    # Add chain-specific fields (e.g., cinema_id)


@dataclass
class ComparisonResult:
    """Result of comparing website theaters against a source."""
    to_remove: set[str]
    to_add: set[str]
    in_sync: set[str]
    source_name: str


class ChainTheaterExtractor:
    """Extracts theater data from the chain's website."""

    @classmethod
    def extract_theaters(cls) -> list[WebsiteTheater]:
        return run_async_safely(cls._extract_theaters_async())

    @classmethod
    async def _extract_theaters_async(cls) -> list[WebsiteTheater]:
        async with AsyncCamoufox(**get_browser_options()) as browser:
            context = await browser.new_context(
                **get_context_options(ignore_https_errors=True),
            )
            # ... extraction logic
            await context.close()
            return theaters


class TheatersJsonManager:
    """Manages reading and writing {chain}_theaters.json."""

    def __init__(self, file_path: Path):
        self.file_path = file_path

    def load(self) -> dict[str, list[dict[str, Any]]]:
        with open(self.file_path) as f:
            return json.load(f)

    def save(self, data: dict[str, list[dict[str, Any]]]) -> None:
        with open(self.file_path, "w") as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
            f.write("\n")


class Command(BaseCommand):
    help = "Review and sync {Chain} theaters with {chain}_theaters.json"

    def add_arguments(self, parser: CommandParser) -> None:
        parser.add_argument(
            "--apply", action="store_true",
            help="Apply changes (default is dry-run)",
        )
        parser.add_argument(
            "--dry-run", action="store_true",
            help="Show what would change without modifying files",
        )

    def handle(self, *args: object, **options: object) -> None:
        apply_changes = bool(options.get("apply"))
        # ... implementation
```

### Standard Behavior

- `--dry-run` (default): Show what would change without modifying files
- `--apply`: Apply changes to the JSON file

### Output Format

The command should display:
1. **JSON vs Website comparison** -- theaters to add/remove from the JSON
2. **Database vs Website comparison** -- theaters to add/remove from the database
3. **Summary** -- count of matching, to-add, to-remove theaters
4. **Next steps** -- remind user to run `theaters_load` after applying, then `theaters_geocode`

## Step 3: Handle SSL Certificate Errors

Some sites have certificate issues. Pass `ignore_https_errors=True`:

```python
context = await browser.new_context(
    **get_context_options(ignore_https_errors=True),
)
```

## Step 4: Test the Command

```bash
python manage.py theater_sync_{chain}           # Dry-run first
python manage.py theater_sync_{chain} --apply   # Only after verifying output
```

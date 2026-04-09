---
name: s-creating-an-api-scraper
description: Use when building a new API-based scraper for a cinema chain — covers API discovery, the two-class architecture (ApiClient + ShowtimeSaver), movie metadata requirements, Celery tasks, management commands, and testing patterns
---

# Creating an API-Based Scraper

API-based scrapers fetch showtimes from a cinema's REST API rather than parsing HTML. They are faster and more reliable.

## Reference Implementation

Emulate the Cine Colombia API scraper:
- **Task:** `movies_app/tasks/cine_colombia_api_download_task.py`
- **Command:** `movies_app/management/commands/scrape_cine_colombia_api.py`
- **Tests:** `movies_app/tasks/tests/test_cine_colombia_api_download_task.py`

## API Discovery

### Step 1: Capture Network Traffic

Use Chrome DevTools MCP or Camoufox MCP:

```
mcp_chrome-devtoo_new_page
mcp_chrome-devtoo_navigate_page         # Go to cinema's showtime page
mcp_chrome-devtoo_list_network_requests # Filter by Fetch/XHR
mcp_chrome-devtoo_get_network_request   # Inspect JSON responses
```

**Common API patterns:**
- `/api/showtimes?cinema_id=123&date=2025-01-15`
- `/api/v1/screenings/{cinema_code}`
- `/ocapi/v1/showtimes/by-business-date/{date}?siteIds={cinema_id}`

### Step 2: Test Multiple Dates

Many APIs return showtimes for one date per request. Test with different dates to ensure coverage for today + 7 days.

### Step 3: Identify Authentication

Common patterns:
1. **No auth:** Public endpoints
2. **Bearer token:** JWT embedded in page JavaScript
3. **API key:** Static key in headers
4. **Session cookies:** Requires browser visit first

For token-based APIs, find where the token is stored:
```javascript
window.initialData?.api?.authToken
localStorage.getItem('auth_token')
document.querySelector('meta[name="api-token"]')?.content
```

## Architecture

API scrapers follow the two-class template pattern:

1. **`*ApiClient`** -- Stateless utility class
   - HTTP requests to API endpoints
   - Token extraction if needed
   - JSON parsing into dataclasses

2. **`*ApiShowtimeSaver`** -- Extends `MovieAndShowtimeSaverTemplate`
   - Implements `_find_movies()`, `_process_showtimes_for_theater()`
   - Uses ApiClient for data fetching

## Implementation

### 1. Dataclasses for API Response

```python
from dataclasses import dataclass
import datetime


@dataclass
class MyApiSession:
    """Single showtime from the API."""
    showtime: datetime.datetime
    screen_name: str
    screen_number: int
    format: str  # "2D", "3D", "IMAX"
    translation_type: str  # "Doblada", "Subtitulada"


@dataclass
class MyApiMovie:
    """Movie data from the API with all its sessions."""
    film_id: str
    title: str
    slug: str | None
    sessions: list[MyApiSession]
```

### 2. API Client

```python
import requests
import datetime
import logging
from movies_app.tasks.download_utilities import run_async_safely, BOGOTA_TZ

logger = logging.getLogger(__name__)

API_BASE_URL = "https://cinema-api.example.com"
SHOWTIMES_ENDPOINT = "/api/v1/showtimes/{date}"
REQUEST_TIMEOUT = 30
MAX_DAYS_AHEAD = 7


class MyApiClient:
    """Client for fetching data from Cinema's API."""

    def __init__(self, auth_token: str):
        self.auth_token = auth_token

    def _headers(self) -> dict[str, str]:
        return {
            "Authorization": f"Bearer {self.auth_token}",
            "Accept": "application/json",
        }

    def fetch_showtimes_for_cinema(self, cinema_id: str) -> list[MyApiMovie]:
        """Fetch all movies and showtimes for a cinema across all dates."""
        today = datetime.datetime.now(BOGOTA_TZ).date()
        all_movies: dict[str, MyApiMovie] = {}

        for day_offset in range(MAX_DAYS_AHEAD + 1):
            target_date = today + datetime.timedelta(days=day_offset)
            date_str = target_date.strftime("%Y-%m-%d")

            url = f"{API_BASE_URL}{SHOWTIMES_ENDPOINT.format(date=date_str)}"
            params = {"cinema_id": cinema_id}

            response = requests.get(
                url,
                headers=self._headers(),
                params=params,
                timeout=REQUEST_TIMEOUT,
            )
            response.raise_for_status()
            data = response.json()

            movies_for_date = self._parse_response(data)

            # Merge by film_id (same movie may appear on multiple dates)
            for movie in movies_for_date:
                if movie.film_id in all_movies:
                    all_movies[movie.film_id].sessions.extend(movie.sessions)
                else:
                    all_movies[movie.film_id] = movie

        return list(all_movies.values())

    @staticmethod
    def _parse_response(data: dict) -> list[MyApiMovie]:
        """Parse API response into movie objects."""
        movies = []
        for item in data.get("films", []):
            sessions = [
                MyApiSession(
                    showtime=datetime.datetime.fromisoformat(s["start_time"]),
                    screen_name=s.get("screen_name", ""),
                    screen_number=s.get("screen_number", 0),
                    format=s.get("format", "2D"),
                    translation_type=s.get("language", ""),
                )
                for s in item.get("sessions", [])
            ]
            movies.append(MyApiMovie(
                film_id=item["id"],
                title=item["title"],
                slug=item.get("slug"),
                sessions=sessions,
            ))
        return movies
```

### 3. Token Extraction (if required)

```python
from camoufox.async_api import AsyncCamoufox
from movies_app.tasks.download_utilities import get_browser_options, get_context_options

BROWSER_TIMEOUT_SECONDS = 60
TOKEN_SOURCE_URL = "https://cinema.example.com/showtimes"


class MyApiClient:
    # ... previous methods ...

    @staticmethod
    def extract_auth_token() -> str:
        """Extract fresh JWT auth token from cinema website."""
        return run_async_safely(MyApiClient._extract_auth_token_async())

    @staticmethod
    async def _extract_auth_token_async() -> str:
        async with AsyncCamoufox(**get_browser_options()) as browser:
            context = await browser.new_context(**get_context_options())
            page = await context.new_page()

            try:
                await page.goto(
                    TOKEN_SOURCE_URL,
                    wait_until="domcontentloaded",
                    timeout=BROWSER_TIMEOUT_SECONDS * 1000,
                )
                token = await page.evaluate("window.initialData?.api?.authToken")

                if not token or not isinstance(token, str):
                    raise MyApiError("No auth token found")

                return token
            finally:
                await context.close()
```

### 4. ShowtimeSaver Class

```python
from movies_app.tasks.showtime_saver_template import (
    MovieAndShowtimeSaverTemplate,
    MovieInfo,
    MovieMetadata,
    MoviesCache,
    ShowtimeData,
)
from movies_app.models import ScraperMovieMetadata, Theater


class MyApiShowtimeSaver(MovieAndShowtimeSaverTemplate):
    """API-based scraper extending the template pattern."""

    def __init__(
        self,
        api_client: MyApiClient,
        tmdb_service: TMDBService,
        storage_service: SupabaseStorageService | None,
    ):
        super().__init__(
            tmdb_service=tmdb_service,
            storage_service=storage_service,
            source_name="my_cinema",
            scraper_type="my_cinema",
            scraper_type_enum=ScraperMovieMetadata.ScraperType.MY_CINEMA,
            task_name="my_cinema_download_task",
        )
        self.api_client = api_client
        self._movie_cache: dict[str, MyApiMovie] = {}

    def _find_movies(self, theater: Theater) -> list[MovieInfo]:
        """Fetch movies from API for this theater."""
        self._movie_cache.clear()

        config = theater.scraper_config or {}
        cinema_id = config.get("cinema_id")

        if not cinema_id:
            raise MyApiError(f"Theater {theater.name} missing cinema_id")

        api_movies = self.api_client.fetch_showtimes_for_cinema(cinema_id)

        movies: list[MovieInfo] = []
        for api_movie in api_movies:
            source_url = self._generate_source_url(api_movie)
            self._movie_cache[source_url] = api_movie
            movies.append(MovieInfo(name=api_movie.title, source_url=source_url))

        return movies

    def _process_showtimes_for_theater(
        self,
        theater: Theater,
        movies_for_theater: list[MovieInfo],
        movies_cache: MoviesCache,
    ) -> int:
        """Convert API session data to showtimes."""
        all_showtimes: list[ShowtimeData] = []

        for movie_info in movies_for_theater:
            cache_entry = movies_cache.get(movie_info.source_url)
            if not cache_entry:
                continue
            movie, scraper_metadata = cache_entry

            api_movie = self._movie_cache.get(movie_info.source_url)
            if not api_movie:
                continue

            for session in api_movie.sessions:
                if not is_date_within_scraper_range(session.showtime.date()):
                    continue

                all_showtimes.append(ShowtimeData(
                    movie=movie,
                    scraper_movie_metadata=scraper_metadata,
                    date=session.showtime.date(),
                    time=session.showtime.time(),
                    format=session.format,
                    translation_type=session.translation_type,
                    screen=session.screen_name,
                    source_url=movie_info.source_url,
                ))

        return self._delete_future_showtimes_then_save_new_showtimes_for_theater(theater, all_showtimes)
```

## Movie Metadata -- Critical Requirement

**DO NOT skip this.** The `MovieLookupService` uses metadata to match scraped movies to TMDB entries. Without metadata:
- **Title-only matching fails** for films with Spanish translations that differ from TMDB's Spanish titles
- **No disambiguation** between remakes or similarly-named films
- **Higher unfindable rate** -- movies end up in `unfindable_urls` instead of being matched

### Required Fields

| Field | Importance | Why |
|-------|------------|-----|
| `original_title` | **Critical** | Enables TMDB search in original language |
| `release_year` | **Critical** | Disambiguates remakes, sequels, same-name films |
| `duration_minutes` | Important | Validates matches against TMDB runtime |

### How to Fetch Metadata

API responses typically lack rich metadata. Options:

1. **Use API** -- if all metadata is in the API response (rare)
2. **Reuse existing HTML scraper** (preferred) -- if one exists for this chain
3. **Create new parser methods** -- visit the cinema's movie detail page with Camoufox

The template only calls `_get_movie_metadata()` for movies **not already** in `ScraperMovieMetadata`, so overhead is minimal (2-5 new movies per week).

**Do NOT return None by default** -- only return None on error:

```python
# Wrong
def _get_movie_metadata(self, movie_info: MovieInfo) -> MovieMetadata | None:
    return None

# Correct
def _get_movie_metadata(self, movie_info: MovieInfo) -> MovieMetadata | None:
    try:
        html = self._download_movie_detail_page(movie_info.source_url)
        return self._parse_metadata(html)
    except Exception as e:
        logger.warning(f"Metadata fetch failed: {e}")
        return None  # Only return None on error
```

## Celery Task

```python
from config.celery_app import app

@app.task
def my_cinema_download_task():
    """Celery task to download showtimes via API."""
    auth_token = MyApiClient.extract_auth_token()
    api_client = MyApiClient(auth_token)
    tmdb_service = TMDBService()
    storage_service = SupabaseStorageService.create_from_settings()

    saver = MyApiShowtimeSaver(api_client, tmdb_service, storage_service)
    return saver.execute(triggered_by=ScraperRunReport.TriggerType.CELERY)
```

## Management Command

See `scrape_cine_colombia_api.py` for the full pattern. Must support `-t` (theater slug) and `-l` (list theaters) options.

After creating the command, add its name to the `SCRAPERS` list in `movies_app/management/commands/scrape_all.py` so it runs as part of the full scrape pipeline.

## Theater Configuration

API scrapers use `scraper_config` JSON field instead of `theater_url`:

```json
{
    "name": "Cinema Example - Mall Location",
    "slug": "cinema-example-mall",
    "scraper_type": "my_cinema",
    "scraper_config": {
        "cinema_id": "12345"
    }
}
```

### Chain Registry

After creating the seed file (e.g., `seed_data/my_cinema_theaters.json`), add the chain to the centralized registry in `movies_app/utils/theater_seed_data.py`. This is the single source of truth used by `theaters_load` and `theaters_geocode`.

## Testing Requirements

### Unit Tests
Test parsing logic with sample API responses.

### Integration Tests
Mock HTTP calls but test full saver flow. Verify showtimes are saved to database.

### Async Safety Test (Required)

```python
@pytest.mark.django_db
class TestAsyncSyncSafety:
    async def test_token_extraction_from_async_context(self):
        """Catches asyncio.run() bug -- fails with 'event loop already running'."""
        with patch.object(
            MyApiClient,
            "_extract_auth_token_async",
            new_callable=AsyncMock,
            return_value="test-token",
        ):
            token = MyApiClient.extract_auth_token()
            assert token == "test-token"
```

## Checklist

- [ ] Discovered API endpoints
- [ ] Created dataclasses for response structure
- [ ] Created `*ApiClient` class
- [ ] Added token extraction if needed
- [ ] Created `*ApiShowtimeSaver` extending template
- [ ] **`_get_movie_metadata()` fetches from movie detail page** (not returning None)
- [ ] Fetches all dates (today + MAX_DAYS_AHEAD)
- [ ] Merges movies across dates by film ID
- [ ] Created Celery task
- [ ] Created management command with `-t` and `-l` options
- [ ] Added new command to `SCRAPERS` list in `scrape_all.py`
- [ ] Added theaters to seed_data with `scraper_config`
- [ ] Added chain to `movies_app/utils/theater_seed_data.py` CHAIN_FILES registry
- [ ] Unit tests for parsing logic
- [ ] Integration tests for full flow
- [ ] Async safety test
- [ ] Ran `ruff check .`, `pyright`, `pytest`
- [ ] Manual test: `python manage.py scrape_{source} -t {slug}`
- [ ] Verified showtimes match website

# arr-mcp — MCP Server Capabilities

## Server Overview

arr-mcp is a unified FastMCP 3.3+ server for the complete *arr automation stack. It provides a single MCP interface to Radarr (movies), Sonarr (TV series), Lidarr (music), Prowlarr (indexers), Readarr (books), Overseerr (media requests), and Bazarr (subtitles). Each service is independently optional — if a service is not configured, its tools are simply not registered.

**The differentiator:** The `arr_orchestrate` tool chains media server availability checks (Jellyfin + Plex + Emby) with automatic *arr routing. Given a media title, it checks configured media servers, and if not found, auto-routes to the correct *arr service for acquisition.

**Stack architecture:** arr-mcp auto-discovers services on default ports when an API key is found in the environment. Service tools are conditionally registered — each service client is independently initialized, and missing services gracefully skip tool registration. The server also provides Prefab UI cards for health status, calendar, and stack statistics.

## Tools

### Radarr: radarr_movies

Portmanteau tool for movie management across the full lifecycle.

**Operations:** `list`, `lookup`, `get`, `add`, `delete`, `update`, `import`

**Parameters (by operation):**
- `list`: No additional parameters needed.
- `lookup`: Requires `term` (str) — search term for movie lookup.
- `get`: Requires `movie_id` (int) — Radarr movie ID.
- `add`: Requires `tmdb_id` (int), `quality_profile_id` (int), `root_folder_path` (str). Optional: `title` (str), `monitored` (bool, default True), `search_for_movie` (bool, default True).
- `delete`: Requires `movie_id` (int). Optional: `delete_files` (bool, default False).
- `update`: Requires `movie_id` (int). Optional: `monitored` (bool).
- `import`: Requires `folder` (str) — path for manual import.

**Return format:** `{"success": bool, "message": str, "data": list|dict}`

### Sonarr: sonarr_series / sonarr_episodes

Two portmanteau tools for TV series and episode management.

**sonarr_series operations:** `list`, `lookup`, `get`, `add`, `delete`, `update`
- `lookup`: Requires `term` (str).
- `add`: Requires `tvdb_id` (int), `quality_profile_id` (int), `root_folder_path` (str). Optional: `title`, `monitored` (default True), `search_for_missing` (default True).
- `delete`: Requires `series_id` (int). Optional: `delete_files` (default False).

**sonarr_episodes operations:** `list`, `get`, `search`, `set_monitored`
- `list`: Requires `series_id` (int). Optional: `season_number` (int) to filter.
- `get`: Requires `episode_id` (int).
- `search`: Requires `episode_id` (int) or `series_id` (int).
- `set_monitored`: Requires `episode_id` (int), `monitored` (bool).

### Lidarr: lidarr_artists / lidarr_albums

Two portmanteau tools for music artist and album management.

**lidarr_artists operations:** `list`, `lookup`, `get`, `add`, `delete`, `update`
- `add`: Requires `foreign_artist_id` (str, MusicBrainz ID), `quality_profile_id` (int), `root_folder_path` (str).

**lidarr_albums operations:** `list`, `get`, `lookup`, `set_monitored`
- `list`: Requires `artist_id` (int).
- `lookup`: Requires `term` (str).

### Readarr: readarr_authors / readarr_books

Two portmanteau tools for author and book management.

**readarr_authors operations:** `list`, `lookup`, `get`, `add`, `delete`, `update`
- `add`: Requires `foreign_author_id` (str), `quality_profile_id` (int), `root_folder_path` (str).

**readarr_books operations:** `list`, `get`, `lookup`, `set_monitored`
- `list`: Requires `author_id` (int).
- `lookup`: Requires `term` (str).

### Prowlarr: prowlarr_indexers / prowlarr_search / prowlarr_applications / prowlarr_history

Four portmanteau tools for indexer backbone management.

**prowlarr_indexers operations:** `list`, `get`, `add`, `update`, `delete`, `test`, `test_all`, `schema`
- `add`: Requires `name` (str), `config_contract` (str, e.g. "NewznabSettings"). Optional: `enable_rss` (default True), `enable_search` (default True), `priority` (default 25).

**prowlarr_search operation:** `search` — unified search across all indexers. Requires `query` (str). Optional: `type` (movie/search/book/tvsearch), `indexer_ids` (list[int]), `limit` (int, default 30).

**prowlarr_applications operations:** `list`, `get`, `sync`, `sync_all`, `test`
- `sync`: Requires `app_id` (int). `sync_all`: No additional params.

**prowlarr_history operations:** `list`, `since`, `by_indexer`
- `since`: Requires `date` (ISO datetime).

### Overseerr: overseerr_requests / overseerr_search / overseerr_users

Three portmanteau tools for media request management.

**overseerr_requests operations:** `list`, `get`, `create`, `approve`, `decline`, `delete`, `count`, `pending`
- `create`: Requires `media_type` ("movie"/"tv"), `media_id` (int). Optional: `seasons` (list[int] for TV), `is4k` (bool).
- `approve`/`decline`/`get`/`delete`: Require `request_id` (int).

**overseerr_search operation:** `search` — search for media to request. Requires `query` (str).

**overseerr_users operations:** `list`, `get`, `create`, `update`, `delete`
- `get`: Requires `user_id` (int).

### Bazarr: bazarr_subtitles

Portmanteau tool for subtitle management across the stack.

**Operations:** `wanted`, `search`, `download`, `history`, `providers`, `languages`
- `search`: Requires `episode_id` (int) or `movie_id` (int). Optional: `language` (str, e.g. "en", "de").
- `download`: Requires `subtitle_path` (str), `episode_id` or `movie_id`. Optional: `language`, `provider` (str), `scene_name` (str).

### Cross-Arr Orchestration: arr_orchestrate

**THE DIFFERENTIATOR** — chains media server checks with auto-routing.

**Operations:** `request`, `status`, `check_jellyfin`, `check_plex`, `check_emby`, `queue`
- `request`: Requires `media_title` (str). Optional: `media_type` (movie/series/album/book — auto-detected), `year` (int), `quality_profile_id`, `root_folder_path`.
- `check_jellyfin`/`check_plex`/`check_emby`: Requires `media_title` (str).
- `status`: Returns full stack status.
- `queue`: Returns active download queues.

### Stack Health: arr_health

**Parameters:**
- `service` (Literal, default "all"): `all`, `radarr`, `sonarr`, `lidarr`, `prowlarr`, `readarr`, `overseerr`, `bazarr`.

**Return format:** `{"success": bool, "message": "X/Y services reachable", "data": {service_name: {"reachable": bool, "version": str, "reason": str}}}`

### Unified Calendar: arr_calendar

Aggregates upcoming releases from Radarr, Sonarr, Lidarr, and Readarr.

**Operations:** `upcoming`, `today`, `week`, `range`
- `range`: Requires `start` (ISO datetime), `end` (ISO datetime). Optional: `types` (list of media types to filter).

### Stack Statistics: arr_stats

**Operations:** `summary`, `disk`, `queues`, `history`

### Agentic Workflows: arr_agentic

Uses FastMCP sampling (SEP-1577) for LLM-powered cross-arr operations. The LLM decides which tools to call and in what order.

**Operations:** `workflow`, `natural_query`
- `workflow`: Multi-step task described in natural language. Examples: "add The Matrix to Radarr with best quality."
- `natural_query`: Ask a question about your media stack. Examples: "what's the most wanted movie right now?"

### Help: arr_help

**Operations:** `discover`, `tool_info`, `quickstart`
- `discover`: Lists all available tools.
- `tool_info`: Requires `tool_name` (str).
- `quickstart`: Step-by-step guide.

### Prefab UI Cards: arr_health_card / arr_calendar_card / arr_stats_card

Rich visual cards rendered in supporting MCP clients.
- `arr_health_card()`: Stack health as a live card.
- `arr_calendar_card(days=14)`: Upcoming releases card.
- `arr_stats_card()`: Consolidated statistics card.

### Health & Diagnostics: api_health / api_logs / api_logs_stream

- `api_health`: Server health check endpoint.
- `api_logs`: Recent server logs (up to 500 entries from ring buffer).
- `api_logs_stream`: SSE endpoint for real-time log streaming.

## Configuration

Each *arr service is configured independently via environment variables. Service discovery uses the following defaults:

| Variable | Default Port | Purpose |
|----------|-------------|---------|
| `RADARR_URL` / `RADARR_API_KEY` | 7878 | Radarr movie management |
| `SONARR_URL` / `SONARR_API_KEY` | 8989 | Sonarr TV series management |
| `LIDARR_URL` / `LIDARR_API_KEY` | 8686 | Lidarr music management |
| `PROWLARR_URL` / `PROWLARR_API_KEY` | 9696 | Prowlarr indexer backbone |
| `READARR_URL` / `READARR_API_KEY` | 8787 | Readarr book management |
| `OVERSEERR_URL` / `OVERSEERR_API_KEY` | 5055 | Overseerr media requests |
| `BAZARR_URL` / `BAZARR_API_KEY` | 6767 | Bazarr subtitle management |
| `JELLYFIN_URL` / `JELLYFIN_API_KEY` | 8096 | Jellyfin media server bridge |
| `PLEX_URL` / `PLEX_TOKEN` | 32400 | Plex media server bridge |
| `EMBY_URL` / `EMBY_API_KEY` | 8096 | Emby media server bridge |
| `ARR_LOG_LEVEL` | INFO | Logging verbosity |

### Ports

| Service | Port |
|---------|------|
| arr-mcp backend | 10938 |
| arr-mcp frontend | 10939 |

## Resources

| Resource | Description |
|----------|-------------|
| `arr://config` | Current server configuration summary |
| `arr://quickstart` | Quickstart guide for new users |
| `arr://help` | Tool documentation index |
| `arr://capabilities` | Available service capabilities |

## Prompts

| Prompt | Description |
|--------|-------------|
| `orchestrate_media` | Guide the LLM through media request orchestration |
| `stack_health_check` | Walkthrough for diagnosing stack health issues |

## Error Handling

All tools return structured dicts with `{"success": bool, "message": str, "data": ...}` pattern. Service-specific errors include descriptive messages. When a service is not configured, tools are simply not registered — no error is raised. Timeouts and network failures return structured error messages with the exception detail. The `arr_agentic` tool falls back gracefully when sampling is unavailable.

# arr-mcp

<p align="center">
  <a href="https://github.com/sandraschi/arr-mcp"><img src="https://img.shields.io/github/stars/sandraschi/arr-mcp?style=flat-square" alt="Stars"></a>
  <a href="https://github.com/sandraschi/arr-mcp/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="License"></a>
  <a href="https://python.org"><img src="https://img.shields.io/badge/Python-3.12+-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python"></a>
  <a href="https://github.com/PrefectHQ/fastmcp"><img src="https://img.shields.io/badge/FastMCP-3.3-7c5cfc?style=flat-square" alt="FastMCP"></a>
  <a href=""><img src="https://img.shields.io/badge/stack-*arr-00ADD8?style=flat-square" alt="*arr Stack"></a>
  <a href=""><img src="https://img.shields.io/badge/tests-143-success?style=flat-square" alt="Tests"></a>
  <a href=""><img src="https://img.shields.io/badge/fleet-SOTA-6366f1?style=flat-square" alt="Fleet SOTA"></a>
</p>

FastMCP 3.3 MCP server for the complete *arr automation stack — Radarr, Sonarr, Lidarr, Prowlarr, Readarr, Overseerr, and Bazarr — under a single MCP interface.

## Features

- **7 services, 1 MCP server** — Radarr (Movies), Sonarr (TV), Lidarr (Music), Prowlarr (Indexers), Readarr (Books), Overseerr (Requests), Bazarr (Subtitles)
- **29 MCP tools** — 22 portmanteau tools + 1 Readarr options tool + 3 Readarr mutation tools + 3 Prefab card tools, 112+ operations
- **Cross-arr orchestration** — request a title, auto-routes to correct arr with Jellyfin availability check
- **Prefab-UI cards** — `arr_health_card`, `arr_calendar_card`, `arr_stats_card` — rich interactive cards in Claude Desktop, Cursor
- **Prowlarr indexer backbone** — unified search across all indexers
- **Auto-discovery** — probes default ports for running *arr services; no `.env` needed for standard setups
- **Optional services** — each arr independently configurable via `.env`, disabled services don't register tools
- **FastMCP 3.3 resources** — `arr://config`, `arr://quickstart`, `arr://help`, `arr://capabilities` for agent self-discovery
- **LLM sampling** — `arr_agentic` tool with Context injection for LLM-powered cross-arr workflows
- **React dashboard** — 15-page webapp with health monitoring, LLM chat, MCP Inspector, live SSE log streaming
- **Local LLM chat** — built-in chat page with Ollama and LM Studio model selection
- **PWA** — installable as desktop/mobile app with offline service worker
- **MCP Inspector** — interactive tool runner: select a tool, set params, execute via `/mcp`, see raw JSON-RPC response
- **Real-time logs** — SSE stream endpoint `/api/logs/stream` for live log viewing
- **Browser notifications** — desktop alerts for download completions / request approvals
- **Tauri 2.0 native** — single `.exe` installer bundling Python backend via PyInstaller
- **CI/CD** — GitHub Actions (ruff + pytest + biome + tsc), 143 tests, Playwright e2e smoke tests

## Quick Start

```bash
# 1. Clone
git clone https://github.com/sandraschi/arr-mcp
cd arr-mcp

# 2. Create config (optional — auto-discovery probes default ports)
cp .env.example .env
# Edit .env — add your *arr URLs and API keys

# 3. Run from the checkout
uv sync
uv run arr-mcp

# 4. Webapp (separate terminal)
cd webapp
npm install
npm run dev        # → http://localhost:10939
```

For a local checkout launcher using STDIO (keep API keys and other secrets in
the external environment), use:

```bash
ARR_MCP_TRANSPORT=stdio uvx --from /absolute/checkout --with-editable /absolute/checkout arr-mcp
```

Restart the local process after changing source files so the editable checkout
is loaded again. This command runs the checkout directly; it does not publish
or install a fork.

### Readarr book mutations

Before adding a book for a new author, call the read-only
`readarr_add_options` tool. It returns the available `root_folders` (including
their paths and default profile/tag IDs), `quality_profiles`,
`metadata_profiles`, and existing `tags` as safe ID/name choices. Use those
IDs and the selected root-folder path in `readarr_add_book`; the discovery
tool filters connection details and does not create tags or change Readarr
settings. It also reports the supported `monitor_modes` and
`monitor_new_books_modes` values. Existing authors keep their current Readarr
configuration.

The dedicated `readarr_add_book`, `readarr_delete_book`, and
`readarr_delete_books` tools operate on exact IDs. Adding a book requires an
existing `author_id` (which reuses that author's quality, metadata, and root
folder configuration) or all four new-author values: `foreign_author_id`,
`quality_profile_id`, `metadata_profile_id`, and `root_folder_path`. Readarr's
exact foreign-book lookup is checked for a single match and duplicates or
ambiguous results are rejected; incomplete lookup identity is enriched through
Readarr's exact search endpoint. A new author is created only as the side
effect of adding the requested book. The default `monitor="specific_book"`
maps to Readarr's `monitor="all"` plus `booksToMonitor=[foreign_book_id]`;
catalogue modes (`all`, `future`, `missing`, `existing`, `first`, `latest`)
omit that override, and `none` leaves the author unmonitored. Use
`monitor_new_books="all"|"none"|"new"` and `tags` to control the new
author. `search_for_new_book` is disabled by default, and no edition selector
is exposed.

To **add and search immediately**, pass `search_for_new_book=true` to
`readarr_add_book`. This uses Readarr's native `addOptions.searchForNewBook`
in the add request, searching only the added book. For example, after looking
up the exact foreign book ID for an existing author:

```python
readarr_add_book(foreign_book_id="<exact-work-id>", author_id=7, search_for_new_book=True)
```

For a book already in the library, use
`readarr_books(operation="search", book_id=123)`. The tool verifies that exact
positive Readarr ID exists, then submits `BookSearch` with `bookIds=[123]` to
`/api/v1/command`. Its response includes the native command ID and status;
submission does not mean a download has completed. Search may automatically
grab a matching release using the author's quality profile (for example,
Spoken for audiobooks). `operation="lookup"` remains a metadata lookup.

Book deletion is a dry run by default and previews the ID, title, author, and
file implications. Actual deletion removes only the requested book record;
the author is retained. Files are kept unless `delete_files=true` is paired
with `confirm_delete_files=true`. Bulk deletion preflights all unique IDs,
accepts at most 100, then deletes sequentially and stops on the first failure.

DELETE calls accept successful empty responses (including HTTP 200/204)
across Radarr, Sonarr, Lidarr, Prowlarr, Readarr and Overseerr. This also covers
book/episode file deletion and bulk blocklist deletion. Empty responses return
`{}` instead of a JSON parsing error. HTTP failures and malformed nonempty
responses still surface as errors.

### Radarr and Sonarr list exclusions

Both delete operations accept `add_import_list_exclusion` (default `false`),
matching the **Add List Exclusion** checkbox. Set it to `true` to prevent
import lists from re-adding the deleted movie or series. File deletion remains
a separate option, `delete_files`, also defaulting to `false`.

```python
radarr_movies(operation="delete", movie_id=42, add_import_list_exclusion=True)
sonarr_series(operation="delete", series_id=42, add_import_list_exclusion=True)
```

The shared tool argument maps to Radarr's `addImportExclusion` query parameter
and Sonarr's `addImportListExclusion`. Pass `delete_files=True` separately if
the associated files should also be deleted.

## Supported Services

| Service | Default Port | Config Prefix | Description |
|---------|-------------|---------------|-------------|
| Radarr | 7878 | `RADARR_` | Movies |
| Sonarr | 8989 | `SONARR_` | TV Series |
| Lidarr | 8686 | `LIDARR_` | Music |
| Prowlarr | 9696 | `PROWLARR_` | Indexer Backbone |
| Readarr | 8787 | `READARR_` | Books |
| Overseerr | 5055 | `OVERSEERR_` | Media Requests & Discovery |
| Bazarr | 6767 | `BAZARR_` | Subtitles |

## Webapp Pages (15)

| Page | Route | What it does |
|------|-------|-------------|
| **Dashboard** | `/` | Live health cards for all 7 services, polls every 15s |
| **Radarr / Sonarr / Lidarr / Prowlarr / Readarr / Overseerr / Bazarr** | `/{service}` | Live per-service data: counts, wanted, queue, disk space |
| **Orchestrate** | `/orchestrate` | Cross-arr pipeline viz, stack overview with total wanted |
| **Chat** | `/chat` | AI chat with Ollama/LM Studio model selection |
| **Inspector** | `/inspector` | Interactive MCP tool runner — select, param, execute |
| **Logger** | `/logger` | Real-time SSE log streaming with filter/export |
| **Help** | `/help` | Setup guide, Docker Compose, per-service install, API keys |
| **Settings** | `/settings` | LLM provider config, backend health, port reference |

## Cross-Arr Orchestration

```
"I want to watch Dune"
       ↓
  Jellyfin check → already in library? → DONE
       ↓ not found
  Type detection → movie → Radarr
       ↓
  Queue check → not already queued?
       ↓
  Add to Radarr → downloading...
```

## Project Structure

```
arr-mcp/
├── src/arr_mcp/             # Python backend (FastMCP 3.3)
│   ├── services/            # 8 arr clients (BaseArrClient + 7 arrs)
│   ├── tools/               # 29 MCP tools (22 portmanteau + 1 Readarr options + 3 mutations + 3 prefab cards)
│   ├── prefabs.py           # Prefab-UI card builders (health, calendar, stats, orchestrate)
│   ├── utils/               # Jellyfin bridge
│   ├── api.py               # REST router with /api/{service}/summary
│   ├── app.py               # FastMCP singleton, lifespan, resources, prompts
│   ├── server.py            # Entry point, auto-discovery, log buffer
│   ├── config.py            # Pydantic v2 config + .env loading
│   └── transport.py         # STDIO/HTTP/SSE + FastMCP 3.3 env vars
├── webapp/                  # React 19 + Vite + Tailwind
│   ├── src/pages/           # 15 page components
│   ├── src/utils/           # apiFetch<T>(), notifications, LLM client
│   └── e2e/                 # Playwright smoke tests (15 pages)
├── native/                  # Tauri 2.0 app wrapper
│   ├── Cargo.toml           # Rust deps (tauri 2, shell/fs/process)
│   ├── src/main.rs          # Entry point, sidecar launch, cleanup
│   ├── tauri.conf.json      # Window config, sidecar path
│   ├── capabilities/        # Tauri 2.0 permission model
│   ├── icons/               # App icons
│   └── build-sidecar.ps1    # PyInstaller → binaries/
├── tests/                   # pytest + pytest-httpx (143 tests)
├── docker-compose.yml       # Full *arr stack (8 services)
├── .github/workflows/ci.yml # GitHub Actions (ruff + pytest + biome + tsc)
├── justfile                 # Fleet-standard recipes
├── arr-mcp-backend.spec     # PyInstaller spec
└── run_server.py            # PyInstaller entry point
```

## Development

```bash
just install       # uv sync + pre-commit
just start         # run MCP server
just webapp        # start React dev server
just lint          # ruff check
just typecheck     # mypy
just test          # pytest with coverage (143 tests)
just e2e           # Playwright e2e (starts backend + webapp)
just ci            # lint + typecheck + test + webapp build
just tauri-build   # full native installer
just tauri-dev     # Tauri hot-reload
just tauri-sidecar # PyInstaller backend only → native/binaries/
```

### E2E Tests (Playwright)

```bash
cd webapp
npm install
npx playwright install chromium
npm run test:e2e          # headless
npm run test:e2e:ui       # interactive UI
npm run test:e2e:headed   # visible browser
```

Playwright auto-starts the backend (`:10938`) and Vite dev server (`:10939`). 15 smoke tests verify every page loads without crashing.

## Ports

- Backend: **10938** (FastMCP HTTP `/mcp` + REST `/api/*` + SSE `/api/logs/stream`)
- Frontend: **10939** (Vite React dashboard)

## License

MIT

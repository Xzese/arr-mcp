# arr-mcp — User Guide

## Quick Start

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/sandraschi/arr-mcp.git
   cd arr-mcp
   ```

2. **Install dependencies:**
   ```bash
   uv sync
   ```

3. **Configure `.env`:**
   ```env
   RADARR_API_KEY=your_radarr_api_key
   SONARR_API_KEY=your_sonarr_api_key
   # Optional services:
   # LIDARR_API_KEY=...
   # PROWLARR_API_KEY=...
   # READARR_API_KEY=...
   # OVERSEERR_API_KEY=...
   # BAZARR_API_KEY=...
   ```

4. **Run the server:**
   ```bash
   uv run run_server.py
   ```

5. **Add to Claude Desktop:**
   ```json
   {
     "arr-mcp": {
       "command": "uv",
       "args": ["run", "--directory", "C:\\path\\to\\arr-mcp", "run_server.py"],
       "env": {"RADARR_API_KEY": "..."}
     }
   }
   ```

6. **Verify:**
   Call `arr_health()` to see which services are reachable.

## Tutorials

### Tutorial 1: Stack-Wide Health Check

The first thing to do is verify which *arr services are reachable.

```python
health = arr_health(service="all")
print(health['message'])
for service_name, status in health['data'].items():
    print(f"{service_name}: {'UP' if status.get('reachable') else 'DOWN'} {status.get('version', '')}")
```

### Tutorial 2: List and Search Movies (Radarr)

Discover what movies Radarr is managing.

```python
# List all movies
movies = radarr_movies(operation="list")
print(f"Total movies: {len(movies['data'])}")
for m in movies['data'][:5]:
    print(f"  {m.get('title')} ({m.get('year')})")

# Lookup a movie to add
results = radarr_movies(operation="lookup", term="Dune")
for r in results['data']:
    print(f"{r.get('title')} ({r.get('year')}) — TMDB: {r.get('tmdbId')}")
```

### Tutorial 3: Add a Movie to Radarr

Search, find, and add a movie with the desired quality profile.

```python
# Step 1: Look up the movie
results = radarr_movies(operation="lookup", term="The Matrix")
tmdb_id = results['data'][0]['tmdbId']

# Step 2: Add it with quality profile
result = radarr_movies(
    operation="add",
    tmdb_id=tmdb_id,
    title="The Matrix",
    quality_profile_id=1,
    root_folder_path="/media/Movies",
    search_for_movie=True
)
print(result['message'])
```

### Tutorial 4: Browse Sonarr Series

View and search TV series managed by Sonarr.

```python
# List all series
series = sonarr_series(operation="list")
print(f"Total series: {len(series['data'])}")

# Look up a specific show
found = sonarr_series(operation="lookup", term="Breaking Bad")
for s in found['data']:
    print(f"{s.get('title')} — TVDB: {s.get('tvdbId')}")
```

### Tutorial 5: Cross-Arr Orchestration

The killer feature — check media servers and auto-route to the correct *arr.

```python
# Check if a movie exists on your media server, then request it
result = arr_orchestrate(
    operation="request",
    media_title="Dune",
    media_type="movie",
    quality_profile_id=1,
    root_folder_path="/media/Movies"
)

# Show the pipeline steps
for step in result.get('pipeline', []):
    print(f"[{step.get('step')}] {step.get('action')}: {step.get('result', '')}")
```

### Tutorial 6: Unified Calendar

See upcoming releases across all *arr services.

```python
# Next two weeks
calendar = arr_calendar(operation="upcoming")
for service, items in calendar.get('data', {}).items():
    for item in items[:3]:
        print(f"[{service}] {item.get('title')} — {item.get('releaseDate')}")

# Specific date range
calendar = arr_calendar(operation="range", start="2026-06-01T00:00:00Z", end="2026-06-30T23:59:59Z")

# Filter by type
calendar = arr_calendar(operation="upcoming", types=["movie", "series"])
```

### Tutorial 7: Stack Statistics

Get consolidated statistics across the entire stack.

```python
# Summary
stats = arr_stats(operation="summary")
for service, data in stats.get('data', {}).items():
    print(f"{service}: {data.get('total', '?')} items, {data.get('wanted', '?')} wanted")

# Disk usage
disk = arr_stats(operation="disk")
for mount, usage in disk.get('data', {}).items():
    print(f"{mount}: {usage.get('freeSpace') / 1e9:.1f} GB free")

# Active queues
queues = arr_stats(operation="queues")
print(f"Queued items: {queues.get('total', 0)}")
```

### Tutorial 8: Prowlarr Indexer Management

Manage your Usenet and Torrent indexers.

```python
# List all configured indexers
indexers = prowlarr_indexers(operation="list")
for idx in indexers['data']:
    print(f"{idx.get('name')} — {idx.get('configContract')} — {'enabled' if idx.get('enableRss') else 'disabled'}")

# Test a specific indexer
test = prowlarr_indexers(operation="test", indexer_id=1)
print(test['message'])

# Test all indexers at once
test_all = prowlarr_indexers(operation="test_all")
```

### Tutorial 9: Manage Media Requests (Overseerr)

Handle user-submitted media requests.

```python
# View pending requests
pending = overseerr_requests(operation="pending")
for r in pending['data'].get('results', []):
    print(f"Request #{r.get('id')}: {r.get('media', {}).get('title')}")

# Approve a request
overseerr_requests(operation="approve", request_id=42)

# Create a new request
result = overseerr_requests(
    operation="create",
    media_type="movie",
    media_id=438631  # TMDB ID
)
print(result['message'])
```

### Tutorial 10: Manage Subtitles (Bazarr)

Search and download subtitles for your media.

```python
# Check what subtitles are wanted/missing
wanted = bazarr_subtitles(operation="wanted")
print(f"Items missing subtitles: {wanted['message']}")

# Search for subtitles
subs = bazarr_subtitles(operation="search", movie_id=42, language="en")
for sub in subs['data']:
    print(f"  {sub.get('name')} — {sub.get('score')}%")

# Download subtitles
bazarr_subtitles(
    operation="download",
    movie_id=42,
    subtitle_path=subs['data'][0]['path'],
    language="en"
)
```

### Tutorial 11: Agentic Cross-Arr Workflows

Use natural language with LLM-powered workflow orchestration.

```python
# Multi-step workflow
result = arr_agentic(
    operation="workflow",
    prompt="add The Matrix to Radarr with best quality"
)
print(result['message'])

# Natural language query
query = arr_agentic(
    operation="natural_query",
    prompt="what's my most wanted movie right now?"
)
print(query.get('data', {}).get('response', ''))
```

### Tutorial 12: Prefab UI Cards

Visual cards for health, calendar, and stats in supporting MCP clients.

```python
# Health card
arr_health_card()

# Calendar card (next 14 days)
arr_calendar_card(days=14)

# Stats card
arr_stats_card()
```

### Tutorial 13: Lidarr Music Management

Search and add music artists via Lidarr.

```python
# Look up an artist
artists = lidarr_artists(operation="lookup", term="Nine Inch Nails")
foreign_id = artists['data'][0].get('foreignArtistId')

# Add the artist
result = lidarr_artists(
    operation="add",
    foreign_artist_id=foreign_id,
    quality_profile_id=1,
    root_folder_path="/media/Music"
)

# List their albums
albums = lidarr_albums(operation="list", artist_id=result['data'].get('id'))
```

### Tutorial 14: Readarr Book Management

Search and add authors via Readarr.

```python
# Look up an author
authors = readarr_authors(operation="lookup", term="Brandon Sanderson")
foreign_id = authors['data'][0].get('foreignAuthorId')

# Add the author
result = readarr_authors(
    operation="add",
    foreign_author_id=foreign_id,
    quality_profile_id=1,
    root_folder_path="/media/Books"
)
```

## Troubleshooting

### "Service not configured — skipping tools"

If a service is not reachable and has no API key configured, its tools are not registered. Check:
- The service is running on its default port
- The API key is set in `.env`
- Auto-discovery: set the API key without the URL to enable port probing

### "Health check fails"

Run `arr_health(service="all")` to see which services fail. Common issues:
- Wrong port: check the service's default port
- API key mismatch: verify the key in the *arr web UI
- Firewall: ensure localhost connections are allowed

### "Operation X requires parameter Y"

All portmanteau tools use Literal types for operations. Each operation has specific required parameters documented in the return schema.

## FAQ

**Q: Do I need all *arr services?**
A: No. Each service is optional. Unconfigured services simply skip tool registration. The server works with any subset.

**Q: How does auto-discovery work?**
A: If an API key is set but no URL, arr-mcp probes the default port for that service. If the port responds, the service URL is auto-configured.

**Q: What happens if I delete a movie with delete_files=True?**
A: The movie is removed from Radarr and its files are permanently deleted from disk.

**Q: Can I search across all indexers at once?**
A: Yes. Use `prowlarr_search(operation="search", query="...")` for unified search across all configured Prowlarr indexers.

**Q: Does arr-mcp support Jellyfin?**
A: Yes. Configure `JELLYFIN_URL` and `JELLYFIN_API_KEY` for cross-arr orchestration.

**Q: Can I approve Overseerr requests automatically?**
A: Yes. Use `overseerr_requests(operation="approve", request_id=N)`. For auto-approval, configure Overseerr's own settings.

**Q: What ports does arr-mcp use?**
A: Backend port 10938, frontend port 10939.

**Q: How do I get quick help within a session?**
A: Use `arr_help(operation="discover")` to list all tools, or `arr_help(operation="quickstart")` for a step-by-step guide.

**Q: Does Bazarr integration feed into other systems?**
A: Yes. Subtitle data from Bazarr can feed into jellyfin-mcp's RAG pipeline for downstream analysis.

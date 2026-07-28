# arr-mcp (MCPB Bundle)

FastMCP 3.3+ MCP server for the complete *arr automation stack

## Usage

Add to \claude_desktop_config.json\:
\\\json
{
  "mcpServers": {
    "arr-mcp": {
      "command": "uv",
      "args": ["run", "--directory", "\D:\Dev\repos", "python", "-m", "arr_mcp"],
      "env": { "PYTHONPATH": "\D:\Dev\repos/src" }
    }
  }
}
\\\

## Tools

- **api_health**: api_health
- **api_logs**: api_logs
- **api_logs_stream**: SSE endpoint for real-time log streaming.
- **radarr_summary**: radarr_summary
- **radarr_movies**: radarr_movies
- **sonarr_summary**: sonarr_summary
- **sonarr_series**: sonarr_series
- **lidarr_summary**: lidarr_summary
- **prowlarr_summary**: prowlarr_summary
- **readarr_summary**: readarr_summary
- **overseerr_summary**: overseerr_summary
- **bazarr_summary**: bazarr_summary
- **orchestrator_summary**: orchestrator_summary
- **health_alias**: health_alias
- **arr_agentic**: arr_agentic
- **register_agentic_tools_workflow**: register_agentic_tools(workflow)
- **register_agentic_tools_natural_query**: register_agentic_tools(natural_query)
- **bazarr_subtitles**: bazarr_subtitles
- **register_bazarr_tools_wanted**: register_bazarr_tools(wanted)
- **register_bazarr_tools_search**: register_bazarr_tools(search)
- **register_bazarr_tools_download**: register_bazarr_tools(download)
- **register_bazarr_tools_history**: register_bazarr_tools(history)
- **register_bazarr_tools_providers**: register_bazarr_tools(providers)
- **register_bazarr_tools_languages**: register_bazarr_tools(languages)
- **arr_orchestrate**: arr_orchestrate
- **arr_calendar**: arr_calendar
- **arr_stats**: arr_stats
- **register_cross_arr_tools_request**: register_cross_arr_tools(request)
- **register_cross_arr_tools_status**: register_cross_arr_tools(status)
- **register_cross_arr_tools_check_jellyfin**: register_cross_arr_tools(check_jellyfin)
- **register_cross_arr_tools_check_plex**: register_cross_arr_tools(check_plex)
- **register_cross_arr_tools_check_emby**: register_cross_arr_tools(check_emby)
- **register_cross_arr_tools_queue**: register_cross_arr_tools(queue)
- **arr_health**: arr_health
- **register_health_tools_all**: register_health_tools(all)
- **register_health_tools_radarr**: register_health_tools(radarr)
- **register_health_tools_sonarr**: register_health_tools(sonarr)
- **register_health_tools_lidarr**: register_health_tools(lidarr)
- **register_health_tools_prowlarr**: register_health_tools(prowlarr)
- **register_health_tools_readarr**: register_health_tools(readarr)
- **register_health_tools_overseerr**: register_health_tools(overseerr)
- **register_health_tools_bazarr**: register_health_tools(bazarr)
- **arr_help**: arr_help
- **register_help_tools_discover**: register_help_tools(discover)
- **register_help_tools_tool_info**: register_help_tools(tool_info)
- **register_help_tools_quickstart**: register_help_tools(quickstart)
- **lidarr_artists**: lidarr_artists
- **lidarr_albums**: lidarr_albums
- **register_lidarr_tools_list**: register_lidarr_tools(list)
- **register_lidarr_tools_lookup**: register_lidarr_tools(lookup)
- **register_lidarr_tools_get**: register_lidarr_tools(get)
- **register_lidarr_tools_add**: register_lidarr_tools(add)
- **register_lidarr_tools_delete**: register_lidarr_tools(delete)
- **register_lidarr_tools_update**: register_lidarr_tools(update)
- **overseerr_requests**: overseerr_requests
- **overseerr_search**: overseerr_search
- **overseerr_users**: overseerr_users
- **register_overseerr_tools_list**: register_overseerr_tools(list)
- **register_overseerr_tools_get**: register_overseerr_tools(get)
- **register_overseerr_tools_create**: register_overseerr_tools(create)
- **register_overseerr_tools_approve**: register_overseerr_tools(approve)
- **register_overseerr_tools_decline**: register_overseerr_tools(decline)
- **register_overseerr_tools_delete**: register_overseerr_tools(delete)
- **register_overseerr_tools_count**: register_overseerr_tools(count)
- **register_overseerr_tools_pending**: register_overseerr_tools(pending)
- **arr_health_card**: arr_health_card
- **arr_calendar_card**: arr_calendar_card
- **arr_stats_card**: arr_stats_card
- **prowlarr_indexers**: prowlarr_indexers
- **prowlarr_search**: prowlarr_search
- **prowlarr_applications**: prowlarr_applications
- **prowlarr_history**: prowlarr_history
- **register_prowlarr_tools_list**: register_prowlarr_tools(list)
- **register_prowlarr_tools_get**: register_prowlarr_tools(get)
- **register_prowlarr_tools_add**: register_prowlarr_tools(add)
- **register_prowlarr_tools_update**: register_prowlarr_tools(update)
- **register_prowlarr_tools_delete**: register_prowlarr_tools(delete)
- **register_prowlarr_tools_test**: register_prowlarr_tools(test)
- **register_prowlarr_tools_test_all**: register_prowlarr_tools(test_all)
- **register_prowlarr_tools_schema**: register_prowlarr_tools(schema)
- **register_radarr_tools**: register_radarr_tools
- **register_radarr_tools_list**: register_radarr_tools(list)
- **register_radarr_tools_lookup**: register_radarr_tools(lookup)
- **register_radarr_tools_get**: register_radarr_tools(get)
- **register_radarr_tools_add**: register_radarr_tools(add)
- **register_radarr_tools_delete**: register_radarr_tools(delete)
- **register_radarr_tools_update**: register_radarr_tools(update)
- **register_radarr_tools_import**: register_radarr_tools(import)
- **readarr_authors**: readarr_authors
- **readarr_books**: readarr_books
- **register_readarr_tools_list**: register_readarr_tools(list)
- **register_readarr_tools_lookup**: register_readarr_tools(lookup)
- **register_readarr_tools_get**: register_readarr_tools(get)
- **register_readarr_tools_add**: register_readarr_tools(add)
- **register_readarr_tools_delete**: register_readarr_tools(delete)
- **register_readarr_tools_update**: register_readarr_tools(update)
- **sonarr_episodes**: sonarr_episodes
- **register_sonarr_tools_list**: register_sonarr_tools(list)
- **register_sonarr_tools_lookup**: register_sonarr_tools(lookup)
- **register_sonarr_tools_get**: register_sonarr_tools(get)
- **register_sonarr_tools_add**: register_sonarr_tools(add)
- **register_sonarr_tools_delete**: register_sonarr_tools(delete)
- **register_sonarr_tools_update**: register_sonarr_tools(update)

## Requirements

- Python 3.12+
- uv

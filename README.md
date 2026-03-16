# vibe-kanban-mobile

A mobile-optimized remote control UI for [vibe-kanban](https://github.com/bjjwwang/vibe-kanban), designed for iPhone Safari in portrait mode.

## Features

- **Workspace list** — flat view of all workspaces with real-time status via WebSocket (active/idle/error indicators)
- **Session control** — send follow-up prompts to AI agents (Ctrl/Cmd+Enter), view process history
- **Stop button** — instantly stop a running workspace task
- **Conversation log** — full markdown-rendered view of agent messages, tool calls, and system events
- **Completion sound** — plays a chime when a task finishes (Web Audio API, no external files)
- **Draft persistence** — textarea content survives accidental navigation via sessionStorage
- **Cookie-based auth** — integrates with nginx `auth_request` for authentication

## Architecture

Single `index.html` file (no build step, no dependencies) that:
1. Connects to vibe-kanban's REST API (`/api/projects`, `/api/sessions`, etc.)
2. Streams real-time updates via WebSocket (JSON Patch protocol)
3. Renders a three-view SPA: Home → Session → Log

## Deployment

Serve `index.html` behind a reverse proxy that routes `/api/` to your vibe-kanban instance.

Example nginx config:

```nginx
server {
    listen 80;
    server_name kanban.example.com;

    # Mobile UI
    location /m/ {
        alias /path/to/vibe-kanban-mobile/;
        index index.html;
        default_type text/html;
    }

    # Proxy API + WebSocket to vibe-kanban
    # Replace the port with your vibe-kanban instance port
    location / {
        proxy_pass http://127.0.0.1:<vibe-kanban-port>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 3600s;
    }
}
```

Then visit `https://kanban.example.com/m/` on your phone.

## API Endpoints Used

| Endpoint | Method | Description |
|---|---|---|
| `/api/projects` | GET | List projects |
| `/api/tasks?project_id=` | GET | List tasks per project |
| `/api/sessions?workspace_id=` | GET | Get sessions for a workspace |
| `/api/sessions` | POST | Create new session |
| `/api/sessions/{id}/follow-up` | POST | Send prompt to agent |
| `/api/task-attempts/{id}/stop` | POST | Stop a running workspace |
| `/api/task-attempts/stream/ws` | WS | Real-time workspace status |
| `/api/execution-processes/stream/session/ws` | WS | Real-time process updates |
| `/api/execution-processes/{id}/normalized-logs/ws` | WS | Conversation log stream |

## License

MIT

# Purdue Gold Chat Frontend + CLI Test Server

Unified setup for the Purdue-themed React frontend and the Python CLI test server. The frontend proxies API calls to the CLI server by default.

## Overview

```
Frontend (Vite React, port 8080) → /api/* (Vite proxy) → CLI Server (Python, port 8000)
```

## Requirements

- Node.js 18+ and npm
- Python 3.10+
- A Gemini API key (starts with `AIzaSy`)

## Quick Start

### 1) Terminal 1 — Run the CLI server from GitHub (rao305/cli-demo)
```bash
git clone https://github.com/rao305/cli-demo.git
cd cli-demo
# Optional: set your key non-interactively
export GEMINI_API_KEY="<your_key>"   # PowerShell: $env:GEMINI_API_KEY="<your_key>"
python cli_server.py
```
The server listens on `http://localhost:8000` and exposes:
- `GET /health`
- `POST /query` with JSON `{ "query": "..." }`

### 2) Terminal 2 — Run the Purdue frontend
```bash
npm install
npm run dev
```
Open `http://localhost:8080`. Messages you send POST to `/api/query` and are proxied to `http://localhost:8000/query`.

## How the connection works

- Vite dev proxy is configured in `purdue-gold-chat-main/vite.config.ts`:
  - `/api` → `http://localhost:8000`
- Chat page `purdue-gold-chat-main/src/pages/Index.tsx` does:
  - `fetch('/api/query', { method: 'POST', body: { query } })`
- The CLI server (`cli_server.py`) echoes the query back. Replace its logic with your own when ready.

## Commands

- Start CLI: `python cli_server.py`
- Start frontend: `cd purdue-gold-chat-main && npm run dev`
- Build frontend: `cd purdue-gold-chat-main && npm run build`

## Changing ports

- Frontend port: edit `purdue-gold-chat-main/vite.config.ts` (`server.port`).
- Backend port: change `HTTPServer(('localhost', 8000), RequestHandler)` in `cli_server.py` and update the proxy target in `vite.config.ts`.

## Troubleshooting

- Chat shows an error toast:
  - Ensure the CLI server is running and listening on `http://localhost:8000`.
  - Verify the Vite proxy in `purdue-gold-chat-main/vite.config.ts`.
- Permission error removing `node_modules` on Windows (e.g., `esbuild.exe locked`):
  - Close any running dev servers or terminals using the folder, then retry.
- CORS during development:
  - The proxy avoids CORS. Always call the backend via `/api/...` from the frontend.

## Project layout

```
purdue-gold-chat-main/
  src/pages/Index.tsx          # Chat UI wired to /api/query
  vite.config.ts               # /api → http://localhost:8000 proxy
```

## Extending the server (plug in your logic)

In `cli_server.py`, update `SimpleBoilerAI.process_query` or `BoilerAICLIServer.process_query` to call your model/RAG pipeline and return its response in the `response` field. The frontend will render it automatically.

---

All set. Start both processes and chat at `http://localhost:8080`.

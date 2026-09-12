# AGENTS.md — Base44 Dev Notes

## Project Overview
AI-Enhanced Honeypot System (Python/Flask). Runs 4 honeypots (SSH, HTTP, FTP, Telnet),
an ML inference engine, and a web dashboard — all from a single `python main.py` entry point.

## Stack
- Python 3.12, Flask (dashboard on port 5000, mapped to host 3000)
- SQLite at `data/honeypot.db` (pre-populated with sample attack events)
- Pre-trained ML models in `ml/model/` (Random Forest classifier + scaler, joblib format)
- Dependencies installed at container startup via `pip install -r requirements.txt`

## Running
```
docker compose -f docker-compose.base44.yml up -d
```
Dashboard: http://localhost:3000 (container port 5000 → host 3000)
API endpoints: `/api/stats`, `/api/events`

## No External Secrets Required
All data is local (SQLite, local files). `SECRET_KEY` has a built-in default.
No external API keys or credentials are needed to run the app.

## Quirks
- `main.py` creates required directories on startup, including `keys/` for the SSH honeypot's
  generated RSA host key (`keys/ssh_host_key`). If missing, the SSH honeypot crashes on import.
- The app does NOT use live-reload. Python code changes require a container restart:
  `docker compose -f docker-compose.base44.yml restart`. Flask templates are re-read on each
  request, so template edits appear on browser refresh without restart.
- Honeypot ports (2222, 8080, 2121, 2323) are internal to the container and not exposed to the host.

## Verification
- `curl -s http://localhost:3000/` → HTTP 200 (dashboard HTML)
- `curl -s http://localhost:3000/api/stats` → JSON with event counts

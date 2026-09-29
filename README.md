# Hermes Docker — Reproducible AI Agent Infrastructure

This repository defines a complete, production-ready Docker stack for running **Hermes Agent** (by Nous Research) as a persistent infrastructure controller. It gives you a reproducible environment to build, test, and run MVPs with real backing services (databases, queues, APIs, workers) — all managed by an AI agent that can execute code, control Docker, and orchestrate multi-container workloads.

---

## What This Stack Runs

| Service | Purpose | Ports |
|---------|---------|-------|
| **Hermes Agent** | AI infrastructure agent (API + Dashboard) | 8642 (API), 9119 (Dashboard) |
| **Docker Socket Proxy** | Least-privilege Docker API for Hermes | 2375 (internal) |
| **WireGuard VPN** | Secure overlay to reach services from your laptop | 51820/udp |

Hermes runs inside the stack with `DOCKER_HOST=tcp://docker-socket-proxy:2375`, so it can create/manage containers, networks, volumes — but only the capabilities the proxy exposes (containers, images, networks, volumes, POST).

---

## Quick Start (Reproduce This Exact Stack)

### 1. Clone & Configure

```bash
git clone https://github.com/gilbertoesp/hermes-docker.git
cd hermes-docker

# Copy the example env and fill in your secrets
cp .env.example .env
# Edit .env with your actual values (see Configuration below)
```

### 2. Required Secrets (`.env`)

| Variable | Source | Notes |
|----------|--------|-------|
| `OPENROUTER_API_KEY` | [OpenRouter](https://openrouter.ai/keys) | LLM provider for Hermes |
| `NVIDIA_API_KEY` | [NVIDIA NGC](https://ngc.nvidia.com/) | Optional alternative provider |
| `GITHUB_TOKEN` | [GitHub Fine-grained PAT](https://github.com/settings/tokens) | Repo access for Hermes skills/plugins |
| `HERMES_DASHBOARD_BASIC_AUTH_USERNAME` | You choose | Dashboard login |
| `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD` | You choose | Strong password |
| `HERMES_DASHBOARD_BASIC_AUTH_SECRET` | `openssl rand -hex 32` | Session signing key |
| `API_SERVER_KEY` | `openssl rand -hex 32` | API server auth key |

### 3. Launch

```bash
docker compose up -d
```

### 4. Access

- **Dashboard**: `http://<host>:9119` (basic auth from `.env`)
- **API Server**: `http://<host>:8642` (OpenAI-compatible, key = `API_SERVER_KEY`)
- **WireGuard**: Configs generated in `~/.hermes/wireguard/config/` — import on your laptop to reach services securely

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Your Laptop                             │
│  ┌─────────────┐    WireGuard (10.13.13.0/24)    ┌────────┐ │
│  │  Browser    │ ◄─────────────────────────────► │ Hermes │ │
│  │  :9119      │                                 │ Dashboard│ │
│  └─────────────┘                                 └────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Docker Host (Server)                      │
│  ┌──────────────────┐   ┌──────────────────────────────┐   │
│  │  wireguard       │   │  hermes_net (bridge)         │   │
│  │  :51820/udp      │   │  ┌────────────────────────┐  │   │
│  └──────────────────┘   │  │ docker-socket-proxy   │  │   │
│                         │  │  :2375 (TCP)           │  │   │
│                         │  └───────────┬────────────┘  │   │
│                         │              │               │   │
│                         │  ┌───────────▼────────────┐  │   │
│                         │  │      hermes           │  │   │
│                         │  │  API :8642            │  │   │
│                         │  │  Dashboard :9119      │  │   │
│                         │  │  DOCKER_HOST=proxy    │  │   │
│                         │  └────────────────────────┘  │   │
│                         └──────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## What Hermes Can Do In This Environment

- **Spin up MVP stacks**: `docker compose` files, databases (Postgres, Redis, etc.), message queues, APIs
- **Run code**: Python, Node, shell — in containers or the host via tools
- **Manage skills**: Install/author Hermes skills for repeatable workflows (deploy, test, monitor)
- **Cron jobs**: Schedule recurring tasks (backups, health checks, data pipelines)
- **Web dashboard**: Chat with the agent, view logs, approve actions
- **API server**: Integrate Hermes into your own tools (OpenAI-compatible `/v1/chat/completions`)

---

## Persisting Data

All Hermes state lives in `~/.hermes/` (mounted to `/opt/data` in-container):

```
~/.hermes/
├── .env                    # Your secrets (gitignored)
├── config.yaml             # Hermes config (auto-generated on first run)
├── skills/                 # Installed skills
├── plugins/                # Custom plugins
├── cron/                   # Scheduled jobs
├── memories/               # Persistent memory across sessions
├── sessions/               # Conversation history
├── wireguard/config/       # WireGuard peer configs
└── state.db                # Agent state (SQLite)
```

Back up `~/.hermes/` to migrate or restore the entire agent.

---

## Updating

```bash
cd hermes-docker
git pull
docker compose pull
docker compose up -d
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Dashboard not loading | Check `docker logs hermes` — verify `.env` vars are set |
| Hermes can't create containers | Check `docker logs docker-socket-proxy` — ensure proxy has `POST=1` |
| WireGuard handshake fails | Verify `SERVERURL` in `.env` (or docker-compose.yml) matches your public IP/DDNS |
| API returns 401 | Confirm `API_SERVER_KEY` matches what you're sending in `Authorization: Bearer` |

---

## References

- **Hermes Agent**: https://github.com/NousResearch/hermes-agent
- **Documentation**: https://hermes-agent.nousresearch.com/docs
- **Docker Socket Proxy**: https://github.com/Tecnativa/docker-socket-proxy
- **WireGuard (linuxserver.io)**: https://github.com/linuxserver/docker-wireguard

---

## License

MIT — use freely for your own infrastructure.
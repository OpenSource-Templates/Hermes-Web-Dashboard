# Deploy and Host Hermes Agent on Railway (with Web Dashboard)

Deploy [Hermes Agent](https://github.com/NousResearch/hermes-agent) on [Railway](https://railway.app) with a web-based admin dashboard for configuration, gateway management, and user pairing.

This template always builds from the latest upstream Hermes Agent source (`HERMES_REF=main` by default). Every redeploy that rebuilds the image pulls the newest `main` branch.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/hermes-web-dashboard)

> Hermes Agent is an autonomous AI agent by [Nous Research](https://nousresearch.com/) that lives on your server, connects to your messaging channels (Telegram, Discord, Slack, WhatsApp, ntfy, and more), and gets more capable the longer it runs.

## Features

- **Admin Dashboard** — dark-themed setup wizard at `/setup` to configure providers, channels, tools, and manage the gateway
- **Full Hermes Dashboard** — the native Hermes web UI (Chat, Keys, Skills, Kanban, Analytics, Console) is proxied at `/`, behind the same login
- **One-Page Setup** — provider dropdown, checkbox-based channel/tool toggles — no config files to edit
- **Gateway Management** — start, stop, restart the Hermes gateway from the browser, with automatic restart if it crashes
- **Live Status** — stat cards for gateway state, uptime, model, and pending pairing requests
- **Live Logs** — streaming gateway log viewer
- **User Pairing** — approve or deny users who message your bot, revoke access anytime
- **Password-Protected** — one cookie-based login guards both the setup wizard and the Hermes dashboard
- **Reset Config** — one-click reset to start fresh
- **Backup & Restore** — download a full snapshot (config, credentials, chat history, memories, skills) as a zip, and restore it — including into a fresh project — to clone a deployment. Not encrypted; a safety snapshot is taken automatically before every restore.
- **Always latest** — defaults to `HERMES_REF=main` so rebuilds track upstream

## Getting Started

### 1. Get an LLM Provider Key

1. Register at [OpenRouter](https://openrouter.ai/) (recommended) or use OpenAI / Anthropic / xAI / others
2. Create an API key
3. You can also pick a model later in the dashboard

### 2. Deploy on Railway

1. Deploy this template (or connect the repo and deploy)
2. Attach a **volume** at `/data` (required — config and sessions live there)
3. Set `ADMIN_PASSWORD` (optional but recommended; if unset a random password is logged)
4. Open the public URL → log in → complete `/setup`

### 3. Connect a messaging channel

Fastest path is usually **Telegram**:

1. Talk to [@BotFather](https://t.me/BotFather) → `/newbot` → copy the token
2. In the dashboard setup, enable Telegram and paste the token
3. Message the bot; approve the pairing request in the dashboard **Users** tab (or set allowlists)

Discord, Slack, WhatsApp, ntfy, and other platforms Hermes supports can be configured the same way from the UI.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | Web server port (set automatically by Railway) |
| `ADMIN_USERNAME` | `admin` | Login username |
| `ADMIN_PASSWORD` | *(auto-generated)* | Login password — if unset, a random password is printed in deploy logs |
| `HERMES_REF` | `main` | Git tag or branch of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) to install. Defaults to **latest `main`**. Override with a release tag (e.g. `v2026.9.24`) to pin. |

All other configuration (LLM provider, model, channels, tools) is managed through the admin dashboard after login.

Optional keys you may still set as Railway variables (also configurable in the UI):

```env
OPENROUTER_API_KEY=
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
TELEGRAM_BOT_TOKEN=
DISCORD_BOT_TOKEN=
# etc.
```

## Architecture

| Component | Role |
|-----------|------|
| `server.py` | Starlette app: login, `/setup` admin UI, reverse-proxy to Hermes dashboard, gateway supervisor |
| `start.sh` | Seeds `/data/.hermes`, stamps install method, launches `server.py` |
| Hermes Agent | Installed from git (`HERMES_REF`) into `/opt/hermes-agent` with dashboard + TUI prebuilt |
| Volume `/data` | Persistent `HERMES_HOME` (config, sessions, pairing, memories, skills) |

One login covers both the setup wizard and the native Hermes dashboard. The gateway is supervised: if it crashes, `server.py` restarts it with backoff.

Config lives on the volume at `/data/.hermes/` and survives redeploys.

## Volumes

| Mount path | What is stored |
|------------|----------------|
| `/data` | Hermes home (`.hermes`), config, sessions, pairing, memories, skills, workspace |

Do **not** remove this volume or you will lose configuration and chat history.

## Updating Hermes

This template defaults to **`HERMES_REF=main`** (always latest upstream).

- **Stay on latest:** redeploy / rebuild the image. The build clones `main` again.
- **Pin a release:** set Railway variable `HERMES_REF=v2026.9.24` (or any [release tag](https://github.com/NousResearch/hermes-agent/releases)), then redeploy.
- The in-dashboard “Update Hermes” button is a **no-op** in this container (immutable image). Change `HERMES_REF` and redeploy instead.

## Running Locally

```bash
docker build -t hermes-agent-dashboard .
docker run --rm -it -p 8080:8080 \
  -e PORT=8080 \
  -e ADMIN_PASSWORD=changeme \
  -v hermes-data:/data \
  hermes-agent-dashboard
```

Open `http://localhost:8080` and log in with `admin` / `changeme`.

To pin a version locally:

```bash
docker build --build-arg HERMES_REF=v2026.9.24 -t hermes-agent-dashboard .
```

## Traps

- **No volume at `/data`** — config and sessions reset on every deploy.
- **Forgot admin password** — check deploy logs for the auto-generated value, or set `ADMIN_PASSWORD` and redeploy.
- **Gateway not starting** — open `/setup`, ensure a provider key and at least one channel are saved, then Start gateway.
- **Build fails on a broken `main` commit** — temporarily pin `HERMES_REF` to a known-good release tag.
- **In-app Update button** — does nothing on Railway; bump `HERMES_REF` / redeploy.

## Why this template

Railway hosts the container and volume; this image adds a password-protected admin UI and proxies the full Hermes dashboard so you can configure providers, channels, pairing, and the gateway from the browser without SSH for day-to-day use.

---

**Template version:** Hermes Agent from git `main` (override with `HERMES_REF`) · Admin dashboard + gateway supervisor

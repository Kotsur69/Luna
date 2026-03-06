# 🌙 Luna Core v1.0 — Setup Instructions

> Your AI companion. A 3D software engineer who lives in your workspace,
> remembers you, writes and runs her own code, and talks to you in voice chat.

---

## What Luna Can Do

| Capability | How |
|---|---|
| Instant chat | Ollama (local, offline, free) |
| Full codebase analysis | Gemini 1.5 Pro + context caching |
| Live web search | Gemini grounding (weather, crypto, news) |
| Write & run code | Sandbox executor (subprocess / Docker / E2B) |
| Read & write your repo files | File manager with path guardrails |
| Self-correct bugs | Propose → Write → Test → Reflect → Fix loop |
| Remember you | SQLite memory engine (humor, preferences, code style) |
| Speak in voice chat | edge-tts + ffmpeg → Discord VC via PyNaCl |
| Learn from feedback | Discord reactions / voice commands feed back into memory |
| Plugin system | MCP — plug in GitHub, Spotify, Slack with one config change |
| Self-documentation | LUNA.md (brain dump) + DEVLOG.md (changelog) |
| Live debug dashboard | React dashboard over WebSocket |

---

## Prerequisites

Before you run anything, make sure you have:

- **Python 3.11 or 3.12** — check with `python3 --version`
- **Git** — to clone/manage your repo
- **Node.js 18+** — needed for MCP servers via `npx`
- **ffmpeg** — for voice audio pipeline
- **Docker** (optional) — for the hardened sandbox backend
- **Ollama** — for local instant chat

---

## Step 1 — Install System Dependencies

### macOS
```bash
brew install ffmpeg ollama node
```

### Ubuntu / Debian
```bash
sudo apt update && sudo apt install -y ffmpeg nodejs npm
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows (WSL2 recommended)
```bash
# Inside WSL2 Ubuntu:
sudo apt install -y ffmpeg nodejs npm
curl -fsSL https://ollama.com/install.sh | sh
```

---

## Step 2 — Pull a Local Model for Ollama

Luna uses Ollama for instant offline chat. Pull at least one model:

```bash
# Fast, good quality (recommended for most machines)
ollama pull llama3.1:8b

# Smarter, needs 16GB+ RAM
ollama pull llama3.1:70b

# If you want code-focused local model
ollama pull codestral
```

Start Ollama (it runs as a background service):
```bash
ollama serve
```

---

## Step 3 — Place Luna Core in Your Project

Copy the `luna_core/` folder into your project:

```
your_project/
├── luna_core/          ← put this folder here
│   ├── main.py
│   ├── config.py
│   ├── discord_bot.py
│   └── ...
├── your_code_file.py
└── ...
```

---

## Step 4 — Install Python Dependencies

```bash
cd luna_core
pip install -r requirements.txt
```

What gets installed:
- `google-generativeai` — Gemini API + context caching
- `discord.py` + `PyNaCl` — Discord bot + voice encryption
- `edge-tts` — Microsoft TTS (free, 300+ voices)
- `httpx` — async HTTP for Ollama
- `python-dotenv` — .env loading
- `websockets` — live dashboard feed
- `mcp` — Model Context Protocol plugin system

---

## Step 5 — Configure Your .env

Copy the template and fill it in:

```bash
cp .env.template .env
nano .env   # or open in your editor
```

Here is every variable explained:

```env
# ── Your codebase ─────────────────────────────────────────
# Absolute path to your project root (the 40k-line repo)
REPO_DIR=/Users/you/your_project

# ── Ollama (local LLM) ────────────────────────────────────
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3.1:8b

# ── Gemini ────────────────────────────────────────────────
# Get your key at: https://aistudio.google.com/app/apikey
GEMINI_API_KEY=your_key_here
GEMINI_MODEL=gemini-1.5-pro-latest
GEMINI_CACHE_TTL_SECONDS=3600

# ── Discord ───────────────────────────────────────────────
# Create a bot at: https://discord.com/developers/applications
# Enable: MESSAGE CONTENT INTENT + SERVER MEMBERS INTENT
# Bot permissions needed: Send Messages, Read Messages,
#   Connect (voice), Speak, Use Voice Activity
DISCORD_TOKEN=your_bot_token_here
DISCORD_GUILD_ID=your_server_id_as_integer

# ── Voice / TTS ───────────────────────────────────────────
TTS_ENGINE=edge-tts
# Good voices to try: en-US-AriaNeural, en-US-JennyNeural, en-GB-SoniaNeural
TTS_VOICE=en-US-AriaNeural

# Optional: ElevenLabs for ultra-realistic voice (paid)
# ELEVENLABS_API_KEY=
# ELEVENLABS_VOICE_ID=

# ── Sandbox backend ───────────────────────────────────────
# subprocess = fast, no setup (default)
# docker     = fully isolated (requires Docker)
# e2b        = cloud sandbox (requires E2B_API_KEY)
SANDBOX_BACKEND=subprocess
SANDBOX_TIMEOUT_SEC=30

# ── Memory ────────────────────────────────────────────────
MEMORY_INJECT_LIMIT=10
```

---

## Step 6 — Get Your API Keys

### Gemini API Key
1. Go to https://aistudio.google.com/app/apikey
2. Click **Create API key**
3. Paste it into `GEMINI_API_KEY=` in your `.env`

### Discord Bot Token
1. Go to https://discord.com/developers/applications
2. Click **New Application** → name it "Luna"
3. Go to **Bot** tab → click **Reset Token** → copy the token
4. Paste into `DISCORD_TOKEN=` in your `.env`
5. Under **Privileged Gateway Intents**, enable:
   - ✅ Message Content Intent
   - ✅ Server Members Intent
6. Go to **OAuth2 → URL Generator**
   - Scopes: `bot`, `applications.commands`
   - Bot permissions: `Send Messages`, `Read Message History`,
     `Connect`, `Speak`, `Use Voice Activity`, `Add Reactions`
7. Copy the generated URL, open it, and invite Luna to your server

### Get Your Discord Server ID
1. In Discord, go to Settings → Advanced → enable **Developer Mode**
2. Right-click your server name → **Copy Server ID**
3. Paste into `DISCORD_GUILD_ID=` in your `.env`

---

## Step 7 — Run the Health Check

Before launching the bot, verify all services are reachable:

```bash
python main.py --check
```

Expected output:
```
✅  Ollama
✅  Gemini API Key
✅  Discord Token
✅  Repo Dir
✅  SQLite DB
✅  ffmpeg
✅  PyNaCl
✅  edge-tts

✅ All systems go!
```

Fix any ✗ items before proceeding.

---

## Step 8 — Launch Luna

```bash
python main.py
```

First boot does three things automatically:
1. Scans all `.py` files in your `REPO_DIR`
2. Uploads them to Gemini's context cache (one-time, ~10-30 seconds)
3. Logs in to Discord

You'll see:
```
[ContextPacker] Packing 20 files...
[ContextPacker] Packed: 40,312 lines, ~52.4k tokens
[ContextPacker] Cache created: cachedContents/abc123 (TTL=3600s)
[Router] Ollama: ✓  |  Gemini: ✓  |  MCP tools: 0
[Luna] Logged in as Luna#1234 (id=...)
```

---

## Step 9 — Talk to Luna

### From Discord (any device including iPhone)

**Wake word — natural language:**
```
Hej Luna fix the bug in auth.py
Hey Luna review my latest changes
Luna what's Bitcoin at right now
Luna show me config.py
```

**Prefix commands:**
```
!analyze why is auth.py slow
!fix add type hints to utils.py
!review db_handler.py check for race conditions
!run python print("hello from sandbox")
!read config.py
!ls
!brain
!devlog
!tools
!status
```

**Feedback (reacts to Luna's messages):**
```
💜 = loved it      👍 = liked it
😂 = funny         🤖 = too robotic
👎 = disliked      💤 = too long/boring
```

### From Terminal (no Discord needed)

```bash
python main.py --cli
```

Type normally. Prefix with `code:` to force Gemini, `chat:` for Ollama:
```
You: Hej Luna what's the weather in Tokyo
You: code: fix the type error in db_handler.py
You: chat: how are you doing today
```

---

## Step 10 — Enable MCP Servers (Optional)

MCP gives Luna plug-in skills. On first run, `mcp_servers.json` is created automatically. Edit it to enable servers:

```json
[
  {
    "name": "github",
    "enabled": true,
    "transport": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {"GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"}
  }
]
```

Add `GITHUB_TOKEN=your_pat_here` to your `.env`, restart Luna. She now has full GitHub access — create PRs, read issues, check commits — without any custom code.

**Available MCP servers:**
```bash
# GitHub
npx @modelcontextprotocol/server-github

# Spotify
npx @modelcontextprotocol/server-spotify

# Brave web search
npx @modelcontextprotocol/server-brave-search

# Slack
npx @modelcontextprotocol/server-slack

# SQLite (Python)
pip install mcp-server-sqlite
```

---

## Step 11 — Docker Sandbox (Recommended for Production)

The default subprocess sandbox is fine for development. For better isolation:

```bash
# Build the sandbox image (one time)
python -c "from sandbox_executor import generate_dockerfile; generate_dockerfile()"
docker build -f Dockerfile.luna-sandbox -t luna-sandbox:latest .

# Switch to Docker backend
# In .env:
SANDBOX_BACKEND=docker
```

Luna's code now runs in a container with: no network, 256 MB RAM cap, read-only filesystem, non-root user, killed on timeout.

---

## Step 12 — Live Debug Dashboard

The dashboard shows Luna's full thought chain in real time:
Intent → Memory injection → Personality blocks → Generation → Voice params → Feedback.

```bash
# Install dependencies
npm install react react-dom

# Copy dashboard.jsx to your React app's src/
# Or run directly with Vite:
npx create-vite luna-dashboard --template react
cp dashboard.jsx luna-dashboard/src/App.jsx
cd luna-dashboard && npm install && npm run dev
```

Open http://localhost:5173 — it auto-connects to `ws://localhost:8765` where Luna's bot broadcasts live events.

---

## File Map

```
luna_core/
│
├── main.py              Entry point. --cli, --check, or Discord bot mode
├── config.py            All settings. Change things here, not in code
├── .env.template        Copy to .env and fill in
│
├── LUNA.md              Luna's brain dump — read at start of every task
├── DEVLOG.md            Luna's changelog — auto-updated after every action
│
├── discord_bot.py       Discord bot with wake word listener
├── engineer_cog.py      !run !write !fix !brain !devlog !tools commands
├── feedback_loop.py     Reaction / voice / command feedback → memory
│
├── llm_router.py        Routes: chat→Ollama, code→Gemini, live→Grounding
├── personality.py       System prompt builder — Luna's full character
├── memory_engine.py     SQLite memory: humor, preferences, code style
├── context_packer.py    Repo scanner + Gemini context cache manager
│
├── sandbox_executor.py  run_code() — subprocess / Docker / E2B backends
├── file_manager.py      read/write/list files with path guardrails
├── self_correction.py   Propose → Write → Test → Reflect → Fix loop
├── luna_brain.py        LUNA.md manager — persistent technical memory
├── dev_log.py           update_dev_log() — changelog + blame queries
│
├── mcp_client.py        MCP toolbox — plug in GitHub, Spotify, etc.
├── mcp_servers.json     Auto-created — edit to enable MCP servers
│
├── voice_handler.py     TTS synthesis + Discord VC streaming
├── luna_logger.py       Decision chain logger + WebSocket broadcaster
├── luna_decisions.db    Auto-created SQLite — decision chain history
├── luna_memory.db       Auto-created SQLite — Luna's memories
├── luna_dev.log         Auto-created JSONL — append-only dev log
├── luna_devlog.db       Auto-created SQLite — queryable dev log index
│
└── dashboard.jsx        React dashboard — live decision chain monitor
```

---

## Troubleshooting

**Luna doesn't respond to "Hej Luna"**
- Check that Message Content Intent is enabled in the Discord developer portal
- Make sure the bot has Read Message History permission in that channel

**Voice not working**
- Run `ffmpeg -version` — must be installed and in PATH
- Run `python -c "import nacl; print('ok')"` — PyNaCl must be installed
- Luna must be in a voice channel before she can speak: `!join`

**Gemini cache errors**
- Free tier limits: cache TTL max 1 hour, min 32k tokens to cache
- If your repo is under 32k tokens, caching won't activate (Luna still works, just re-sends context each time)
- Check your API quota at https://aistudio.google.com

**Ollama not responding**
- Make sure `ollama serve` is running in a separate terminal
- Check with: `curl http://localhost:11434/api/tags`

**"MCP SDK not installed"**
- `pip install mcp` — the official Anthropic SDK
- If no servers are enabled in `mcp_servers.json`, this warning is harmless

**Sandbox execution fails**
- The subprocess backend needs Python 3 in PATH
- For JavaScript: `node --version` must work
- For Docker backend: `docker ps` must work without sudo

---

## Cost Reference

| Feature | Cost |
|---|---|
| Ollama (chat) | Free — runs on your machine |
| Gemini 1.5 Pro input (cached) | ~$0.35 / 1M tokens |
| Gemini 1.5 Pro input (non-cached) | ~$3.50 / 1M tokens |
| Gemini context cache storage | ~$1.00 / 1M tokens / hour |
| Gemini output | ~$10.50 / 1M tokens |
| Gemini grounding (search) | ~$35 / 1k requests after free tier |
| edge-tts | Free |
| ElevenLabs (optional) | $5–$22/month depending on plan |

With context caching, a typical coding session (10-20 Gemini calls) costs **under $0.10**.

---

*Luna Core v1.0 — Built session by session.*

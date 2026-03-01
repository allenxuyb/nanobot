# CLAUDE.md — AI Assistant Guide for nanobot

## Project Overview

**nanobot** (`nanobot-ai` on PyPI) is an ultra-lightweight personal AI assistant framework (~4,000 lines of core agent code). It provides a multi-channel chatbot that can run on Telegram, Discord, WhatsApp, Feishu, Slack, Email, DingTalk, QQ, Matrix, and a CLI. The project prioritizes being small, readable, and research-friendly — 99% smaller than comparable frameworks.

- **Current version**: 0.1.4.post3
- **Python**: ≥ 3.11
- **License**: MIT
- **PyPI package**: `nanobot-ai`

## Repository Structure

```
nanobot/
├── nanobot/                 # Main Python package
│   ├── agent/               # Core agent logic
│   │   ├── loop.py          # AgentLoop: LLM ↔ tool execution engine
│   │   ├── context.py       # ContextBuilder: system prompt assembly
│   │   ├── memory.py        # MemoryStore: two-layer persistent memory
│   │   ├── skills.py        # SkillsLoader: skills discovery & loading
│   │   ├── subagent.py      # SubagentManager: background task execution
│   │   └── tools/           # Built-in tool implementations
│   │       ├── base.py      # Tool base class
│   │       ├── registry.py  # ToolRegistry: dynamic tool management
│   │       ├── filesystem.py # read_file, write_file, edit_file, list_dir
│   │       ├── shell.py     # exec tool (shell command execution)
│   │       ├── web.py       # web_search, web_fetch
│   │       ├── message.py   # message tool (send to channel)
│   │       ├── spawn.py     # spawn tool (launch subagents)
│   │       ├── cron.py      # cron tool (schedule tasks)
│   │       └── mcp.py       # MCP tool integration
│   ├── channels/            # Chat platform integrations
│   │   ├── base.py          # Channel base class
│   │   ├── manager.py       # ChannelManager
│   │   ├── telegram.py      # Telegram bot
│   │   ├── discord.py       # Discord bot (WebSocket gateway)
│   │   ├── whatsapp.py      # WhatsApp (via Node.js bridge)
│   │   ├── feishu.py        # Feishu/Lark (WebSocket)
│   │   ├── dingtalk.py      # DingTalk (Stream mode)
│   │   ├── slack.py         # Slack (Socket mode)
│   │   ├── email.py         # Email (IMAP/SMTP)
│   │   ├── qq.py            # QQ (botpy SDK)
│   │   ├── mochat.py        # Mochat/Claw IM (Socket.IO)
│   │   └── matrix.py        # Matrix/Element (nio, optional E2EE)
│   ├── providers/           # LLM provider integrations
│   │   ├── registry.py      # ProviderSpec registry (single source of truth)
│   │   ├── base.py          # LLMProvider base class
│   │   ├── litellm_provider.py  # LiteLLM-based provider
│   │   ├── custom_provider.py   # Direct OpenAI-compatible provider
│   │   ├── openai_codex_provider.py  # OAuth-based Codex provider
│   │   └── transcription.py     # Voice transcription (Groq/Whisper)
│   ├── config/              # Configuration system
│   │   ├── schema.py        # Pydantic config models (Config, ProvidersConfig, etc.)
│   │   └── loader.py        # Config load/save helpers
│   ├── session/             # Conversation session management
│   │   └── manager.py       # SessionManager, Session
│   ├── bus/                 # Message routing
│   │   ├── events.py        # InboundMessage, OutboundMessage
│   │   └── queue.py         # MessageBus
│   ├── cron/                # Scheduled task service
│   │   ├── service.py       # CronService
│   │   └── types.py         # CronJob type
│   ├── heartbeat/           # Proactive periodic wake-up
│   │   └── service.py       # HeartbeatService (checks HEARTBEAT.md every 30 min)
│   ├── skills/              # Built-in skills (Markdown instructions)
│   │   ├── README.md        # Skill format documentation
│   │   ├── github/          # GitHub CLI skill
│   │   ├── weather/         # Weather skill
│   │   ├── summarize/       # URL/file/YouTube summarization
│   │   ├── tmux/            # tmux session control
│   │   ├── clawhub/         # ClawHub skill registry
│   │   ├── skill-creator/   # Create new skills
│   │   ├── memory/          # Memory management skill
│   │   └── cron/            # Cron scheduling skill
│   ├── templates/           # Workspace bootstrap templates
│   │   ├── AGENTS.md        # Default agent instructions
│   │   ├── SOUL.md          # Agent personality
│   │   ├── USER.md          # User preferences
│   │   ├── TOOLS.md         # Tool usage instructions
│   │   ├── HEARTBEAT.md     # Heartbeat task template
│   │   └── memory/          # Memory subdirectory template
│   ├── utils/               # Shared utilities
│   │   └── helpers.py       # Common helper functions
│   ├── cli/
│   │   └── commands.py      # Typer CLI app (all CLI commands)
│   ├── __init__.py          # Package version, logo
│   └── __main__.py          # Entry point
├── bridge/                  # Node.js WhatsApp bridge (TypeScript)
│   ├── src/                 # Bridge source
│   ├── package.json
│   └── tsconfig.json
├── tests/                   # Test suite
├── pyproject.toml           # Build config, dependencies, tool settings
├── Dockerfile               # Container build
├── docker-compose.yml       # Docker Compose config
├── core_agent_lines.sh      # Script to count core agent lines
├── README.md
├── SECURITY.md
└── COMMUNICATION.md
```

## Development Setup

### Install from source (recommended for development)

```bash
git clone https://github.com/HKUDS/nanobot.git
cd nanobot
pip install -e .

# Install dev dependencies
pip install -e ".[dev]"

# For Matrix support (optional)
pip install -e ".[matrix]"
```

### Build system

Uses **hatchling** as the build backend (configured in `pyproject.toml`). The wheel includes:
- All Python files under `nanobot/`
- Template Markdown files (`nanobot/templates/**/*.md`)
- Skill files (`nanobot/skills/**/*.md`, `nanobot/skills/**/*.sh`)
- The Node.js `bridge/` directory (force-included as `nanobot/bridge`)

## Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_commands.py

# Run with verbose output
pytest -v

# Run async tests (asyncio_mode=auto is set in pyproject.toml)
pytest tests/test_heartbeat_service.py
```

Tests use `pytest-asyncio` with `asyncio_mode = "auto"` — async test functions work without additional decorators. The `testpaths = ["tests"]` is configured in `pyproject.toml`.

## Linting and Formatting

Uses **ruff** for linting and formatting:

```bash
# Lint
ruff check nanobot/

# Format
ruff format nanobot/

# Fix auto-fixable issues
ruff check --fix nanobot/
```

Ruff config (in `pyproject.toml`):
- Line length: 100
- Target version: py311
- Enabled rules: E (pycodestyle), F (pyflakes), I (isort), N (naming), W (warnings)
- Ignored: E501 (line too long — covered by line-length setting)

## CLI Commands

```bash
nanobot onboard              # Initialize config & workspace at ~/.nanobot/
nanobot agent                # Interactive chat
nanobot agent -m "..."       # Single-shot chat
nanobot agent --no-markdown  # Plain-text output
nanobot agent --logs         # Show runtime logs during chat
nanobot gateway              # Start the multi-channel gateway
nanobot status               # Show provider/channel status
nanobot provider login openai-codex     # OAuth login
nanobot channels login       # Link WhatsApp (QR scan)
nanobot channels status      # Channel status
nanobot cron add --name "x" --message "..." --cron "0 9 * * *"
nanobot cron list
nanobot cron remove <job_id>
```

Interactive mode exits: `exit`, `quit`, `/exit`, `/quit`, `:q`, `Ctrl+D`.

## Architecture & Key Concepts

### Agent Loop (`nanobot/agent/loop.py`)

The `AgentLoop` is the core processing engine. It:
1. Receives `InboundMessage` from the `MessageBus`
2. Loads session history via `SessionManager`
3. Builds system prompt via `ContextBuilder`
4. Calls the LLM via `LLMProvider`
5. Executes tool calls via `ToolRegistry`
6. Sends `OutboundMessage` responses back to the bus
7. Persists memory via `MemoryStore`

Key parameters:
- `max_iterations`: 40 (safety limit on tool call loops)
- `temperature`: 0.1 (default for precision)
- `max_tokens`: 4096
- `memory_window`: 100 (session history messages kept in context)

### Provider System (`nanobot/providers/`)

**Single source of truth**: `nanobot/providers/registry.py` defines all providers as `ProviderSpec` dataclass entries in the `PROVIDERS` tuple.

To **add a new provider** (2 steps only):
1. Add a `ProviderSpec` entry to `PROVIDERS` in `nanobot/providers/registry.py`
2. Add a matching field to `ProvidersConfig` in `nanobot/config/schema.py`

Provider types:
- **Standard** (`is_gateway=False`): matched by model-name keywords (anthropic, openai, deepseek, gemini, groq, zhipu, dashscope, moonshot, minimax)
- **Gateway** (`is_gateway=True`): routes any model, detected by API key prefix or base URL (openrouter, aihubmix, siliconflow, volcengine)
- **Local** (`is_local=True`): matched by config key name (vllm)
- **Direct** (`is_direct=True`): bypasses LiteLLM entirely (custom)
- **OAuth** (`is_oauth=True`): no API key, uses OAuth flow (openai_codex, github_copilot)

Most providers route through **LiteLLM**. The `litellm_prefix` field auto-prefixes model names (e.g., `deepseek-chat` → `deepseek/deepseek-chat`).

### Configuration (`nanobot/config/schema.py`)

All config is Pydantic v2 models that accept **both camelCase and snake_case** keys (via `alias_generator=to_camel`). Config is stored at `~/.nanobot/config.json`.

Top-level config structure:
```python
Config
├── providers: ProvidersConfig     # API keys per provider
├── agents: AgentsConfig           # Model selection, parameters
│   └── defaults: AgentConfig     # model, provider, temperature, max_tokens, etc.
├── channels: ChannelsConfig       # Channel-specific configs
│   ├── telegram: TelegramConfig
│   ├── discord: DiscordConfig
│   ├── whatsapp: WhatsAppConfig
│   ├── feishu: FeishuConfig
│   ├── dingtalk: DingTalkConfig
│   ├── slack: SlackConfig
│   ├── email: EmailConfig
│   ├── qq: QQConfig
│   ├── mochat: MochatConfig
│   └── matrix: MatrixConfig
└── tools: ToolsConfig             # Tool settings
    ├── restrictToWorkspace: bool  # Sandbox to workspace dir
    ├── exec: ExecToolConfig       # Shell tool config
    └── mcpServers: dict           # MCP server definitions
```

### Memory System (`nanobot/agent/memory.py`)

Two-layer memory stored in `~/.nanobot/workspace/memory/`:
- **`MEMORY.md`**: Long-term facts (structured markdown, always in system prompt)
- **`HISTORY.md`**: Timestamped conversation log (grep-searchable)

Memory is consolidated by the LLM using a `save_memory` tool call that updates both layers.

### Context Builder (`nanobot/agent/context.py`)

Builds the system prompt by assembling:
1. Core identity (workspace path, OS, Python version)
2. Bootstrap files from workspace: `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md`, `IDENTITY.md`
3. Memory content from `MemoryStore`
4. "Always-active" skills (loaded at startup)
5. Skills summary (available skills listed for on-demand loading)

### Skills System (`nanobot/skills/`)

Skills are directories containing a `SKILL.md` file with YAML frontmatter and Markdown instructions. Skills extend the agent's capabilities without changing core code. Built-in skills:

| Skill | Directory | Purpose |
|-------|-----------|---------|
| GitHub | `github/` | Interact with GitHub via `gh` CLI |
| Weather | `weather/` | Weather via wttr.in and Open-Meteo |
| Summarize | `summarize/` | Summarize URLs, files, YouTube |
| tmux | `tmux/` | Remote-control tmux sessions |
| ClawHub | `clawhub/` | Search/install public skills |
| Skill Creator | `skill-creator/` | Create new skills |
| Memory | `memory/` | Memory management |
| Cron | `cron/` | Schedule task management |

Custom skills can be installed in `~/.nanobot/workspace/skills/`.

### Message Bus (`nanobot/bus/`)

`MessageBus` is an async queue-based message router:
- Channels publish `InboundMessage` events
- `AgentLoop` subscribes and processes them
- Responses are sent as `OutboundMessage` events back to the originating channel

### Session Management (`nanobot/session/manager.py`)

`SessionManager` tracks conversation history per `session_key` (e.g., `telegram:user123`). Each `Session` holds the message list and metadata (last active channel, etc.).

### MCP Support (`nanobot/agent/tools/mcp.py`)

Integrates Model Context Protocol (MCP) servers. Supports:
- **Stdio transport**: `command` + `args` (local process via `npx`/`uvx`)
- **HTTP transport**: `url` + optional `headers` (remote endpoints)

MCP tools are auto-discovered on startup and registered in `ToolRegistry`.

### Built-in Tools

| Tool | Class | Description |
|------|-------|-------------|
| `read_file` | `ReadFileTool` | Read workspace files |
| `write_file` | `WriteFileTool` | Write workspace files |
| `edit_file` | `EditFileTool` | Patch/replace file content |
| `list_dir` | `ListDirTool` | List directory contents |
| `exec` | `ExecTool` | Run shell commands |
| `web_search` | `WebSearchTool` | Brave Search API |
| `web_fetch` | `WebFetchTool` | Fetch and parse URLs |
| `message` | `MessageTool` | Send messages to channels |
| `spawn` | `SpawnTool` | Launch background subagents |
| `cron` | `CronTool` | Schedule tasks |

### Heartbeat (`nanobot/heartbeat/service.py`)

`HeartbeatService` wakes up every 30 minutes, reads `~/.nanobot/workspace/HEARTBEAT.md`, and executes periodic tasks listed there. Results are delivered to the most recently active chat channel.

### Subagents (`nanobot/agent/subagent.py`)

`SubagentManager` handles background task execution via the `spawn` tool. Subagents run with a subset of tools (no `message` or `spawn` — to avoid recursion) and report results back via the message bus.

## Code Conventions

### Python style
- Python 3.11+ features used (`X | Y` union types, `from __future__ import annotations`)
- Pydantic v2 for all data models
- `loguru` for logging (not standard `logging`)
- `asyncio` for all async I/O
- Type hints everywhere; use `TYPE_CHECKING` guard for circular imports
- `from __future__ import annotations` at the top of files with forward references

### Async patterns
- All channel integrations, tools, and the agent loop are async
- `asyncio.Task` used for background subagents and heartbeat
- Channels use WebSocket connections (websockets/websocket-client libraries)

### Config / schema conventions
- All config classes extend `Base` (which extends Pydantic `BaseModel`)
- `Base` uses `alias_generator=to_camel` — configs accept both `camelCase` (JSON) and `snake_case` (Python)
- Default values set on fields — no required fields in channel configs

### Tool conventions
- Each tool extends `Tool` base class from `nanobot/agent/tools/base.py`
- Tools implement `to_schema()` (OpenAI function format) and `execute(**params)`
- `ToolRegistry.execute()` wraps errors with a hint: `[Analyze the error above and try a different approach.]`
- Tool results exceeding `_TOOL_RESULT_MAX_CHARS` (500 chars) are truncated in session history

### Adding a new channel
1. Create `nanobot/channels/<name>.py` extending `base.Channel`
2. Add a config class to `nanobot/config/schema.py`
3. Add the field to `ChannelsConfig` in `schema.py`
4. Register in `nanobot/channels/manager.py`

### Adding a new tool
1. Create a class extending `Tool` in `nanobot/agent/tools/`
2. Implement `name`, `to_schema()`, and `execute()`
3. Register in `AgentLoop.__init__()` in `nanobot/agent/loop.py`

### Adding a new provider
1. Add `ProviderSpec` to `PROVIDERS` in `nanobot/providers/registry.py`
2. Add field to `ProvidersConfig` in `nanobot/config/schema.py`

## Testing Conventions

- Tests live in `tests/`
- `pytest` with `pytest-asyncio` (`asyncio_mode = "auto"`)
- Use `unittest.mock.patch` for mocking config paths (see `test_commands.py` for the pattern)
- Test file naming: `test_<module_name>.py`
- Fixtures for path isolation: mock `get_config_path`, `save_config`, `load_config`, `get_workspace_path`

## Docker

```bash
# Build
docker build -t nanobot .

# First-time setup
docker run -v ~/.nanobot:/root/.nanobot --rm nanobot onboard

# Run gateway
docker run -v ~/.nanobot:/root/.nanobot -p 18790:18790 nanobot gateway

# Docker Compose
docker compose run --rm nanobot-cli onboard
docker compose up -d nanobot-gateway
docker compose logs -f nanobot-gateway
```

The `-v ~/.nanobot:/root/.nanobot` mount persists config and workspace across container restarts.

## Key Files for Common Tasks

| Task | File(s) |
|------|---------|
| Understand agent core | `nanobot/agent/loop.py` |
| Modify system prompt | `nanobot/agent/context.py` |
| Add/modify a tool | `nanobot/agent/tools/` + register in `loop.py` |
| Add a new LLM provider | `nanobot/providers/registry.py` + `nanobot/config/schema.py` |
| Add a new chat channel | `nanobot/channels/` + `nanobot/config/schema.py` + `nanobot/channels/manager.py` |
| Modify config schema | `nanobot/config/schema.py` |
| Add/modify a skill | `nanobot/skills/<skill>/SKILL.md` |
| Modify CLI commands | `nanobot/cli/commands.py` |
| Understand memory | `nanobot/agent/memory.py` |
| Modify workspace templates | `nanobot/templates/` |

## Workspace Layout (Runtime)

At runtime, nanobot uses `~/.nanobot/` as its home:

```
~/.nanobot/
├── config.json          # Main configuration file
└── workspace/           # Agent's working directory
    ├── AGENTS.md        # Agent instructions (editable by user)
    ├── SOUL.md          # Agent personality
    ├── USER.md          # User preferences
    ├── TOOLS.md         # Tool usage instructions
    ├── HEARTBEAT.md     # Periodic task list (checked every 30 min)
    ├── memory/
    │   ├── MEMORY.md    # Long-term facts
    │   └── HISTORY.md   # Session history log
    └── skills/          # User-installed custom skills
```

## Security Notes

- `tools.restrictToWorkspace: true` sandboxes all file/shell tools to the workspace directory
- `channels.*.allowFrom` whitelists user IDs per channel (empty = allow all)
- `email.consentGranted` is a required explicit flag before mailbox access
- Keep `claw_token` (Mochat) private — treat like an API key
- For production, always set `restrictToWorkspace: true`

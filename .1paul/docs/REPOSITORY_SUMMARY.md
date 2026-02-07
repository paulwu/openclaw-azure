# Repository Analysis: OpenClaw

## Overview

**OpenClaw** is a personal AI assistant platform designed to run on your own devices. It acts as a unified gateway that connects to the messaging channels you already use—WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, Matrix, and more—transforming them into interfaces for AI-powered assistance.

The project solves a fundamental problem: fragmented communication channels and the desire for a **local-first, privacy-conscious AI assistant** that doesn't require cloud dependency for the control plane.

**Key Value Proposition:** A single control plane (Gateway) manages sessions, routing, tools, and events across all connected channels, with the product being the assistant experience itself rather than the infrastructure.

---

## Architecture

OpenClaw follows a **modular monorepo architecture** organized into distinct layers:

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Applications                      │
│  (macOS App, iOS App, Android App, Control UI, CLI)         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Gateway (Core)                          │
│  - WebSocket control plane                                   │
│  - Session management                                        │
│  - Channel routing                                           │
│  - Tool execution                                            │
│  - Cron scheduling                                           │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Channels   │     │    Agents    │     │  Extensions  │
│  (Built-in)  │     │ (Pi Runtime) │     │  (Plugins)   │
│              │     │              │     │              │
│ - WhatsApp   │     │ - Embedded   │     │ - MS Teams   │
│ - Telegram   │     │   Runner     │     │ - Matrix     │
│ - Discord    │     │ - System     │     │ - BlueBubbles│
│ - Slack      │     │   Prompt     │     │ - Zalo       │
│ - Signal     │     │ - Tools      │     │ - Voice Call │
│ - iMessage   │     │              │     │ - Memory     │
└──────────────┘     └──────────────┘     └──────────────┘
```

### Core Directories

| Directory | Purpose |
|-----------|---------|
| `src/` | Core source code (CLI, gateway, channels, agents, tools) |
| `apps/` | Native applications (macOS, iOS, Android) |
| `extensions/` | Plugin channels and integrations (30 extensions) |
| `docs/` | Mintlify-powered documentation site |
| `skills/` | Bundled AI capabilities and tools |
| `scripts/` | Build, release, and utility scripts |
| `packages/` | Internal packages (clawdbot, moltbot shims) |

---

## Key Components

### Gateway (`src/gateway/`)
The central WebSocket control plane that orchestrates all system functionality:
- **Session Management**: Maintains conversation state across channels
- **Channel Routing**: Routes messages to appropriate agents based on rules
- **Tool Execution**: Provides AI agents with capabilities (browser, canvas, cron)
- **Event System**: Handles webhooks, cron jobs, and real-time events

### Agent Runtime (`src/agents/`)
The AI agent execution environment:
- **Pi Embedded Runner**: Executes AI models with tool calling support
- **System Prompt**: Constructs context-aware prompts for agents
- **Pi Tools**: Defines available tools (browser, file operations, messaging)
- **Multi-Agent Routing**: Supports isolated agents per workspace

### Channel Implementations (`src/` + `extensions/`)
**Built-in Channels:**
- WhatsApp (Baileys library)
- Telegram (grammY framework)
- Discord (discord.js)
- Slack (Bolt framework)
- Signal (signal-cli)
- iMessage (imsg integration)
- Google Chat (Chat API)
- LINE

**Extension Channels:**
- Microsoft Teams, Matrix, BlueBubbles, Zalo, Mattermost, Nostr, Twitch, Nextcloud Talk, Tlon

### CLI (`src/cli/` + `src/commands/`)
Command-line interface for:
- Onboarding (`openclaw onboard`)
- Gateway management (`openclaw gateway`)
- Agent interaction (`openclaw agent`)
- Message sending (`openclaw message send`)
- Diagnostics (`openclaw doctor`)

### Native Applications (`apps/`)
- **macOS**: Menu bar app with Canvas host, Voice Wake
- **iOS**: Companion app with node hosting
- **Android**: Companion app with node hosting
- **Shared**: OpenClawKit for cross-platform code sharing

### Media Pipeline (`src/media/`)
Handles multimedia across channels:
- Image/audio/video processing
- Transcription hooks
- Size caps and temp file lifecycle
- Link understanding

---

## Technologies Used

### Core Runtime
- **Language**: TypeScript (ESM modules)
- **Runtime**: Node.js 22+ (primary), Bun (supported)
- **Package Manager**: pnpm (with bun support)

### Frameworks & Libraries
- **Agent SDK**: pi-ai (custom RPC agent runtime)
- **Web Server**: Native Node.js HTTP/WebSocket
- **CLI Framework**: Custom with @clack/prompts
- **Testing**: Vitest with V8 coverage

### Channel SDKs
- grammY (Telegram)
- discord.js (Discord)
- @slack/bolt (Slack)
- @anthropic-ai/sdk, openai (AI providers)

### Native Apps
- **macOS/iOS**: Swift, SwiftUI, Observation framework
- **Android**: Kotlin, Jetpack Compose

### Build & CI
- **TypeScript Build**: tsdown + tsgo
- **Linting**: Oxlint, Oxfmt
- **Swift Formatting**: swiftformat
- **CI**: GitHub Actions

### Infrastructure
- Docker support with compose
- Fly.io deployment configs
- Render, Railway, Northflank guides

---

## Data Flow

```
┌──────────────┐     ┌─────────────┐     ┌──────────────┐
│   User on    │     │   Gateway   │     │   AI Model   │
│   Channel    │────>│   (WS/HTTP) │────>│  (Anthropic/ │
│ (WhatsApp,   │     │             │     │   OpenAI)    │
│  Telegram)   │<────│   Routing   │<────│              │
└──────────────┘     │   & Tools   │     └──────────────┘
                     └─────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Session  │ │  Tools   │ │ Memory   │
        │  Store   │ │ (Browser,│ │ (Local   │
        │          │ │  Canvas) │ │  Search) │
        └──────────┘ └──────────┘ └──────────┘
```

1. **Inbound**: Message arrives via channel SDK (e.g., WhatsApp/Baileys)
2. **Routing**: Gateway routes to appropriate agent based on sender/channel rules
3. **Session**: Session context loaded (conversation history, memory)
4. **Agent**: Pi runtime constructs prompt and calls AI model
5. **Tools**: AI may invoke tools (browser, file operations, channel actions)
6. **Response**: Reply routed back through originating channel

---

## Team and Ownership

### Primary Maintainer
- **Peter Steinberger** – 80.8% of commits (6,953/8,601)
  - Core architecture, gateway, agent runtime
  - macOS/iOS native apps
  - Release management

### Core Contributors (100+ commits)
- **Shadow** (172) – Infrastructure, CI, community management
- **Tyler Yust** (55) – Mobile platforms, BlueBubbles
- **Ayaan Zaidi** (49) – Features and integrations
- **cpojer** (45) – Build tooling, performance

### Community
- **393 unique contributors** over the project lifetime
- **429 acknowledged community contributions** (commits with "thanks")
- **401 merged pull requests**

### Ownership by Area
| Area | Primary Owner | Key Contributors |
|------|---------------|------------------|
| Gateway Core | Peter Steinberger | Shadow, Glucksberg |
| Telegram | Peter Steinberger | @obviyus, @ThanhNguyxn |
| Discord | Peter Steinberger | @thewilloftheshadow, @kentaro |
| macOS App | Peter Steinberger | @fal3, @kennyklee |
| Security | Peter Steinberger | @vignesh07, @HassanFleyah |
| Documentation | Peter Steinberger | @joshp123, @hougangdev |

---

## Repository Statistics

| Metric | Value |
|--------|-------|
| Total Commits | 8,601 |
| Contributors | 393 |
| Lines of Code | ~150,000+ (TypeScript) |
| Extensions | 30 |
| Supported Channels | 15+ |
| Documentation Pages | 100+ |
| Test Coverage Target | 70% (lines/branches/functions) |

---

## Key Files

| File | Purpose | Changes |
|------|---------|---------|
| [CHANGELOG.md](CHANGELOG.md) | Release notes and version history | 2,013 |
| [package.json](package.json) | npm package configuration | 328 |
| [src/gateway/server.ts](src/gateway/server.ts) | Core gateway implementation | 242 |
| [src/agents/pi-embedded-runner.ts](src/agents/pi-embedded-runner.ts) | AI agent execution | 185 |
| [src/config/zod-schema.ts](src/config/zod-schema.ts) | Configuration validation | 167 |
| [src/telegram/bot.ts](src/telegram/bot.ts) | Telegram channel implementation | 153 |
| [AGENTS.md](AGENTS.md) | AI agent guidelines for codebase | 125 |

# OpenClaw Project Architecture Blueprint

> **Generated:** 2026-02-07  
> **Version:** 2026.1.30  
> **Technology:** TypeScript/Node.js (ESM)  
> **Architecture Pattern:** Plugin-Based Modular Architecture with Gateway-Agent Pattern

---

## Table of Contents

1. [Architectural Overview](#1-architectural-overview)
2. [Architecture Visualization](#2-architecture-visualization)
3. [Core Architectural Components](#3-core-architectural-components)
4. [Architectural Layers and Dependencies](#4-architectural-layers-and-dependencies)
5. [Data Architecture](#5-data-architecture)
6. [Cross-Cutting Concerns](#6-cross-cutting-concerns)
7. [Service Communication Patterns](#7-service-communication-patterns)
8. [Technology-Specific Patterns](#8-technology-specific-patterns)
9. [Implementation Patterns](#9-implementation-patterns)
10. [Testing Architecture](#10-testing-architecture)
11. [Deployment Architecture](#11-deployment-architecture)
12. [Extension and Evolution Patterns](#12-extension-and-evolution-patterns)
13. [Code Examples](#13-code-examples)
14. [Architectural Decision Records](#14-architectural-decision-records)
15. [Architecture Governance](#15-architecture-governance)
16. [Blueprint for New Development](#16-blueprint-for-new-development)

---

## 1. Architectural Overview

### 1.1 System Purpose

OpenClaw is a multi-channel AI agent gateway system that provides:
- **WhatsApp Gateway CLI** using Baileys web protocol
- **Pi RPC Agent** for AI-powered conversational interactions
- **Multi-Channel Messaging** across Telegram, Discord, Slack, Signal, iMessage, and more
- **Plugin Architecture** for extensible channel and feature support
- **Native Apps** for macOS, iOS, and Android

### 1.2 Guiding Architectural Principles

| Principle | Implementation |
|-----------|----------------|
| **Modularity** | Plugin-based extensions, clear module boundaries |
| **Extensibility** | Plugin SDK, channel adapters, hook system |
| **Type Safety** | TypeScript strict mode, Zod schema validation |
| **Testability** | Colocated tests, dependency injection, mocking patterns |
| **Configuration-Driven** | YAML/JSON config with environment variable substitution |
| **Multi-Platform** | Core Node.js + Native apps (Swift/Kotlin) |

### 1.3 High-Level Architecture Pattern

The system follows a **Plugin-Based Modular Architecture** combined with a **Gateway-Agent Pattern**:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         OpenClaw System                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Native    │  │   Native    │  │   Native    │  │    Web      │ │
│  │ macOS App   │  │  iOS App    │  │ Android App │  │    UI       │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘ │
│         │                │                │                │        │
│         └────────────────┴────────────────┴────────────────┘        │
│                                   │                                  │
│                          ┌────────▼────────┐                        │
│                          │  Gateway Server │                        │
│                          │   (WebSocket)   │                        │
│                          └────────┬────────┘                        │
│         ┌─────────────────────────┼─────────────────────────┐       │
│         │                         │                         │       │
│  ┌──────▼──────┐  ┌───────────────▼───────────────┐  ┌─────▼─────┐ │
│  │   Channel   │  │        Agent Runtime          │  │  Plugins  │ │
│  │   Manager   │  │  (Pi Embedded, Subagents)     │  │  System   │ │
│  └──────┬──────┘  └───────────────┬───────────────┘  └─────┬─────┘ │
│         │                         │                         │       │
│  ┌──────▼──────────────┐  ┌───────▼───────┐  ┌─────────────▼─────┐ │
│  │ Channel Adapters    │  │  AI Providers │  │ Extension Plugins │ │
│  │ (WhatsApp, Telegram,│  │ (Anthropic,   │  │ (Memory, Auth,    │ │
│  │  Discord, Slack...) │  │  OpenAI...)   │  │  Voice, Matrix...)│ │
│  └─────────────────────┘  └───────────────┘  └───────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Architecture Visualization

### 2.1 Component Diagram (C4 Level 2)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              OpenClaw                                     │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                         CLI Layer (src/cli/)                       │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │  │
│  │  │ program  │ │ gateway  │ │ channels │ │  config  │ │ plugins  │  │  │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘  │  │
│  └───────┼────────────┼────────────┼────────────┼────────────┼────────┘  │
│          │            │            │            │            │           │
│  ┌───────▼────────────▼────────────▼────────────▼────────────▼────────┐  │
│  │                      Commands Layer (src/commands/)                │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │  │
│  │  │  agent  │ │ doctor  │ │ onboard │ │ status  │ │ sessions│       │  │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘       │  │
│  └───────┼───────────┼───────────┼───────────┼───────────┼────────────┘  │
│          │           │           │           │           │              │
│  ┌───────▼───────────▼───────────▼───────────▼───────────▼────────────┐  │
│  │                        Core Services                               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │  │
│  │  │   Gateway    │  │    Agents    │  │   Channels   │              │  │
│  │  │ (src/gateway)│  │ (src/agents) │  │(src/channels)│              │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │  │
│  └─────────┼─────────────────┼─────────────────┼──────────────────────┘  │
│            │                 │                 │                        │
│  ┌─────────▼─────────────────▼─────────────────▼──────────────────────┐  │
│  │                      Infrastructure Layer                          │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ │  │
│  │  │ config │ │  infra │ │  hooks │ │ logging│ │ routing│ │ plugins│ │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                     Extensions (extensions/*)                       │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ │  │
│  │  │telegram│ │ slack  │ │ matrix │ │msteams │ │ memory │ │voice-  │ │  │
│  │  │        │ │        │ │        │ │        │ │-lancedb│ │  call  │ │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow Diagram

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    User      │     │   Channel    │     │   Gateway    │
│  (WhatsApp,  │────▶│   Adapter    │────▶│   Server     │
│  Telegram..) │     │              │     │              │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                 │
                     ┌───────────────────────────┘
                     │
              ┌──────▼──────┐     ┌──────────────┐
              │   Routing   │────▶│  Session     │
              │   Engine    │     │  Manager     │
              └──────┬──────┘     └──────┬───────┘
                     │                   │
              ┌──────▼──────┐     ┌──────▼───────┐
              │  Auto-Reply │     │   Agent      │
              │   Handler   │────▶│   Runtime    │
              └──────┬──────┘     └──────┬───────┘
                     │                   │
              ┌──────▼──────┐     ┌──────▼───────┐
              │    Hooks    │     │ AI Provider  │
              │   System    │     │  (Anthropic) │
              └──────┬──────┘     └──────────────┘
                     │
              ┌──────▼──────┐
              │  Response   │
              │  Delivery   │
              └─────────────┘
```

---

## 3. Core Architectural Components

### 3.1 Gateway Server (`src/gateway/`)

**Purpose:** Central hub for all client connections and message routing.

**Responsibilities:**
- WebSocket server management
- Client authentication and authorization
- Message routing between channels and agents
- Plugin service lifecycle management
- Health monitoring and diagnostics

**Key Abstractions:**
- `GatewayServer` - Main server implementation
- `GatewayRequestHandler` - HTTP/WS request handling
- `ChannelManager` - Channel lifecycle management
- `NodeRegistry` - Connected node tracking
- `ExecApprovalManager` - Command execution approvals

**Internal Structure:**
```
src/gateway/
├── server.ts              # Main server entry
├── server.impl.ts         # Server implementation
├── server-methods/        # RPC method handlers
├── server-channels.ts     # Channel management
├── server-chat.ts         # Chat event handling
├── server-plugins.ts      # Plugin loading
├── protocol/              # Protocol definitions
└── client.ts              # Gateway client
```

### 3.2 Agent System (`src/agents/`)

**Purpose:** AI agent execution and management.

**Responsibilities:**
- Agent lifecycle management
- AI provider integration (Anthropic, OpenAI, Google, etc.)
- Tool execution and sandboxing
- Session management and persistence
- System prompt generation

**Key Abstractions:**
- `PiEmbeddedRunner` - Main agent execution engine
- `AgentScope` - Agent context and configuration
- `AuthProfiles` - Provider credential management
- `ToolPolicy` - Tool access control
- `SandboxContext` - Secure execution environment

**Internal Structure:**
```
src/agents/
├── pi-embedded-runner.ts     # Main agent runner
├── pi-embedded-subscribe.ts  # Stream subscription
├── pi-tools.ts               # Tool definitions
├── sandbox/                  # Sandbox execution
├── auth-profiles/            # Auth management
├── skills/                   # Skill system
└── cli-runner/               # CLI execution
```

### 3.3 Channels System (`src/channels/`)

**Purpose:** Multi-channel message transport abstraction.

**Responsibilities:**
- Channel adapter registration
- Message normalization across channels
- Allowlist and permission management
- Typing indicators and reactions
- Session key resolution

**Key Abstractions:**
- `ChannelPlugin` - Channel adapter interface
- `ChannelDock` - Channel runtime context
- `ChannelMessagingAdapter` - Message send/receive
- `ChannelPairingAdapter` - Account linking

**Built-in Channels:**
- WhatsApp (Baileys web)
- Telegram
- Discord
- Slack
- Signal
- iMessage

### 3.4 Plugin System (`src/plugins/`)

**Purpose:** Extensibility framework for all system components.

**Responsibilities:**
- Plugin discovery and loading
- Plugin lifecycle management
- Service registration and injection
- Configuration schema validation
- HTTP route registration

**Key Abstractions:**
- `OpenClawPluginApi` - Plugin interface
- `PluginRuntime` - Plugin execution context
- `PluginLoader` - Dynamic loading
- `PluginRegistry` - Plugin registration

**Plugin Types:**
- Channel plugins (messaging platforms)
- Memory plugins (storage backends)
- Auth plugins (authentication providers)
- Tool plugins (agent capabilities)

### 3.5 CLI Framework (`src/cli/`)

**Purpose:** Command-line interface and user interaction.

**Responsibilities:**
- Command parsing and dispatch
- Interactive prompts and wizards
- Progress reporting
- Terminal output formatting

**Key Abstractions:**
- `buildProgram()` - Commander.js program builder
- `createDefaultDeps()` - Dependency injection
- `WizardPrompter` - Interactive prompts

### 3.6 Configuration System (`src/config/`)

**Purpose:** Application configuration management.

**Responsibilities:**
- YAML/JSON configuration loading
- Environment variable substitution
- Schema validation (Zod)
- Migration from legacy formats
- Runtime configuration overrides

**Key Abstractions:**
- `OpenClawConfig` - Main configuration type
- `loadConfig()` - Configuration loader
- Configuration schemas in `zod-schema.*.ts`

---

## 4. Architectural Layers and Dependencies

### 4.1 Layer Structure

```
┌────────────────────────────────────────────────────┐
│              Presentation Layer                     │
│  (CLI, Native Apps, Web UI)                        │
├────────────────────────────────────────────────────┤
│              Application Layer                      │
│  (Commands, Workflows, Orchestration)              │
├────────────────────────────────────────────────────┤
│              Domain Layer                           │
│  (Agents, Channels, Sessions, Routing)             │
├────────────────────────────────────────────────────┤
│              Infrastructure Layer                   │
│  (Config, Infra, Logging, Persistence)             │
└────────────────────────────────────────────────────┘
```

### 4.2 Dependency Rules

1. **Upper layers depend on lower layers** - Never the reverse
2. **Domain layer is dependency-free** - No framework dependencies
3. **Infrastructure provides interfaces** - Domain consumes abstractions
4. **Plugins follow adapter pattern** - Implement standard interfaces

### 4.3 Dependency Injection Pattern

```typescript
// src/cli/deps.ts - Central dependency creation
export function createDefaultDeps(): CliDeps {
  return {
    sendMessageWhatsApp,
    sendMessageTelegram,
    sendMessageDiscord,
    sendMessageSlack,
    sendMessageSignal,
    sendMessageIMessage,
  };
}

export function createOutboundSendDeps(deps: CliDeps): OutboundSendDeps {
  return {
    sendWhatsApp: deps.sendMessageWhatsApp,
    sendTelegram: deps.sendMessageTelegram,
    // ...
  };
}
```

---

## 5. Data Architecture

### 5.1 Domain Model Structure

```
┌─────────────────────────────────────────────────────────────┐
│                      Domain Entities                         │
├──────────────┬──────────────┬──────────────┬────────────────┤
│    Agent     │   Session    │   Channel    │    Message     │
│  - agentId   │  - sessionKey│  - channelId │  - messageId   │
│  - config    │  - history   │  - accountId │  - content     │
│  - identity  │  - context   │  - state     │  - sender      │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

### 5.2 Data Storage Patterns

| Data Type | Storage Location | Format |
|-----------|------------------|--------|
| Configuration | `~/.openclaw/config.yml` | YAML |
| Sessions | `~/.openclaw/agents/<id>/sessions/` | JSONL |
| Credentials | `~/.openclaw/credentials/` | JSON (encrypted) |
| Auth Profiles | `~/.openclaw/agents/<id>/auth-profiles.json` | JSON |
| Skills | `~/.openclaw/skills/` or workspace `.openclaw/skills/` | Markdown |

### 5.3 Data Validation

All configuration and data uses **Zod** schema validation:

```typescript
// src/config/zod-schema.core.ts
export const DmPolicySchema = z.enum(["open", "allowlist"]);
export const GroupPolicySchema = z.enum(["closed", "mention", "broadcast"]);
```

---

## 6. Cross-Cutting Concerns

### 6.1 Authentication & Authorization

**Implementation:**
- Auth profiles stored per-agent in `auth-profiles.json`
- Multiple credential types: API keys, OAuth tokens, device codes
- Credential rotation and failover support
- Per-channel allowlist enforcement

**Security Boundaries:**
- Sandbox execution for agent tools
- Command gating by sender identity
- Elevated mode for privileged operations

### 6.2 Error Handling & Resilience

**Exception Handling:**
```typescript
// src/infra/errors.ts
export function formatUncaughtError(error: unknown): string

// Global handler in src/index.ts
process.on("uncaughtException", (error) => {
  console.error("[openclaw] Uncaught exception:", formatUncaughtError(error));
  process.exit(1);
});
```

**Retry Patterns:**
- `src/infra/retry.ts` - Configurable retry policy
- `src/infra/backoff.ts` - Exponential backoff
- Auth profile rotation on failures

### 6.3 Logging & Monitoring

**Logging System:**
```typescript
// src/logging/subsystem.ts
export function createSubsystemLogger(name: string): Logger

// Usage
const log = createSubsystemLogger("gateway");
log.info("Starting gateway server");
```

**Diagnostic Events:**
- `src/infra/diagnostic-events.ts` - Event emission
- `src/infra/diagnostic-flags.ts` - Feature flags
- OpenTelemetry integration via `extensions/diagnostics-otel`

### 6.4 Validation

**Input Validation:**
- Zod schemas for all configuration
- Runtime type checking with `safeParse`
- Plugin config schema validation

**Business Rule Validation:**
- Allowlist checks in `src/channels/allowlists/`
- Command gating in `src/channels/command-gating.ts`
- Tool policy enforcement in `src/agents/tool-policy.ts`

### 6.5 Configuration Management

**Configuration Sources:**
1. Default values in code
2. Configuration file (`~/.openclaw/config.yml`)
3. Environment variables
4. Runtime overrides

**Environment Variable Substitution:**
```yaml
# config.yml
anthropic:
  apiKey: ${ANTHROPIC_API_KEY}
```

---

## 7. Service Communication Patterns

### 7.1 Protocol Definitions

**Gateway Protocol:**
- WebSocket-based RPC
- JSON message format
- Defined in `src/gateway/protocol/`

**Message Types:**
```typescript
// Request/Response pattern
type GatewayRequest = {
  id: string;
  method: string;
  params?: unknown;
};

type GatewayResponse = {
  id: string;
  result?: unknown;
  error?: { code: number; message: string };
};
```

### 7.2 Communication Patterns

| Pattern | Usage | Implementation |
|---------|-------|----------------|
| Request/Response | CLI to Gateway | WebSocket RPC |
| Pub/Sub | Agent Events | Event emitters |
| Streaming | AI responses | Async iterators |
| Webhook | External integrations | HTTP handlers |

### 7.3 Native App Communication

**macOS/iOS:**
- Swift protocol definitions generated from TypeScript
- `scripts/protocol-gen-swift.ts`
- Shared via `apps/shared/OpenClawKit/`

---

## 8. Technology-Specific Patterns

### 8.1 Node.js Patterns

**ESM Modules:**
- Pure ESM (`"type": "module"` in package.json)
- `.js` extensions in imports
- Top-level await support

**Async Patterns:**
```typescript
// Async iterators for streaming
async function* streamResponse(): AsyncGenerator<string> {
  // ...
}

// Promise-based APIs
export async function loadConfig(): Promise<OpenClawConfig>
```

### 8.2 TypeScript Patterns

**Strict Type Checking:**
```json
{
  "compilerOptions": {
    "strict": true,
    "noEmit": true
  }
}
```

**Type Guards:**
```typescript
function isChannelPlugin(plugin: unknown): plugin is ChannelPlugin {
  return typeof plugin === "object" && plugin !== null && "meta" in plugin;
}
```

### 8.3 Swift Patterns (macOS/iOS)

**SwiftUI with Observation:**
- Prefer `@Observable` over `ObservableObject`
- Use `@Bindable` for two-way bindings
- Shared code in `apps/shared/OpenClawKit`

---

## 9. Implementation Patterns

### 9.1 Interface Design Patterns

**Adapter Pattern for Channels:**
```typescript
// src/channels/plugins/types.ts
export interface ChannelMessagingAdapter {
  sendMessage(ctx: ChannelOutboundContext, ...): Promise<void>;
  sendTyping?(ctx: ChannelOutboundContext): Promise<void>;
}

export interface ChannelPairingAdapter {
  loginWithQrStart(ctx: ...): Promise<ChannelLoginWithQrStartResult>;
  loginWithQrWait(ctx: ...): Promise<ChannelLoginWithQrWaitResult>;
}
```

### 9.2 Service Implementation Patterns

**Factory Pattern:**
```typescript
// Create dependencies through factory functions
export function createDefaultDeps(): CliDeps
export function createOutboundSendDeps(deps: CliDeps): OutboundSendDeps
```

**Registry Pattern:**
```typescript
// src/channels/registry.ts
export function getChatChannelMeta(channelId: ChannelId): ChannelMeta | undefined
export function listChannelPlugins(): ChannelPlugin[]
```

### 9.3 Plugin Implementation Pattern

```typescript
// Extension plugin structure (e.g., extensions/telegram/)
export default {
  meta: {
    id: "telegram",
    name: "Telegram",
    version: "1.0.0",
  },
  // Implement required adapters
  messaging: TelegramMessagingAdapter,
  pairing: TelegramPairingAdapter,
  // ...
} satisfies ChannelPlugin;
```

---

## 10. Testing Architecture

### 10.1 Testing Strategy

| Test Type | Location | Framework |
|-----------|----------|-----------|
| Unit Tests | Colocated `*.test.ts` | Vitest |
| E2E Tests | `*.e2e.test.ts` | Vitest |
| Live Tests | `*.live.test.ts` | Vitest |
| Docker Tests | `scripts/e2e/*.sh` | Bash + Vitest |

### 10.2 Test File Naming

```
src/
├── config/
│   ├── config.ts
│   ├── config.test.ts              # Unit tests
│   └── config.backup-rotation.test.ts  # Specific scenario
├── gateway/
│   ├── server.ts
│   ├── server.reload.e2e.test.ts   # E2E test
│   └── gateway-cli-backend.live.test.ts  # Live test
```

### 10.3 Coverage Thresholds

```javascript
// vitest.config.ts
coverage: {
  thresholds: {
    lines: 70,
    branches: 70,
    functions: 70,
    statements: 70,
  }
}
```

### 10.4 Test Helpers

```typescript
// src/test-helpers/ - Shared test utilities
// src/gateway/test-helpers.ts - Gateway-specific mocks
// src/agents/test-helpers/ - Agent test utilities
```

---

## 11. Deployment Architecture

### 11.1 Deployment Modes

| Mode | Description | Configuration |
|------|-------------|---------------|
| Local | Development/personal use | `gateway.mode=local` |
| VPS | Remote server deployment | `gateway.mode=vps` |
| Docker | Containerized deployment | `Dockerfile` |
| Fly.io | Cloud deployment | `fly.toml` |
| Render | Cloud deployment | `render.yaml` |

### 11.2 Build Process

```bash
# Build TypeScript to JavaScript
pnpm build

# Output structure
dist/
├── index.js           # Main entry
├── cli/               # CLI commands
├── gateway/           # Gateway server
├── agents/            # Agent runtime
└── ...
```

### 11.3 Native App Packaging

**macOS:**
```bash
scripts/package-mac-app.sh
# Produces dist/OpenClaw.app
```

**iOS/Android:**
```bash
pnpm ios:build
pnpm android:assemble
```

---

## 12. Extension and Evolution Patterns

### 12.1 Adding New Channels

1. **Create extension directory:**
   ```
   extensions/newchannel/
   ├── package.json
   ├── index.ts
   ├── send.ts
   └── receive.ts
   ```

2. **Implement channel interfaces:**
   ```typescript
   import type { ChannelPlugin } from "openclaw/plugin-sdk";
   
   export default {
     meta: { id: "newchannel", name: "New Channel" },
     messaging: { sendMessage, sendTyping },
     pairing: { loginWithQrStart, loginWithQrWait },
   } satisfies ChannelPlugin;
   ```

3. **Register in configuration schema**

4. **Update documentation in `docs/channels/`**

### 12.2 Adding New AI Providers

1. **Add auth profile type in `src/agents/auth-profiles/`**
2. **Implement provider config in `src/agents/models-config.ts`**
3. **Add CLI auth flow in `src/commands/auth-choice.apply.*.ts`**
4. **Update model catalog**

### 12.3 Adding New Tools

```typescript
// src/agents/tools/my-tool.ts
import type { AnyAgentTool } from "./common.js";

export const myTool: AnyAgentTool = {
  name: "my_tool",
  description: "Does something useful",
  parameters: { ... },
  execute: async (params) => { ... },
};
```

---

## 13. Code Examples

### 13.1 Layer Separation Example

**Interface Definition (Domain Layer):**
```typescript
// src/channels/plugins/types.ts
export interface ChannelOutboundAdapter {
  send(ctx: ChannelOutboundContext, payload: ReplyPayload): Promise<void>;
}
```

**Implementation (Infrastructure Layer):**
```typescript
// extensions/telegram/send.ts
export const sendMessage: ChannelOutboundAdapter["send"] = async (ctx, payload) => {
  const bot = await getTelegramBot(ctx.config);
  await bot.sendMessage(ctx.chatId, payload.text);
};
```

### 13.2 Dependency Injection Example

```typescript
// Command with injected dependencies
export async function runAgentCommand(deps: CliDeps, options: AgentOptions) {
  const config = await loadConfig();
  const sendDeps = createOutboundSendDeps(deps);
  
  // Use injected send functions
  await sendDeps.sendTelegram(ctx, message);
}
```

### 13.3 Plugin Registration Example

```typescript
// Plugin exports standard interface
// extensions/memory-lancedb/index.ts
export default {
  meta: {
    id: "memory-lancedb",
    kind: "memory",
    name: "LanceDB Memory",
  },
  createMemoryAdapter(ctx) {
    return new LanceDBMemoryAdapter(ctx.config);
  },
} satisfies OpenClawPluginApi;
```

---

## 14. Architectural Decision Records

### 14.1 ADR-001: Plugin-Based Extensibility

**Context:** Need to support many messaging channels without bloating core.

**Decision:** Implement channel adapters as plugins in `extensions/`.

**Consequences:**
- ✅ Core stays lean
- ✅ Channels can be added independently
- ✅ Each channel has isolated dependencies
- ⚠️ Plugin loading adds complexity

### 14.2 ADR-002: TypeScript ESM-Only

**Context:** Node.js ecosystem transitioning to ESM.

**Decision:** Pure ESM with no CommonJS fallback.

**Consequences:**
- ✅ Modern syntax, tree-shaking
- ✅ Top-level await
- ⚠️ Some legacy packages require workarounds

### 14.3 ADR-003: Gateway-Agent Architecture

**Context:** Need to support multiple clients (CLI, apps, web).

**Decision:** Central gateway server with WebSocket protocol.

**Consequences:**
- ✅ Single source of truth for state
- ✅ Multiple clients can connect
- ✅ Enables mobile/desktop apps
- ⚠️ Gateway must be running for full functionality

### 14.4 ADR-004: Zod for Runtime Validation

**Context:** TypeScript types only exist at compile time.

**Decision:** Use Zod for runtime schema validation.

**Consequences:**
- ✅ Runtime type safety
- ✅ Automatic type inference
- ✅ Detailed error messages
- ⚠️ Schema definitions can be verbose

---

## 15. Architecture Governance

### 15.1 Automated Checks

| Check | Tool | Configuration |
|-------|------|---------------|
| Linting | oxlint | `pnpm lint` |
| Formatting | oxfmt | `pnpm format` |
| Type Checking | TypeScript | `pnpm build` |
| Tests | Vitest | `pnpm test` |
| Coverage | V8 | 70% threshold |

### 15.2 Pre-Commit Hooks

```bash
# git-hooks/pre-commit
pnpm lint
pnpm format --check
pnpm build
pnpm test
```

### 15.3 File Size Guidelines

- Target: Files under ~500 LOC
- Maximum guideline: ~700 LOC (not enforced)
- Script check: `pnpm check:loc`

---

## 16. Blueprint for New Development

### 16.1 Development Workflow

1. **Setup:**
   ```bash
   pnpm install
   prek install  # Pre-commit hooks
   ```

2. **Development:**
   ```bash
   pnpm dev              # Run in dev mode
   pnpm gateway:dev      # Run gateway in dev
   pnpm test:watch       # Watch tests
   ```

3. **Before Commit:**
   ```bash
   pnpm lint && pnpm build && pnpm test
   ```

### 16.2 Component Creation Sequence

**For New Feature:**
1. Define types in `src/<domain>/types.ts`
2. Implement in `src/<domain>/<feature>.ts`
3. Add tests in `src/<domain>/<feature>.test.ts`
4. Add CLI command if needed in `src/cli/`
5. Update documentation in `docs/`

**For New Channel Plugin:**
1. Create `extensions/<channel>/`
2. Implement `ChannelPlugin` interface
3. Add config schema
4. Add documentation in `docs/channels/`
5. Update `.github/labeler.yml`

### 16.3 Common Pitfalls

| Pitfall | Avoidance |
|---------|-----------|
| Circular dependencies | Use dependency injection |
| Breaking layer boundaries | Review imports |
| Missing async handling | Use async/await consistently |
| Skipping validation | Always use Zod schemas |
| Large files | Split when clarity improves |

### 16.4 Documentation Requirements

- **Code:** Brief comments for tricky logic
- **Types:** JSDoc for exported interfaces
- **Commands:** Help text via Commander
- **Docs:** Update `docs/` for user-facing features

---

## Appendix: Key File Locations

| Purpose | Location |
|---------|----------|
| Main entry | `src/index.ts` |
| CLI program | `src/cli/program.ts` |
| Gateway server | `src/gateway/server.impl.ts` |
| Agent runner | `src/agents/pi-embedded-runner.ts` |
| Channel registry | `src/channels/registry.ts` |
| Plugin SDK | `src/plugin-sdk/index.ts` |
| Configuration | `src/config/config.ts` |
| Type definitions | `src/config/types.ts` |

---

*This blueprint reflects the OpenClaw architecture as of version 2026.1.30. Update this document when making significant architectural changes.*

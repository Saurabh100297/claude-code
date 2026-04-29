# Claude Code: The Complete Guide

**A Comprehensive Book on the Claude Code Codebase**

---

## About This Book

This book provides a comprehensive guide to the Claude Code codebase - Anthropic's official CLI tool for interacting with Claude directly from the terminal. This documentation is organized by the functional components of the codebase, providing both high-level architecture and detailed implementation insights.

**Book Information:**
- **Subject**: Claude Code Leaked Source Code
- **Version**: Based on March 31, 2026 leak
- **Language**: TypeScript (strict)
- **Runtime**: Bun
- **Scale**: ~1,900 files · 512,000+ lines of code

---

## Table of Contents

1. [Chapter 1: Introduction and Overview](#chapter-1-introduction-and-overview)
2. [Chapter 2: Core Architecture](#chapter-2-core-architecture)
3. [Chapter 3: The Tool System](#chapter-3-the-tool-system)
4. [Chapter 4: The Command System](#chapter-4-the-command-system)
5. [Chapter 5: Service Layer and Integrations](#chapter-5-service-layer-and-integrations)
6. [Chapter 6: Major Subsystems](#chapter-6-major-subsystems)
7. [Chapter 7: State Management and UI](#chapter-7-state-management-and-ui)
8. [Chapter 8: Design Patterns and Best Practices](#chapter-8-design-patterns-and-best-practices)
9. [Chapter 9: Getting Started with the Codebase](#chapter-9-getting-started-with-the-codebase)
10. [Appendix: Key Files Reference](#appendix-key-files-reference)

---

# Chapter 1: Introduction and Overview

## 1.1 What is Claude Code?

Claude Code is Anthropic's official CLI tool for interacting with Claude directly from the terminal. It enables developers to:

- Edit files and run commands
- Search codebases intelligently
- Manage git workflows
- Execute complex multi-step tasks
- Integrate with IDEs (VS Code, JetBrains)

## 1.2 How It Leaked

On March 31, 2026, security researcher Chaofan Shou (@Fried_rice) discovered that the published npm package for Claude Code included a `.map` file referencing the full, unobfuscated TypeScript source — downloadable as a zip from Anthropic's R2 storage bucket.

## 1.3 Technical Stack

| Component | Technology |
|-----------|-----------|
| **Runtime** | Bun |
| **Language** | TypeScript (strict mode) |
| **Terminal UI** | React + Ink |
| **CLI Parsing** | Commander.js (extra-typings) |
| **Schema Validation** | Zod v4 |
| **Code Search** | ripgrep (via GrepTool) |
| **Protocols** | MCP SDK, LSP |
| **API** | Anthropic SDK |
| **Telemetry** | OpenTelemetry + gRPC |
| **Feature Flags** | GrowthBook |
| **Auth** | OAuth 2.0, JWT, macOS Keychain |

## 1.4 Directory Structure

```
src/
├── main.tsx                 # Entrypoint — Commander.js CLI parser + React/Ink renderer
├── QueryEngine.ts           # Core LLM API caller (~46K lines)
├── Tool.ts                  # Tool type definitions (~29K lines)
├── commands.ts              # Command registry (~25K lines)
├── tools.ts                 # Tool registry
├── context.ts               # System/user context collection
├── cost-tracker.ts          # Token cost tracking
│
├── tools/                   # Agent tool implementations (~40)
├── commands/                # Slash command implementations (~50)
├── components/              # Ink UI components (~140)
├── services/                # External service integrations
├── hooks/                   # React hooks (incl. permission checks)
├── types/                   # TypeScript type definitions
├── utils/                   # Utility functions
├── screens/                 # Full-screen UIs (Doctor, REPL, Resume)
│
├── bridge/                  # IDE integration (VS Code, JetBrains)
├── coordinator/             # Multi-agent orchestration
├── plugins/                 # Plugin system
├── skills/                  # Skill system
├── server/                  # Server mode
├── remote/                  # Remote sessions
├── memdir/                  # Persistent memory directory
├── tasks/                   # Task management
├── state/                   # State management
│
├── voice/                   # Voice input
├── vim/                     # Vim mode
├── keybindings/             # Keybinding configuration
├── schemas/                 # Config schemas (Zod)
├── migrations/              # Config migrations
├── entrypoints/             # Initialization logic
├── query/                   # Query pipeline
├── ink/                     # Ink renderer wrapper
├── buddy/                   # Companion sprite (Easter egg 🐣)
├── native-ts/               # Native TypeScript utils
├── outputStyles/            # Output styling
└── upstreamproxy/           # Proxy configuration
```

## 1.5 Scale and Metrics

- **Total Files**: ~1,900 source files
- **Total Lines**: 512,000+ lines of TypeScript
- **Tools**: ~40 agent tools
- **Commands**: ~85 slash commands
- **Components**: ~140 React/Ink UI components
- **Services**: 20+ external integrations

---

# Chapter 2: Core Architecture

## 2.1 High-Level Pipeline

Claude Code follows a clean pipeline architecture:

```
User Input → CLI Parser → Query Engine → LLM API → Tool Execution Loop → Terminal UI
```

The entire UI layer is built with **React + Ink**, making it a fully reactive CLI application with components, hooks, state management, and all the patterns you'd expect in a React web app — just rendered to the terminal.

## 2.2 Startup Sequence

### Phase 1: Parallel Prefetch
Before heavy module imports, the entrypoint fires parallel side-effects:
- MDM settings read
- Keychain prefetch
- API preconnect
- GrowthBook initialization

### Phase 2: CLI Parsing
- Commander.js parses arguments and flags
- Environment detection (TTY, IDE bridge mode)
- Configuration loading

### Phase 3: React/Ink Initialization
- Render tree setup
- Context providers initialization
- REPL launcher activation

## 2.3 Core Components

### 2.3.1 Main Entrypoint (`src/main.tsx`)

The primary entrypoint that:
- Sets up the Commander.js CLI parser
- Initializes parallel prefetch operations
- Bootstraps the React/Ink rendering system
- Routes to appropriate mode (REPL, bridge, server, etc.)

### 2.3.2 Query Engine (`src/QueryEngine.ts`, ~46K lines)

The heart of Claude Code, responsible for:

**Streaming Responses**
- Handles real-time streaming from Anthropic API
- Progressive rendering of responses
- Chunked token processing

**Tool-Call Loops**
- Detects when LLM requests a tool
- Executes the tool with appropriate permissions
- Feeds results back to the LLM
- Continues until task completion

**Thinking Mode**
- Extended thinking with budget management
- Longer reasoning chains for complex problems
- Token budget tracking

**Retry Logic**
- Automatic retries with exponential backoff
- Handles transient API failures
- Error recovery strategies

**Token Management**
- Tracks input/output tokens per turn
- Cost calculation and reporting
- Usage quota enforcement

**Context Management**
- Conversation history maintenance
- Context window management
- Automatic summarization when needed

### 2.3.3 Tool Registry (`src/Tool.ts` + `src/tools.ts`)

Central registry of all executable tools:
- Type definitions for tool interfaces
- Input schema validation (Zod)
- Permission model definitions
- Concurrency safety flags
- Tool discovery and lookup

### 2.3.4 Command Registry (`src/commands.ts`)

Manages all slash commands:
- Command registration and discovery
- Type-safe command definitions
- Conditional loading based on feature flags
- Command execution orchestration

## 2.4 Execution Flow

1. **User Input** → REPL or CLI arguments
2. **Command Parsing** → Identify slash command or natural language
3. **Permission Check** → Validate user permissions
4. **Query Submission** → Send to Query Engine
5. **LLM Processing** → Anthropic API streaming
6. **Tool Execution** → Execute requested tools
7. **Result Rendering** → Display in terminal UI
8. **State Update** → Update conversation history

## 2.5 Feature Flags

Dead code elimination at build time via Bun's `bun:bundle`:

```typescript
import { feature } from 'bun:bundle'

const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

**Notable Flags:**
- `PROACTIVE` — Proactive agent mode
- `KAIROS` — Advanced scheduling
- `BRIDGE_MODE` — IDE integration
- `DAEMON` — Background daemon mode
- `VOICE_MODE` — Voice input/output
- `AGENT_TRIGGERS` — Automated agent triggers
- `MONITOR_TOOL` — System monitoring

---

# Chapter 3: The Tool System

## 3.1 Tool Architecture

Every tool in Claude Code is a self-contained module with:
- **Input schema** (Zod validation)
- **Permission model** (user approval rules)
- **Execution logic** (implementation)
- **UI components** (terminal rendering)
- **Concurrency safety** (parallelization rules)

## 3.2 Tool Definition Pattern

```typescript
export const MyTool = buildTool({
  name: 'MyTool',
  aliases: ['my_tool'],
  description: 'What this tool does',
  inputSchema: z.object({
    param: z.string(),
  }),
  async call(args, context, canUseTool, parentMessage, onProgress) {
    // Execute and return { data: result, newMessages?: [...] }
  },
  async checkPermissions(input, context) { /* Permission checks */ },
  isConcurrencySafe(input) { /* Can run in parallel? */ },
  isReadOnly(input) { /* Non-destructive? */ },
  prompt(options) { /* System prompt injection */ },
  renderToolUseMessage(input, options) { /* UI for invocation */ },
  renderToolResultMessage(content, progressMessages, options) { /* UI for result */ },
})
```

## 3.3 Tool Categories

### 3.3.1 File System Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **FileReadTool** | Read file contents (text, images, PDFs, notebooks). Supports line ranges | Yes |
| **FileWriteTool** | Create or overwrite files | No |
| **FileEditTool** | Partial file modification via string replacement | No |
| **GlobTool** | Find files matching glob patterns (e.g. `**/*.ts`) | Yes |
| **GrepTool** | Content search using ripgrep (regex-capable) | Yes |
| **NotebookEditTool** | Edit Jupyter notebook cells | No |
| **TodoWriteTool** | Write to a structured todo/task file | No |

**Key Features:**
- Multi-format support (text, images, PDFs, notebooks)
- Line-range reading for large files
- Atomic write operations
- Glob pattern matching with wildcards
- Ripgrep-powered content search

### 3.3.2 Shell & Execution Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **BashTool** | Execute shell commands in bash | No |
| **PowerShellTool** | Execute PowerShell commands (Windows) | No |
| **REPLTool** | Run code in a REPL session (Python, Node, etc.) | No |

**Capabilities:**
- Async/sync execution modes
- Interactive command support
- Output streaming
- Background process management
- Session persistence

### 3.3.3 Agent & Orchestration Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **AgentTool** | Spawn a sub-agent for complex tasks | No |
| **SendMessageTool** | Send messages between agents | No |
| **TeamCreateTool** | Create a team of parallel agents | No |
| **TeamDeleteTool** | Remove a team agent | No |
| **EnterPlanModeTool** | Switch to planning mode (no execution) | No |
| **ExitPlanModeTool** | Exit planning mode, resume execution | No |
| **EnterWorktreeTool** | Isolate work in a git worktree | No |
| **ExitWorktreeTool** | Exit worktree isolation | No |
| **SleepTool** | Pause execution (proactive mode) | Yes |
| **SyntheticOutputTool** | Generate structured output | Yes |

**Use Cases:**
- Multi-agent workflows
- Parallel task execution
- Planning before implementation
- Git worktree isolation
- Inter-agent communication

### 3.3.4 Task Management Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **TaskCreateTool** | Create a new background task | No |
| **TaskUpdateTool** | Update a task's status or details | No |
| **TaskGetTool** | Get details of a specific task | Yes |
| **TaskListTool** | List all tasks | Yes |
| **TaskOutputTool** | Get output from a completed task | Yes |
| **TaskStopTool** | Stop a running task | No |

### 3.3.5 Web Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **WebFetchTool** | Fetch content from a URL | Yes |
| **WebSearchTool** | Search the web | Yes |

### 3.3.6 MCP (Model Context Protocol) Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **MCPTool** | Invoke tools on connected MCP servers | Varies |
| **ListMcpResourcesTool** | List resources exposed by MCP servers | Yes |
| **ReadMcpResourceTool** | Read a specific MCP resource | Yes |
| **McpAuthTool** | Handle MCP server authentication | No |
| **ToolSearchTool** | Discover deferred/dynamic tools from MCP servers | Yes |

### 3.3.7 Integration Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **LSPTool** | Language Server Protocol operations (go-to-definition, find references, etc.) | Yes |
| **SkillTool** | Execute a registered skill | Varies |

### 3.3.8 Scheduling & Triggers

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **ScheduleCronTool** | Create a scheduled cron trigger | No |
| **RemoteTriggerTool** | Fire a remote trigger | No |

### 3.3.9 Utility Tools

| Tool | Description | Read-Only |
|------|-------------|-----------|
| **AskUserQuestionTool** | Prompt the user for input during execution | Yes |
| **BriefTool** | Generate a brief/summary | Yes |
| **ConfigTool** | Read or modify Claude Code configuration | No |

## 3.4 Permission Model

Every tool invocation passes through the permission system (`src/hooks/toolPermission/`).

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Prompt the user for each potentially destructive operation |
| `plan` | Show the full plan, ask once for batch approval |
| `bypassPermissions` | Auto-approve everything (dangerous) |
| `auto` | ML-based classifier decides automatically |

### Permission Rules

Rules use wildcard patterns:

```
Bash(git *)           # Allow all git commands
FileEdit(/src/*)      # Allow edits to anything in src/
FileRead(*)           # Allow reading any file
```

Each tool implements `checkPermissions()` returning:
```typescript
{ granted: boolean, reason?: string, prompt?: string }
```

## 3.5 Tool Implementation Structure

Typical tool directory structure:
```
src/tools/MyTool/
├── MyTool.ts        # Main implementation
├── UI.tsx           # Terminal rendering
├── prompt.ts        # System prompt contribution
└── utils.ts         # Tool-specific helpers
```

## 3.6 Concurrency and Safety

Tools declare concurrency safety via `isConcurrencySafe()`:
- **Safe**: Read-only operations, independent tasks
- **Unsafe**: File writes, shell commands, state mutations

The Query Engine uses this to parallelize safe operations while serializing unsafe ones.

---

# Chapter 4: The Command System

## 4.1 Command Architecture

Commands are user-facing slash commands invoked in the REPL with `/` prefix (e.g., `/commit`, `/review`).

### Command Types

| Type | Description | Example |
|------|-------------|---------|
| **PromptCommand** | Sends a formatted prompt to the LLM with injected tools | `/review`, `/commit` |
| **LocalCommand** | Runs in-process, returns plain text | `/cost`, `/version` |
| **LocalJSXCommand** | Runs in-process, returns React JSX | `/install`, `/doctor` |

## 4.2 Command Categories

### 4.2.1 Git & Version Control

| Command | Description |
|---------|-------------|
| `/commit` | Create a git commit with an AI-generated message |
| `/commit-push-pr` | Commit, push, and create a PR in one step |
| `/branch` | Create or switch git branches |
| `/diff` | View file changes (staged, unstaged, or against a ref) |
| `/pr_comments` | View and address PR review comments |
| `/rewind` | Revert to a previous state |

### 4.2.2 Code Quality

| Command | Description |
|---------|-------------|
| `/review` | AI-powered code review of staged/unstaged changes |
| `/security-review` | Security-focused code review |
| `/advisor` | Get architectural or design advice |
| `/bughunter` | Find potential bugs in the codebase |

### 4.2.3 Session & Context

| Command | Description |
|---------|-------------|
| `/compact` | Compress conversation context to fit more history |
| `/context` | Visualize current context (files, memory, etc.) |
| `/resume` | Restore a previous conversation session |
| `/session` | Manage sessions (list, switch, delete) |
| `/share` | Share a session via link |
| `/export` | Export conversation to a file |
| `/summary` | Generate a summary of the current session |
| `/clear` | Clear the conversation history |

### 4.2.4 Configuration & Settings

| Command | Description |
|---------|-------------|
| `/config` | View or modify Claude Code settings |
| `/permissions` | Manage tool permission rules |
| `/theme` | Change the terminal color theme |
| `/output-style` | Change output formatting style |
| `/color` | Toggle color output |
| `/keybindings` | View or customize keybindings |
| `/vim` | Toggle vim mode for input |
| `/effort` | Adjust response effort level |
| `/model` | Switch the active model |
| `/privacy-settings` | Manage privacy/data settings |
| `/fast` | Toggle fast mode (shorter responses) |
| `/brief` | Toggle brief output mode |

### 4.2.5 Memory & Knowledge

| Command | Description |
|---------|-------------|
| `/memory` | Manage persistent memory (CLAUDE.md files) |
| `/add-dir` | Add a directory to the project context |
| `/files` | List files in the current context |

### 4.2.6 MCP & Plugins

| Command | Description |
|---------|-------------|
| `/mcp` | Manage MCP server connections |
| `/plugin` | Install, remove, or manage plugins |
| `/reload-plugins` | Reload all installed plugins |
| `/skills` | View and manage skills |

### 4.2.7 Authentication

| Command | Description |
|---------|-------------|
| `/login` | Authenticate with Anthropic |
| `/logout` | Sign out |
| `/oauth-refresh` | Refresh OAuth tokens |

### 4.2.8 Tasks & Agents

| Command | Description |
|---------|-------------|
| `/tasks` | Manage background tasks |
| `/agents` | Manage sub-agents |
| `/ultraplan` | Generate a detailed execution plan |
| `/plan` | Enter planning mode |

### 4.2.9 Diagnostics & Status

| Command | Description |
|---------|-------------|
| `/doctor` | Run environment diagnostics |
| `/status` | Show system and session status |
| `/stats` | Show session statistics |
| `/cost` | Display token usage and estimated cost |
| `/version` | Show Claude Code version |
| `/usage` | Show detailed API usage |

### 4.2.10 IDE & Desktop Integration

| Command | Description |
|---------|-------------|
| `/bridge` | Manage IDE bridge connections |
| `/bridge-kick` | Force-restart the IDE bridge |
| `/ide` | Open in IDE |
| `/desktop` | Hand off to the desktop app |
| `/mobile` | Hand off to the mobile app |
| `/teleport` | Transfer session to another device |

## 4.3 Command Implementation Pattern

```typescript
const command = {
  type: 'prompt',
  name: 'my-command',
  description: 'What this command does',
  progressMessage: 'working...',
  allowedTools: ['Bash(git *)', 'FileRead(*)'],
  source: 'builtin',
  async getPromptForCommand(args, context) {
    return [{ type: 'text', text: '...' }]
  },
} satisfies Command
```

## 4.4 Command Execution Flow

1. **User Types Command** → `/command-name args`
2. **Parser** → Extracts command name and arguments
3. **Command Lookup** → Finds command in registry
4. **Type Check** → Routes to appropriate handler
5. **Execution** → Runs command logic
6. **Result Rendering** → Displays output in terminal

For PromptCommands:
1. Generate prompt via `getPromptForCommand()`
2. Inject allowed tools
3. Submit to Query Engine
4. Execute resulting tool calls
5. Display final response

---

# Chapter 5: Service Layer and Integrations

## 5.1 Service Architecture

The service layer (`src/services/`) provides external integrations and shared infrastructure.

## 5.2 Core Services

### 5.2.1 API Service (`src/services/api/`)

**Components:**
- Anthropic SDK client wrapper
- File upload handling
- Bootstrap configuration
- Rate limiting
- Error handling and retries

**Key Files:**
- `client.ts` — Main API client
- `fileApi.ts` — File upload/download
- `bootstrap.ts` — Initial configuration fetch

### 5.2.2 MCP Service (`src/services/mcp/`)

**Functionality:**
- MCP server connection management
- Tool discovery from MCP servers
- Resource browsing
- Authentication flows
- Connection health monitoring

**Features:**
- Automatic reconnection
- Server approval workflow
- Multi-server support
- Dynamic tool loading

### 5.2.3 OAuth Service (`src/services/oauth/`)

**Capabilities:**
- OAuth 2.0 authentication flow
- Token management
- Token refresh
- Device authorization
- Keychain integration (macOS)

### 5.2.4 LSP Service (`src/services/lsp/`)

**Language Server Protocol Integration:**
- Go-to-definition
- Find references
- Symbol search
- Hover documentation
- Code completion
- Multi-language support

**Supported Languages:**
- TypeScript/JavaScript
- Python
- Go
- Rust
- Java
- And more...

### 5.2.5 Analytics Service (`src/services/analytics/`)

**Components:**
- GrowthBook feature flags
- Usage telemetry
- Error tracking
- Performance metrics
- A/B testing support

### 5.2.6 Plugin Service (`src/services/plugins/`)

**Functionality:**
- Plugin discovery
- Installation management
- Version checking
- Auto-updates
- Marketplace integration

### 5.2.7 Compact Service (`src/services/compact/`)

**Context Compression:**
- Conversation summarization
- History truncation
- Smart context pruning
- Token budget management

### 5.2.8 Extract Memories Service (`src/services/extractMemories/`)

**Automatic Memory Extraction:**
- Parse conversations for facts
- Extract project conventions
- Identify user preferences
- Store in CLAUDE.md files

### 5.2.9 Team Memory Sync (`src/services/teamMemorySync/`)

**Collaborative Knowledge:**
- Synchronize team memories
- Shared conventions
- Cross-user learning
- Conflict resolution

### 5.2.10 Token Estimation (`src/services/tokenEstimation.ts`)

**Token Counting:**
- Estimate tokens before API call
- Track usage per turn
- Cost calculation
- Quota enforcement

### 5.2.11 Policy Limits (`src/services/policyLimits/`)

**Organization Policies:**
- Rate limiting
- Usage quotas
- Feature restrictions
- Compliance enforcement

### 5.2.12 Remote Managed Settings (`src/services/remoteManagedSettings/`)

**Enterprise Configuration:**
- Centralized settings management
- Policy enforcement
- Remote configuration updates
- Organization-wide defaults

## 5.3 Additional Services

| Service | Purpose |
|---------|---------|
| **Tips** | Contextual usage tips |
| **Agent Summary** | Agent work summaries |
| **Prompt Suggestion** | Suggested follow-up prompts |
| **Session Memory** | Session-level memory |
| **Magic Docs** | Documentation generation |
| **Auto Dream** | Background ideation |
| **x402** | x402 payment protocol |

---

# Chapter 6: Major Subsystems

## 6.1 Bridge (IDE Integration)

**Location:** `src/bridge/`

The bridge is a bidirectional communication layer connecting Claude Code's CLI with IDE extensions.

### Architecture

```
┌──────────────────┐         ┌──────────────────────┐
│   IDE Extension  │◄───────►│   Bridge Layer       │
│  (VS Code, JB)   │  JWT    │  (src/bridge/)       │
│                  │  Auth   │                      │
│  - UI rendering  │         │  - Session mgmt      │
│  - File watching │         │  - Message routing    │
│  - Diff display  │         │  - Permission proxy   │
└──────────────────┘         └──────────┬───────────┘
                                        │
                                        ▼
                              ┌──────────────────────┐
                              │   Claude Code Core   │
                              │  (QueryEngine, Tools) │
                              └──────────────────────┘
```

### Key Components

| File | Purpose |
|------|---------|
| `bridgeMain.ts` | Main bridge loop |
| `bridgeMessaging.ts` | Message protocol |
| `bridgePermissionCallbacks.ts` | Permission routing to IDE |
| `bridgeApi.ts` | API surface for IDE |
| `replBridge.ts` | REPL-to-bridge connector |
| `jwtUtils.ts` | JWT authentication |
| `sessionRunner.ts` | Session execution management |

### Features
- JWT-based authentication
- Bidirectional messaging
- Permission proxy to IDE
- File synchronization
- Diff display support

## 6.2 MCP (Model Context Protocol)

**Location:** `src/services/mcp/`

Claude Code acts as both an MCP **client** (consuming tools) and **server** (exposing tools).

### Client Features
- Tool discovery from MCP servers
- Resource browsing
- Dynamic tool loading
- Authentication handling
- Connection monitoring

### Server Mode
Exposes Claude Code tools via MCP protocol through `src/entrypoints/mcp.ts`.

### Related Tools
- `MCPTool` — Invoke MCP server tools
- `ListMcpResourcesTool` — List MCP resources
- `ReadMcpResourceTool` — Read MCP resources
- `McpAuthTool` — Authenticate with servers
- `ToolSearchTool` — Discover deferred tools

## 6.3 Permission System

**Location:** `src/hooks/toolPermission/`

Centralized permission checking for all tool invocations.

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Prompt per operation |
| `plan` | Batch approval |
| `bypassPermissions` | Auto-approve all (dangerous) |
| `auto` | ML classifier decides |

### Permission Flow

1. Tool invoked by Query Engine
2. `checkPermissions()` called
3. Check against configured rules
4. Prompt user if needed (terminal or IDE)
5. Return grant/deny decision

### Rule Examples
```
Bash(git *)           # Allow all git commands
FileEdit(/src/*)      # Allow src/ edits
FileRead(*)           # Allow all reads
```

## 6.4 Plugin System

**Location:** `src/plugins/`, `src/services/plugins/`

Extensible plugin architecture for adding capabilities.

### Plugin Lifecycle
1. **Discovery** — Scan directories and marketplace
2. **Installation** — Download and register
3. **Loading** — Initialize at startup or on-demand
4. **Execution** — Contribute tools, commands, prompts
5. **Auto-update** — Version checking and updates

### Built-in Plugins
Shipped with Claude Code in `src/plugins/bundled/`

## 6.5 Skill System

**Location:** `src/skills/`

Reusable workflows that bundle prompts and tool configurations.

### Bundled Skills (16)

| Skill | Purpose |
|-------|---------|
| `batch` | Batch operations across files |
| `claudeApi` | Direct API interaction |
| `claudeInChrome` | Chrome extension integration |
| `debug` | Debugging workflows |
| `keybindings` | Keybinding configuration |
| `loop` | Iterative refinement |
| `loremIpsum` | Placeholder text |
| `remember` | Persist to memory |
| `scheduleRemoteAgents` | Schedule agents |
| `simplify` | Simplify code |
| `skillify` | Create new skills |
| `stuck` | Get unstuck |
| `updateConfig` | Config modification |
| `verify` | Verify correctness |

### Execution
- Via `SkillTool`
- Via `/skills` command
- Custom user skills supported

## 6.6 Task System

**Location:** `src/tasks/`

Manages background and parallel work.

### Task Types

| Type | Purpose |
|------|---------|
| `LocalShellTask` | Background shell commands |
| `LocalAgentTask` | Sub-agent execution |
| `RemoteAgentTask` | Remote agent execution |
| `InProcessTeammateTask` | Parallel teammates |
| `DreamTask` | Background ideation |
| `LocalMainSessionTask` | Main session as task |

### Task Tools
- `TaskCreateTool` — Create tasks
- `TaskUpdateTool` — Update status
- `TaskGetTool` — Get details
- `TaskListTool` — List all
- `TaskOutputTool` — Get output
- `TaskStopTool` — Stop running task

## 6.7 Memory System

**Location:** `src/memdir/`

Persistent memory based on `CLAUDE.md` files.

### Memory Hierarchy

| Scope | Location | Purpose |
|-------|----------|---------|
| **Project** | `CLAUDE.md` in project root | Project facts, conventions |
| **User** | `~/.claude/CLAUDE.md` | User preferences, cross-project |
| **Extracted** | Auto-extracted from conversations | Automatic learning |
| **Team** | Synchronized across team | Shared knowledge |

### Features
- `/memory` command management
- `remember` skill for persisting
- Auto-extraction from conversations
- Team synchronization

## 6.8 Coordinator (Multi-Agent)

**Location:** `src/coordinator/`

Orchestrates multiple agents working in parallel.

### Components
- `coordinatorMode.ts` — Lifecycle management
- `TeamCreateTool` — Create agent teams
- `TeamDeleteTool` — Remove team agents
- `SendMessageTool` — Inter-agent communication
- `AgentTool` — Sub-agent spawning

### Use Cases
- Parallel research tasks
- Multi-file refactoring
- Distributed problem solving
- Specialized agent teams

## 6.9 Voice System

**Location:** `src/voice/`

Voice input/output for hands-free interaction.

### Components
- `src/services/voice.ts` — Core processing
- `src/services/voiceStreamSTT.ts` — Speech-to-text
- `src/services/voiceKeyterms.ts` — Domain vocabulary
- Voice hooks and UI components

### Features
- Real-time speech recognition
- Context-aware keywords
- Voice commands
- Audio output (TTS)

---

# Chapter 7: State Management and UI

## 7.1 State Architecture

Claude Code uses **React context + custom store** pattern.

### Core State Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `AppState` | `src/state/AppStateStore.ts` | Global mutable state |
| Context Providers | `src/context/` | React context distribution |
| Selectors | `src/state/` | Derived state functions |
| Change Observers | `src/state/onChangeAppState.ts` | Side-effects on changes |

### AppState Structure
```typescript
{
  conversation: Message[],
  config: UserConfig,
  permissions: PermissionState,
  tasks: TaskState[],
  mcpServers: MCPServerConnection[],
  currentMode: Mode,
  // ... and more
}
```

## 7.2 UI Layer (React + Ink)

### Component Architecture

**Components** (`src/components/`, ~140 components):
- Functional React components
- Ink primitives (`Box`, `Text`, `useInput()`)
- Hooks for state and side-effects
- Styled with inline props

### Key Components

| Component | Purpose |
|-----------|---------|
| `ConversationView` | Main chat display |
| `InputPrompt` | User input field |
| `ToolInvocation` | Tool call rendering |
| `ToolResult` | Tool result display |
| `StatusBar` | Bottom status bar |
| `Spinner` | Loading indicators |
| `ProgressBar` | Progress visualization |
| `ErrorBoundary` | Error handling |

### Screens (`src/screens/`)

Full-screen UIs:
- `Doctor` — Environment diagnostics
- `REPL` — Main interactive prompt
- `Resume` — Session restoration
- `Onboarding` — First-time setup

## 7.3 Rendering Pipeline

1. **State Change** → AppState mutation
2. **React Re-render** → Component tree update
3. **Ink Reconciliation** → Terminal diff calculation
4. **Terminal Update** → ANSI escape codes sent
5. **Display** → User sees changes

## 7.4 Input Handling

### Input Modes
- **Normal Mode** — Standard text input
- **Vim Mode** — Vim keybindings
- **Voice Mode** — Speech input
- **Multi-line Mode** — Extended input

### Keybindings
Configurable via `src/keybindings/`:
- Ctrl+C — Cancel
- Ctrl+D — Exit
- Tab — Autocomplete
- Arrow keys — Navigation
- Custom bindings via `/keybindings`

## 7.5 Styling and Themes

### Output Styles (`src/outputStyles/`)
- Markdown rendering
- Syntax highlighting
- Code blocks
- Tables
- Lists

### Themes
- Dark themes
- Light themes
- Custom color schemes
- Accessible high-contrast modes

---

# Chapter 8: Design Patterns and Best Practices

## 8.1 Parallel Prefetch

**Pattern:** Optimize startup by firing side-effects before heavy imports.

```typescript
// main.tsx
startMdmRawRead()
startKeychainPrefetch()
startApiPreconnect()
// ... then load heavy modules
```

**Benefits:**
- Reduced perceived startup time
- Parallel I/O operations
- Better resource utilization

## 8.2 Lazy Loading

**Pattern:** Defer heavy module imports until needed.

```typescript
// Load OpenTelemetry only when tracing is enabled
if (tracingEnabled) {
  const { initTracing } = await import('./telemetry')
  initTracing()
}
```

**Benefits:**
- Smaller initial bundle
- Faster startup
- Conditional feature loading

## 8.3 Feature Flags

**Pattern:** Build-time dead code elimination.

```typescript
import { feature } from 'bun:bundle'

const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

**Benefits:**
- Smaller production builds
- Clean feature gating
- Zero runtime overhead

## 8.4 Agent Swarms

**Pattern:** Multi-agent orchestration for parallel work.

```typescript
// Spawn multiple agents for different tasks
await Promise.all([
  agentTool.call({ task: 'research backend' }),
  agentTool.call({ task: 'research frontend' }),
  agentTool.call({ task: 'research database' }),
])
```

**Benefits:**
- Parallel task execution
- Specialized agent roles
- Faster completion

## 8.5 Tool Composition

**Pattern:** Compose complex workflows from simple tools.

```typescript
// Example: Code review workflow
async function reviewCode() {
  const files = await globTool.call({ pattern: '**/*.ts' })
  const diffs = await bashTool.call({ command: 'git diff' })
  const review = await llm.analyze(diffs)
  await fileWriteTool.call({ path: 'REVIEW.md', content: review })
}
```

## 8.6 Permission Delegation

**Pattern:** Hierarchical permission checking.

```typescript
// Tools check permissions before execution
async checkPermissions(input, context) {
  if (isReadOnly(input)) return { granted: true }
  if (context.mode === 'bypassPermissions') return { granted: true }
  return await promptUser(input)
}
```

## 8.7 Streaming Response Handling

**Pattern:** Progressive rendering of streaming LLM responses.

```typescript
for await (const chunk of stream) {
  if (chunk.type === 'text') {
    appendText(chunk.content)
  } else if (chunk.type === 'tool_call') {
    executeTool(chunk.tool, chunk.input)
  }
}
```

## 8.8 Error Recovery

**Pattern:** Automatic retry with exponential backoff.

```typescript
async function callWithRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn()
    } catch (error) {
      if (i === maxRetries - 1) throw error
      await sleep(2 ** i * 1000)
    }
  }
}
```

## 8.9 Context Management

**Pattern:** Smart context window management.

```typescript
// Automatic summarization when approaching limits
if (contextTokens > threshold) {
  const summary = await compactService.summarize(history)
  replaceHistoryWith(summary)
}
```

## 8.10 Type Safety

**Pattern:** End-to-end type safety with Zod.

```typescript
const inputSchema = z.object({
  filePath: z.string(),
  content: z.string(),
})

type Input = z.infer<typeof inputSchema>
```

---

# Chapter 9: Getting Started with the Codebase

## 9.1 Development Setup

### Prerequisites
- Bun runtime installed
- Git
- Node.js (for MCP server)

### Clone Repository
```bash
git clone https://github.com/codeaashu/claude-code.git
cd claude-code
```

### Install Dependencies
```bash
bun install
```

### Build
```bash
bun run build
```

## 9.2 Exploration Strategies

### 9.2.1 Top-Down Approach

**Start with:**
1. `README.md` — Overview and structure
2. `docs/` — Architecture documentation
3. `src/main.tsx` — Entrypoint
4. `src/QueryEngine.ts` — Core engine
5. `src/tools.ts` — Tool registry
6. `src/commands.ts` — Command registry

### 9.2.2 Bottom-Up Approach

**Start with:**
1. Pick a tool in `src/tools/`
2. Read its implementation
3. Trace how it's registered
4. Follow execution path
5. Understand integration points

### 9.2.3 Feature-Based Approach

**Pick a feature:**
1. Identify relevant command (e.g., `/commit`)
2. Find command source in `src/commands/`
3. Trace tool invocations
4. Follow data flow
5. Understand UI rendering

## 9.3 Key Exploration Tools

### Using the MCP Server

Install and query the codebase:
```bash
# Install MCP server
cd mcp-server
npm install && npm run build

# Register with Claude Code
claude mcp add claude-code-explorer -- node /path/to/mcp-server/dist/index.js
```

**Available MCP Tools:**
- `list_tools` — List all tools
- `list_commands` — List all commands
- `get_tool_source` — Read tool source
- `search_source` — Grep the codebase

### Using Grep Patterns

```bash
# Find all tool definitions
grep -r "buildTool" src/tools/

# Find command definitions
grep -r "satisfies Command" src/commands/

# Find permission checks
grep -r "checkPermissions" src/
```

## 9.4 Understanding Key Flows

### 9.4.1 Tool Execution Flow

```
User Message
  ↓
Query Engine
  ↓
LLM Response (tool call)
  ↓
Permission Check (src/hooks/toolPermission/)
  ↓
Tool Execution (src/tools/*/index.ts)
  ↓
Result Rendering (src/tools/*/UI.tsx)
  ↓
Feed back to LLM
```

### 9.4.2 Command Execution Flow

```
User Types /command
  ↓
Command Parser (src/commands.ts)
  ↓
Command Lookup
  ↓
Type Check (Prompt/Local/LocalJSX)
  ↓
Execute Command Logic
  ↓
Render Result
```

### 9.4.3 State Update Flow

```
Action (tool call, user input)
  ↓
AppState Mutation (src/state/AppStateStore.ts)
  ↓
Change Observers (src/state/onChangeAppState.ts)
  ↓
React Re-render
  ↓
Ink Reconciliation
  ↓
Terminal Update
```

## 9.5 Common Development Tasks

### Adding a New Tool

1. Create directory: `src/tools/MyTool/`
2. Implement tool: `MyTool.ts`
3. Create UI: `UI.tsx`
4. Register in: `src/tools.ts`
5. Add tests if needed

### Adding a New Command

1. Create file: `src/commands/mycommand.ts`
2. Define command object
3. Import in: `src/commands.ts`
4. Add to registry
5. Test in REPL

### Modifying Permissions

1. Edit: `src/hooks/toolPermission/`
2. Update permission rules
3. Test with various modes
4. Update documentation

## 9.6 Testing

### Running Tests
```bash
bun test
```

### Test Structure
- Unit tests for tools
- Integration tests for flows
- E2E tests for commands

## 9.7 Contributing

See `CONTRIBUTING.md` for guidelines on:
- Code style
- Pull request process
- Documentation standards
- Testing requirements

---

# Appendix: Key Files Reference

## A.1 Core Files

| File | Lines | Purpose |
|------|------:|---------|
| `src/main.tsx` | Large | CLI entrypoint, React/Ink renderer |
| `src/QueryEngine.ts` | ~46K | LLM API engine, streaming, tool loops |
| `src/Tool.ts` | ~29K | Tool type definitions and interfaces |
| `src/commands.ts` | ~25K | Command registry and execution |
| `src/tools.ts` | Medium | Tool registry |
| `src/context.ts` | Medium | Context collection |
| `src/cost-tracker.ts` | Medium | Token cost tracking |

## A.2 Major Tool Files

| Tool | Location | Purpose |
|------|----------|---------|
| `FileReadTool` | `src/tools/FileReadTool/` | File reading |
| `FileWriteTool` | `src/tools/FileWriteTool/` | File writing |
| `FileEditTool` | `src/tools/FileEditTool/` | File editing |
| `BashTool` | `src/tools/BashTool/` | Shell execution |
| `AgentTool` | `src/tools/AgentTool/` | Sub-agent spawning |
| `GlobTool` | `src/tools/GlobTool/` | File pattern matching |
| `GrepTool` | `src/tools/GrepTool/` | Content search |
| `MCPTool` | `src/tools/MCPTool/` | MCP tool invocation |
| `LSPTool` | `src/tools/LSPTool/` | LSP integration |

## A.3 Major Command Files

| Command | Location | Purpose |
|---------|----------|---------|
| `/commit` | `src/commands/commit.ts` | Git commit |
| `/review` | `src/commands/review.ts` | Code review |
| `/mcp` | `src/commands/mcp/` | MCP management |
| `/config` | `src/commands/config/` | Configuration |
| `/doctor` | `src/commands/doctor/` | Diagnostics |
| `/memory` | `src/commands/memory/` | Memory management |

## A.4 Service Files

| Service | Location | Purpose |
|---------|----------|---------|
| API Client | `src/services/api/client.ts` | Anthropic API |
| MCP Manager | `src/services/mcp/` | MCP connections |
| OAuth | `src/services/oauth/` | Authentication |
| LSP Manager | `src/services/lsp/` | Language servers |
| Analytics | `src/services/analytics/` | Telemetry |

## A.5 State Files

| File | Location | Purpose |
|------|----------|---------|
| AppState | `src/state/AppStateStore.ts` | Global state |
| Selectors | `src/state/` | Derived state |
| Observers | `src/state/onChangeAppState.ts` | Side-effects |

## A.6 Bridge Files

| File | Location | Purpose |
|------|----------|---------|
| Bridge Main | `src/bridge/bridgeMain.ts` | Main loop |
| Messaging | `src/bridge/bridgeMessaging.ts` | Protocol |
| Permissions | `src/bridge/bridgePermissionCallbacks.ts` | Permission proxy |
| JWT Utils | `src/bridge/jwtUtils.ts` | Authentication |

---

# Conclusion

This book has provided a comprehensive overview of the Claude Code codebase, organized by functionality:

- **Chapter 1**: Introduction and overview of the leaked source
- **Chapter 2**: Core architecture and execution pipeline
- **Chapter 3**: The 40+ agent tools that power Claude's capabilities
- **Chapter 4**: The 85+ slash commands for user interaction
- **Chapter 5**: Service layer and external integrations
- **Chapter 6**: Major subsystems (bridge, MCP, permissions, plugins, skills, tasks, memory, coordinator, voice)
- **Chapter 7**: State management and React/Ink UI architecture
- **Chapter 8**: Design patterns and best practices
- **Chapter 9**: Getting started guide for exploration and development

## Resources

- **Repository**: https://github.com/codeaashu/claude-code
- **Original Leak Date**: March 31, 2026
- **Documentation**: `/docs` directory
- **MCP Server**: `/mcp-server` directory
- **Contributing**: `CONTRIBUTING.md`

## Disclaimer

This book documents source code leaked from Anthropic's npm registry. All original source code is the property of Anthropic. This is not an official release and is not licensed for redistribution.

---

**End of Book**

*Generated: April 29, 2026*

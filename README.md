# Claude Code Source (claude-code-src)

This repository contains the TypeScript/React source code for the Claude Code CLI and runtime. It includes the interactive terminal UI, tool orchestration, remote control modes, and supporting services used by Claude Code.

## What this repo provides

- **CLI + REPL runtime** with command routing and interactive UI.
- **Tooling and task orchestration** (bash, file, web, MCP, agent tools).
- **Terminal UI system** built on a custom Ink-based renderer.
- **Services layer** for API access, telemetry, policies, MCP, LSP, and plugins.
- **Skills and plugins** for extensibility.
- **Remote control and direct-connect** session support.
- **Session and memory persistence** with project-scoped storage.

## Entry points and runtime modes

| Entry point | Purpose |
| --- | --- |
| `entrypoints/cli.tsx` | Lightweight CLI bootstrap and fast-path flags. |
| `main.tsx` | Primary CLI runtime (command routing, UI boot, main loop). |
| `setup.ts` | Session setup and environment preparation. |
| `entrypoints/init.ts` | Startup initialization (configs, telemetry, networking). |
| `entrypoints/mcp.ts` | MCP server over stdio exposing Claude Code tools. |
| `entrypoints/agentSdkTypes.ts` + `entrypoints/sdk/*` | SDK schemas/types for integrations. |
| `entrypoints/sandboxTypes.ts` | Sandbox integration types. |
| `replLauncher.tsx` | REPL launcher for interactive sessions. |
| `bridge/bridgeMain.ts` | Remote control/bridge runtime. |
| `server/createDirectConnectSession.ts` | Direct-connect session setup. |

## High-level architecture

1. **Bootstrap** (`entrypoints/cli.tsx`) handles fast paths and loads the main runtime.
2. **Initialization** (`entrypoints/init.ts`, `setup.ts`) loads config, telemetry, and environment state.
3. **Command routing** (`commands.ts`, `commands/`) builds the command registry for CLI/REPL.
4. **Query pipeline** (`query.ts`, `query/`) manages prompt construction, tool execution, and streaming responses.
5. **Tools and tasks** (`Tool.ts`, `tools.ts`, `Task.ts`, `tasks/`) define capabilities and background execution.
6. **UI layer** (`ink/`, `components/`, `screens/`) renders terminal experiences and dialogs.
7. **Services** (`services/`) implement API calls, MCP, plugins, policy gates, and telemetry.
8. **Remote and bridge modes** (`bridge/`, `remote/`, `server/`) enable remote control and distributed sessions.

## Repository layout

| Path | Description |
| --- | --- |
| `assistant/` | Assistant (Kairos) mode implementation. |
| `bootstrap/` | Global session state and bootstrap helpers. |
| `bridge/` | Remote control/bridge runtime and networking. |
| `buddy/` | Companion/buddy UI behavior. |
| `cli/` | CLI transports, output helpers, and handlers. |
| `commands/` | Built-in slash/CLI commands. |
| `components/` | Shared UI components. |
| `constants/` | Product and feature constants. |
| `context/` | System/user context assembly for prompts. |
| `coordinator/` | Coordinator/multi-agent mode. |
| `entrypoints/` | Runtime entrypoints and SDK schemas. |
| `hooks/` | React hooks and runtime hook helpers. |
| `ink/` | Custom terminal renderer and primitives. |
| `keybindings/` | Keybinding parser and resolver. |
| `memdir/` | Memory storage paths and helpers. |
| `migrations/` | Config and data migrations. |
| `moreright/` | UI layout helpers (e.g., right-side panels). |
| `native-ts/` | Native bindings (yoga-layout, file-index, color-diff). |
| `outputStyles/` | Output style loaders. |
| `plugins/` | Built-in plugin registry and loaders. |
| `query/` + `query.ts` | Query pipeline and message orchestration. |
| `remote/` | Remote session manager and transport adapters. |
| `schemas/` | JSON schemas (settings, hooks, etc.). |
| `screens/` | Screen-level UI flows. |
| `server/` | Direct-connect server/session support. |
| `services/` | API, analytics, MCP, LSP, and policy services. |
| `skills/` | Skill system (bundled + dynamic). |
| `state/` | Application state store. |
| `tasks/` + `Task.ts` | Task execution and lifecycle management. |
| `tools/` + `Tool.ts` | Tool implementations and interface. |
| `types/` | Shared types. |
| `upstreamproxy/` | Upstream proxy support for constrained environments. |
| `utils/` | Cross-cutting utilities (config, auth, file ops). |
| `vim/` | Vim-style input/motion helpers. |
| `voice/` | Voice mode support. |

## Configuration and storage

### Config directories

- **Config home**: `~/.claude` by default, or `CLAUDE_CONFIG_DIR` if set.
- **Global config file**: `~/.claude.json` (or `~/.claude-<suffix>.json` when OAuth variants are used). Legacy fallback: `~/.claude/.config.json`.

### Settings files

Settings are merged from multiple sources in this order (later wins):

1. **User settings**: `~/.claude/settings.json` (or `cowork_settings.json` in cowork mode).
2. **Project settings**: `<repo>/.claude/settings.json`.
3. **Local settings** (gitignored): `<repo>/.claude/settings.local.json`.
4. **CLI settings flag**: `--settings <path>`.
5. **Managed/policy settings**: OS-specific `managed-settings.json` + `managed-settings.d/*.json`.

Managed settings locations:

- macOS: `/Library/Application Support/ClaudeCode`
- Windows: `C:\Program Files\ClaudeCode`
- Linux: `/etc/claude-code`

Settings schema: <https://json.schemastore.org/claude-code-settings.json>

### Data storage

- **Projects & transcripts**: `~/.claude/projects/` (project-scoped sessions and logs).
- **Teams**: `~/.claude/teams/`.
- **Memory**: `~/.claude/memory/` (auto-memory), with overrides via settings or env vars.

### Runtime flags and environment variables (selected)

- `CLAUDE_CONFIG_DIR`: override config home directory.
- `CLAUDE_CODE_SIMPLE` / `--bare`: disable most background features.
- `CLAUDE_CODE_REMOTE`: enable remote/CCR-specific behavior.
- `CLAUDE_CODE_REMOTE_MEMORY_DIR`: override memory base directory in remote environments.

## Commands, tools, and extensions

- **Commands** live in `commands/` and are registered in `commands.ts`. They cover session management, configuration, tasks, plugins, MCP, review workflows, and more.
- **Tools** implement the capability surface for Claude Code (`tools/`, `Tool.ts`, `tools.ts`). Examples include Bash, file read/write/edit, glob/grep, web fetch/search, MCP, and agent tools.
- **Skills** (`skills/`) and **plugins** (`plugins/`) extend functionality; bundled skills/plugins are loaded at startup and optional sources are discoverable via settings.

## Development notes

- **Runtime requirement**: Node.js 18+ (checked at startup in `setup.ts`).
- **Feature flags**: Compile-time feature gates are wired through `bun:bundle` (`feature('FLAG')`).
- **Build/test scripts**: This repository does not include package manifests or build/test scripts; it assumes an external build pipeline.


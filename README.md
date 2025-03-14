# Cursor History

<p align="center">
  <img src="docs/readme-banner.png" alt="cursor-history — One interface for your entire Cursor history. Composer, Agent transcripts, and Store / ACP sources feed a unified CLI and Node.js API." width="960">
</p>

[![npm version](https://img.shields.io/npm/v/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![npm downloads](https://img.shields.io/npm/dm/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-20%2C%2022--26-green.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0%2B-blue.svg)](https://www.typescriptlang.org/)

🇺🇸 [English](./README.md) | 🇨🇳 [中文](./docs/readme_zh.md) | 🇫🇷 [Français](./docs/readme_fr.md) | 🇪🇸 [Español](./docs/readme_es.md) | 🇯🇵 [日本語](./docs/readme_ja.md) | 🇰🇷 [한국어](./docs/readme_ko.md) | 🇺🇦 [Українська](./docs/readme_uk.md)

**One interface for your entire Cursor history.**

Cursor conversations can be spread across workspaces, IDE databases, Agent transcripts, and newer CLI / ACP session stores. `cursor-history` discovers supported local sources and brings them together through one CLI and Node.js library.

Search conversation content across workspaces, inspect messages and available tool activity, and export to Markdown or JSON. Back up and restore Composer history, or migrate supported Composer sessions when projects move.

**Already have months of Cursor history? No prior capture or indexing setup is required.** Search runs locally, with no embeddings or API key.

**Prefer an MCP server interface?** Connect [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp#quick-start) to expose history as MCP tools. Agents can also call this project's CLI or Node.js API directly—choose the interface that fits your workflow. The packages release independently; check [MCP storage compatibility and release information](https://github.com/S2thend/cursor-history-mcp#compatibility).

## Quick Start

```bash
npm install -g cursor-history

cursor-history list --all
cursor-history search "authentication"
cursor-history show 1
cursor-history export 1
cursor-history backup
```

Requires Node.js 20.x or 22.x–26.x and existing local Cursor history. To try without a permanent install, run `npx cursor-history list --all`.

The numbers in `show` and `export` refer to the list from the same data source and workspace scope; use the session UUID for saved commands. The `backup` command archives Composer databases, not Store databases or transcripts.

[Comparison](#comparison) · [Installation](#installation) · [Usage](#usage) · [Example output](#example-output) · [Library API](#library-api) · [Roadmap](#roadmap) · [Compatibility and safe upgrades](#compatibility-and-safe-upgrades)

<a id="comparison"></a>

## Why choose cursor-history?

Choose `cursor-history` when you want to find, inspect, preserve, and reuse Cursor history through one CLI and Node.js API.

- **Multiple Cursor sources, one interface** — Read supported Composer, Agent transcript, Store / CLI, and ACP sources.
- **From search to preservation** — Search conversation content across workspaces, inspect available diffs and tool activity, and export Markdown or JSON. Back up and restore Composer data, or preview eligible Composer migrations with `--dry-run`.
- **Ready for your workflow** — Use the Node.js API in your own tools, connect the separate [MCP companion](https://github.com/S2thend/cursor-history-mcp#compatibility), or follow the [WSL configuration](#where-cursor-stores-data) to read Windows-side or WSL-local Store data.

| Dimension | **cursor-history (this project)** | [deja-vu](https://github.com/vshulcz/deja-vu) | [cursaves](https://github.com/Callum-Ward/cursaves) | [johnlindquist/cursor-history](https://github.com/johnlindquist/cursor-history) |
|---|---|---|---|---|
| Focus | Cursor history access and management; CLI + Node.js API | Cross-agent memory | Git / S3 sync | Browsing, export, clipboard |
| Documented Cursor sources | Composer, Agent transcripts, Store / CLI, ACP | IDE SQLite, CLI transcripts¹ | Workspace/global SQLite | Not documented |
| Search | Conversation content across workspaces | Cross-agent indexed search | Not documented | Fuzzy title search |
| Preservation and moving history | Composer backup/restore; eligible Composer migration | Memory sync/handoff | Snapshot restore; workspace copy | Not documented |

Documentation comparison checked **2026-09-09**, using the linked project READMEs and ¹ [deja-vu's Cursor format registry](https://github.com/vshulcz/deja-vu/blob/main/docs/registry/cursor.md); not a comparative runtime test. “Not documented” means the reviewed sources do not describe the capability, not that it is unsupported.

Our backup and restore cover Composer databases only. Migration supports eligible Composer sessions; Store-only and merged-source sessions are excluded. Store / ACP sessions and transcripts can be exported when readable, but those exports are not restorable backup archives. See the [compatibility contract](./docs/compatibility.md).

## Why this exists

You may remember solving a problem with Cursor without remembering which project, session, or Cursor interface you used. A keyword search across the local history can bring that conversation back.

Cursor's [Agent CLI](https://cursor.com/docs/cli/reference/parameters) provides `agent ls`, `agent resume`, and `agent --resume=<id>` to find or resume CLI sessions. `cursor-history` adds workflows for reading and managing supported local history across workspaces and storage formats.

| What you want to do | Where to start |
|---|---|
| Continue a conversation in Cursor Agent CLI | Cursor's `agent ls` or `agent --resume=<id>` |
| Find a phrase in conversation content across workspaces | `cursor-history search "connection pool"` |
| Read or export a discovered local session | `cursor-history show 1` or `cursor-history export 1` |
| Back up or restore Composer history | `cursor-history backup` / `cursor-history restore` ([usage](#backup--restore)) |
| Move supported Composer sessions to another workspace | `cursor-history migrate-session` ([usage](#migrate-sessions)) |
| Build on history in your own application | The [Node.js API](#library-api) |

## Works across Cursor storage generations and surfaces

Different Cursor versions and interfaces can leave different local representations on the same machine. `cursor-history` discovers these supported sources:

| Source | Local files | How they are used |
|---|---|---|
| Legacy / Composer | `workspaceStorage/*/state.vscdb` and `globalStorage/state.vscdb` under the Cursor user data directory | Workspace records and global conversation data |
| Agent transcripts | `~/.cursor/projects/**/agent-transcripts/**/*.jsonl` | Conversation text and tool calls available in the transcript |
| Store / CLI | `~/.cursor/chats/**/store.db` | Per-session Store conversation data |
| ACP sessions | `~/.cursor/acp-sessions/**/store.db` | Per-session Store data discovered under the ACP root |

See [Where Cursor Stores Data](#where-cursor-stores-data) for platform paths, custom roots, and WSL configuration.

These representations do not always contain the same fields. A transcript may omit timestamps or tool results. When a usable Store database and transcript coexist within scope, the database supplies the Store conversation and the transcript remains provenance. Source details and timestamp provenance distinguish stored information from inferred values and partial views.

Reading support does not imply backup or migration support: current backup archives contain Composer data only, and Store-only or merged-source sessions cannot be migrated. Broader coverage is on the [roadmap](#roadmap). See the [Compatibility and Data-Integrity Contract](./docs/compatibility.md) for the full boundaries.

## Three things you can do with your history

### Find it

```bash
cursor-history list --all
cursor-history search "connection pool"
cursor-history show 1
```

Find the conversation where you already solved the problem, even if it belongs to another workspace. Existing supported local history is searchable without having installed this tool beforehand.

### Preserve it

```bash
cursor-history export 1
cursor-history backup
cursor-history migrate-session 1 /path/to/new/workspace --dry-run
```

Export readable sessions to Markdown or JSON. Back up Composer history and preview migration of supported Composer sessions before moving them to another workspace.

### Reuse it

Use the [Node.js library](#library-api) directly, or connect the separate [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp) server so an MCP-capable assistant can search your existing development history.

Months of decisions, fixes, prompts, and tool activity may already be on disk. Make that context easier to find the next time you need it.

## Features

- **Dual interface** - Use as CLI tool or import as a library in your Node.js projects
- **List sessions** - View all chat sessions across workspaces
- **View conversations** - Inspect the content available in each source, including:
  - AI responses with natural language explanations
  - **Full diff display** for file edits and writes with syntax highlighting
  - **Detailed tool calls** showing all parameters (file paths, search patterns, commands, etc.)
  - AI reasoning and thinking blocks
  - Message timestamps with explicit stored/inferred provenance
- **Search** - Search conversation content across workspaces by keyword with highlighted matches
- **Export** - Save sessions as Markdown or JSON files
- **Migrate** - Move or copy supported Composer sessions between workspaces (e.g., when renaming projects)
- **Backup & Restore** - Back up Composer databases and restore them when needed
- **Cross-platform & WSL support** - Works on macOS, Windows, Linux, and WSL. From WSL, configure the data path to read Cursor Store sessions stored on Windows or inside WSL. See [WSL configuration](#where-cursor-stores-data).

## Installation

### From NPM (Recommended)

```bash
# Install globally
npm install -g cursor-history

# Use the CLI
cursor-history list
```

If you prefer `pnpm`:

```bash
# Install globally
pnpm add -g cursor-history

# Use the CLI
cursor-history list
```

### From Source

```bash
# Clone and build
git clone https://github.com/S2thend/cursor_chat_history.git
cd cursor_chat_history
npm install
npm run build

# Run directly
node dist/cli/index.js list

# Or link globally
npm link
cursor-history list
```

Equivalent `pnpm` workflow:

```bash
# Clone and build
git clone https://github.com/S2thend/cursor_chat_history.git
cd cursor_chat_history
pnpm install
pnpm build

# Run directly
node dist/cli/index.js list
node dist/cli/index.js show 1 --json
```

## Requirements

- Node.js 20.x or 22.x–26.x (Node 21 is not supported; Node.js 22.5+ is recommended for built-in SQLite support)
- Existing local history from Cursor IDE or Agent CLI in a supported format

## SQLite Driver Configuration

cursor-history supports two SQLite drivers for maximum compatibility:

| Driver | Description | Node.js capability boundary |
|--------|-------------|----------------------------|
| `node:sqlite` | Built-in module; selected only when it provides every API required by the operation | Import/read support starts in 22.5; online backup starts in 22.16.0 and 23.8.0 |
| `better-sqlite3` | Native binding and automatic fallback when capable | Supported project majors: 20 and 22–26 |

### Automatic Driver Selection

Driver selection is per operation and probes capabilities, not only whether a module imports. In
automatic mode cursor-history:

1. prefers **node:sqlite** when it implements all APIs required by that operation; then
2. falls back to an installed, capable **better-sqlite3**.

If neither provider is capable, the operation returns an actionable typed error. Store snapshot
capability failures are fatal and are not silently converted into an empty/partial session or a
transcript fallback.

### Manual Driver Selection

You can force a specific driver using the environment variable:

```bash
# Force better-sqlite3
CURSOR_HISTORY_SQLITE_DRIVER=better-sqlite3 cursor-history list

# Force node:sqlite (the runtime must support every API needed by this operation)
CURSOR_HISTORY_SQLITE_DRIVER=node:sqlite cursor-history list
```

A forced driver never falls back. If it lacks a required capability, cursor-history reports the
driver, operation, missing capability, available alternative, and remedy.

### Debug Driver Selection

To see which driver is being used:

```bash
DEBUG=cursor-history:* cursor-history list
```

### Library API Driver Control

When using cursor-history as a library, you can control the driver programmatically:

```typescript
import { setDriver, getActiveDriver, listSessions } from 'cursor-history';

// Force a specific driver before any operations
setDriver('better-sqlite3');

// Check which driver is active
const driver = getActiveDriver();
console.log(`Using driver: ${driver}`);

// Or configure via LibraryConfig
const result = await listSessions({
  sqliteDriver: 'node:sqlite'  // Force node:sqlite for this call
});
```

## Usage

### List Sessions

```bash
# List recent sessions (default: 20)
cursor-history list

# List all sessions
cursor-history list --all

# List with composer IDs (for external tools)
cursor-history list --ids

# Limit results
cursor-history list -n 10

# List workspaces only
cursor-history list --workspaces
```

`list --workspaces` is intentionally unscoped discovery and cannot be combined with `--workspace`.
Choose a path from its output, then use that same `--workspace` value for scoped session commands.

### View a Session

```bash
# Show session by index number
cursor-history show 1

# Show with truncated messages (for quick overview)
cursor-history show 1 --short

# Show full AI thinking/reasoning text
cursor-history show 1 --think

# Show full tool call details (commands, content, results)
cursor-history show 1 --tool

# Show full error messages (not truncated to 300 chars)
cursor-history show 1 --error

# Filter by message type (user, assistant, tool, thinking, error)
cursor-history show 1 --only user
cursor-history show 1 --only user,assistant
cursor-history show 1 --only tool,error

# Combine options
cursor-history show 1 --short --think --tool --error
cursor-history show 1 --only user,assistant --short

# Output as JSON
cursor-history show 1 --json
```

### Search

```bash
# Search for keyword
cursor-history search "react hooks"

# Limit results
cursor-history search "api" -n 5

# Adjust context around matches
cursor-history search "error" --context 100
```

### Export

```bash
# Export single session to Markdown
cursor-history export 1

# Export to specific file
cursor-history export 1 -o ./my-chat.md

# Export as JSON
cursor-history export 1 --format json

# Export all sessions to directory
cursor-history export --all -o ./exports/

# Overwrite existing files
cursor-history export 1 --force
```

### Migrate Sessions

Migration supports eligible Composer sessions. Store-only, merged-source, and ambiguous sessions are rejected; use `--dry-run` to preview a migration.

```bash
# Move a single session to another workspace
cursor-history migrate-session 1 /path/to/new/project

# Move multiple sessions (comma-separated indices or IDs)
cursor-history migrate-session 1,3,5 /path/to/project

# Copy instead of move (keeps original)
cursor-history migrate-session --copy 1 /path/to/project

# Preview what would happen without making changes
cursor-history migrate-session --dry-run 1 /path/to/project

# Move all sessions from one workspace to another
cursor-history migrate /old/project /new/project

# Copy all sessions (backup)
cursor-history migrate --copy /project /backup/project

# Force merge with existing sessions at destination
cursor-history migrate --force /old/project /existing/project
```

### Backup & Restore

Backup archives contain Composer `state.vscdb` data. They do not include Store databases, Agent transcripts, or ACP session stores. Export readable sessions from those sources to Markdown or JSON when you need a portable copy; exports are not restorable backup archives.

```bash
# Create a backup of Composer chat history
cursor-history backup

# Create backup to specific file
cursor-history backup -o ~/my-backup.zip

# Overwrite existing backup
cursor-history backup --force

# List available backups
cursor-history list-backups

# List backups in a specific directory
cursor-history list-backups -d /path/to/backups

# Restore from a backup
cursor-history restore ~/cursor-history-backups/backup.zip

# Restore to a custom location
cursor-history restore backup.zip --target /custom/cursor/data

# Force overwrite existing data
cursor-history restore backup.zip --force

# View sessions from a backup without restoring
cursor-history list --backup ~/backup.zip
cursor-history show 1 --backup ~/backup.zip
cursor-history search "query" --backup ~/backup.zip
cursor-history export 1 --backup ~/backup.zip
```

### Global Options

```bash
# Output as JSON (works with all commands)
cursor-history --json list

# Use custom Cursor data path
cursor-history --data-path ~/.cursor-alt list

# Filter by workspace
cursor-history --workspace /path/to/project list
```

## Example Output

### List Sessions

<pre>
<span style="color: #888">cursor-history list</span>

<span style="color: #5fd7ff">cursor-history</span> - Chat History Browser

<span style="color: #5fd7ff">Sessions (showing 3 of 42):</span>

  <span style="color: #af87ff">#1</span>  <span style="color: #87d787">12/26 09:15 AM</span>  <span style="color: #d7d787">cursor_chat_history</span>
      <span style="color: #888">15 messages · Updated 2 min ago</span>
      <span style="color: #fff">"Help me fix the migration path issue..."</span>

  <span style="color: #af87ff">#2</span>  <span style="color: #87d787">12/25 03:22 PM</span>  <span style="color: #d7d787">my-react-app</span>
      <span style="color: #888">8 messages · Updated 18 hours ago</span>
      <span style="color: #fff">"Add authentication to the app..."</span>

  <span style="color: #af87ff">#3</span>  <span style="color: #87d787">12/24 11:30 AM</span>  <span style="color: #d7d787">api-server</span>
      <span style="color: #888">23 messages · Updated 2 days ago</span>
      <span style="color: #fff">"Create REST endpoints for users..."</span>
</pre>

### Show Session Details

<pre>
<span style="color: #888">cursor-history show 1</span>

<span style="color: #5fd7ff">Session #1</span> · <span style="color: #d7d787">cursor_chat_history</span>
<span style="color: #888">15 messages · Created 12/26 09:15 AM</span>

────────────────────────────────────────

<span style="color: #87d787">You:</span> <span style="color: #888">09:15:23 AM</span>

Help me fix the migration path issue in the codebase

────────────────────────────────────────

<span style="color: #af87ff">Assistant:</span> <span style="color: #888">09:15:45 AM</span>

I'll help you fix the migration path issue. Let me first examine
the relevant files.

────────────────────────────────────────

<span style="color: #d7af5f">Tool:</span> <span style="color: #888">09:15:46 AM</span>
<span style="color: #d7af5f">🔧 Read File</span>
   <span style="color: #888">File:</span> <span style="color: #5fd7ff">src/core/migrate.ts</span>
   <span style="color: #888">Content:</span> <span style="color: #fff">export function migrateSession(sessionId: string...</span>
   <span style="color: #87d787">Status: ✓ completed</span>

────────────────────────────────────────

<span style="color: #d7af5f">Tool:</span> <span style="color: #888">09:16:02 AM</span>
<span style="color: #d7af5f">🔧 Edit File</span>
   <span style="color: #888">File:</span> <span style="color: #5fd
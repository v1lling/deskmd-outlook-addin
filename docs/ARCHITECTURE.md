# Orbit Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                     ORBIT (Tauri)                        │
├─────────────────────────────────────────────────────────┤
│  Next.js App Router                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ /        │  │/projects │  │/settings │              │
│  │ (tasks)  │  │  /notes  │  │          │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                      │                                   │
│  ┌───────────────────▼───────────────────┐              │
│  │         lib/orbit/* (CRUD)            │              │
│  │  areas.ts  projects.ts  tasks.ts      │              │
│  │  notes.ts  meetings.ts                │              │
│  └───────────────────┬───────────────────┘              │
│                      │                                   │
│  ┌───────────────────▼───────────────────┐              │
│  │         tauri-fs.ts                   │              │
│  │  (Tauri plugin-fs / mock for browser) │              │
│  └───────────────────┬───────────────────┘              │
└──────────────────────┼──────────────────────────────────┘
                       │
                       ▼
            ┌─────────────────┐
            │   ~/Orbit/      │
            │  (Markdown FS)  │
            └─────────────────┘
```

## Key Design Decisions

### Why Markdown Files?

| Approach | Pros | Cons |
|----------|------|------|
| **Markdown** | Portable, grep-able, offline, version-controllable | Slower for large datasets |
| SQLite | Fast queries | Binary, harder to inspect |
| Cloud DB | Sync, collaboration | Requires internet |

**Decision**: Markdown. Use case (~100s of tasks) doesn't need database performance, portability matters more.

### Dual-Mode Operation

- `npm run dev` - Browser mode with mock data (fast UI dev)
- `npm run tauri dev` - Desktop mode with real file system

Detection via `isTauri()` in `tauri-fs.ts`.

### State Management

- **TanStack Query**: Server state (tasks, projects, notes, meetings)
- **Zustand**: Client state (settings, current area)

## Module Structure

### Core Data Layer (`src/lib/orbit/`)

| File | Purpose |
|------|---------|
| `tauri-fs.ts` | File system abstraction (Tauri/mock) |
| `parser.ts` | Markdown frontmatter parsing, date helpers |
| `constants.ts` | Magic strings (SPECIAL_DIRS, PATH_SEGMENTS) |
| `search.ts` | Cross-area search helpers |
| `calculations.ts` | Business logic (task stats, grouping) |
| `areas.ts` | Area CRUD operations |
| `projects.ts` | Project CRUD operations |
| `tasks.ts` | Task CRUD operations |
| `notes.ts` | Note CRUD operations |
| `meetings.ts` | Meeting CRUD operations |

### Design System (`src/lib/`)

| File | Purpose |
|------|---------|
| `design-tokens.ts` | Colors for status, priority, areas |
| `utils.ts` | Utility functions (cn for classnames) |

### State Management (`src/stores/`)

- React Query hooks for server state
- Zustand stores for client state
- Grouped task helpers

## File Format

All entities use YAML frontmatter + Markdown body:

```markdown
---
title: Setup webhook
status: doing
priority: high
created: 2024-01-15
---

Task content here...
```

Compatible with Obsidian for manual editing.

## Special Directories

- `_unassigned` - Tasks/notes not belonging to any project
- `context/` - AI knowledge files (future)

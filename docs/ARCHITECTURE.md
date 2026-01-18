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

- **TanStack Query**: Server state (tasks, projects, notes)
- **Zustand**: Client state (settings, current area)

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

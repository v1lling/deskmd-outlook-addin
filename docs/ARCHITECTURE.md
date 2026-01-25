# Orbit Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                     ORBIT (Tauri)                        │
├─────────────────────────────────────────────────────────┤
│  Next.js App Router                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ /        │  │  /docs   │  │/settings │              │
│  │(dashboard)│  │  /tasks  │  │/personal │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                      │                                   │
│  ┌───────────────────▼───────────────────┐              │
│  │         lib/orbit/* (CRUD)            │              │
│  │  workspaces.ts  projects.ts  tasks.ts │              │
│  │  docs.ts  meetings.ts  personal.ts    │              │
│  │  dashboard.ts  search-index.ts        │              │
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

## Data Hierarchy

The application has a clear three-level hierarchy:

```
Personal Space (not workspace-scoped)
├── Capture (quick capture, triage from here)
├── Tasks
└── Docs

Workspace (Client/Context)
├── Workspace-level Docs (shared across all projects)
└── Projects
    └── Project
        ├── Tasks
        ├── Docs
        └── Meetings
```

### Scopes Explained

| Scope | Description | Location |
|-------|-------------|----------|
| **Personal** | Private items not tied to any client/workspace | `personal/` |
| **Workspace** | Shared docs for the whole workspace | `workspaces/{id}/docs/` |
| **Project** | Items belonging to a specific project | `workspaces/{id}/projects/{id}/` |
| **Unassigned** | Workspace items not yet assigned to a project | `workspaces/{id}/_unassigned/` |

## File Structure

```
~/Orbit/
├── config.json                     # App settings (theme, current workspace)
├── personal/                       # Personal space (PERSONAL_SPACE_ID)
│   ├── capture/
│   │   └── tasks/*.md              # Quick capture items
│   ├── tasks/*.md                  # Personal tasks
│   └── docs/                       # Personal docs (tree structure)
│       ├── folder/
│       │   └── doc.md
│       └── doc.md
├── workspaces/
│   └── {workspace-id}/
│       ├── workspace.md            # Workspace metadata
│       ├── .view.json              # UI state (All Tasks ordering)
│       ├── docs/                   # Workspace-level docs (tree structure)
│       │   ├── folder/
│       │   └── doc.md
│       ├── _unassigned/            # Items not assigned to a project
│       │   ├── tasks/*.md
│       │   └── docs/*.md
│       └── projects/
│           └── {project-id}/
│               ├── project.md      # Project metadata
│               ├── .view.json      # UI state (task order)
│               ├── tasks/*.md
│               ├── docs/           # Project docs (tree structure)
│               │   ├── folder/
│               │   └── doc.md
│               └── meetings/*.md
```

## Path Constants

All paths are centralized in `src/lib/orbit/constants.ts`:

```typescript
export const PATH_SEGMENTS = {
  WORKSPACES: "workspaces",
  PERSONAL: "personal",
  PROJECTS: "projects",
  TASKS: "tasks",
  DOCS: "docs",
  MEETINGS: "meetings",
  CAPTURE: "capture",
} as const;

export const SPECIAL_DIRS = {
  UNASSIGNED: "_unassigned",
} as const;

export const FILE_NAMES = {
  WORKSPACE_MD: "workspace.md",
  PROJECT_MD: "project.md",
  VIEW_STATE: ".view.json",
} as const;

export const PERSONAL_SPACE_ID = "__personal__" as const;
```

**All library files MUST use these constants** - never hardcode path strings.

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

- **TanStack Query**: Server state (tasks, projects, docs, meetings)
- **Zustand**: Client state (settings, current workspace)

### Content vs Display Data

| Type | Format | Location | Purpose |
|------|--------|----------|---------|
| Content data | Markdown + YAML frontmatter | `*.md` files | Tasks, docs, projects |
| Global config | JSON | `~/Orbit/config.json` | App settings |
| View state | JSON | `.view.json` per folder | UI preferences (task order, expanded folders) |

**Key principle:** Content data belongs in markdown (portable). UI preferences belong in `.view.json` (can be regenerated).

### Custom Scrollbars

Uses **OverlayScrollbars** instead of native scrollbars for consistent styling across Chrome and Tauri (WKWebView on macOS, which doesn't support `::-webkit-scrollbar`).

- `ScrollArea` component wraps scrollable content
- Theme defined in `globals.css` as `.os-theme-orbit`
- Always visible (`autoHide: "never"`)
- Popup components (Select, DropdownMenu, Command) use native scroll (intentional)

## Module Structure

### Core Data Layer (`src/lib/orbit/`)

| File | Purpose |
|------|---------|
| `tauri-fs.ts` | File system abstraction (Tauri/mock) |
| `parser.ts` | Markdown frontmatter parsing, date helpers |
| `constants.ts` | Path segments, special dirs, file names |
| `workspaces.ts` | Workspace CRUD operations |
| `projects.ts` | Project CRUD operations |
| `tasks.ts` | Task CRUD operations |
| `docs.ts` | Doc CRUD + tree operations (folders, import) |
| `meetings.ts` | Meeting CRUD operations |
| `personal.ts` | Personal space CRUD (capture, tasks, docs) |
| `dashboard.ts` | Cross-workspace data aggregation |
| `search.ts` | Cross-workspace search helpers |
| `search-index.ts` | In-memory Fuse.js search index |
| `watcher.ts` | File system watcher service (Tauri) |
| `calculations.ts` | Business logic (task stats, grouping) |

### Design System (`src/lib/`)

| File | Purpose |
|------|---------|
| `design-tokens.ts` | Colors for status, priority, workspaces |
| `utils.ts` | Utility functions (cn for classnames) |

### State Management (`src/stores/`)

- React Query hooks for server state
- Zustand stores for client state
- Grouped task helpers

### Hooks (`src/hooks/`)

| File | Purpose |
|------|---------|
| `use-auto-save.ts` | Debounced auto-save with error-only status |
| `use-file-watcher.ts` | React hook for file watcher integration |
| `use-search-index.ts` | React hook for search index management |
| `use-view-mode.ts` | Persisted view mode (kanban/list) |

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

## Docs System

Docs support a hierarchical folder structure with unlimited nesting:

```typescript
type DocScope = 'personal' | 'workspace' | 'project';

interface DocTreeNode =
  | { type: 'folder'; folder: DocFolder }
  | { type: 'doc'; doc: Doc };

interface DocFolder {
  name: string;
  path: string;
  children: DocTreeNode[];
}
```

### Doc Scopes

| Scope | Route | Data Path |
|-------|-------|-----------|
| Personal | `/personal/docs` | `personal/docs/` |
| Workspace | `/docs` (Workspace tab) | `workspaces/{id}/docs/` |
| All Projects | `/docs` (All tab) | Aggregate view |
| Project | `/projects/view?id=xxx` | `workspaces/{id}/projects/{id}/docs/` |

### Future: AI Context

Docs can be flagged for AI context in the future. This is not yet implemented, but the plan is:
- Any doc can be marked as "AI context"
- AI features will read flagged docs when generating content
- No separate "context" folder - just metadata on docs

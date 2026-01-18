# Orbit Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                              ORBIT                                  │
│                        (Tauri Desktop App)                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│  │   Areas     │  │  Projects   │  │   Tasks     │  │   Notes   │  │
│  │   Module    │  │   Module    │  │   Module    │  │   Module  │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └─────┬─────┘  │
│         │                │                │                │        │
│         ▼                ▼                ▼                ▼        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Next.js App Router                        │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │   │
│  │  │ /        │  │ /tasks   │  │ /project │  │ /settings    │ │   │
│  │  │ (dashboard)│ │ (kanban) │  │ /[id]    │  │              │ │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────┘ │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    API Routes (Node.js)                      │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐ │   │
│  │  │ /api/     │  │ /api/     │  │ /api/     │  │ /api/     │ │   │
│  │  │  areas    │  │  projects │  │  tasks    │  │  notes    │ │   │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘ │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
└──────────────────────────────┼──────────────────────────────────────┘
                               │
                               ▼
                    ┌─────────────────┐
                    │   ~/Orbit/      │
                    │   (Local FS)    │
                    │                 │
                    │  Markdown files │
                    │  + config.json  │
                    └─────────────────┘
```

## Directory Structure

```
orbit/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── layout.tsx                # Root layout (area switcher)
│   │   ├── page.tsx                  # Dashboard / All Tasks
│   │   ├── tasks/
│   │   │   └── page.tsx              # Kanban board (current area)
│   │   ├── projects/
│   │   │   ├── page.tsx              # Project list
│   │   │   └── [id]/
│   │   │       ├── page.tsx          # Project overview
│   │   │       ├── tasks/page.tsx    # Project tasks
│   │   │       ├── notes/page.tsx    # Project notes
│   │   │       └── context/page.tsx  # Project AI context
│   │   ├── inbox/
│   │   │   └── page.tsx              # Unassigned tasks
│   │   ├── settings/
│   │   │   └── page.tsx              # App settings
│   │   ├── api/
│   │   │   ├── areas/
│   │   │   │   └── route.ts          # CRUD areas
│   │   │   ├── projects/
│   │   │   │   ├── route.ts          # List/create projects
│   │   │   │   └── [id]/route.ts     # Get/update/delete project
│   │   │   ├── tasks/
│   │   │   │   ├── route.ts          # List/create tasks
│   │   │   │   └── [id]/route.ts     # Get/update/delete task
│   │   │   └── notes/
│   │   │       ├── route.ts          # List/create notes
│   │   │       └── [id]/route.ts     # Get/update/delete note
│   │   └── globals.css
│   │
│   ├── components/
│   │   ├── ui/                       # shadcn/ui components
│   │   ├── layout/
│   │   │   ├── sidebar.tsx           # Main navigation
│   │   │   ├── area-switcher.tsx     # Area dropdown
│   │   │   └── header.tsx            # Top bar
│   │   ├── kanban/
│   │   │   ├── board.tsx             # Kanban container
│   │   │   ├── column.tsx            # Status column
│   │   │   └── card.tsx              # Task card
│   │   ├── tasks/
│   │   │   ├── task-form.tsx         # Create/edit task
│   │   │   └── task-quick-add.tsx    # Quick add modal
│   │   ├── projects/
│   │   │   ├── project-card.tsx      # Project in list
│   │   │   └── project-form.tsx      # Create/edit project
│   │   ├── notes/
│   │   │   ├── note-list.tsx         # Notes list
│   │   │   └── note-editor.tsx       # Markdown editor
│   │   └── context/
│   │       ├── file-tree.tsx         # Context folder view
│   │       └── file-viewer.tsx       # View/edit context files
│   │
│   ├── lib/
│   │   ├── orbit/
│   │   │   ├── areas.ts              # Area file operations
│   │   │   ├── projects.ts           # Project file operations
│   │   │   ├── tasks.ts              # Task file operations
│   │   │   ├── notes.ts              # Note file operations
│   │   │   └── parser.ts             # Markdown frontmatter parsing
│   │   ├── config.ts                 # App configuration
│   │   └── utils.ts                  # Shared utilities
│   │
│   ├── stores/
│   │   ├── area.ts                   # Current area state
│   │   ├── projects.ts               # Projects state
│   │   ├── tasks.ts                  # Tasks state
│   │   ├── notes.ts                  # Notes state
│   │   └── settings.ts               # App settings
│   │
│   └── types/
│       ├── area.ts
│       ├── project.ts
│       ├── task.ts
│       ├── note.ts
│       └── config.ts
│
├── src-tauri/                        # Tauri Rust backend
│   ├── src/
│   │   └── main.rs
│   ├── tauri.conf.json
│   └── Cargo.toml
│
├── docs/
└── public/
```

## Data Flow

### Reading Tasks (Example)

```
User opens app
       │
       ▼
┌──────────────────┐
│  Area Store      │  ← Reads current area from config
│  (currentArea)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  GET /api/tasks  │  ← Query: ?areaId=slsp
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  lib/orbit/      │  ← Reads ~/Orbit/areas/slsp/projects/*/tasks/*.md
│  tasks.ts        │  ← Parses frontmatter + content
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Tasks Store     │  ← Caches parsed tasks
│  (TanStack Query)│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Kanban Board    │  ← Groups by status, renders cards
└──────────────────┘
```

### Creating a Task

```
User clicks "Add Task"
       │
       ▼
┌──────────────────┐
│  Task Form       │  ← Title, project, priority
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  POST /api/tasks │  ← { title, projectId, areaId, ... }
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  lib/orbit/      │  ← Generates filename: {date}-{slug}.md
│  tasks.ts        │  ← Writes frontmatter + content
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  File System     │  ← ~/Orbit/areas/slsp/projects/slskey/tasks/2024-01-15-setup-webhook.md
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Invalidate      │  ← TanStack Query refetches
│  Cache           │
└──────────────────┘
```

## Key Design Decisions

### Why Local Markdown Files?

| Approach | Pros | Cons |
|----------|------|------|
| **Markdown files** | Portable, grep-able, works offline, version-controllable | Slower for large datasets |
| SQLite | Fast queries, relations | Binary format, harder to inspect |
| Cloud DB | Sync, collaboration | Requires internet, privacy concerns |

**Decision**: Markdown files. The use case (freelancer, ~100s of tasks) doesn't need database performance, and portability matters more.

### Why Area as Top-Level?

Areas provide complete isolation between clients:
- Different projects, different tasks
- Future: different email configs, different integrations
- Switching areas = switching contexts entirely

### Why Next.js + Tauri?

- **Next.js**: Fast development with App Router, API routes in same codebase
- **Tauri**: Tiny binary (~15MB vs Electron's 150MB+), native performance
- **Both**: TypeScript throughout, familiar tooling

### File Watching

External edits (e.g., editing a markdown file directly) need to sync:

1. **Poll on focus** - Check file mtimes when window gains focus
2. **Manual refresh** - Refresh button in UI
3. **Future**: Tauri file watcher for real-time updates

## Future Extensions

### Email Integration (Planned)
```
areas/slsp/area.md:
  email:
    provider: microsoft365
    clientId: ...
    folder: offen
```

Each area can have its own email config. Email threads get assigned to projects.

### AI Context (Planned)
```
projects/slskey/context/
├── CLAUDE.md         # Main prompt
├── technical-docs/   # Knowledge files
└── archived-emails/  # Past conversations
```

Claude CLI reads context folder when generating responses.

### Integrations (Future)
- GitHub: Link repo to project, show PRs/issues
- Calendar: Show meetings related to project
- Slack: Archive relevant threads

# Desk Architecture

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
│  │         lib/desk/* (CRUD)            │              │
│  │  workspaces.ts  projects.ts  tasks.ts │              │
│  │  content.ts  meetings.ts  personal.ts │              │
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
            │   ~/Desk/      │
            │  (Markdown FS)  │
            └─────────────────┘
```

## Data Hierarchy

All content lives in workspaces, including Personal:

```
Workspace (Client or Personal)
├── Workspace-level Docs (shared across all projects)
├── _unassigned/              (items not assigned to a project)
├── _capture/                 (Personal only - triage inbox)
└── Projects
    └── Project
        ├── Tasks
        ├── Docs
        └── Meetings

Personal = workspace "_personal" (always first in list)
```

**"Work Mode" Navigation**: User selects active workspace via bottom selector. Tasks, Docs, Meetings views filter to that workspace automatically.

### Scopes Explained

| Scope | Description | Location |
|-------|-------------|----------|
| **Personal** | Private workspace (treated like any other) | `workspaces/_personal/` |
| **Workspace** | Shared docs for the whole workspace | `workspaces/{id}/docs/` |
| **Project** | Items belonging to a specific project | `workspaces/{id}/projects/{id}/` |
| **Unassigned** | Workspace items not yet assigned to a project | `workspaces/{id}/_unassigned/` |
| **Capture** | Quick triage inbox (Personal workspace only) | `workspaces/_personal/_capture/` |

## File Structure

```
~/Desk/
├── config.json                     # App settings (theme, current workspace)
├── workspaces/
│   ├── _personal/                  # Personal workspace (always first in list)
│   │   ├── workspace.md            # Personal workspace metadata
│   │   ├── .view.json              # UI state
│   │   ├── _capture/               # Triage inbox (Personal only)
│   │   │   └── tasks/*.md          # Quick capture items to triage
│   │   ├── _unassigned/            # Personal items without a project
│   │   │   ├── tasks/*.md
│   │   │   └── docs/*.md
│   │   ├── docs/                   # Personal docs (tree structure)
│   │   │   ├── folder/
│   │   │   └── doc.md
│   │   └── projects/               # Personal projects
│   │       └── {project-id}/
│   │           ├── project.md
│   │           ├── tasks/*.md
│   │           ├── docs/
│   │           └── meetings/*.md
│   └── {workspace-id}/             # Client workspaces
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

All paths are centralized in `src/lib/desk/constants.ts`:

```typescript
export const PATH_SEGMENTS = {
  WORKSPACES: "workspaces",
  PROJECTS: "projects",
  TASKS: "tasks",
  DOCS: "docs",
  MEETINGS: "meetings",
} as const;

export const SPECIAL_DIRS = {
  UNASSIGNED: "_unassigned",
  PERSONAL: "_personal",
  CAPTURE: "_capture",
} as const;

export const FILE_NAMES = {
  WORKSPACE_MD: "workspace.md",
  PROJECT_MD: "project.md",
  VIEW_STATE: ".view.json",
} as const;

export const PERSONAL_WORKSPACE_ID = "_personal" as const;

// Helper functions
export function isPersonalWorkspace(workspaceId: string | null): boolean;
export function isCapture(projectId: string): boolean;
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
| Global config | JSON | `~/Desk/config.json` | App settings |
| View state | JSON | `.view.json` per folder | UI preferences (task order, expanded folders) |

**Key principle:** Content data belongs in markdown (portable). UI preferences belong in `.view.json` (can be regenerated).

### Custom Scrollbars

Uses **OverlayScrollbars** instead of native scrollbars for consistent styling across Chrome and Tauri (WKWebView on macOS, which doesn't support `::-webkit-scrollbar`).

- `ScrollArea` component wraps scrollable content
- Theme defined in `globals.css` as `.os-theme-desk`
- Always visible (`autoHide: "never"`)
- Popup components (Select, DropdownMenu, Command) use native scroll (intentional)

## Module Structure

### Core Data Layer (`src/lib/desk/`)

| File | Purpose |
|------|---------|
| `tauri-fs.ts` | File system abstraction (Tauri/mock) |
| `parser.ts` | Markdown frontmatter parsing, date helpers |
| `constants.ts` | Path segments, special dirs, file names |
| `workspaces.ts` | Workspace CRUD operations |
| `projects.ts` | Project CRUD operations |
| `tasks.ts` | Task CRUD operations |
| `content.ts` | Doc/Asset CRUD + content tree operations (folders, import) |
| `meetings.ts` | Meeting CRUD operations |
| `personal.ts` | Capture inbox CRUD (triage to Personal or workspaces) |
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
| `use-editor-session.ts` | Editor state with manual save (Cmd+S), external change detection |
| `use-query-invalidator.ts` | Routes file watcher events to editors or TanStack Query |
| `use-window-close.ts` | Handles window close with unsaved changes protection |
| `use-search-index.ts` | React hook for search index management |
| `use-project-name.ts` | Project name lookup by ID |
| `use-open-from-query.ts` | Handle `?open=id` URL params |
| `use-grouped-items.ts` | Group items by a key function |
| `use-editor-tab.ts` | Manage editor tab title/dirty state |

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

## Content System

The content tree supports markdown docs (editable) and assets (non-markdown files that open externally):

```typescript
type ContentScope = 'personal' | 'workspace' | 'project';

type FileTreeNode =
  | { type: 'folder'; folder: ContentFolder }
  | { type: 'doc'; doc: Doc }
  | { type: 'asset'; asset: Asset };

interface ContentFolder {
  name: string;
  path: string;
  children: FileTreeNode[];
}

interface Asset {
  id: string;        // filename (display name)
  path: string;      // relative path
  filePath: string;  // absolute path
  extension: string; // e.g., "pdf", "png"
}
```

### Doc Scopes

| Scope | Route | Data Path |
|-------|-------|-----------|
| Personal | `/docs` (with Personal workspace selected) | `workspaces/_personal/docs/` |
| Workspace | `/docs` (Workspace tab) | `workspaces/{id}/docs/` |
| All Projects | `/docs` (All tab) | Aggregate view |
| Project | `/projects/view?id=xxx` | `workspaces/{id}/projects/{id}/docs/` |

### Future: AI Context

Docs can be flagged for AI context in the future. This is not yet implemented, but the plan is:
- Any doc can be marked as "AI context"
- AI features will read flagged docs when generating content
- No separate "context" folder - just metadata on docs

## File System Integration

The app uses a dual-layer architecture for file access: one for **closed files** (list views, sidebar) and one for **open files** (editor tabs).

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FILE SYSTEM (tauri-fs.ts)                       │
└─────────────────────────────────────────────────────────────────────────────┘
        │                                               │
        ▼                                               ▼
┌───────────────────┐                         ┌───────────────────┐
│   FILE WATCHER    │                         │ DOMAIN OPERATIONS │
│   (watcher.ts)    │                         │ (move, delete)    │
└───────────────────┘                         └───────────────────┘
        │                                               │
        ▼                                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        OPEN EDITOR REGISTRY                                  │
│                                                                              │
│  isOpen(path) ──► YES: Route to EditorEventBus → Editor updates             │
│              └──► NO:  Route to QueryInvalidator → TanStack refetch         │
└─────────────────────────────────────────────────────────────────────────────┘
        │                                       │
        ▼                                       ▼
┌─────────────────────────┐             ┌─────────────────────────┐
│   OPEN FILES (Editors)  │             │  CLOSED FILES (Lists)   │
│   useEditorSession()    │             │  FileTreeService cache  │
│   Direct writeTextFile  │             │  TanStack Query         │
└─────────────────────────┘             └─────────────────────────┘
```

### Key Principle

```
When file is OPEN in editor tab:
  → Editor owns state (useEditorSession hook)
  → File watcher routes changes to editor via EditorEventBus
  → Manual save with Cmd+S (no auto-save)
  → Metadata changes (status, priority, etc.) save immediately, passing body from editor
  → Dirty indicator shown in tab until saved
  → Unsaved changes prompt on tab close and app quit
  → Detects external changes via lastSavedContent comparison

When file is CLOSED:
  → TanStack Query owns state
  → FileTreeService provides cached content
  → File watcher invalidates queries for refetch
```

### Key Files

| File | Purpose |
|------|---------|
| `src/stores/open-editor-registry.ts` | Zustand store tracking all open editor sessions by path |
| `src/stores/editor-event-bus.ts` | Pub/sub for routing external changes to editors |
| `src/hooks/use-editor-session.ts` | Hook for editor state with manual save (Cmd+S) |
| `src/hooks/use-query-invalidator.ts` | Routes file watcher events to editors or TanStack |
| `src/lib/desk/file-cache/` | Cached file tree for list views (LRU cache, 50MB limit) |
| `src/lib/desk/watcher.ts` | Tauri file system watcher |

### Editor State Management

Editors (DocEditor, TaskEditor, MeetingEditor) use `useEditorSession()`:

```typescript
const {
  content,           // Current editor content
  setContent,        // Update content (marks dirty, no auto-save)
  getCurrentContent, // Get current content for metadata saves
  isDirty,           // Has unsaved changes
  saveStatus,        // "idle" | "saving" | "error"
  pathChanged,       // File was moved/renamed externally
  newPath,           // New path after move
  fileDeleted,       // File was deleted externally
  acknowledgePathChange,  // Accept new path
  acknowledgeDeleted,     // Accept deletion
  save,              // Save to disk (called via Cmd+S)
} = useEditorSession({
  type: "doc",       // "doc" | "task" | "meeting"
  entityId: docId,
  filePath: doc?.filePath,
  initialContent: "",
  enabled: !!doc,
  onSaveComplete,    // Callback after save (e.g., for RAG indexing)
});
```

**Save behavior:**
- Content saves only on explicit Cmd+S (or clicking save button)
- Metadata changes (status, priority, due date, title) save immediately
- When saving metadata, pass `content: getCurrentContent()` to preserve unsaved body changes
- Tab shows dirty indicator (dot) when there are unsaved changes
- Closing dirty tab shows "Save/Don't Save/Cancel" dialog
- Quitting app with dirty tabs shows confirmation dialog

### How External Changes Are Detected

1. File watcher detects change
2. `useQueryInvalidator` checks `OpenEditorRegistry.isOpen(path)`
3. If open: read file, compare with `lastSavedContent`
   - Matches → our save, ignore
   - Different → external change, publish via `EditorEventBus`
4. If closed: invalidate TanStack Query

### Path Safety in Domain Operations

When moving/deleting files, domain operations notify open editors:

```typescript
// In tasks.ts, content.ts, meetings.ts
import { useOpenEditorRegistry } from "@/stores/open-editor-registry";
import { publishPathChange, publishDeleted } from "@/stores/editor-event-bus";

// Before delete
if (registry.isOpen(filePath)) {
  registry.handlePathDeleted(filePath);
  publishDeleted(filePath);
}

// After move
if (registry.isOpen(oldPath)) {
  registry.handlePathChange(oldPath, newPath);
  publishPathChange(oldPath, newPath);
}
```

### Debugging File System Issues

| Symptom | Likely Cause | Check |
|---------|--------------|-------|
| Content lost after metadata change | Body not passed to mutation | Ensure `content: getCurrentContent()` passed to update mutation |
| External edits not showing | Event bus not connected | Check subscribeToEditorEvents call |
| File move breaks editor | Path safety missing | Check domain operation notifies registry |
| Infinite re-renders | Zustand dependency loop | Use `getState()` not hook in effects |
| Stale list data | Cache not invalidated | Check watcher → QueryInvalidator flow |
| Dirty indicator not clearing | Save callback not updating state | Check save() clears isDirty |
| Close dialog not showing | Tab not marked dirty | Check setTabDirty called when content changes |

### Important Implementation Notes

1. **Editors write directly** to disk via `writeTextFile()`, NOT through FileTreeService
2. **lastSavedContent** must be updated:
   - After loading content from disk in `useEditorSession`
   - After explicit save via Cmd+S
3. **getRegistry()** pattern avoids Zustand re-render loops in effects
4. **Save behavior**:
   - Content: Manual save via Cmd+S only
   - Metadata: Saves immediately, passing `content: getCurrentContent()` to preserve editor state
   - No auto-save, no debounce - simpler and more predictable
5. **Dirty tab protection**:
   - Tab bar checks `isDirty` before closing, shows SaveChangesDialog
   - Window close intercepted by Rust, emits event to check for dirty tabs
   - WindowCloseProvider in providers.tsx handles the confirmation dialog
6. **FileMovedBanner/FileDeletedBanner** handle external file changes gracefully

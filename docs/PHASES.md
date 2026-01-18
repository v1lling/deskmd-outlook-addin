# Orbit Implementation Phases

## Overview

The implementation is divided into 5 phases, building from foundation to full feature set. Each phase results in a usable (if incomplete) application.

**Current Status**: Phases 1-5 complete (UI with mock data). Next: file system integration via Tauri.

---

## Phase 1: Foundation ✅

**Goal**: Project setup with basic layout, area switching, and settings.

### Tasks

- [x] **1.1 Initialize Next.js project**
  ```bash
  npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir
  ```

- [x] **1.2 Add Tauri**
  ```bash
  npm install -D @tauri-apps/cli
  npx tauri init
  ```

- [x] **1.3 Install core dependencies**
  ```bash
  npm install zustand @tanstack/react-query gray-matter
  npm install clsx tailwind-merge lucide-react date-fns
  npx shadcn@latest init
  npx shadcn@latest add button card dialog dropdown-menu input label
  ```

- [x] **1.4 Set up directory structure**
  ```
  src/
  ├── app/
  ├── components/ui/
  ├── components/layout/
  ├── lib/orbit/
  ├── stores/
  └── types/
  ```

- [x] **1.5 Build layout components**
  - Sidebar with navigation
  - Area switcher dropdown
  - Header bar
  - Main content area

- [x] **1.6 Implement settings store**
  - Define config types
  - Zustand store for settings
  - Settings page UI (Theme toggle, data path display)
  - Theme persistence (light/dark/system)

- [x] **1.7 Implement area store**
  - List areas (mock data)
  - Current area selection
  - Area switcher component

- [ ] **1.8 First-run setup wizard** (Deferred)
  - Detect if `~/Orbit/` exists
  - Welcome screen
  - Create data folder
  - Create first area
  - Save config

### Deliverable
App shell with sidebar, area switcher, and settings. Can switch between areas.

---

## Phase 2: Tasks & Kanban ✅

**Goal**: Full task management with kanban board.

### Tasks

- [x] **2.1 Build markdown parser library**
  - `lib/orbit/parser.ts`: Parse frontmatter with gray-matter
  - Generic parse/write functions for all entity types

- [x] **2.2 Build task library**
  - `lib/orbit/tasks.ts`: CRUD operations (mock data)
  - Generate filename pattern defined

- [ ] **2.3 Create task API routes** (Not needed - direct lib calls)

- [x] **2.4 Build tasks store**
  - TanStack Query for fetching
  - Filter by project, status
  - Optimistic updates for drag-drop

- [x] **2.5 Install dnd-kit**
  ```bash
  npm install @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
  ```

- [x] **2.6 Build kanban components**
  - `components/tasks/kanban-board.tsx`
  - `components/tasks/kanban-column.tsx`
  - `components/tasks/task-card.tsx`

- [x] **2.7 Implement drag-and-drop**
  - Drag between columns updates status
  - Visual feedback during drag
  - Optimistic update

- [x] **2.8 Build task detail panel**
  - Sheet slide-over
  - Edit all task fields
  - Save/delete with toast notifications

- [x] **2.9 Build quick add modal**
  - Quick add task dialog
  - Title, project, priority, due date, notes

- [ ] **2.10 Build inbox view** (Deferred)

### Deliverable
Working kanban board with drag-and-drop. Can create, edit, drag tasks.

---

## Phase 3: Projects ✅

**Goal**: Project management with overview and detail views.

### Tasks

- [x] **3.1 Build project library**
  - `lib/orbit/projects.ts`: CRUD operations (mock data)

- [ ] **3.2 Create project API routes** (Not needed - direct lib calls)

- [x] **3.3 Build projects store**
  - TanStack Query
  - Task counts per project

- [x] **3.4 Build projects list page**
  - Card grid layout
  - Status badges
  - Task counts
  - Filter by status

- [x] **3.5 Build project detail page**
  - Tabs: Overview, Tasks, Notes
  - Overview with stats and description
  - Tasks tab = filtered kanban
  - Notes tab = project notes

- [x] **3.6 Build create project form**
  - Name, description, status
  - New project modal

- [x] **3.7 Connect tasks to projects**
  - Project dropdown in task form
  - Filter kanban by project

### Deliverable
Projects page with list and detail views. Tasks linked to projects.

---

## Phase 4: Notes ✅

**Goal**: Project notes for meeting logs and documentation.

### Tasks

- [x] **4.1 Build notes library**
  - `lib/orbit/notes.ts`: CRUD operations (mock data)

- [ ] **4.2 Create notes API routes** (Not needed - direct lib calls)

- [x] **4.3 Build notes store**
  - TanStack Query
  - Project filter support

- [x] **4.4 Build notes list component**
  - Card-based list
  - Title, date, preview
  - Project grouping
  - Project filter dropdown

- [x] **4.5 Build note editor**
  - Sheet slide-over with Edit/Preview tabs
  - Title input
  - Markdown preview with proper rendering
  - Save/delete with toast notifications

- [x] **4.6 Add Notes tab to project detail**
  - List project notes
  - Create new note
  - Open editor

- [ ] **4.7 Quick note shortcut** (Deferred with keyboard shortcuts)

### Deliverable
Notes system integrated into project detail. Notes page with filtering. Can create and edit notes.

---

## Phase 5: Polish & Settings ✅

**Goal**: Settings, theme support, and UI polish.

### Tasks

- [ ] **5.1 Build context file browser** (Deferred to Phase 6)
- [ ] **5.2 Build context file viewer/editor** (Deferred to Phase 6)
- [ ] **5.3 Add Context tab to project detail** (Deferred to Phase 6)

- [ ] **5.4 Implement keyboard shortcuts** (Deferred)

- [ ] **5.5 Add file watching** (Requires Tauri integration)

- [x] **5.6 Theme polish**
  - Dark/light/system mode with persistence
  - ThemeProvider with system preference detection
  - Loading states
  - Empty states

- [x] **5.7 Toast notifications**
  - Sonner integration
  - Success/error toasts for all mutations

- [ ] **5.8 Tauri configuration** (Next phase)
- [ ] **5.9 Build and test** (Next phase)

### Deliverable
Settings page with theme toggle. Toast notifications throughout. UI polish.

---

## Phase Summary

| Phase | Focus | Status |
|-------|-------|--------|
| 1 | Foundation | ✅ Complete |
| 2 | Tasks | ✅ Complete |
| 3 | Projects | ✅ Complete |
| 4 | Notes | ✅ Complete |
| 5 | Polish & Settings | ✅ Complete |

---

## What's Done

- Full app shell with sidebar navigation
- Area switcher (mock data)
- Projects list with filtering and stats
- Project detail page with tabs (Overview, Tasks, Notes)
- Kanban board with drag-and-drop
- Task detail panel with full editing
- Quick add task modal
- Notes list page with project filtering
- Note editor with markdown preview
- Settings page with theme toggle
- Toast notifications (sonner)
- Dark/light/system theme support
- TanStack Query for all data fetching
- Zustand for settings persistence

## What's Next

### Phase 6: Tauri Integration
- Replace mock data with file system operations
- Read/write markdown files via Tauri
- First-run setup wizard
- File watching for external changes
- Production build

### Phase 7: Context & AI
- Context file browser per project
- CLAUDE.md support
- AI integration (future)

### Phase 8: Advanced Features
- Keyboard shortcuts
- Global search
- Inbox view
- Email integration

---

## Dependencies

```
Phase 1 (Foundation) ✅
    │
    ▼
Phase 2 (Tasks & Kanban) ✅
    │
    ▼
Phase 3 (Projects) ✅
    │
    ▼
Phase 4 (Notes) ✅
    │
    ▼
Phase 5 (Polish & Settings) ✅
    │
    ▼
Phase 6 (Tauri Integration) ← NEXT
```

---

## Future Phases (Post-MVP)

### Phase 7: Email Integration
- Per-area email config
- Microsoft Graph API for SLSP
- IMAP for SSS
- Email list in sidebar
- Assign email to project
- Archive to context

### Phase 8: AI Integration
- Claude CLI wrapper
- Load project context automatically
- Generate email drafts
- Refinement loop

### Phase 9: Advanced Features
- Command palette (`Cmd+K`)
- Global search
- Templates
- GitHub integration
- Calendar integration

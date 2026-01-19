# Orbit Restructure Plan

## Completed

### Phase 1: Rename Areas → Workspaces
- [x] Rename `Area` type to `Workspace`
- [x] Update all `areaId` → `workspaceId`
- [x] Rename file paths `areas/` → `workspaces/`
- [x] Update `area.md` → `workspace.md`
- [x] Update all stores, hooks, components
- [x] Update documentation

### Phase 2: Personal Space
- [x] Add `PERSONAL` and `INBOX` constants to `PATH_SEGMENTS`
- [x] Create `src/lib/orbit/personal.ts` with CRUD operations
- [x] Create `src/stores/personal.ts` with TanStack Query hooks
- [x] Add personal routes (`/personal/inbox`, `/personal/tasks`, `/personal/notes`)
- [x] Update sidebar with Personal section above workspace navigation
- [x] Initialize personal directory in `tauri-fs.ts`
- [x] Add mock data for browser development mode

---

## Phase 3: Dashboard

**Goal**: Add a dedicated "Personal" section in the sidebar for quick capture and personal items that don't belong to any workspace/client.

### Concept

```
Sidebar:
┌─────────────────────┐
│ [Workspace Switcher]│
├─────────────────────┤
│ Personal            │  ← New section
│   ├─ Inbox          │  ← Quick capture
│   ├─ Notes          │  ← Personal notes
│   └─ Tasks          │  ← Personal tasks
├─────────────────────┤
│ Workspace: Acme     │
│   ├─ Projects       │
│   ├─ All Tasks      │
│   ├─ Notes          │
│   └─ Meetings       │
└─────────────────────┘
```

### File Structure

```
~/Orbit/
├── personal/                    # New: Personal space (no workspace)
│   ├── inbox/                   # Quick capture items
│   │   └── tasks/*.md
│   ├── tasks/*.md               # Personal tasks
│   ├── notes/*.md               # Personal notes
│   └── .view.json
├── workspaces/
│   └── {workspace}/...
└── config.json
```

### Features

1. **Inbox**: Quick capture for tasks that need to be triaged later
   - No project assignment required
   - Can be moved to any workspace/project later

2. **Personal Tasks**: Tasks not tied to any client work
   - Standalone task list (not Kanban - simpler view)

3. **Personal Notes**: Quick notes, scratchpad
   - Same editor as workspace notes

### Implementation

1. **New routes**:
   - `/personal` - Personal space home
   - `/personal/inbox` - Inbox view
   - `/personal/tasks` - Personal tasks
   - `/personal/notes` - Personal notes

2. **New lib functions**:
   - `src/lib/orbit/personal.ts` - Personal space CRUD

3. **Sidebar changes**:
   - Add "Personal" section above workspace content
   - Different styling to distinguish from workspace

4. **Navigation**:
   - Personal items always accessible regardless of selected workspace

---

## Phase 3: Dashboard

**Goal**: Cross-workspace overview showing aggregated data from all workspaces.

### Concept

```
Dashboard:
┌─────────────────────────────────────────┐
│ Today's Focus                           │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐    │
│ │ Task 1  │ │ Task 2  │ │ Task 3  │    │
│ │ (Acme)  │ │ (Beta)  │ │ (Pers)  │    │
│ └─────────┘ └─────────┘ └─────────┘    │
├─────────────────────────────────────────┤
│ Upcoming                                │
│ • Meeting with Acme (Tomorrow 2pm)      │
│ • Project deadline (Friday)             │
├─────────────────────────────────────────┤
│ Recent Notes                            │
│ • Call notes - Acme                     │
│ • Ideas for Beta project                │
└─────────────────────────────────────────┘
```

### Features

1. **Today's Focus**: Tasks due today or in "doing" status across all workspaces
2. **Upcoming**: Meetings and deadlines from all workspaces
3. **Recent Activity**: Recently modified notes/tasks
4. **Quick Actions**: Create task, create note (with workspace picker)

### Implementation

1. **New route**: `/dashboard`

2. **New components**:
   - `DashboardPage`
   - `TodaysFocus`
   - `UpcomingSection`
   - `RecentActivity`

3. **Navigation**:
   - Dashboard as top-level nav item (above Personal)
   - Or: Dashboard as default home when no workspace selected

---

## Phase 4: Navigation Restructure

**Goal**: Clean up sidebar navigation for the new structure.

### New Sidebar Layout

```
┌─────────────────────┐
│ ☰ Orbit             │
├─────────────────────┤
│ ◉ Dashboard         │  ← Cross-workspace view
│ ◉ Personal          │  ← Personal space
│   ├─ Inbox          │
│   ├─ Tasks          │
│   └─ Notes          │
├─────────────────────┤
│ WORKSPACES          │  ← Section header
│ ◉ Acme Corp    [●]  │  ← Color indicator
│ ◉ Beta Inc     [●]  │
│ + New Workspace     │
├─────────────────────┤
│ [Selected: Acme]    │  ← Current workspace context
│ ◉ Projects          │
│ ◉ All Tasks         │
│ ◉ Notes             │
│ ◉ Meetings          │
├─────────────────────┤
│ ⚙ Settings          │
└─────────────────────┘
```

### Changes

1. Remove workspace switcher dropdown
2. Show all workspaces in sidebar (collapsed by default)
3. Click workspace to expand/select
4. Personal always visible at top
5. Dashboard as entry point

---

## Priority Order

1. **Phase 2: Personal Space** - Most useful for daily workflow
2. **Phase 4: Navigation** - Better UX for switching contexts
3. **Phase 3: Dashboard** - Nice to have, lower priority

---

## Notes

- Each phase should be a separate PR
- Keep backward compatibility during transitions
- Test in both browser (mock) and Tauri (real fs) modes

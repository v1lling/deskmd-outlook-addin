# UX Audit: Navigation & Information Architecture

> Analysis of current app structure and proposed "Work Mode" redesign.

## Current Mental Model

```
Workspace (including Personal as "_personal")
├── Tasks           ← Aggregates project tasks + unassigned
├── Docs            ← Workspace-level docs
├── Meetings        ← Aggregates project meetings
├── _capture/       ← Triage inbox (Personal workspace only)
├── _unassigned/    ← Items not yet assigned to a project
└── Projects
    └── Project X
        ├── Tasks
        ├── Docs
        └── Meetings
```

**Key insight**: Personal is now a workspace (`_personal`) with the same structure as client workspaces. It can have projects, and includes a special `_capture` area for quick triage.

---

## Problems Identified

### 1. Deep Nesting Creates "Where Am I?" Confusion

To get to a project's tasks, a user must:
1. Find the workspace in sidebar
2. Click to select it
3. Expand the Projects section
4. Click the project
5. Then see tabs for Tasks/Docs/Meetings

That's **5 steps** to reach what might be the most common destination.

### 2. "All Tasks" vs "Tasks" Naming is Misleading

| Location | Label | Actually Shows |
|----------|-------|----------------|
| Workspace level | "All Tasks" | Workspace-scoped tasks only |
| Personal level | "Tasks" | Personal tasks only |
| Project level | "Tasks" tab | Project tasks only |

A user might think "All Tasks" means *all* their tasks across everything.

### 3. Workspace-Level vs Project-Level Content Split

| Content | Workspace Level | Project Level |
|---------|-----------------|---------------|
| Tasks | "All Tasks" (aggregation) | Yes |
| Docs | Yes (shared) | Yes (project-specific) |
| Meetings | Yes (shared) | Yes (project-specific) |

**User question**: "Should I put this doc at workspace level or project level?"

No clear guidance, creates cognitive overhead.

### 4. Implicit Workspace Context

When clicking "All Tasks" under Workspace A, the page shows Workspace A's tasks. If then clicking Workspace B in sidebar (without clicking "All Tasks" again), **the page content changes silently**. URL stays `/tasks` but content differs.

Violates the principle of least surprise.

### 5. Projects Are Hidden

Projects are the **atomic unit of work** for a freelancer, yet they're buried in a collapsible subsection.

Dashboard shows:
- Capture widget
- Focus widget (in-progress tasks)
- Workspaces widget

But **no Projects widget**. Can't see projects at a glance.

### 6. Tab System + Sidebar = Two Navigation Models

The Desk tab tries to bridge this, but creates confusion:
- Sidebar navigates pages
- Clicking items opens editor tabs
- "Where am I?" becomes "What tab am I on?" + "What page is in the Desk tab?"

---

## Real-World Usage Analysis

Based on actual data from `~/DeskMD`:

### User Profile: Multi-Client Freelancer

| Workspace | Projects | Unassigned Tasks | Pattern |
|-----------|----------|------------------|---------|
| SLSP | 20 | 78 | Heavy workload, needs workspace filtering |
| Savignano | 1 | 0 | Simple, focused |
| MKG Göbel | 2 | 0 | Simple, focused |

### Key Observations

1. **Workspace-level organization IS needed** for the heavy client (SLSP)
   - 78 unassigned tasks = cross-project coordination work
   - Can't eliminate workspace-level task view

2. **Workspace-level docs are used** but sparingly (3 total)
   - Contracts, organization docs, client-wide references
   - Must keep three doc scopes: personal, workspace, project

3. **"Work mode" is natural** - user works for one client most of each day
   - Monday = SLSP, Tuesday = Savignano
   - Wants clean context switching, not constant navigation

4. **Personal space is reference library** (41 files)
   - Health research, planning, admin
   - Separate from work contexts

### Why "Project-First" Doesn't Fit

The original proposal to flip to project-first navigation assumes all freelancers think in projects. But:

- With 20+ projects for one client, you need workspace grouping
- Cross-project tasks (unassigned) are essential for coordination
- Client context matters for docs (contracts, templates)

**Better approach**: Keep workspaces, but make context switching explicit and friction-free.

---

## Proposed Redesign: "Work Mode" Navigation

**Core principle**: Explicit workspace context with global views that filter automatically.

### New Sidebar Structure

```
┌──────────────────────────────────┐
│  🔍 Search...              ⌘K   │
├──────────────────────────────────┤
│                                  │
│  Dashboard                       │
│                                  │
│  ─────────────────────────────── │
│                                  │
│  Tasks                    (78)   │  ← Filtered by active workspace
│  Docs                     (23)   │  ← Filtered by active workspace
│  Meetings                 (12)   │  ← Filtered by active workspace
│                                  │
│  ─────────────────────────────── │
│  PROJECTS                        │
│  + New Project                   │
│  ─────────────────────────────── │
│  • API Redesign           (15)   │  ← Active workspace's projects
│  • Billing System          (8)   │
│  • Data Migration         (34)   │
│  • Documentation           (3)   │
│  • Mobile App             (18)   │
│                                  │
├──────────────────────────────────┤
│  ⚙️ Settings                     │
├──────────────────────────────────┤
│  ┌────────────────────────────┐  │
│  │ ● SLSP                  ▼ │  │  ← Workspace selector (includes Personal)
│  └────────────────────────────┘  │
│  Work Mode                       │
└──────────────────────────────────┘
```

**Note**: Personal is now a workspace option in the selector dropdown (always first), not a separate sidebar section.

### Key Changes

| Current | Proposed | Why |
|---------|----------|-----|
| Workspaces in mid-sidebar, expandable | Workspace selector at bottom | Clear "work mode" mental model |
| Tasks/Docs/Meetings nested under workspace | Global views at top level | Reduces clicks, explicit filtering |
| Projects buried in collapsible section | Projects listed directly in sidebar | Primary work unit is visible |
| Implicit workspace context | Explicit badge in page headers | No silent content changes |
| "All Tasks" naming | Just "Tasks" (context is explicit) | Less confusion |

### Click Count Comparison

| Action | Current | Proposed |
|--------|---------|----------|
| View project tasks | 5 clicks | 2 clicks (sidebar → project) |
| View workspace tasks | 3 clicks | 1 click (sidebar → Tasks) |
| Switch workspaces | 1 click | 1 click (selector dropdown) |

---

## Workspace Selector ("Work Mode")

### Design

At bottom of sidebar, always visible:

```
┌────────────────────────────┐
│ ● SLSP                  ▼ │
└────────────────────────────┘
Work Mode
```

### Behavior

1. **Click** opens dropdown with all workspaces (showing colors)
2. **Select** a workspace:
   - Sets active workspace (persisted)
   - Tasks/Docs/Meetings views filter automatically
   - Projects list updates to show that workspace's projects
3. **Visual**: Workspace color dot always visible
4. **Personal** is a workspace (`_personal`) - first option in dropdown with indigo color

### State Management

Uses existing `useSettingsStore.currentWorkspaceId` - no new stores needed.

---

## Page Headers: Explicit Context

All workspace-filtered pages show the active workspace:

```
┌─────────────────────────────────────────┐
│ ● Tasks                         SLSP   │
│   ─────────────────────────────────────│
│   [Filter dropdown] [View toggle]      │
└─────────────────────────────────────────┘
```

- Workspace color dot
- Workspace name as badge
- User always knows what context they're viewing

---

## Doc Scopes (Unified)

| Scope | Access Point | Use Case |
|-------|--------------|----------|
| Personal | Select Personal workspace → Docs | Private notes, research, planning |
| Workspace | Global Docs → "Workspace" scope | Contracts, client templates, shared refs |
| Project | Project detail → Docs tab | Project-specific documentation |

The Docs page keeps its scope dropdown:
- `{Workspace Name} (Workspace)` - workspace-level docs
- Individual project names - project-level docs
- "All" - combined view

**Note**: Personal docs are now accessed by selecting the Personal workspace in the Work Mode selector, then viewing Docs. Same pattern as any other workspace.

---

## Long Projects List Handling

For workspaces with many projects (SLSP has 20):

1. **Scrollable section** with max-height
2. **Optional**: Search/filter within projects section
3. **Optional**: Collapse inactive projects or group by status

---

## Implementation Phases

### Phase 1: Sidebar Restructure ✅
- Create `WorkspaceSelector` component at bottom
- Move Tasks/Docs/Meetings to top as "Global Views"
- List projects directly (remove workspace nesting)

**Files**: `src/components/layout/sidebar.tsx`, new `workspace-selector.tsx`

### Phase 2: Explicit Context Headers ✅
- Add workspace badge to Tasks, Docs, Meetings pages
- Use workspace color for visual consistency

**Files**: `src/app/tasks/page.tsx`, `src/app/docs/client.tsx`, `src/app/meetings/page.tsx`

### Phase 3: Meetings Aggregation ✅
- Update Meetings to aggregate workspace + project meetings
- Match how Tasks already works

**Files**: `src/stores/meetings.ts`, `src/lib/desk/meetings.ts`, `src/app/meetings/page.tsx`

### Phase 4: Polish ✅
- Default project detail to Tasks tab (most common use)
- Rename any remaining "All Tasks" labels
- Add task/doc counts to sidebar items

### Phase 5: Personal as Workspace ✅
- Treat Personal as a workspace (`_personal`) not separate entity
- Personal can have projects like other workspaces
- Capture remains as special triage inbox (`_capture`)
- Removed dedicated `/personal/` routes

**Files**: `src/lib/desk/workspaces.ts`, `src/lib/desk/personal.ts`, `src/stores/personal.ts`

---

## Decision: Option C (Work Mode)

This approach is better than the original options because:

| Option A (Project-First) | Option B (Iterative) | Option C (Work Mode) |
|--------------------------|----------------------|----------------------|
| Doesn't fit 20+ project clients | Band-aids, doesn't fix mental model | Explicit context, clean navigation |
| Loses workspace-level features | Still has implicit context issues | Keeps all three doc scopes |
| Major data restructure | No clear end state | Minimal route changes |

**Recommendation**: Implement Option C (Work Mode).

---

## Next Steps

- [x] Analyze real-world usage patterns
- [x] Design "Work Mode" navigation structure
- [x] Implement Phase 1: Sidebar restructure
- [x] Implement Phase 2: Explicit context headers
- [x] Implement Phase 3: Meetings aggregation
- [x] Implement Phase 4: Polish
- [x] Implement Phase 5: Personal as workspace
- [x] Update documentation (CLAUDE.md, ARCHITECTURE.md, FEATURES.md)
- [x] Update code comments for consistency
- [x] Migrate files: `~/DeskMD/personal/` → `~/DeskMD/workspaces/_personal/`
- [x] Test with actual workflow

---

## Technical Notes

### Files Modified

| File | Changes |
|------|---------|
| `src/components/layout/sidebar.tsx` | Major restructure - global views at top, projects list, workspace selector at bottom |
| `src/components/layout/workspace-selector.tsx` | New component for bottom "Work Mode" selector (includes Personal) |
| `src/components/layout/projects-list.tsx` | New component for current workspace's projects |
| `src/components/patterns/filtered-list-page.tsx` | Added workspace context header support |
| `src/components/patterns/page-header.tsx` | New reusable header with workspace badge |
| `src/app/tasks/page.tsx` | Uses FilteredListPage with workspace context |
| `src/app/docs/client.tsx` | Added workspace context header |
| `src/app/meetings/page.tsx` | Added workspace context header |
| `src/app/projects/view/client.tsx` | Default tab changed to "tasks" |
| `src/stores/meetings.ts` | Added `useMeetings()` for workspace-level aggregation |
| `src/lib/desk/meetings.ts` | Added `getMeetings()`, fixed unassigned meetings bug |
| `src/lib/desk/paths.ts` | New centralized path builders (DRY) |
| `src/lib/desk/workspaces.ts` | Added `PERSONAL_WORKSPACE` constant, Personal returned first in `getWorkspaces()` |
| `src/lib/desk/personal.ts` | Simplified to capture-only (triage inbox) |
| `src/stores/personal.ts` | Rewritten for capture-only: `captureKeys`, `useCaptureTasks()`, etc. |
| `src/stores/content.ts` | Fixed over-invalidation (now workspace-scoped) |
| `src/lib/desk/constants.ts` | Added `PERSONAL_WORKSPACE_ID`, `WORKSPACE_LEVEL_PROJECT_ID`, `isPersonalWorkspace()`, `isCapture()` |
| `src/lib/desk/content.ts` | Fixed `getContentBasePath` to use correct Personal path, uses constants |
| `src/lib/desk/view-state.ts` | Fixed Personal view state path, uses constants |
| `src/stores/view-state.ts` | Uses `WORKSPACE_LEVEL_PROJECT_ID` constant |
| `src/app/page.tsx` | Fixed FocusWidget and WorkspacesWidget to use workspace context switching |
| `src/components/tabs/tab-bar.tsx` | Removed dead code for old `/personal/` routes |
| `src/app/docs/client.tsx` | Uses `WORKSPACE_LEVEL_PROJECT_ID` constant |

### Bugs Fixed During Implementation

1. **Meetings didn't support unassigned** - `_unassigned` directory was explicitly excluded
2. **Content keys over-invalidated** - Used `contentKeys.all` instead of workspace-scoped
3. **Personal store hardcoded query key** - Used `["tasks", workspaceId]` instead of `taskKeys`
4. **Duplicated path construction** - 7+ instances of same path patterns across files
5. **Personal docs used old path** - `getContentBasePath` used `~/Desk/personal/` instead of `~/Desk/workspaces/_personal/`
6. **Personal view state used old path** - Same issue in `view-state.ts`
7. **Dashboard used wrong Personal ID** - Compared against `"__personal__"` instead of `"_personal"`
8. **Dashboard routed to deleted routes** - Referenced `/personal/tasks` which no longer exists
9. **Deprecated constant cleanup** - Removed `PERSONAL_SPACE_ID`, added `WORKSPACE_LEVEL_PROJECT_ID`

### Architecture Change: Personal as Workspace

Personal space is now a workspace (`_personal`) rather than a separate entity:

- **Before**: `~/Desk/personal/` with separate CRUD functions
- **After**: `~/Desk/workspaces/_personal/` using standard workspace stores

Benefits:
- Personal can have projects (same as client workspaces)
- Same task/doc/meeting stores work for Personal
- Simplified codebase - no dual-path logic
- Capture (`_capture`) remains as special triage inbox within Personal

### Removed

- `/src/app/personal/` routes (Personal accessed via WorkspaceSelector)
- `usePersonalTasks`, `useCreatePersonalTask`, etc. (use standard task hooks)
- `personalKeys` (renamed to `captureKeys` for capture-only)

### No Changes Needed

- Route structure (`/tasks`, `/docs`, `/meetings`, `/projects/view`)
- Data models (Task, Doc, Meeting types)
- `ContentScope` type or scoping logic
- Settings store (already has `currentWorkspaceId`)

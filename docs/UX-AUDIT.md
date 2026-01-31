# UX Audit: Navigation & Information Architecture

> Analysis of current app structure and proposed "Work Mode" redesign.

## Current Mental Model

```
Personal Space
├── Tasks
├── Docs

Workspace (e.g., "Client A")
├── All Tasks      ← Aggregates project tasks
├── Docs           ← Workspace-level only
├── Meetings       ← Workspace-level only
└── Projects
    └── Project X
        ├── Tasks
        ├── Docs
        └── Meetings
```

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
│  ─────────────────────────────── │
│                                  │
│  ▸ PERSONAL                      │  ← Collapsible, always accessible
│    Tasks                         │
│    Docs                          │
│                                  │
├──────────────────────────────────┤
│  ⚙️ Settings                     │
├──────────────────────────────────┤
│  ┌────────────────────────────┐  │
│  │ ● SLSP                  ▼ │  │  ← Workspace selector
│  └────────────────────────────┘  │
│  Work Mode                       │
└──────────────────────────────────┘
```

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
4. **Personal space** remains separate section (not a "workspace")

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

## Three Doc Scopes (Preserved)

| Scope | Access Point | Use Case |
|-------|--------------|----------|
| Personal | Sidebar → Personal → Docs | Private notes, research, planning |
| Workspace | Global Docs → "Workspace" scope | Contracts, client templates, shared refs |
| Project | Project detail → Docs tab | Project-specific documentation |

The Docs page keeps its scope dropdown:
- `{Workspace Name} (Workspace)` - workspace-level docs
- Individual project names - project-level docs
- "All" - combined view

---

## Long Projects List Handling

For workspaces with many projects (SLSP has 20):

1. **Scrollable section** with max-height
2. **Optional**: Search/filter within projects section
3. **Optional**: Collapse inactive projects or group by status

---

## Implementation Phases

### Phase 1: Sidebar Restructure
- Create `WorkspaceSelector` component at bottom
- Move Tasks/Docs/Meetings to top as "Global Views"
- List projects directly (remove workspace nesting)
- Keep Personal as separate section

**Files**: `src/components/layout/sidebar.tsx`, new `workspace-selector.tsx`

### Phase 2: Explicit Context Headers
- Add workspace badge to Tasks, Docs, Meetings pages
- Use workspace color for visual consistency

**Files**: `src/app/tasks/page.tsx`, `src/app/docs/client.tsx`, `src/app/meetings/page.tsx`

### Phase 3: Meetings Aggregation
- Update Meetings to aggregate workspace + project meetings
- Match how Tasks already works

**Files**: `src/stores/meetings.ts`, `src/lib/desk/meetings.ts`, `src/app/meetings/page.tsx`

### Phase 4: Polish
- Default project detail to Tasks tab (most common use)
- Rename any remaining "All Tasks" labels
- Add task/doc counts to sidebar items

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
- [ ] Implement Phase 1: Sidebar restructure
- [ ] Implement Phase 2: Explicit context headers
- [ ] Implement Phase 3: Meetings aggregation
- [ ] Implement Phase 4: Polish
- [ ] Test with actual workflow

---

## Technical Notes

### Files to Modify

| File | Changes |
|------|---------|
| `src/components/layout/sidebar.tsx` | Major restructure - remove workspace nesting, add global views |
| `src/components/layout/workspace-selector.tsx` | New component for bottom selector |
| `src/app/tasks/page.tsx` | Add workspace context header |
| `src/app/docs/client.tsx` | Add workspace context header |
| `src/app/meetings/page.tsx` | Add workspace context header, aggregate meetings |
| `src/stores/meetings.ts` | Add workspace-level meetings query |

### No Changes Needed

- Route structure (`/tasks`, `/docs`, `/meetings`, `/projects/view`)
- File system structure (`~/Desk/workspaces/{id}/...`)
- Data models (Task, Doc, Meeting types)
- `ContentScope` type or scoping logic
- Settings store (already has `currentWorkspaceId`)

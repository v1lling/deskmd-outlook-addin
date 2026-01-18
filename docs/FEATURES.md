# Orbit Features Specification

## Overview

Orbit has four core modules plus global features:

1. **Area Management** - Switch between clients/workspaces
2. **Tasks Module** - Kanban board and task management
3. **Projects Module** - Project organization with notes and context
4. **Notes Module** - Meeting logs and project documentation
5. **Global Features** - Search, shortcuts, settings

---

## 1. Area Management

### 1.1 Area Switcher

**Location**: Top-left of sidebar

**Features**:
- Dropdown showing all areas
- Current area highlighted
- Switch instantly loads new area's data
- Color indicator per area (optional)

**UI**:
```
┌──────────────────┐
│ [SLSP ▾]         │  ← Click to switch
├──────────────────┤
│  ● SLSP          │
│  ○ SSS           │
│  ───────────     │
│  + New Area      │
└──────────────────┘
```

### 1.2 Area Settings

**Per-area configuration**:
- Display name
- Color (for visual distinction)
- Default project (optional)
- Email config (future)

---

## 2. Tasks Module

### 2.1 All Tasks View (Kanban)

**Purpose**: Main working view - see all tasks across all projects in current area.

**Features**:
- Three columns: Todo, Doing, Done
- Drag tasks between columns (updates status)
- Each card shows: title, project tag, priority indicator
- Filter by project (dropdown)
- Quick add button per column
- Click card to open task detail

**UI**:
```
┌─────────────────────────────────────────────────────────────────────┐
│ All Tasks                           Project: [All ▾]  [+ New Task]  │
├─────────────────────────────────────────────────────────────────────┤
│  TODO (3)          │  DOING (2)          │  DONE (5)                │
│ ┌────────────────┐ │ ┌────────────────┐  │ ┌────────────────┐       │
│ │ Setup webhook  │ │ │ Review docs    │  │ │ Fix auth bug   │       │
│ │ ● SLSKey       │ │ │ ● Alma         │  │ │ ✓ SLSKey       │       │
│ │ ▲ high         │ │ │                │  │ │                │       │
│ └────────────────┘ │ └────────────────┘  │ └────────────────┘       │
│ ┌────────────────┐ │ ┌────────────────┐  │                          │
│ │ Write specs    │ │ │ Email follow-up│  │                          │
│ │ ● Swisscovery  │ │ │ ● SLSKey       │  │                          │
│ └────────────────┘ │ └────────────────┘  │                          │
│    [+ Add]         │    [+ Add]          │                          │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Task Detail / Editor

**Opens as**: Slide-over panel or modal

**Features**:
- Edit title (inline)
- Change status, priority, project (dropdowns)
- Due date picker
- Markdown content editor
- Delete task (with confirmation)
- Link to project

**UI**:
```
┌─────────────────────────────────────────────────────────────────────┐
│                                                          [×] Close  │
├─────────────────────────────────────────────────────────────────────┤
│ # Setup webhook for new library                                     │
│                                                                     │
│ Status: [Doing ▾]   Priority: [High ▾]   Project: [SLSKey ▾]       │
│ Due: [Jan 20, 2024]                                                 │
├─────────────────────────────────────────────────────────────────────┤
│ ## Notes                                                            │
│                                                                     │
│ Configure Alma webhook for ZB Winterthur.                           │
│                                                                     │
│ - [ ] Get API credentials                                           │
│ - [ ] Set up endpoint                                               │
│ - [ ] Test with sample user                                         │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│ [Delete]                                              [Save]        │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3 Quick Add Task

**Trigger**: `Cmd+N` or "+" button

**Features**:
- Minimal modal: title + project selector + priority
- Creates task immediately
- Defaults to current project filter (if any)
- Status defaults to "todo"

**UI**:
```
┌─────────────────────────────────────────┐
│ New Task                            [×] │
├─────────────────────────────────────────┤
│ Title: [________________________]       │
│ Project: [SLSKey ▾]                     │
│ Priority: ○ Low  ● Medium  ○ High       │
├─────────────────────────────────────────┤
│                      [Cancel] [Create]  │
└─────────────────────────────────────────┘
```

### 2.4 Inbox

**Purpose**: Tasks not yet assigned to a project

**Location**: `areas/{area}/_inbox/tasks/`

**Features**:
- List view of unassigned tasks
- Quick assign to project
- Same task editing as elsewhere

---

## 3. Projects Module

### 3.1 Projects List

**Purpose**: Overview of all projects in current area

**Features**:
- Card grid or list view
- Show: name, status badge, task count, recent activity
- Filter by status (active, paused, completed, archived)
- Search by name
- Click to open project

**UI**:
```
┌─────────────────────────────────────────────────────────────────────┐
│ Projects                        Status: [All ▾]     [+ New Project] │
├─────────────────────────────────────────────────────────────────────┤
│ ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────┐ │
│ │ SLSKey              │ │ Alma Migration      │ │ Swisscovery     │ │
│ │ ● Active            │ │ ● Active            │ │ ○ Paused        │ │
│ │                     │ │                     │ │                 │ │
│ │ 5 tasks · 3 doing   │ │ 2 tasks · 1 doing   │ │ 8 tasks         │ │
│ │ Updated: Today      │ │ Updated: Yesterday  │ │ Updated: -3d    │ │
│ └─────────────────────┘ └─────────────────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 Project Detail

**Purpose**: Everything about one project

**Features**:
- Tabs: Overview, Tasks, Notes, Context
- Overview: description, stats, quick actions
- Tasks: filtered kanban for this project only
- Notes: list of project notes
- Context: AI knowledge files

**UI**:
```
┌─────────────────────────────────────────────────────────────────────┐
│ ← Projects    SLSKey                        ● Active    [⚙ Edit]   │
├─────────────────────────────────────────────────────────────────────┤
│ [Overview]  [Tasks]  [Notes]  [Context]                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ Authentication service for Swiss library patrons.                   │
│                                                                     │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                 │
│ │ 5 Tasks      │ │ 12 Notes     │ │ 8 Context    │                 │
│ │ 3 in progress│ │ Last: Today  │ │    Files     │                 │
│ └──────────────┘ └──────────────┘ └──────────────┘                 │
│                                                                     │
│ Recent Activity                                                     │
│ • Task "Setup webhook" moved to Doing (2h ago)                      │
│ • Note "Weekly sync" added (Yesterday)                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.3 Create Project

**Features**:
- Name input
- Status selector (defaults to Active)
- Optional description
- Creates folder structure automatically

---

## 4. Notes Module

### 4.1 Project Notes List

**Location**: Project detail → Notes tab

**Features**:
- Chronological list (newest first)
- Show: title, date, preview
- Search within project notes
- Click to open/edit

**UI**:
```
┌─────────────────────────────────────────────────────────────────────┐
│ Notes                               Search: [________]  [+ New Note]│
├─────────────────────────────────────────────────────────────────────┤
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ Weekly Sync - Jan 15, 2024                                      │ │
│ │ Discussed upcoming deadlines and webhook requirements...        │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ Architecture Decision - Jan 10, 2024                            │ │
│ │ Decided to use SAML over OIDC for the integration because...    │ │
│ └─────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 Note Editor

**Features**:
- Title input
- Full markdown editor
- Auto-save
- Delete option

### 4.3 Quick Note

**Trigger**: `Cmd+Shift+N`

**Features**:
- Creates dated note in current project
- Opens editor immediately
- Filename: `YYYY-MM-DD-{slug}.md`

---

## 5. Context Module (AI Knowledge)

### 5.1 Context Explorer

**Location**: Project detail → Context tab

**Features**:
- File tree view of `project/context/` folder
- Create files/folders
- Upload files (drag & drop)
- Special treatment for `CLAUDE.md`

**UI**:
```
┌─────────────────────────────────────────────────────────────────────┐
│ Context                                            [+ New] [Upload] │
├─────────────────────────────────────────────────────────────────────┤
│ 📄 CLAUDE.md                                    ← Main AI context   │
│ 📁 technical-docs/                                                  │
│    📄 service-description.md                                        │
│    📄 api-reference.md                                              │
│ 📁 archived-emails/                                                 │
│    📁 britannica/                                                   │
│    📁 statista/                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 File Viewer/Editor

**Features**:
- Markdown rendering (view mode)
- Raw editing (edit mode)
- Syntax highlighting
- Save/cancel

---

## 6. Global Features

### 6.1 Sidebar Navigation

```
┌──────────────────┐
│ [SLSP ▾]         │  ← Area switcher
├──────────────────┤
│ ◉ All Tasks      │  ← Main kanban
│ ○ Inbox          │  ← Unassigned
│ ○ Projects       │  ← Project list
├──────────────────┤
│ PROJECTS         │
│ • SLSKey         │  ← Quick access
│ • Alma           │
│ • Swisscovery    │
├──────────────────┤
│ ⚙ Settings       │
└──────────────────┘
```

### 6.2 Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Cmd+N` | New task |
| `Cmd+Shift+N` | New note (in current project) |
| `Cmd+P` | New project |
| `Cmd+K` | Command palette (future) |
| `Cmd+1` | Go to All Tasks |
| `Cmd+2` | Go to Projects |
| `Cmd+,` | Settings |
| `Escape` | Close modal/panel |

### 6.3 Settings

**Sections**:
- **General**: Data folder path, theme (light/dark/system)
- **Areas**: Manage areas (rename, delete, reorder)
- **About**: Version, links

### 6.4 First Run Setup

**Wizard steps**:
1. Welcome screen
2. Set data folder (default: `~/Orbit`)
3. Create first area
4. Done - open app

---

## 7. Future Features (Not MVP)

- Email integration (per-area config)
- AI draft generation with project context
- Command palette (`Cmd+K`)
- Global search across all areas
- Calendar integration
- GitHub/GitLab integration
- Time tracking
- Templates for projects/tasks
- Export/backup functionality
- Mobile companion app (read-only)

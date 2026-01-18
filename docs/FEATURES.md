# Orbit Features

## Current (v0.2)

### Areas
- Dropdown switcher in sidebar
- Color-coded for visual distinction
- Complete isolation between areas
- Create/edit/delete areas

### Projects
- Card grid with status badges (active/paused/completed/archived)
- Tabbed detail page: Overview, Tasks, Notes, Meetings
- Task counts and progress indicators
- Quick actions from overview tab

### Tasks
- 4-column Kanban: Todo, Doing, Waiting, Done
- Drag-drop between columns
- Quick add modal (title, project, priority, due date)
- Detail panel with full editing
- Priority badges (high/medium/low)
- Due dates
- Project reassignment from detail panel
- Filter by project or show all/unassigned

### Notes
- WYSIWYG markdown editor (Tiptap)
- Project grouping and filtering
- Card-based list view with preview
- Project reassignment
- Filter by project or show unassigned

### Meetings
- List view by project
- Markdown editor for notes
- Date tracking

### Unassigned Items
- Tasks and notes can exist without a project
- `_unassigned` special directory
- Filter option on All Tasks and Notes pages
- Easy reassignment to projects

### Settings
- Theme: Light/Dark/System
- Data folder path display and change
- Reset to default path option

### Setup
- First-run wizard
- Detect existing Orbit data
- Custom data path selection

## Planned

### Global Search (Cmd+K)
- Search across tasks, notes, projects
- Quick navigation
- Recent items

### Keyboard Shortcuts
- `Cmd+N` - New task
- `Cmd+Shift+N` - New note
- `Cmd+P` - New project
- `Cmd+K` - Search

### AI Context
- Per-project `context/` folder
- `CLAUDE.md` for AI instructions
- Knowledge files for project context

### Email (Future)
- Per-area email config
- Microsoft 365 / IMAP support
- Assign emails to projects

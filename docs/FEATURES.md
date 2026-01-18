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

---

## Planned

### Global Search (Cmd+K)
- Search tasks, notes, projects, meetings in one place
- Fuzzy matching for titles and content
- Quick navigation to any item
- Recent items list
- Keyboard-first interaction

### Keyboard Shortcuts
- `Cmd+K` - Open search
- `Cmd+N` - New task
- `Cmd+Shift+N` - New note
- `Cmd+P` - New project
- `Cmd+,` - Settings
- `Esc` - Close modals/panels
- Shortcut hints in UI tooltips

### Dashboard/Home Page
- Today's tasks (due today, overdue)
- Tasks in progress across all projects
- Recent activity feed
- Quick stats (tasks completed this week, etc.)
- Quick capture without leaving dashboard

### AI Context Integration
- Per-project `context/` folder
- `CLAUDE.md` for AI instructions per project
- Knowledge files (.md) for project context
- Export project context for AI tools

### Testing Infrastructure
- Unit tests for `lib/orbit/` (calculations, parser, search)
- Integration tests for CRUD operations
- Component tests for key UI flows

---

## Future Ideas

### Email Integration
- Per-area email config
- Microsoft 365 / IMAP support
- Assign emails to projects
- Email-to-task conversion

### Calendar View
- Task due dates on calendar
- Meeting scheduling
- Drag tasks to reschedule

### Time Tracking
- Log time on tasks
- Project time reports
- Optional Pomodoro timer

### Templates
- Project templates (common project structures)
- Task templates (recurring task patterns)
- Note templates (meeting notes, decisions)

### Mobile Companion
- Read-only web view
- Quick capture from phone
- Sync via file system (iCloud/Dropbox)

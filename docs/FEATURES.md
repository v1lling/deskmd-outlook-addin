# Orbit Features

## Current (v0.3)

### Dashboard
- **Capture widget**: Quick task capture with triage workflow
  - Quick add input for instant task capture
  - Amber highlighting when tasks await triage
  - Triage menu: Move to Personal Tasks, Workspace/Project, or Delete
  - Post-triage detail modal for adding priority and notes
- **Focus widget**: Shows all in-progress tasks across workspaces
- **Workspaces widget**: Overview of all workspaces with completion progress

### Personal Space
- Tasks and notes not tied to any workspace
- Inbox for quick capture (triaged from Dashboard)
- Separate from workspace-scoped content

### Workspaces
- Color-coded for visual distinction
- Inline in sidebar with expandable sub-navigation
- Complete isolation between workspaces
- Create/edit/delete workspaces

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

### Global Search (Cmd+K)
- Search tasks, notes, projects, meetings in one place
- Fuzzy matching for titles and content
- Quick navigation to any item
- In-memory Fuse.js index for fast search

### File Watcher
- Auto-refresh UI when files change externally
- Works with external editors (Obsidian, VS Code, etc.)

### Auto-Save
- Obsidian-like silent persistence
- Error-only status indicator
- Debounced saves to prevent excessive writes

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

### Testing Infrastructure
- Unit tests for `lib/orbit/` (calculations, parser, search)
- Integration tests for CRUD operations
- Component tests for key UI flows
- End-to-end tests for main user journeys

### Keyboard Shortcuts
- `Cmd+K` - Open search (implemented)
- `Cmd+N` - New task
  - Or new quick capture if in dashboard? Or always quick capture?
- `Cmd+Shift+N` - New note
- `Cmd+P` - New project
- `Cmd+,` - Settings
- `Esc` - Close modals/panels
- Shortcut hints in UI tooltips

### Knowledge Base Integration
- Per-project `context/` folder
- Knowledge files (.md) for project context
- Each Project should have "Knowledge Base" section
- Export project context for AI tools
- Nice structure and display for context files per project
- Global Knowledge Base for general info
- AI reads context files when generating content

### Email Integration
- Per-workspace email config
- Microsoft 365 / IMAP support
- Assign emails to projects
- Email-to-task conversion
- AI features:
  - AI-assisted draft replies using Knowledge Base context
  - Summarize email threads into tasks/notes
  - Chat with AI about email content, get suggestions
- Focus on task extraction and email drafting, not full email client
- AI features are optional, opt-in
- AI features are visually minimalistic
  
### AI Assistance
- For Knowledge Base, Email, Task/Note creation
- Context-aware suggestions
- Nice framework which hands over context to AI models
- Minimal UI impact
- Configure AI providers in Settings
- Technical:
  - Modular AI integration layer
  - Support for multiple AI providers
  - Claude Code is a must, not sure how to do it as it need to access terminal?

### Mobile Companion
- Read-only web view
- Quick capture from phone
- Sync via file system (iCloud/WebDAV/Dropbox)
- Responsive design for small screens
  
---

## Future Ideas

### Time Tracking
- Log time on tasks
- Project time reports
- Optional Pomodoro timer

### Templates
- Project templates (common project structures)
- Task templates (recurring task patterns)
- Note templates (meeting notes, decisions)
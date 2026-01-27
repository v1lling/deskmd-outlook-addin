# Orbit Features

## Current (v0.4)

### Dashboard
- **Capture widget**: Quick task capture with triage workflow to Personal or Workspace/Project
- **Focus widget**: Shows all in-progress tasks across workspaces
- **Workspaces widget**: Overview of all workspaces with completion progress

### Personal Space
- Tasks and docs not tied to any workspace
- Capture widget on Dashboard for quick capture (triage to Personal or Workspace)
- Separate from workspace-scoped content

### Workspaces
- Color-coded for visual distinction
- Inline in sidebar with expandable sub-navigation
- Complete isolation between workspaces
- Workspace-level docs (shared across all projects)
- Create/edit/delete workspaces

### Projects
- Listed inline in sidebar under each workspace (alphabetically sorted)
- Tabbed detail page: Overview, Tasks, Docs, Meetings
- Status badges (active/paused/completed/archived)
- Task counts and progress indicators
- Quick actions from overview tab
- New Project button in sidebar

### Tasks
- 4-column Kanban: Todo, Doing, Waiting, Done
- Drag-drop between columns
- Quick add modal (title, project, priority, due date)
- Detail panel with full editing
- Priority badges (high/medium/low)
- Due dates
- Project reassignment from detail panel
- Filter by project or show all/unassigned

### Docs
- **Tree structure**: Folders with unlimited nesting
- **Multiple scopes**: Personal, Workspace-level, Project-level
- WYSIWYG markdown editor (Tiptap)
- Drag-drop file import
- Folder operations: create, rename, delete
- Project reassignment (for project docs)
- Filter by project or show unassigned
- Tabbed view: Workspace docs vs All Projects

### Meetings
- List view by project
- Markdown editor for notes
- Date tracking

### Unassigned Items
- Tasks and docs can exist without a project
- `_unassigned` special directory
- Filter option on All Tasks and Docs pages
- Easy reassignment to projects

### Global Search (Cmd+K)
- Search tasks, docs, projects, meetings in one place
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

### Tab-Based Editing (Obsidian-style)
- "Orbit" tab is always pinned, showing current app view
- Clicking docs/tasks/meetings opens them in new tabs
- Full-width editors with markdown WYSIWYG (Tiptap)
- Tab state persists in localStorage
- Keyboard shortcuts: Cmd+W close, Cmd+Shift+[ ] switch tabs

### AI Chat
- Slide panel accessible from any view
- **Providers**: Claude Code CLI or Anthropic API
- **Doc context**: Attach docs from current scope for AI awareness
- Conversation history per session
- Configure AI provider in Settings

### Editor State Management
- 400ms debounced auto-save (Obsidian-like fast saves)
- External change detection (works with Obsidian, VS Code editing)
- Path change and deletion handling with user notifications
- Cursor-stable editing (no jumps during external changes)

### UI/UX
- **Custom scrollbars**: OverlayScrollbars for consistent cross-platform styling (Chrome + Tauri/WKWebView)
- List/Kanban view toggle for tasks
- Collapsible sidebar sections

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
- `Cmd+Shift+N` - New doc
- `Cmd+P` - New project
- `Cmd+,` - Settings
- `Esc` - Close modals/panels
- Shortcut hints in UI tooltips

### AI Context Integration
- Any doc can be flagged as "AI context"
- AI features read flagged docs when generating content
- Per-project context selection
- Export project context for AI tools
- Global context docs for general info

### Email Integration
- Per-workspace email config
- Microsoft 365 / IMAP support
- Assign emails to projects
- Email-to-task conversion
- AI features:
  - AI-assisted draft replies using context
  - Summarize email threads into tasks/docs
  - Chat with AI about email content
- Focus on task extraction and email drafting

### AI Assistance
- Context-aware suggestions
- Nice framework which hands over context to AI models
- Minimal UI impact
- Configure AI providers in Settings

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
- Doc templates (meeting notes, decisions)

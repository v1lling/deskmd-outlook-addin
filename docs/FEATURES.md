# Desk Features

## Current (v0.4)

### Dashboard
- **Capture widget**: Quick task capture with triage workflow to Personal or Workspace/Project
- **Focus widget**: Shows all in-progress tasks across workspaces
- **Workspaces widget**: Overview of all workspaces with completion progress

### Personal Workspace
- Personal is a workspace (`_personal`) - first option in Work Mode selector
- Can have projects, tasks, docs like any other workspace
- **Capture inbox** (`_capture`): Quick triage to Personal or other workspaces
- Indigo color distinguishes from client workspaces

### Workspaces ("Work Mode" Navigation)
- **Workspace selector** at bottom of sidebar for explicit context switching
- Color-coded for visual distinction (color dot in selector and page headers)
- Complete isolation between workspaces
- Workspace-level docs (shared across all projects)
- Create/edit/delete workspaces

### Projects
- **Listed directly in sidebar** for current workspace (alphabetically sorted)
- Scrollable list with max-height for workspaces with many projects
- Tabbed detail page: Tasks (default), Overview, Docs, Meetings
- Status badges (active/paused/completed/archived)
- Task counts in sidebar and progress indicators
- New Project button in projects section

### Tasks
- **Page header** with workspace color dot and name badge
- 4-column Kanban: Todo, Doing, Waiting, Done
- Drag-drop between columns
- Quick add modal (title, project, priority, due date)
- Detail panel with full editing
- Priority badges (high/medium/low)
- Due dates
- Project reassignment from detail panel
- Filter by project or show all/unassigned
- Active task count in sidebar

### Docs
- **Page header** with workspace color dot and name badge
- **Tree structure**: Folders with unlimited nesting
- **Multiple scopes**: Personal, Workspace-level, Project-level
- WYSIWYG markdown editor (Tiptap)
- Drag-drop file import
- Folder operations: create, rename, delete
- Project reassignment (for project docs)
- Filter by project or show unassigned
- Doc count in sidebar

### Meetings
- **Page header** with workspace color dot and name badge
- **Aggregates meetings** from all projects in workspace (+ unassigned)
- Grouped view by project when showing all
- Filter by project
- Markdown editor for notes
- Date tracking
- Meeting count in sidebar

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

### Manual Save
- **Cmd+S** to save content (or click save button in header)
- Metadata changes (status, priority, title, etc.) save immediately
- Dirty indicator in tab when unsaved
- Save/Don't Save/Cancel dialog on tab close
- Confirmation dialog when quitting with unsaved changes

### Settings
- Theme: Light/Dark/System
- Data folder path display and change
- Reset to default path option

### Setup
- First-run wizard
- Detect existing Desk data
- Custom data path selection

### Tab-Based Editing (Obsidian-style)
- "Desk" tab is always pinned, showing current app view
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
- Manual save with Cmd+S (no auto-save)
- External change detection (works with Obsidian, VS Code editing)
- Path change and deletion handling with user notifications
- Unsaved changes protection on tab close and app quit
- Immediate metadata saves preserve unsaved body content

### Email Integration
- **Deep links**: External mail clients send emails via `desk://email?data=...`
- **Outlook Add-in**: "Open in Desk" button in Outlook ribbon (hosted on GitHub Pages)
- **Email viewer**: Session-only tab display (not persisted)
- **Draft Reply**: AI-assisted draft generation with editable To/CC/Subject fields
- **Send**: Opens mailto: in default mail client
- *Coming soon*: Project context for AI drafts, Extract Tasks from emails

### UI/UX
- **"Work Mode" navigation**: Explicit workspace context with selector at bottom
- **Global views**: Tasks/Docs/Meetings at top of sidebar, filtered by active workspace
- **Page headers**: Workspace color dot + name badge on all workspace-scoped pages
- **Personal in Work Mode**: Personal is first option in workspace selector (indigo color)
- **Custom scrollbars**: OverlayScrollbars for consistent cross-platform styling (Chrome + Tauri/WKWebView)
- List/Kanban view toggle for tasks

---

## Planned

### Testing Infrastructure
- Unit tests for `lib/desk/` (calculations, parser, search)
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

## Move away from Nextjs
- Migrate to pure Tauri + React setup
- We don't need server-side rendering
- Simplify build and deployment

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

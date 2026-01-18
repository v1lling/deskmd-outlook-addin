# Orbit Data Models

## Overview

All data is stored as markdown files with YAML frontmatter. This document defines the structure for each entity type.

---

## 1. File Structure

```
~/Orbit/
├── config.json                         # App configuration
└── areas/
    └── {area-id}/
        ├── area.md                     # Area metadata
        ├── _inbox/
        │   └── tasks/
        │       └── *.md                # Unassigned tasks
        └── projects/
            └── {project-id}/
                ├── project.md          # Project metadata
                ├── tasks/
                │   └── *.md            # Task files
                ├── notes/
                │   └── *.md            # Note files
                └── context/
                    ├── CLAUDE.md       # AI context (optional)
                    └── **/*            # Knowledge files
```

---

## 2. App Configuration

**File**: `~/Orbit/config.json`

```typescript
interface OrbitConfig {
  // Data location
  dataPath: string;              // e.g., "/Users/sascha/Orbit"

  // Current state
  currentAreaId: string | null;  // Last selected area

  // UI preferences
  theme: 'light' | 'dark' | 'system';
  sidebarCollapsed: boolean;

  // First run
  setupCompleted: boolean;
}
```

**Example**:
```json
{
  "dataPath": "/Users/sascha/Orbit",
  "currentAreaId": "slsp",
  "theme": "system",
  "sidebarCollapsed": false,
  "setupCompleted": true
}
```

---

## 3. Area

**File**: `~/Orbit/areas/{area-id}/area.md`

### TypeScript Interface

```typescript
interface Area {
  // Identity
  id: string;              // Folder name, e.g., "slsp"

  // From frontmatter
  name: string;            // Display name, e.g., "SLSP"
  description?: string;    // Optional description
  color?: string;          // Hex color for UI, e.g., "#3b82f6"
  created: string;         // ISO date

  // Future: email config
  email?: {
    provider: 'microsoft365' | 'imap';
    // Provider-specific fields...
  };

  // Computed
  projectCount?: number;
}
```

### Markdown File Format

```markdown
---
name: SLSP
description: Swiss Library Service Platform - Technical Support
color: "#3b82f6"
created: 2024-01-15
---

# SLSP

Main client for library platform support work.

## Notes

- Uses Microsoft 365 for email
- Multiple ongoing projects
```

---

## 4. Project

**File**: `~/Orbit/areas/{area-id}/projects/{project-id}/project.md`

### TypeScript Interface

```typescript
interface Project {
  // Identity
  id: string;              // Folder name, e.g., "slskey"
  areaId: string;          // Parent area ID

  // From frontmatter
  name: string;            // Display name
  status: 'active' | 'paused' | 'completed' | 'archived';
  description?: string;
  created: string;         // ISO date

  // Computed
  taskCount?: number;
  tasksByStatus?: {
    todo: number;
    doing: number;
    done: number;
  };
  noteCount?: number;
  lastActivity?: string;   // ISO date
}
```

### Markdown File Format

```markdown
---
name: SLSKey
status: active
description: Authentication service for Swiss library patrons
created: 2024-01-10
---

# SLSKey

## Overview

SLSKey enables private users to access e-resources through Swiss libraries using Switch edu-ID authentication.

## Key Links

- [Confluence](https://confluence.slsp.ch/slskey)
- [GitHub](https://github.com/Swiss-Library-Service-Platform/slskey)
```

---

## 5. Task

**File**: `~/Orbit/areas/{area-id}/projects/{project-id}/tasks/{date}-{slug}.md`

**Filename format**: `YYYY-MM-DD-{slug}.md` where slug is derived from title.

### TypeScript Interface

```typescript
interface Task {
  // Identity
  id: string;              // Filename without .md
  projectId: string;       // Parent project ID (or "_inbox")
  areaId: string;          // Parent area ID
  filePath: string;        // Full path to file

  // From frontmatter
  title: string;
  status: 'todo' | 'doing' | 'done';
  priority?: 'low' | 'medium' | 'high';
  due?: string;            // ISO date
  created: string;         // ISO date

  // From content
  content: string;         // Markdown body after frontmatter
}
```

### Markdown File Format

```markdown
---
title: Setup webhook for ZB Winterthur
status: doing
priority: high
due: 2024-01-20
created: 2024-01-15
---

Configure Alma webhook endpoint for the new library.

## Steps

- [ ] Get Alma API credentials from library
- [ ] Configure webhook endpoint in Alma
- [ ] Test with sample patron
- [ ] Document the setup
```

---

## 6. Note

**File**: `~/Orbit/areas/{area-id}/projects/{project-id}/notes/{date}-{slug}.md`

**Filename format**: `YYYY-MM-DD-{slug}.md`

### TypeScript Interface

```typescript
interface Note {
  // Identity
  id: string;              // Filename without .md
  projectId: string;       // Parent project ID
  areaId: string;          // Parent area ID
  filePath: string;        // Full path to file

  // From frontmatter
  title: string;
  created: string;         // ISO date

  // From content
  content: string;         // Markdown body

  // Computed
  preview?: string;        // First ~100 chars for list view
}
```

### Markdown File Format

```markdown
---
title: Weekly Sync Meeting
created: 2024-01-15
---

# Weekly Sync Meeting

**Date**: January 15, 2024
**Attendees**: Sascha, Maria, Thomas

## Discussion

- Reviewed webhook implementation progress
- Discussed timeline for Britannica integration
- Maria to follow up on API credentials

## Action Items

- [ ] Sascha: Complete webhook setup by Friday
- [ ] Maria: Send API docs to team
- [ ] Thomas: Schedule follow-up with library
```

---

## 7. Context File

**Location**: `~/Orbit/areas/{area-id}/projects/{project-id}/context/`

Context files are free-form markdown or text files used for AI assistance. The structure is flexible, but `CLAUDE.md` has special significance.

### TypeScript Interface

```typescript
interface ContextFile {
  // Identity
  path: string;            // Relative path within context folder
  name: string;            // Filename

  // Metadata
  type: 'md' | 'txt' | 'json' | 'other';
  size: number;            // Bytes
  modified: string;        // ISO date

  // Content (when loaded)
  content?: string;
}

interface ProjectContext {
  projectId: string;
  files: ContextFile[];
  claudeMd?: string;       // Content of CLAUDE.md if exists
}
```

### CLAUDE.md Template

```markdown
# {Project Name} - AI Context

## Overview

Brief description of the project for AI context.

## Key Terms

- **Term 1**: Definition
- **Term 2**: Definition

## Common Issues

### Issue Category 1

Description and typical resolution.

### Issue Category 2

Description and typical resolution.

## Response Guidelines

- Tone: Professional but friendly
- Language: English (or German if recipient writes in German)
- Sign off: Best regards, Sascha
```

---

## 8. API Types

### Standard Response

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
}
```

### Task Operations

```typescript
// GET /api/tasks?areaId=x&projectId=y
interface TasksResponse {
  tasks: Task[];
  total: number;
}

// POST /api/tasks
interface CreateTaskRequest {
  areaId: string;
  projectId: string;       // Use "_inbox" for unassigned
  title: string;
  priority?: 'low' | 'medium' | 'high';
  due?: string;
  content?: string;
}

// PATCH /api/tasks/[id]
interface UpdateTaskRequest {
  title?: string;
  status?: 'todo' | 'doing' | 'done';
  priority?: 'low' | 'medium' | 'high';
  projectId?: string;      // Move to different project
  due?: string;
  content?: string;
}
```

### Project Operations

```typescript
// GET /api/projects?areaId=x
interface ProjectsResponse {
  projects: Project[];
  total: number;
}

// POST /api/projects
interface CreateProjectRequest {
  areaId: string;
  name: string;
  description?: string;
  status?: 'active' | 'paused';
}

// PATCH /api/projects/[id]
interface UpdateProjectRequest {
  name?: string;
  description?: string;
  status?: 'active' | 'paused' | 'completed' | 'archived';
}
```

### Note Operations

```typescript
// GET /api/notes?projectId=x
interface NotesResponse {
  notes: Note[];
  total: number;
}

// POST /api/notes
interface CreateNoteRequest {
  projectId: string;
  areaId: string;
  title: string;
  content?: string;
}

// PATCH /api/notes/[id]
interface UpdateNoteRequest {
  title?: string;
  content?: string;
}
```

---

## 9. Zustand Store Types

### Area Store

```typescript
interface AreaStore {
  // Data
  areas: Area[];
  currentAreaId: string | null;
  isLoading: boolean;

  // Actions
  fetchAreas: () => Promise<void>;
  setCurrentArea: (id: string) => void;
  createArea: (data: CreateAreaRequest) => Promise<Area>;
}
```

### Tasks Store

```typescript
interface TasksStore {
  // Data
  tasks: Task[];
  isLoading: boolean;

  // Filters
  projectFilter: string | null;  // null = all projects
  statusFilter: string | null;   // null = all statuses

  // Actions
  fetchTasks: (areaId: string) => Promise<void>;
  createTask: (data: CreateTaskRequest) => Promise<Task>;
  updateTask: (id: string, data: UpdateTaskRequest) => Promise<Task>;
  deleteTask: (id: string) => Promise<void>;
  setProjectFilter: (projectId: string | null) => void;
  setStatusFilter: (status: string | null) => void;
}
```

### Projects Store

```typescript
interface ProjectsStore {
  // Data
  projects: Project[];
  currentProject: Project | null;
  isLoading: boolean;

  // Actions
  fetchProjects: (areaId: string) => Promise<void>;
  setCurrentProject: (id: string | null) => void;
  createProject: (data: CreateProjectRequest) => Promise<Project>;
  updateProject: (id: string, data: UpdateProjectRequest) => Promise<Project>;
  deleteProject: (id: string) => Promise<void>;
}
```

### Notes Store

```typescript
interface NotesStore {
  // Data
  notes: Note[];
  currentNote: Note | null;
  isLoading: boolean;

  // Actions
  fetchNotes: (projectId: string) => Promise<void>;
  setCurrentNote: (id: string | null) => void;
  createNote: (data: CreateNoteRequest) => Promise<Note>;
  updateNote: (id: string, data: UpdateNoteRequest) => Promise<Note>;
  deleteNote: (id: string) => Promise<void>;
}
```

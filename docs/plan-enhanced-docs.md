# Enhanced Docs - Implementation Plan

## Overview

Rename "Notes" to "Docs" and enhance with:
- **Tree structure** - Folders with unlimited nesting
- **Import features** - Drag & drop, folder import, smart paste

**Not in this scope:**
- AI context toggle (separate future feature)

## Why This Approach

- **Single source of truth** - All documents in one place (Docs)
- **No duplication** - Company info, project specs, personal templates all live together
- **Simpler mental model** - "Docs" for everything
- **Clean migration** - Rename data folders too, no legacy cruft

## Changes Summary

| Current | After |
|---------|-------|
| "Notes" in sidebar | "Docs" in sidebar |
| Flat list of notes | Tree with folders |
| Card grid view | Tree view + editor panel |

## File Structure (clean rename)

```
~/Orbit/
├── personal/
│   └── docs/                       # Renamed from notes/
│       ├── .view.json              # Tree expansion state
│       ├── my-company/
│       │   ├── about.md
│       │   └── finances.md
│       └── templates/
│           └── email-style.md
├── workspaces/{workspace}/
│   ├── docs/                       # Workspace-level docs (NEW)
│   │   ├── .view.json
│   │   ├── team-contacts.md
│   │   └── processes/
│   │       └── onboarding.md
│   └── projects/{project}/
│       └── docs/                   # Renamed from notes/
│           ├── .view.json
│           ├── requirements.md
│           └── tech/
│               └── architecture.md
```

**Migration:** Simply rename `notes/` folders to `docs/` in your Orbit data directory.

## Data Model

```typescript
interface Doc {
  id: string;
  path: string;                    // Relative path with folders (e.g., "tech/architecture.md")
  scope: 'personal' | 'workspace' | 'project';
  workspaceId: string | null;
  projectId: string | null;
  filePath: string;                // Full absolute path
  title: string;
  created: string;
  content: string;
  preview?: string;
}

interface DocFolder {
  name: string;
  path: string;
  children: DocTreeNode[];
}

type DocTreeNode =
  | { type: 'folder'; folder: DocFolder }
  | { type: 'doc'; doc: Doc };
```

**Frontmatter (unchanged from Notes):**
```yaml
---
title: Architecture Overview
created: 2024-01-15
---
```

## UI Layout

```
┌─────────────────────────────────────────────────────┐
│ Docs                       [Import ▾] [+ New ▾]    │
├─────────────┬───────────────────────────────────────┤
│ Tree        │ Editor                                │
│             │                                       │
│ ▼ tech/     │ Title: Architecture Overview          │
│   📄 arch.. │                                       │
│   📄 stack..│ # Architecture Overview               │
│ ▶ processes │                                       │
│ 📄 overview │ This project uses Next.js with...    │
│             │                                       │
│ [+ Folder]  │                                       │
│             │ Drop files here to import             │
└─────────────┴───────────────────────────────────────┘
```

## Route Structure (Clean & Comprehensive)

**Current structure (Notes):**
```
/notes                     → Workspace notes (all projects, grouped)
/personal/notes            → Personal notes (flat list)
/projects/view?id=xxx      → Project detail with Notes tab
```

**New structure (Docs):**
```
/docs                      → Workspace docs page with tabs:
                              - "Workspace" tab: workspace-level docs tree
                              - "All" tab: aggregate all docs across projects (grouped)
/personal/docs             → Personal docs tree
/projects/view?id=xxx      → Project detail with "Docs" tab (was Notes)
```

**Sidebar nav changes:**
- Personal section: "Notes" → "Docs"
- Workspace section: "Notes" → "Docs"
- Project detail: "Notes" tab → "Docs" tab

## UI Access Points

| Level | Sidebar | Route | View |
|-------|---------|-------|------|
| Personal | Personal → "Docs" | `/personal/docs` | Tree with folders |
| Workspace (shared) | Workspace → "Docs" | `/docs` (Workspace tab) | Tree with folders |
| Workspace (all) | Workspace → "Docs" | `/docs` (All tab) | Grouped by project |
| Project | Project detail | `/projects/view?id=xxx` | Docs tab with tree |

## Import Methods

### 1. Drag & Drop Files
- Drop `.md` files onto tree or editor area
- Copies content, preserves folder structure if dropping folder

### 2. Import Folder
- "Import Folder..." in Import dropdown
- Recursively imports all `.md` files
- Preview before confirming

### 3. Smart Paste
- Tiptap converts pasted HTML → markdown
- "New from Clipboard" creates doc with clipboard content

## Implementation Phases

### Phase 1: Rename Notes → Docs (full rename)
**Goal:** Rename everything - UI, code, and data paths.

**Files to modify:**
- `src/components/layout/sidebar.tsx` - "Notes" → "Docs" in nav items
- `src/app/notes/` → `src/app/docs/` (rename route folder)
- `src/app/personal/notes/` → `src/app/personal/docs/` (rename route folder)
- `src/components/notes/` → `src/components/docs/` (rename component folder)
- `src/lib/orbit/notes.ts` → `src/lib/orbit/docs.ts`
- `src/stores/notes.ts` → `src/stores/docs.ts`
- `src/lib/orbit/constants.ts` - `notes` → `docs` in PATH_SEGMENTS
- Update all imports referencing old paths
- `src/app/projects/view/client.tsx` - rename "Notes" tab to "Docs"

### Phase 2: Library Enhancement
**Files:**
- `src/lib/orbit/docs.ts`

**New functions:**
- `buildDocTree()` - recursive tree from filesystem
- `getDocTree(scope, workspaceId?, projectId?)`
- `createDocFolder()`
- `renameDocFolder()`
- `deleteDocFolder()`
- `moveDoc()` - move between folders

**Enhanced functions:**
- `createDoc()` - support parentPath for folder placement

### Phase 3: Types & Store Updates
**Files:**
- `src/types/index.ts` - Note → Doc, add DocFolder, DocTreeNode
- `src/stores/docs.ts`

**New hooks:**
- `useDocTree(scope, workspaceId?, projectId?)`
- `useCreateDocFolder()`
- `useDeleteDocFolder()`

### Phase 4: Tree Component
**Files:**
- `src/components/docs/doc-tree.tsx`
- `src/components/docs/doc-tree-item.tsx`
- `src/components/docs/index.ts`

**Features:**
- Recursive folder rendering
- Expand/collapse with chevrons
- Click to select doc
- Context menu (rename, delete, new subfolder)

### Phase 5: Editor Updates
**Files:**
- `src/components/docs/doc-editor.tsx` (adapt existing note-editor)

**Updates:**
- Shows folder path breadcrumb
- Works with tree selection

### Phase 6: Workspace Docs Page (with tabs)
**Files:**
- `src/app/docs/page.tsx` - wrapper
- `src/app/docs/client.tsx` - main component

**Features:**
- Two tabs: "Workspace" and "All"
- "Workspace" tab: Tree view of workspace-level docs (`workspaces/{id}/docs/`)
- "All" tab: Aggregate view of all project docs (grouped by project, like current Notes page)

### Phase 7: Personal Docs Page
**Files:**
- `src/app/personal/docs/page.tsx` - wrapper
- `src/app/personal/docs/client.tsx` - tree view for personal docs

### Phase 8: Project Docs Tab
**Files:**
- `src/app/projects/view/client.tsx` - update Notes tab → Docs tab with tree view

### Phase 9: Drag & Drop Import
**Files:**
- `src/components/docs/doc-drop-zone.tsx`

### Phase 10: Folder Import
**Files:**
- `src/components/docs/import-folder-modal.tsx`
- Add `importFolder()` to docs.ts

### Phase 11: View State & Polish
- Persist expanded folders in `.view.json`
- Empty states with helpful hints
- File watcher integration for external changes

## Key Files to Modify

| File | Changes |
|------|---------|
| **Routes** | |
| `src/app/notes/` → `src/app/docs/` | Rename folder, update to tree+tabs layout |
| `src/app/personal/notes/` → `src/app/personal/docs/` | Rename folder, update to tree layout |
| `src/app/projects/view/client.tsx` | Rename Notes tab → Docs, add tree view |
| **Components** | |
| `src/components/notes/` → `src/components/docs/` | Rename folder |
| `src/components/docs/doc-tree.tsx` | NEW: Tree component |
| `src/components/docs/doc-editor.tsx` | Adapt from note-editor |
| `src/components/layout/sidebar.tsx` | "Notes" → "Docs" in nav |
| **Library** | |
| `src/lib/orbit/notes.ts` → `docs.ts` | Rename, add tree building, folder ops |
| `src/lib/orbit/constants.ts` | `notes` → `docs` in PATH_SEGMENTS |
| **Types & Store** | |
| `src/types/index.ts` | Note → Doc, add DocFolder, DocTreeNode |
| `src/stores/notes.ts` → `docs.ts` | Rename, add tree hooks, folder mutations |

## Data Path Strategy (Clean)

**Everything renamed to `docs/`:**

| Level | Data Path |
|-------|-----------|
| Personal Docs | `personal/docs/*.md` |
| Workspace Docs | `workspaces/{id}/docs/*.md` |
| Project Docs | `workspaces/{id}/projects/{id}/docs/*.md` |

**Migration:** User manually renames `notes/` → `docs/` in their Orbit data folder. One-time, simple.

## Verification

1. **Rename check**: All UI shows "Docs" instead of "Notes"
2. **Create folder**: In Docs, create a folder, create doc inside
3. **Tree navigation**: Expand/collapse folders, select docs
4. **Edit doc**: Select doc, edit content, verify auto-save works
5. **Import file**: Drag .md file onto tree, verify it imports
6. **Import folder**: Import a folder with nested .md files
7. **Workspace tabs**: `/docs` page shows both "Workspace" and "All" tabs correctly

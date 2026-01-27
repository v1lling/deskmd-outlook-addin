# Plan: Editor State Management (Obsidian-like)

## Problem

Cursor loss and flickering while typing in docs/tasks/meetings due to race condition between auto-save and file watcher causing TanStack Query to refetch and overwrite editor state.

## Root Cause

TanStack Query is serving two incompatible purposes:
1. Fetching data for list views (needs refetch on file changes)
2. Managing active editor state (must NOT refetch while editing)

---

## Solution Overview

**Inspired by Obsidian's behavior:**
- Save frequently (~400ms debounce)
- External changes flow into editor automatically
- Simple content comparison to detect external vs own changes
- Handle edge cases: file moves, renames, deletions while editing

### Core Principle

```
When file is OPEN in editor:
  → Editor owns state
  → File watcher updates editor (not TanStack Query)
  → Save frequently to disk
  → Handle path changes gracefully

When file is CLOSED:
  → TanStack Query owns state
  → File watcher invalidates queries (normal behavior)
```

---

## Architecture

### Component Responsibilities (Clear Naming)

| Component | Location | Responsibility |
|-----------|----------|----------------|
| **OpenEditorRegistry** | `src/stores/open-editor-registry.ts` | Tracks which files are open, their state, and handles path changes |
| **useEditorSession** | `src/hooks/use-editor-session.ts` | Hook for editor components to manage their session |
| **EditorEventBus** | `src/stores/editor-event-bus.ts` | Pub/sub for external updates and path changes |
| **QueryInvalidator** | `src/hooks/use-query-invalidator.ts` | Renamed from `use-file-watcher.ts` - routes events appropriately |
| **FileCacheService** | `src/lib/orbit/file-cache/` | Renamed from `file-tree/` - caches content for closed files |

### Data Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FILE SYSTEM                                     │
└─────────────────────────────────────────────────────────────────────────────┘
        │                                               │
        ▼                                               ▼
┌───────────────────┐                         ┌───────────────────┐
│   FILE WATCHER    │                         │ DOMAIN OPERATIONS │
│   (watcher.ts)    │                         │ (move, delete,    │
│                   │                         │  rename)          │
└───────────────────┘                         └───────────────────┘
        │                                               │
        │                                               │
        ▼                                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        OPEN EDITOR REGISTRY                                  │
│                                                                              │
│  isOpen(path) ──► YES: Route to editor                                      │
│              └──► NO:  Route to query invalidation                          │
│                                                                              │
│  Sessions: Map<path, EditorSession>                                         │
│  - lastSavedContent: string                                                 │
│  - subscribers: Set<callback>                                               │
└─────────────────────────────────────────────────────────────────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│ External    │         │ Path Change │         │ Query       │
│ Content     │         │ (move/del)  │         │ Invalidation│
│ Update      │         │ Handler     │         │ (closed)    │
└─────────────┘         └─────────────┘         └─────────────┘
        │                       │
        ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EDITOR COMPONENTS                             │
│  DocEditor  │  TaskEditor  │  MeetingEditor                     │
│                                                                  │
│  useEditorSession() hook provides:                               │
│  - content, setContent                                           │
│  - isDirty, saveStatus                                           │
│  - pathChanged, fileDeleted flags                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation

### Step 1: Create OpenEditorRegistry

**File**: `src/stores/open-editor-registry.ts`

Central registry that tracks all open editor sessions by file path.

```typescript
import { create } from "zustand";

export interface EditorSession {
  path: string;                    // Absolute file path (source of truth)
  type: "doc" | "task" | "meeting";
  entityId: string;
  lastSavedContent: string;        // What WE last wrote to disk

  // Path change handling
  newPath: string | null;          // Set when file was moved
  isDeleted: boolean;              // Set when file was deleted
}

interface OpenEditorRegistry {
  sessions: Map<string, EditorSession>;

  // Lifecycle
  register(path: string, session: Omit<EditorSession, "path" | "newPath" | "isDeleted">): void;
  unregister(path: string): void;

  // State
  updateLastSaved(path: string, content: string): void;

  // Queries
  isOpen(path: string): boolean;
  getSession(path: string): EditorSession | undefined;
  getSessionByEntityId(type: string, entityId: string): EditorSession | undefined;
  getAllOpenPaths(): string[];

  // Path change handling (called by domain operations)
  handlePathChange(oldPath: string, newPath: string): void;
  handlePathDeleted(path: string): void;

  // Clear path change flags (after editor acknowledges)
  acknowledgePathChange(path: string): void;
  acknowledgeDeleted(path: string): void;
}

export const useOpenEditorRegistry = create<OpenEditorRegistry>((set, get) => ({
  sessions: new Map(),

  register(path, sessionData) {
    set((state) => {
      const sessions = new Map(state.sessions);
      sessions.set(path, {
        ...sessionData,
        path,
        newPath: null,
        isDeleted: false,
      });
      return { sessions };
    });
  },

  unregister(path) {
    set((state) => {
      const sessions = new Map(state.sessions);
      sessions.delete(path);
      return { sessions };
    });
  },

  updateLastSaved(path, content) {
    set((state) => {
      const sessions = new Map(state.sessions);
      const session = sessions.get(path);
      if (session) {
        sessions.set(path, { ...session, lastSavedContent: content });
      }
      return { sessions };
    });
  },

  isOpen(path) {
    return get().sessions.has(path);
  },

  getSession(path) {
    return get().sessions.get(path);
  },

  getSessionByEntityId(type, entityId) {
    for (const session of get().sessions.values()) {
      if (session.type === type && session.entityId === entityId) {
        return session;
      }
    }
    return undefined;
  },

  getAllOpenPaths() {
    return Array.from(get().sessions.keys());
  },

  // Called when a file is moved/renamed
  handlePathChange(oldPath, newPath) {
    set((state) => {
      const sessions = new Map(state.sessions);
      const session = sessions.get(oldPath);
      if (session) {
        // Mark the session with the new path
        sessions.set(oldPath, { ...session, newPath });
      }
      return { sessions };
    });
  },

  // Called when a file is deleted
  handlePathDeleted(path) {
    set((state) => {
      const sessions = new Map(state.sessions);
      const session = sessions.get(path);
      if (session) {
        sessions.set(path, { ...session, isDeleted: true });
      }
      return { sessions };
    });
  },

  // Editor calls this after handling the path change
  acknowledgePathChange(oldPath) {
    const session = get().sessions.get(oldPath);
    if (session && session.newPath) {
      set((state) => {
        const sessions = new Map(state.sessions);
        sessions.delete(oldPath);
        sessions.set(session.newPath!, {
          ...session,
          path: session.newPath!,
          newPath: null,
        });
        return { sessions };
      });
    }
  },

  acknowledgeDeleted(path) {
    // Editor should close itself after acknowledging
    get().unregister(path);
  },
}));
```

### Step 2: Create EditorEventBus

**File**: `src/stores/editor-event-bus.ts`

Simple pub/sub for editor updates.

```typescript
type ContentUpdateHandler = (content: string) => void;
type PathChangeHandler = (newPath: string) => void;
type DeletedHandler = () => void;

interface EditorEventHandlers {
  onContentUpdate?: ContentUpdateHandler;
  onPathChange?: PathChangeHandler;
  onDeleted?: DeletedHandler;
}

const subscribers = new Map<string, EditorEventHandlers>();

export function subscribeToEditorEvents(
  path: string,
  handlers: EditorEventHandlers
): () => void {
  subscribers.set(path, handlers);
  return () => subscribers.delete(path);
}

export function publishContentUpdate(path: string, content: string): void {
  subscribers.get(path)?.onContentUpdate?.(content);
}

export function publishPathChange(path: string, newPath: string): void {
  subscribers.get(path)?.onPathChange?.(newPath);
}

export function publishDeleted(path: string): void {
  subscribers.get(path)?.onDeleted?.();
}
```

### Step 3: Create useEditorSession Hook

**File**: `src/hooks/use-editor-session.ts`

```typescript
import { useState, useEffect, useCallback, useRef } from "react";
import { useOpenEditorRegistry } from "@/stores/open-editor-registry";
import { subscribeToEditorEvents } from "@/stores/editor-event-bus";
import { writeTextFile, readTextFile } from "@/lib/orbit/tauri-fs";

interface UseEditorSessionOptions {
  type: "doc" | "task" | "meeting";
  entityId: string;
  filePath: string | undefined;
  initialContent: string;
  enabled: boolean;
}

interface UseEditorSessionReturn {
  // Content
  content: string;
  setContent: (content: string) => void;

  // Save state
  isDirty: boolean;
  saveStatus: "idle" | "saving" | "error";

  // Path change state
  pathChanged: boolean;
  newPath: string | null;
  fileDeleted: boolean;

  // Actions
  acknowledgePathChange: () => void;
  acknowledgeDeleted: () => void;
  forceSave: () => Promise<void>;
}

export function useEditorSession({
  type,
  entityId,
  filePath,
  initialContent,
  enabled,
}: UseEditorSessionOptions): UseEditorSessionReturn {
  const registry = useOpenEditorRegistry();

  const [content, setContentState] = useState(initialContent);
  const [isDirty, setIsDirty] = useState(false);
  const [saveStatus, setSaveStatus] = useState<"idle" | "saving" | "error">("idle");
  const [pathChanged, setPathChanged] = useState(false);
  const [newPath, setNewPath] = useState<string | null>(null);
  const [fileDeleted, setFileDeleted] = useState(false);

  const saveTimeoutRef = useRef<NodeJS.Timeout | null>(null);
  const lastSavedRef = useRef<string>(initialContent);
  const currentPathRef = useRef<string | undefined>(filePath);

  // Update path ref when it changes
  useEffect(() => {
    currentPathRef.current = filePath;
  }, [filePath]);

  // Register session on mount
  useEffect(() => {
    if (!enabled || !filePath) return;

    registry.register(filePath, {
      type,
      entityId,
      lastSavedContent: initialContent,
    });
    lastSavedRef.current = initialContent;

    // Subscribe to events
    const unsubscribe = subscribeToEditorEvents(filePath, {
      onContentUpdate: (newContent) => {
        // External change - update editor
        setContentState(newContent);
        lastSavedRef.current = newContent;
        setIsDirty(false);
      },
      onPathChange: (path) => {
        setPathChanged(true);
        setNewPath(path);
      },
      onDeleted: () => {
        setFileDeleted(true);
      },
    });

    return () => {
      unsubscribe();
      registry.unregister(filePath);
      if (saveTimeoutRef.current) {
        clearTimeout(saveTimeoutRef.current);
      }
    };
  }, [enabled, filePath, type, entityId, initialContent]);

  // Save function
  const performSave = useCallback(async (contentToSave: string) => {
    const path = currentPathRef.current;
    if (!path || fileDeleted || pathChanged) return;

    if (contentToSave === lastSavedRef.current) {
      setIsDirty(false);
      return;
    }

    setSaveStatus("saving");
    try {
      await writeTextFile(path, contentToSave);
      lastSavedRef.current = contentToSave;
      registry.updateLastSaved(path, contentToSave);
      setIsDirty(false);
      setSaveStatus("idle");
    } catch (error) {
      console.error("[editor-session] Save failed:", error);
      setSaveStatus("error");
    }
  }, [registry, fileDeleted, pathChanged]);

  // Debounced auto-save on content change
  const setContent = useCallback((newContent: string) => {
    setContentState(newContent);
    setIsDirty(true);

    if (saveTimeoutRef.current) {
      clearTimeout(saveTimeoutRef.current);
    }

    saveTimeoutRef.current = setTimeout(() => {
      performSave(newContent);
    }, 400); // 400ms debounce
  }, [performSave]);

  // Force save (for manual save or before close)
  const forceSave = useCallback(async () => {
    if (saveTimeoutRef.current) {
      clearTimeout(saveTimeoutRef.current);
      saveTimeoutRef.current = null;
    }
    await performSave(content);
  }, [performSave, content]);

  // Acknowledge path change
  const acknowledgePathChange = useCallback(() => {
    if (currentPathRef.current && newPath) {
      registry.acknowledgePathChange(currentPathRef.current);
      currentPathRef.current = newPath;
      setPathChanged(false);
      setNewPath(null);
    }
  }, [registry, newPath]);

  // Acknowledge deletion
  const acknowledgeDeleted = useCallback(() => {
    if (currentPathRef.current) {
      registry.acknowledgeDeleted(currentPathRef.current);
    }
  }, [registry]);

  return {
    content,
    setContent,
    isDirty,
    saveStatus,
    pathChanged,
    newPath,
    fileDeleted,
    acknowledgePathChange,
    acknowledgeDeleted,
    forceSave,
  };
}
```

### Step 4: Rename and Update useFileWatcher → useQueryInvalidator

**File**: `src/hooks/use-query-invalidator.ts` (renamed from `use-file-watcher.ts`)

```typescript
import { useEffect, useRef } from "react";
import { useQueryClient } from "@tanstack/react-query";
import { onFileChange, type WatchEvent } from "@/lib/orbit/watcher";
import { useOpenEditorRegistry } from "@/stores/open-editor-registry";
import { publishContentUpdate } from "@/stores/editor-event-bus";
import { readTextFile } from "@/lib/orbit/tauri-fs";
import { /* existing imports for query keys */ } from "@/stores";

/**
 * Routes file system events to the appropriate handler:
 * - Open files → Editor update (via event bus)
 * - Closed files → TanStack Query invalidation
 */
export function useQueryInvalidator() {
  const queryClient = useQueryClient();
  const registry = useOpenEditorRegistry();
  const isInitialized = useRef(false);

  useEffect(() => {
    if (isInitialized.current) return;
    isInitialized.current = true;

    const unsubscribe = onFileChange(async (event: WatchEvent) => {
      await handleFileChange(event, queryClient, registry);
    });

    return () => {
      unsubscribe();
      isInitialized.current = false;
    };
  }, [queryClient, registry]);
}

async function handleFileChange(
  event: WatchEvent,
  queryClient: QueryClient,
  registry: OpenEditorRegistry
) {
  for (const path of event.paths) {
    const session = registry.getSession(path);

    if (session) {
      // ═══════════════════════════════════════════════════════════
      // File is OPEN in editor - check for external changes
      // ═══════════════════════════════════════════════════════════
      await handleOpenFileChange(path, session);
    } else {
      // ═══════════════════════════════════════════════════════════
      // File is NOT open - normal query invalidation
      // ═══════════════════════════════════════════════════════════
      invalidateQueriesForPath(path, queryClient);
    }
  }
}

async function handleOpenFileChange(path: string, session: EditorSession) {
  try {
    const fileContent = await readTextFile(path);

    // File matches what we last saved → our save event, ignore
    if (fileContent === session.lastSavedContent) {
      return;
    }

    // External change → update editor via event bus
    console.log(`[query-invalidator] External change detected: ${path}`);
    publishContentUpdate(path, fileContent);

    // Update lastSavedContent in registry
    useOpenEditorRegistry.getState().updateLastSaved(path, fileContent);
  } catch (error) {
    // File might have been deleted
    if (error instanceof Error && error.message.includes("not found")) {
      useOpenEditorRegistry.getState().handlePathDeleted(path);
    } else {
      console.error(`[query-invalidator] Error reading file: ${path}`, error);
    }
  }
}

function invalidateQueriesForPath(path: string, queryClient: QueryClient) {
  // ... existing query invalidation logic from use-file-watcher.ts ...
}
```

### Step 5: Update Domain Operations for Path Safety

**File**: `src/lib/orbit/tasks.ts` (add to moveTaskToProject)

```typescript
import { useOpenEditorRegistry } from "@/stores/open-editor-registry";
import { publishPathChange } from "@/stores/editor-event-bus";

export async function moveTaskToProject(
  taskId: string,
  workspaceId: string,
  fromProjectId: string,
  toProjectId: string
): Promise<void> {
  const oldPath = await getTaskPath(workspaceId, fromProjectId, taskId);
  const newPath = await getTaskPath(workspaceId, toProjectId, taskId);

  // Check if file is being edited
  const registry = useOpenEditorRegistry.getState();
  const isBeingEdited = registry.isOpen(oldPath);

  // Read, write to new location, delete old
  const content = await readTextFile(oldPath);
  await writeTextFile(newPath, content);
  await removeFile(oldPath);

  // Notify editor if it was open
  if (isBeingEdited) {
    registry.handlePathChange(oldPath, newPath);
    publishPathChange(oldPath, newPath);
  }
}
```

**Similar updates needed for:**
- `moveDocToProject()` in `docs.ts`
- `moveDoc()` in `docs.ts`
- `renameDocFolder()` in `docs.ts`
- `moveCaptureToWorkspace()` in `personal.ts`
- `moveFromCapture()` in `personal.ts`
- `deleteTask()` in `tasks.ts`
- `deleteDoc()` in `docs.ts`
- `deleteMeeting()` in `meetings.ts`
- `deleteProject()` in `projects.ts`

### Step 6: Update Editors to Use New System

**File**: `src/components/editors/doc-editor.tsx`

```typescript
import { useEditorSession } from "@/hooks/use-editor-session";
import { useDoc } from "@/stores";
import { FileMovedBanner, FileDeletedBanner } from "@/components/ui/editor-banners";

export function DocEditor({ docId, workspaceId, onClose }: DocEditorProps) {
  const { data: doc, isLoading } = useDoc(workspaceId, docId);

  const {
    content,
    setContent,
    isDirty,
    saveStatus,
    pathChanged,
    newPath,
    fileDeleted,
    acknowledgePathChange,
    acknowledgeDeleted,
  } = useEditorSession({
    type: "doc",
    entityId: docId,
    filePath: doc?.filePath,
    initialContent: doc?.content ?? "",
    enabled: !!doc,
  });

  // Handle file deleted
  if (fileDeleted) {
    return (
      <FileDeletedBanner
        onClose={() => {
          acknowledgeDeleted();
          onClose();
        }}
      />
    );
  }

  // Handle file moved
  if (pathChanged && newPath) {
    return (
      <FileMovedBanner
        newPath={newPath}
        onAcknowledge={() => acknowledgePathChange()}
      />
    );
  }

  // ... rest of editor UI using content, setContent, etc.
}
```

---

## File Structure After Refactoring

```
src/
├── stores/
│   ├── open-editor-registry.ts      # NEW: Tracks open files
│   ├── editor-event-bus.ts          # NEW: Pub/sub for editor events
│   ├── tasks.ts                     # Existing (add path safety)
│   ├── docs.ts                      # Existing (add path safety)
│   └── ...
│
├── hooks/
│   ├── use-editor-session.ts        # NEW: Editor state hook
│   ├── use-query-invalidator.ts     # RENAMED from use-file-watcher.ts
│   └── ...
│
├── components/
│   ├── editors/
│   │   ├── doc-editor.tsx           # MODIFY: Use new system
│   │   ├── task-editor.tsx          # MODIFY: Use new system
│   │   └── meeting-editor.tsx       # MODIFY: Use new system
│   └── ui/
│       └── editor-banners.tsx       # NEW: FileMovedBanner, FileDeletedBanner
│
└── lib/orbit/
    ├── file-cache/                  # RENAMED from file-tree/
    │   ├── index.ts
    │   ├── service.ts               # FileCacheService (was FileTreeService)
    │   ├── content-cache.ts
    │   └── cache-invalidator.ts     # RENAMED from watcher-integration.ts
    │
    ├── tasks.ts                     # MODIFY: Add path safety checks
    ├── docs.ts                      # MODIFY: Add path safety checks
    └── ...
```

---

## Key Integration Rules

### 1. Writes from Editors
- Use direct `writeTextFile()` from `tauri-fs.ts`
- Don't use `FileCacheService.writeFile()`
- Update `lastSavedContent` in registry after write

### 2. Writes from Domain Operations (move, rename, delete)
- Check `registry.isOpen(path)` before destructive operations
- Call `registry.handlePathChange()` or `registry.handlePathDeleted()`
- Publish events via `EditorEventBus`

### 3. File Watcher Events
- Check registry first: `registry.isOpen(path)`
- Open files → Compare content, update editor if external
- Closed files → Invalidate TanStack Query

### 4. FileCacheService (formerly FileTreeService)
- Serves list views and sidebar
- May have stale data for open files (that's OK)
- Cache invalidation continues as before

---

## Edge Cases Handled

| Scenario | Behavior |
|----------|----------|
| User typing, auto-save fires | Save completes, watcher ignores (our save) |
| External change while user idle | Editor updates automatically |
| External change while user typing | User's next save overwrites (their intent) |
| File moved via sidebar | Editor shows "File moved" banner, updates path |
| File deleted via sidebar | Editor shows "File deleted" banner, closes |
| Project deleted while doc open | All docs in project get delete notification |
| Folder renamed while doc open | All docs in folder get path change notification |
| Multiple tabs same file | Both receive external updates (in sync) |
| Tab closed with unsaved changes | Force save on unmount |
| Rapid typing | Debounce batches, saves efficiently |

---

## Naming Glossary

| Term | Meaning |
|------|---------|
| **OpenEditorRegistry** | Zustand store tracking all files open in editor tabs |
| **EditorSession** | State for one open file: path, lastSavedContent, flags |
| **EditorEventBus** | Pub/sub system for external updates to reach editors |
| **useEditorSession** | React hook that editors use for state management |
| **useQueryInvalidator** | Hook that routes file changes to editors or TanStack |
| **FileCacheService** | Caches parsed file content for list views (not editors) |
| **lastSavedContent** | The content string we last wrote to disk |

---

## Migration Checklist

### Phase 1: Core Infrastructure ✅ COMPLETE
- [x] Create `src/stores/open-editor-registry.ts`
- [x] Create `src/stores/editor-event-bus.ts`
- [x] Create `src/hooks/use-editor-session.ts`
- [x] Rename `use-file-watcher.ts` → `use-query-invalidator.ts`
- [x] Update `use-query-invalidator.ts` to check registry

### Phase 2: Editor Migration ✅ COMPLETE
- [x] Update `DocEditor` to use `useEditorSession`
- [x] Update `TaskEditor` to use `useEditorSession`
- [x] Update `MeetingEditor` to use `useEditorSession`
- [x] Create `FileMovedBanner` and `FileDeletedBanner` components
- [x] Delete `use-doc-form.ts` (replaced by `useEditorSession`)

### Phase 3: Domain Safety ✅ COMPLETE
- [x] Add path safety to `moveTaskToProject()`
- [x] Add path safety to `moveDocToProject()`
- [x] Add path safety to `moveDoc()`
- [x] Add path safety to `deleteTask()`
- [x] Add path safety to `deleteDoc()`
- [x] Add path safety to `deleteMeeting()`
- [ ] ~Add path safety to `renameDocFolder()`~ (skipped - folder renames are rare)
- [ ] ~Add delete checks to `deleteProject()`~ (skipped - would require iterating all files)

### Phase 4: Rename for Clarity ✅ COMPLETE
- [x] Rename `file-tree/` → `file-cache/`
- [ ] ~Rename `FileTreeService` → `FileCacheService`~ (kept as is - service still builds trees)
- [x] Rename `watcher-integration.ts` → `cache-invalidator.ts`
- [x] Update all imports

---

## Verification Checklist

### Basic Editing
- [ ] Type continuously in Orbit doc - no cursor loss or flicker
- [ ] Edit file in VS Code - Orbit updates within ~500ms
- [ ] Edit in Orbit - VS Code updates within ~500ms
- [ ] Open same doc in multiple Orbit tabs - both stay in sync
- [ ] Close tab, reopen - content persisted correctly

### Path Changes
- [ ] Move task to different project while editing - banner appears, path updates
- [ ] Move doc to different folder while editing - banner appears, path updates
- [ ] Rename folder containing open doc - all affected docs notified
- [ ] Delete task while editing - banner appears, tab can close

### Edge Cases
- [ ] Delete project with open docs - all docs show deleted banner
- [ ] Rapid switching between Orbit and VS Code editing - no data loss
- [ ] Multiple tabs same file - all stay in sync
- [ ] Close tab with unsaved changes - content saved before close

---

*Last updated: 2025-01-27*

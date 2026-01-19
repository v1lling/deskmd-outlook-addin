# Plan: Filesystem Layer & Global Search

## Goal
Build a solid foundation for file watching, search, and auto-save.

---

## Phase 1: File Watcher
**Status:** Complete ✓

- ✓ Add Tauri fs-watch plugin (enabled `watch` feature in Cargo.toml)
- ✓ Watch `~/Orbit/` directory recursively
- ✓ On file change → invalidate relevant TanStack Query caches
- ✓ Handle create/modify/delete events
- ✓ Debounce rapid changes (150ms)

**Files created:**
- `src/lib/orbit/watcher.ts` - Core watcher service
- `src/hooks/use-file-watcher.ts` - React hook for cache invalidation

**Outcome:** UI auto-updates when files change externally (e.g., edited in VS Code)

---

## Phase 2: Search Index
**Status:** Complete ✓

- ✓ Build in-memory index on app startup
- ✓ Index: title, content preview, metadata (status, priority, due date)
- ✓ Update index via file watcher events
- ✓ Support fuzzy title matching (fuse.js)

**Files created:**
- `src/lib/orbit/search-index.ts` - In-memory index with Fuse.js
- `src/hooks/use-search-index.ts` - React hook for index management

**Outcome:** Fast search across all items without N×M file scans

---

## Phase 3: Cmd+K Global Search
**Status:** Complete ✓

- ✓ Command palette UI (cmdk + shadcn Command component)
- ✓ Search across tasks, notes, projects, meetings
- ✓ Recent items section (when no query)
- ✓ Keyboard navigation (arrows, enter, esc)
- ✓ Navigate to item on selection
- ✓ Search button in header with ⌘K hint

**Files created:**
- `src/components/ui/command.tsx` - Command palette primitives
- `src/components/global-search.tsx` - Global search component

**Outcome:** Users can find anything instantly with Cmd+K

---

## Phase 4: Auto-Save (Optional)
**Status:** Not started

- Debounced writes (buffer 1-2 seconds)
- "Saving..." / "Saved" indicator
- Handle conflicts (file changed while editing)
- Remove explicit Save button from editors

**Outcome:** Obsidian-like instant persistence

---

## Dependencies

```
Phase 1 (Watcher) ──┬──→ Phase 2 (Index) ──→ Phase 3 (Cmd+K)
                    │
                    └──→ Phase 4 (Auto-save)
```

Watcher is the foundation. Index and auto-save both depend on it.

---

## Notes
- Browser mode won't have watcher (Tauri only)
- Consider SQLite/IndexedDB for persistent index later
- Keep index simple first, optimize if needed

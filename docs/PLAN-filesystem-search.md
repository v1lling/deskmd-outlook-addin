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
**Status:** Not started

- Build in-memory index on app startup
- Index: title, content preview, metadata (status, priority, due date)
- Update index via file watcher events
- Support fuzzy title matching
- Support content search (grep-like)

**Outcome:** Fast search across all items without N×M file scans

---

## Phase 3: Cmd+K Global Search
**Status:** Not started

- Command palette UI (shadcn Command component)
- Search across tasks, notes, projects, meetings
- Recent items section
- Keyboard navigation (arrows, enter, esc)
- Navigate to item on selection

**Outcome:** Users can find anything instantly

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

# File Cache Layer

> **Status**: ✅ Phases 1-4 Complete | ⚠️ Phase 5 Partial | ❌ Phase 6 Not Started
>
> **Related**: [EDITOR-STATE-MANAGEMENT.md](./EDITOR-STATE-MANAGEMENT.md) (✅ Complete)

---

## What This Layer Does

Provides efficient file system access with caching for **list views and sidebar** (closed files). It does NOT manage state for files open in editor tabs.

**Location**: `src/lib/orbit/file-cache/`

## Current Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     FILE SYSTEM (tauri-fs.ts)                    │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────────────┐
│   FileTreeService        │    │   OpenEditorRegistry (PLANNED)   │
│   (file-tree/service.ts) │    │   (see Editor State Management)  │
│                          │    │                                  │
│   → Caches CLOSED files  │    │   → Manages OPEN files in tabs   │
│   → Serves list views    │    │   → Handles external changes     │
│   → Tree building        │    │   → Path move/delete safety      │
└──────────────────────────┘    └──────────────────────────────────┘
```

---

## Implementation Status

### ✅ Phase 1: Core Service (COMPLETE)
- [x] `service.ts` - FileTreeService singleton
- [x] `tree-builder.ts` - Recursive tree building
- [x] `types.ts` - TreeNode, CachedContent types

### ✅ Phase 2: Watcher Integration (COMPLETE)
- [x] `cache-invalidator.ts` - Connects watcher to cache (renamed from watcher-integration.ts)
- [x] Cache invalidation on file changes

### ✅ Phase 3: Content Caching (COMPLETE)
- [x] `content-cache.ts` - LRU cache (50MB limit, 30min TTL)
- [x] Invalidation on modify events
- [x] Prefix invalidation for directories

### ✅ Phase 4: React Hooks (COMPLETE)
- [x] `hooks.ts` - useFileTree, useFileContent

### ⚠️ Phase 5: Domain Migration (PARTIAL)
- [x] `docs.ts` uses getContentByAbsolutePath()
- [x] `tasks.ts` uses getContentByAbsolutePath()
- [x] `meetings.ts` uses getContentByAbsolutePath()
- [ ] Remove remaining direct file I/O in domains

### ❌ Phase 6: Optimization (NOT STARTED)
- [ ] Preloading for likely-needed content
- [ ] Partial tree loading at startup
- [ ] Web Worker for large file parsing

---

## Key Files

| File | Purpose | Status |
|------|---------|--------|
| `service.ts` | Main service: getTree, getContent, writeFile | ✅ |
| `content-cache.ts` | LRU cache with size/age limits | ✅ |
| `tree-builder.ts` | Builds TreeNode from filesystem | ✅ |
| `cache-invalidator.ts` | Invalidates cache on file changes | ✅ |
| `hooks.ts` | React hooks for components | ✅ |
| `types.ts` | TypeScript interfaces | ✅ |
| `parsers.ts` | Markdown/JSON parsers | ✅ |

---

## Completed Renames (Editor State Management Phase 4)

| Old Name | New Name | Status |
|----------|----------|--------|
| `file-tree/` | `file-cache/` | ✅ Done |
| `FileTreeService` | — | Kept as-is (still builds trees) |
| `watcher-integration.ts` | `cache-invalidator.ts` | ✅ Done |

---

## Integration with Editor State Management

When the Editor State Management plan is implemented:

**File OPEN in editor tab:**
- OpenEditorRegistry owns the state
- FileTreeService cache may be stale (that's OK)
- File watcher routes changes to editor, not cache

**File CLOSED:**
- FileTreeService serves content from cache
- File watcher invalidates cache
- TanStack Query refetches via FileTreeService

**Important**: Editors will use direct `writeTextFile()`, NOT `FileTreeService.writeFile()`, to avoid triggering cache notifications that could cause loops.

---

## Original Problem (Solved)

```
BEFORE: File Change → Watcher → Invalidate ALL queries → Re-read 50 files
AFTER:  File Change → Watcher → Invalidate 1 cache entry → Re-read 1 file
```

---

## API Reference

```typescript
// Get tree structure
getTree(relativePath?: string): Promise<TreeNode | null>
getChildren(relativePath: string): Promise<TreeNode[]>
getNode(relativePath: string): Promise<TreeNode | null>

// Get file content (cached)
getContent<T>(relativePath: string, parser?: ContentParser<T>): Promise<T | null>
getContentByAbsolutePath<T>(path: string, parser?): Promise<T | null>

// Write operations (updates cache + disk)
writeFile(relativePath: string, content: string): Promise<void>
createDirectory(relativePath: string): Promise<void>
deleteNode(relativePath: string): Promise<void>
renameNode(oldPath: string, newPath: string): Promise<void>

// Cache management
getCacheStats(): CacheStats
clearCache(): void
invalidateCache(relativePath: string): void
```

---

## Next Steps

1. ~~**Complete Editor State Management**~~ ✅ Done
2. ~~Phase 4 includes renaming this layer for clarity~~ ✅ Done
3. Phase 5 migration can continue (remove remaining direct file I/O)
4. Phase 6 optimization is lower priority

---

*Last updated: 2025-01-27*

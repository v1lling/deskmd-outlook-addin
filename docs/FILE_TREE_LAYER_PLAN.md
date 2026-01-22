# File Tree Layer - Architecture Plan

## Problem Statement

### Current Issues
1. **Redundant file reads**: Opening a doc, closing it, and reopening triggers full re-read from disk
2. **No content caching**: Large markdown files are parsed repeatedly
3. **Inefficient invalidation**: Any file change invalidates entire query, causing re-read of ALL files
4. **Scattered traversal logic**: Each domain (docs, tasks, meetings) has its own file iteration code

### Example Performance Issue
```
User opens large doc (500KB) → Parsed from disk
User closes panel
User reopens same doc → Parsed from disk AGAIN (no cache)
```

### Current Data Flow
```
File Change → Watcher → Invalidate TanStack Query → Re-fetch ALL files → Parse ALL → UI Update
```

**Problem**: Changing 1 file triggers re-reading 50 files.

## Proposed Architecture

### New Layer: File Tree Service

```
┌─────────────────────────────────────────────────────────────┐
│                      File System                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  File Tree Service (NEW)                                     │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Watcher Integration                                     ││
│  │  - Owns the file watcher                                 ││
│  │  - Receives create/modify/delete events                  ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  In-Memory Tree                                          ││
│  │  - Mirrors ~/Orbit directory structure                   ││
│  │  - Nodes: { path, type, metadata, content?, children? }  ││
│  │  - Lazy loading: metadata always, content on-demand      ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Content Cache (LRU)                                     ││
│  │  - Parsed file contents                                  ││
│  │  - Eviction based on size/age                            ││
│  │  - Invalidated on file modify                            ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Query API                                               ││
│  │  - getTree(path) → TreeNode[]                            ││
│  │  - getFile(path) → { metadata, content }                 ││
│  │  - subscribe(path, callback) → unsubscribe              ││
│  └─────────────────────────────────────────────────────────┘│
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  Domain Libraries (docs.ts, tasks.ts, etc.)                  │
│  - Query tree service instead of direct file I/O            │
│  - Apply domain-specific transformations                     │
│  - No longer own traversal logic                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  TanStack Query / Stores                                     │
│  - Still provides React integration                          │
│  - Queries backed by tree service (fast, cached)            │
└─────────────────────────────────────────────────────────────┘
```

### New Data Flow
```
File Change → Tree Service detects → Update ONLY affected node → Notify subscribers → UI Update
```

**Improvement**: Changing 1 file re-reads only 1 file.

## Detailed Design

### 1. Tree Node Structure

```typescript
interface TreeNode {
  path: string;           // Absolute path
  relativePath: string;   // Path relative to Orbit root
  name: string;           // File/folder name
  type: "file" | "directory";

  // For files
  extension?: string;     // ".md", ".json", etc.
  size?: number;          // File size in bytes
  mtime?: number;         // Last modified timestamp

  // For directories
  children?: TreeNode[];  // Lazy loaded
  childrenLoaded?: boolean;

  // Parsed content (cached separately)
  // Not stored in tree - fetched via getContent()
}
```

### 2. Content Cache

```typescript
interface CachedContent {
  path: string;
  mtime: number;          // For invalidation
  raw: string;            // Raw file content
  parsed: unknown;        // Domain-specific parsed data
  size: number;           // For LRU eviction
}

interface ContentCache {
  get(path: string): CachedContent | null;
  set(path: string, content: CachedContent): void;
  invalidate(path: string): void;

  // LRU management
  maxSize: number;        // e.g., 50MB
  currentSize: number;
  evictOldest(): void;
}
```

### 3. File Tree Service API

```typescript
interface FileTreeService {
  // Lifecycle
  initialize(): Promise<void>;
  shutdown(): void;

  // Tree queries
  getTree(path?: string): Promise<TreeNode>;
  getChildren(path: string): Promise<TreeNode[]>;

  // File content (with caching)
  getContent<T>(path: string, parser?: (raw: string) => T): Promise<T>;

  // Subscriptions (for React integration)
  subscribe(path: string, callback: (node: TreeNode) => void): () => void;
  subscribeTree(path: string, callback: (tree: TreeNode[]) => void): () => void;

  // Write operations (updates cache + disk)
  writeFile(path: string, content: string): Promise<void>;
  createDirectory(path: string): Promise<void>;
  delete(path: string): Promise<void>;
  rename(oldPath: string, newPath: string): Promise<void>;
}
```

### 4. React Integration

```typescript
// New hooks that use tree service
function useTreeNode(path: string): TreeNode | null;
function useTreeChildren(path: string): TreeNode[];
function useFileContent<T>(path: string, parser?: (raw: string) => T): T | null;

// Example usage in docs
function useDoc(docPath: string) {
  return useFileContent(docPath, parseMarkdownDoc);
}

function useDocTree(basePath: string) {
  const children = useTreeChildren(basePath);
  return children.filter(node => node.extension === ".md" || node.type === "directory");
}
```

## Implementation Phases

### Phase 1: Core Tree Service (Foundation)
- [ ] Create `src/lib/orbit/file-tree/` directory structure
- [ ] Implement basic `TreeNode` and tree building from disk
- [ ] Implement `getTree()` and `getChildren()` (no caching yet)
- [ ] Add unit tests

### Phase 2: Watcher Integration
- [ ] Move watcher logic into tree service
- [ ] Implement incremental tree updates on file events
- [ ] Handle edge cases: renames, moves, rapid changes
- [ ] Remove old `use-file-watcher.ts` (logic moves to service)

### Phase 3: Content Caching
- [ ] Implement LRU content cache
- [ ] Add `getContent()` with cache lookup
- [ ] Invalidate cache on file modify events
- [ ] Add cache stats/monitoring

### Phase 4: React Hooks
- [ ] Create `useTreeNode`, `useTreeChildren`, `useFileContent` hooks
- [ ] Implement subscription-based reactivity
- [ ] Integration with existing TanStack Query (or replace)

### Phase 5: Migrate Domains
- [ ] Migrate `docs.ts` to use tree service
- [ ] Migrate `tasks.ts` to use tree service
- [ ] Migrate `meetings.ts`, `projects.ts`, `workspaces.ts`
- [ ] Remove duplicated traversal code from domains
- [ ] Update stores to use new hooks

### Phase 6: Optimization
- [ ] Add preloading for likely-needed content
- [ ] Implement partial tree loading (don't load entire Orbit at startup)
- [ ] Profile and optimize hot paths
- [ ] Consider Web Worker for parsing large files

## File Structure

```
src/lib/orbit/file-tree/
├── index.ts              # Public exports
├── types.ts              # TreeNode, CachedContent, etc.
├── service.ts            # FileTreeService implementation
├── tree-builder.ts       # Build tree from disk
├── watcher.ts            # File watcher integration (moved from orbit/)
├── content-cache.ts      # LRU content cache
├── parsers/
│   ├── index.ts
│   ├── markdown.ts       # Markdown frontmatter parser
│   └── json.ts           # JSON parser
└── hooks/
    ├── index.ts
    ├── use-tree-node.ts
    ├── use-tree-children.ts
    └── use-file-content.ts
```

## Migration Strategy

### Gradual Migration
1. Build tree service alongside existing code
2. Add feature flag to switch between old/new
3. Migrate one domain at a time
4. Verify performance improvements
5. Remove old code once stable

### Backwards Compatibility
- Keep existing TanStack Query keys working during transition
- Domain libraries can internally switch to tree service
- External API remains unchanged initially

## Performance Targets

| Metric | Current | Target |
|--------|---------|--------|
| Open same doc twice | 2x disk read | 1x disk read (cached) |
| Single file change | Re-read all files | Re-read 1 file |
| Large doc (500KB) | ~100ms parse each time | ~100ms first, ~1ms cached |
| Startup time | Load all metadata | Load on-demand |

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Tree gets out of sync | Watcher handles all file events; periodic full refresh as fallback |
| Memory usage grows | LRU cache with size limit; evict old content |
| Complexity increase | Phased rollout; feature flags; comprehensive tests |
| Race conditions | Single service instance; queue file operations |

## Open Questions

1. **Should tree service be a singleton or created per-workspace?**
   - Singleton is simpler; per-workspace allows workspace-specific caching

2. **How to handle very large files (>1MB)?**
   - Stream parsing? Don't cache content, only metadata?

3. **Should we use Web Workers for parsing?**
   - Prevents UI blocking on large files
   - Adds complexity

4. **Integration with TanStack Query vs. replace?**
   - TanStack Query provides nice devtools and patterns
   - Could wrap tree service in TanStack queries
   - Or use Zustand/custom subscriptions directly

## Next Steps

1. Review this plan
2. Decide on Phase 1 scope
3. Create `file-tree/` directory structure
4. Implement basic tree building
5. Add tests
6. Iterate

---

*Last updated: 2026-01-22*

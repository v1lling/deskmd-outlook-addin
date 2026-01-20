# Orbit Codebase Review - Production Readiness

> Review Date: 2025-01-19
> Current State: v0.3 (70% production-ready)

## Executive Summary

The codebase demonstrates **solid architectural foundations** with excellent patterns (TanStack Query factory, file abstraction layer, design tokens). However, it needs **production hardening** in testing, error handling, and validation.

**Overall Score: 7/10** - Good foundation, needs polish.

---

## Strengths

### 1. Architecture (9/10)
- **Clean layered structure**: `lib/orbit/` (data), `stores/` (state), `components/` (UI), `hooks/` (cross-cutting)
- **Single responsibility**: Each module has focused purpose
- **Excellent abstraction**: Tauri/browser runtime handled transparently via `tauri-fs.ts`

### 2. State Management (8/10)
- **Hybrid approach**: TanStack Query for server state, Zustand for UI state
- **Query key factory pattern**: Prevents cache misses, enables precise invalidation
- **Optimistic updates**: Implemented for drag-drop operations

### 3. Data Layer (8/10)
- **Portable markdown storage**: YAML frontmatter + content
- **Convention-based paths**: Predictable file locations via constants
- **Smart mock system**: Development works without Tauri

### 4. Code Organization (8/10)
- **Feature-based components**: Easy to navigate
- **Centralized types**: `src/types/index.ts`
- **Design tokens**: `src/lib/design-tokens.ts`

---

## Issues to Address

### Critical (Must Fix)

| Issue | Location | Impact |
|-------|----------|--------|
| No tests | Entire codebase | Can't refactor safely |
| No error boundaries | React components | App crashes on errors |
| Type safety gaps | `parser.ts:21` uses `any` | Runtime errors possible |
| Inconsistent error handling | Multiple files | Errors silently swallowed |

### Important (Should Fix)

| Issue | Location | Impact |
|-------|----------|--------|
| Console.log in production code | `tauri-fs.ts`, `watcher.ts` | Noisy logs, potential info leak |
| No runtime validation | Markdown parsing | Malformed files cause issues |
| Missing loading states | Various components | Poor UX |
| No structured logging | Throughout | Hard to debug in production |

### Minor (Nice to Have)

| Issue | Location | Impact |
|-------|----------|--------|
| Some large components | `kanban-board.tsx` | Harder to maintain |
| Query naming inconsistency | `useProjectTasks` vs `useTask` | Minor confusion |

---

## Actionable Tasks

### Phase 1: Foundation (Do Now)

These tasks establish the baseline for a clean codebase:

#### 1.1 Add Testing Infrastructure
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

**Files to create:**
- `vitest.config.ts` - Test configuration
- `src/lib/orbit/__tests__/parser.test.ts` - Parser unit tests
- `src/lib/orbit/__tests__/tasks.test.ts` - Task operations tests

**Why:** Can't safely refactor without tests. Start with pure functions in `lib/orbit/`.

#### 1.2 Add Runtime Validation with Zod
```bash
# Already have zod in node_modules (transitive dep)
```

**Files to modify:**
- `src/lib/orbit/parser.ts` - Add Zod schemas for frontmatter
- `src/types/schemas.ts` - New file with Zod schemas matching types

**Pattern:**
```typescript
// src/types/schemas.ts
import { z } from "zod";

export const TaskFrontmatterSchema = z.object({
  title: z.string(),
  status: z.enum(["todo", "doing", "waiting", "done"]),
  priority: z.enum(["low", "medium", "high"]).optional(),
  due: z.string().optional(),
  created: z.string(),
});
```

**Why:** Prevents runtime errors from malformed markdown files.

#### 1.3 Add Error Boundary
**Files to create:**
- `src/components/error-boundary.tsx` - React error boundary component
- Update `src/app/layout.tsx` - Wrap app in error boundary

**Why:** Single component error shouldn't crash entire app.

#### 1.4 Standardize Error Handling
**Pattern to apply across all `lib/orbit/*.ts`:**
```typescript
// Bad (current)
catch (e) {
  console.warn(`Failed to read task:`, e);
}

// Good (target)
catch (e) {
  const error = e instanceof Error ? e : new Error(String(e));
  logger.warn("Failed to read task", { filename: entry.name, error: error.message });
  // Still continue processing other files
}
```

**Files to update:**
- `src/lib/orbit/tasks.ts:65-67`
- `src/lib/orbit/notes.ts:60-62`
- `src/lib/orbit/meetings.ts:64-66`
- `src/lib/orbit/projects.ts:104-106`
- `src/lib/orbit/personal.ts:168-170, 416-418`
- `src/lib/orbit/workspaces.ts:59-61`

#### 1.5 Clean Up Console Logs
**Files to update:**
- `src/lib/orbit/tauri-fs.ts` - Remove or gate behind DEBUG flag
- `src/lib/orbit/watcher.ts` - Use proper logger
- `src/lib/orbit/search-index.ts` - Use proper logger

**Pattern:**
```typescript
// Create src/lib/logger.ts
const isDev = process.env.NODE_ENV === "development";

export const logger = {
  debug: (...args: unknown[]) => isDev && console.log("[orbit]", ...args),
  info: (...args: unknown[]) => console.info("[orbit]", ...args),
  warn: (...args: unknown[]) => console.warn("[orbit]", ...args),
  error: (...args: unknown[]) => console.error("[orbit]", ...args),
};
```

---

### Phase 2: Type Safety (Do Next)

#### 2.1 Fix `any` Types
**Files to update:**
- `src/lib/orbit/parser.ts:21` - Replace `any` with generic

```typescript
// Current
export function serializeMarkdown(data: any, content: string): string

// Better
export function serializeMarkdown<T extends Record<string, unknown>>(
  data: T,
  content: string
): string
```

#### 2.2 Add Strict TypeScript Options
**Update `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

---

### Phase 3: Code Quality (Refactoring)

#### 3.1 Extract Shared Patterns

The following pattern appears in tasks.ts, notes.ts, meetings.ts, personal.ts:
```typescript
for (const entry of entries) {
  if (entry.isFile && entry.name.endsWith(".md")) {
    try {
      // read and parse
    } catch (e) {
      console.warn(...)
    }
  }
}
```

**Create:** `src/lib/orbit/utils/read-markdown-files.ts`
```typescript
export async function readMarkdownFiles<T, R>(
  dirPath: string,
  parser: (content: string, filename: string) => R
): Promise<R[]> {
  // Shared implementation
}
```

#### 3.2 Component Size Reduction

**Large components to split:**
- `src/components/tasks/kanban-board.tsx` - Extract column logic
- `src/components/layout/sidebar.tsx` - Extract nav sections

#### 3.3 Add Missing Loading States

**Components needing loading states:**
- `src/components/projects/project-card.tsx`
- `src/components/notes/note-list.tsx`
- `src/components/meetings/meeting-list.tsx`

---

### Phase 4: Production Polish

#### 4.1 Add Vitest Config
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

#### 4.2 Add Test Script to package.json
```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

---

## Files Summary

### Create These Files
| File | Purpose |
|------|---------|
| `vitest.config.ts` | Test configuration |
| `src/test/setup.ts` | Test setup |
| `src/lib/logger.ts` | Structured logging |
| `src/types/schemas.ts` | Zod validation schemas |
| `src/components/error-boundary.tsx` | React error boundary |
| `src/lib/orbit/__tests__/parser.test.ts` | Parser tests |
| `src/lib/orbit/utils/read-markdown-files.ts` | Shared file reading |

### Modify These Files
| File | Change |
|------|--------|
| `package.json` | Add test scripts, vitest dep |
| `tsconfig.json` | Stricter options |
| `src/app/layout.tsx` | Add error boundary |
| `src/lib/orbit/parser.ts` | Add Zod validation, fix any |
| `src/lib/orbit/tauri-fs.ts` | Use logger |
| `src/lib/orbit/tasks.ts` | Standardize error handling |
| `src/lib/orbit/notes.ts` | Standardize error handling |
| `src/lib/orbit/meetings.ts` | Standardize error handling |
| `src/lib/orbit/projects.ts` | Standardize error handling |
| `src/lib/orbit/personal.ts` | Standardize error handling |
| `src/lib/orbit/workspaces.ts` | Standardize error handling |
| `src/lib/orbit/watcher.ts` | Use logger |
| `src/lib/orbit/search-index.ts` | Use logger |

---

## Priority Order

1. **Testing infrastructure** - Enables safe refactoring
2. **Error boundary** - Prevents full app crashes
3. **Logger utility** - Clean up console noise
4. **Zod schemas** - Runtime safety
5. **Standardize error handling** - Consistent patterns
6. **Fix `any` types** - Full type safety
7. **Component refactoring** - Maintainability

---

## Metrics After Completion

| Metric | Before | After |
|--------|--------|-------|
| Test coverage | 0% | 60%+ |
| Type safety | 95% | 99% |
| Error handling | 60% | 90% |
| Production ready | 70% | 90% |

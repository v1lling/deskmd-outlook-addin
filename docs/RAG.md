# Task: RAG Integration and better AI Context Handling

> Status: Planning (not yet implemented)

## Goal

Introduce Retrieval-Augmented Generation (RAG) into Desk to enhance AI capabilities. Allow users to flag specific documents as "AI context," which the AI can utilize when generating content or providing suggestions.

## Requirements

- **Local-only** - No cloud dependencies, ships with the app
- **Lightweight** - Minimal bundle size impact
- **Markdown-aware** - Understand frontmatter, headers, structure
- **Find relevant context** - Instead of sending all data to AI

## Recommended Stack

| Component | Recommendation | Why |
|-----------|----------------|-----|
| **Vector DB** | `sqlite-vec` | SQLite extension, no separate process, ships as single file |
| **Embeddings** | Voyage (cloud) or `nomic-embed-text` (local) | Voyage = best quality; nomic via Ollama for fully local |
| **Chunking** | Custom markdown-aware splitter | Preserve frontmatter, respect headers |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     AI Chat Panel                        │
├─────────────────────────────────────────────────────────┤
│  User Query                                              │
│       │                                                  │
│       ▼                                                  │
│  ┌─────────────┐    ┌──────────────┐    ┌────────────┐ │
│  │   Embed     │───▶│  sqlite-vec  │───▶│  Top-K     │ │
│  │   Query     │    │   Search     │    │  Chunks    │ │
│  └─────────────┘    └──────────────┘    └────────────┘ │
│                                                │        │
│                                                ▼        │
│                                         ┌────────────┐  │
│                                         │   Claude   │  │
│                                         │  + Context │  │
│                                         └────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Two Paths

### Option A: Hybrid (Recommended)

- Use Anthropic's Voyage embeddings API (fast, high quality)
- Store vectors in sqlite-vec locally
- Falls back gracefully if offline

### Option B: Fully Local

- Ollama with `nomic-embed-text` (384 dims, fast)
- Requires Ollama running (same as current Claude Code CLI setup)
- Larger bundle, slower embeddings

## sqlite-vec Setup

```typescript
// Install: npm install sqlite-vec better-sqlite3
import Database from "better-sqlite3";
import * as sqliteVec from "sqlite-vec";

const db = new Database("~/Desk/.index/vectors.db");
sqliteVec.load(db);

db.exec(`
  CREATE VIRTUAL TABLE IF NOT EXISTS doc_chunks USING vec0(
    embedding float[384]  -- or 1024 for voyage
  );

  CREATE TABLE IF NOT EXISTS chunks (
    id INTEGER PRIMARY KEY,
    doc_path TEXT,
    content TEXT,
    chunk_index INTEGER
  );
`);
```

## Implementation Phases

1. **Phase 1**: sqlite-vec integration + manual "Add to AI context" button
2. **Phase 2**: Auto-index flagged docs on save
3. **Phase 3**: Smart chunking (respect markdown structure)
4. **Phase 4**: Background indexing on file watcher events

## Open Questions

1. **Embedding source**: Cloud (Voyage) or local-only (Ollama)?
2. **Scope**: Just flagged docs, or all docs by default?
3. **UI**: How to flag docs? Frontmatter field? Toggle in editor?

## Notes

- sqlite-vec is a SQLite extension - no separate process needed
- `nomic-embed-text` is small (274MB) and fast
- Could start with simple keyword search, add embeddings later
- hab gehört sqlite könnte gut sein sqlite vec oder sowas. dan irgendne kleineeres ollama embedding model oder so.

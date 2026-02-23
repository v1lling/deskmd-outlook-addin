# Task: RAG Integration and better AI Context Handling

> Status: **Phase 5 Complete** - Core RAG integration working. Phase 6-7 pending (UI polish, extended file types).
>
> **Update**: Smart Index was added as an alternative context strategy (see GitHub issue #1). Users now choose between **Smart Index**, **Embeddings (RAG)**, or **None** in Settings > Context. RAG remains fully functional as the "Embeddings" option.

## Goal

Introduce Retrieval-Augmented Generation (RAG) into Desk to enhance AI capabilities. Instead of manually attaching docs to chat, the AI automatically finds relevant context from your indexed documents.

## Smart Index (Alternative Strategy)

Smart Index was added as a lighter alternative to RAG. Instead of vector embeddings, it:
1. Builds an AI-summarized catalog of all files (stored in `.desk/index/indexes.json`)
2. When context is needed, AI selects relevant files from the catalog
3. Full file content is read and passed to the AI

**Key files**: `src/lib/context-index/`, `src/stores/context-index.ts`, `src/hooks/use-context-index-sync.ts`

The unified hook `src/hooks/use-context-search.ts` branches on the selected strategy.

## Decisions Summary

| Decision | Choice |
|----------|--------|
| **Embedding providers** | User's choice: Ollama (local) / OpenAI / Voyage - swappable in settings |
| **Vector DB** | sqlite-vec (Rust implementation in Tauri) |
| **What gets indexed** | Docs, Tasks, Meetings (all content types) |
| **Scope** | All workspaces, cross-workspace search (simplest approach) |
| **Default inclusion** | Everything included by default |
| **Exclusion method** | `.aiignore` files + frontmatter `ai: false` |
| **Chunking** | Hybrid (headers for large files, single chunk for small) |
| **Retrieval count** | Default 5, configurable in settings |
| **Show sources** | Yes, collapsible "Sources" in AI responses with similarity % |
| **Pin docs feature** | Replaced by RAG |
| **Non-MD files** | Phase 7: Add PDF, Word, txt support |

---

## Embedding Providers

**User chooses in settings** - fully swappable:

| Provider | Model | Cost | Quality | Dims | Offline |
|----------|-------|------|---------|------|---------|
| **Ollama** | nomic-embed-text | Free | Good | 384 | ✓ |
| **OpenAI** | text-embedding-3-small | $0.02/1M | Good | 1536 | ✗ |
| **Voyage** | voyage-3.5-lite | $0.02/1M | Better | 1024 | ✗ |

**Strategy**:
1. User selects provider in Settings → RAG tab
2. Default: "Auto" (tries Ollama first, then falls back to configured cloud)
3. Switching providers requires re-indexing (different vector dimensions)

**Note**: OpenAI and Voyage are same price. Voyage is technically better for RAG (built by Stanford retrieval researchers), OpenAI is more mainstream. User decides.

---

## Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend (React)                          │
├─────────────────────────────────────────────────────────────────┤
│  src/lib/rag/           │  src/stores/rag.ts                    │
│  - chunker.ts           │  - RAG settings                       │
│  - aiignore.ts          │  - Index status                       │
│  - index.ts (API)       │                                       │
└────────────────────────────┬────────────────────────────────────┘
                             │ Tauri Commands (invoke)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Tauri Backend (Rust)                         │
├─────────────────────────────────────────────────────────────────┤
│  src-tauri/src/rag/                                              │
│  ├── mod.rs             # Module exports + Tauri commands       │
│  ├── db.rs              # sqlite-vec database operations        │
│  ├── embeddings.rs      # Ollama + Voyage HTTP clients          │
│  └── search.rs          # Similarity search                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  {dataPath}/.desk/rag/vectors.db (sqlite-vec)                   │
│  (dataPath from user settings, e.g. ~/Desk or ~/MyData)         │
└─────────────────────────────────────────────────────────────────┘
```

### Indexing Flow

```
Document (md/docx/pdf)
     │
     ▼ (Frontend)
┌─────────────┐     ┌─────────────────┐
│  Extract    │────▶│  Chunk by       │
│  text       │     │  headers/size   │
└─────────────┘     └─────────────────┘
                             │
                             ▼ Tauri command: rag_index_chunks
┌─────────────────────────────────────────────────────────────────┐
│                     Rust Backend                                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐     ┌─────────────────┐     ┌──────────────┐  │
│  │  Embed via  │────▶│  Store in       │────▶│  Return      │  │
│  │  Ollama or  │     │  sqlite-vec     │     │  status      │  │
│  │  Voyage     │     │                 │     │              │  │
│  └─────────────┘     └─────────────────┘     └──────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Query Flow

```
User question
     │
     ▼ Tauri command: rag_search
┌─────────────────────────────────────────────────────────────────┐
│                     Rust Backend                                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐     ┌─────────────────┐     ┌──────────────┐  │
│  │  Embed      │────▶│  sqlite-vec     │────▶│  Return      │  │
│  │  question   │     │  KNN search     │     │  top-K       │  │
│  └─────────────┘     └─────────────────┘     └──────────────┘  │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼ (Frontend)
┌──────────────────────────────────────────────────────────────┐
│  Claude prompt with context + question                        │
│  → Response + collapsible "Sources" section                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Rust Implementation Details

### Cargo Dependencies

Add to `src-tauri/Cargo.toml`:

```toml
[dependencies]
# ... existing deps ...

# Vector database
rusqlite = { version = "0.32", features = ["bundled"] }
sqlite-vec = "0.1"  # Or use zerocopy for manual loading

# HTTP client for embeddings
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### Tauri Commands

```rust
// src-tauri/src/rag/mod.rs

#[tauri::command]
async fn rag_init_db(data_path: String) -> Result<(), String>;

#[tauri::command]
async fn rag_index_chunks(chunks: Vec<ChunkInput>) -> Result<IndexResult, String>;

#[tauri::command]
async fn rag_delete_doc(doc_path: String) -> Result<(), String>;

#[tauri::command]
async fn rag_search(
    query: String,
    limit: usize,
    provider: String,  // "ollama" | "voyage"
    settings: EmbeddingSettings,
) -> Result<Vec<SearchResult>, String>;

#[tauri::command]
async fn rag_get_status(data_path: String) -> Result<IndexStatus, String>;

#[tauri::command]
async fn rag_clear_index(data_path: String) -> Result<(), String>;

#[tauri::command]
async fn rag_check_ollama(url: String) -> Result<bool, String>;
```

### Data Structures (Rust)

```rust
// src-tauri/src/rag/mod.rs

#[derive(Debug, Serialize, Deserialize)]
pub struct ChunkInput {
    pub doc_path: String,
    pub workspace_id: String,
    pub content_type: String,  // "doc" | "task" | "meeting"
    pub title: String,
    pub content: String,
    pub content_hash: String,  // SHA-256 of original doc for change detection
    pub chunk_index: u32,
    pub total_chunks: u32,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct SearchResult {
    pub doc_path: String,
    pub workspace_id: String,
    pub content_type: String,
    pub title: String,
    pub content: String,
    pub chunk_index: u32,
    pub score: f32,            // Similarity score (0-1), shown in UI as percentage
}

#[derive(Debug, Serialize, Deserialize)]
pub struct IndexStatus {
    pub document_count: u32,
    pub chunk_count: u32,
    pub last_indexed_at: Option<String>,
    pub index_size_bytes: u64,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct EmbeddingSettings {
    pub provider: String,        // "ollama" | "openai" | "voyage"
    pub ollama_url: String,
    pub ollama_model: String,
    pub openai_api_key: Option<String>,
    pub voyage_api_key: Option<String>,
}
```

### SQLite Schema

```sql
-- Created by rag_init_db
-- Database location: {dataPath}/.desk/rag/vectors.db

-- Index metadata (tracks provider, dimensions, etc.)
CREATE TABLE IF NOT EXISTS index_meta (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL
);
-- Keys: provider, model, dimensions, last_full_index, version

-- Chunks table with content hash for incremental indexing
CREATE TABLE IF NOT EXISTS chunks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    doc_path TEXT NOT NULL,
    workspace_id TEXT NOT NULL,
    content_type TEXT NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    content_hash TEXT NOT NULL,      -- SHA-256 of original doc, for change detection
    chunk_index INTEGER NOT NULL,
    total_chunks INTEGER NOT NULL,
    updated_at TEXT NOT NULL,
    UNIQUE(doc_path, chunk_index)
);

CREATE INDEX idx_chunks_doc_path ON chunks(doc_path);
CREATE INDEX idx_chunks_workspace ON chunks(workspace_id);
CREATE INDEX idx_chunks_hash ON chunks(content_hash);

-- sqlite-vec virtual table for vectors
-- Dimensions vary by provider: Ollama=384, Voyage=1024, OpenAI=1536
-- Table is DROPPED and recreated when provider changes (requires re-index)
CREATE VIRTUAL TABLE IF NOT EXISTS chunk_vectors USING vec0(
    chunk_id INTEGER PRIMARY KEY,
    embedding FLOAT[384]  -- dimensions set based on provider at creation time
);
```

### Embedding Provider Clients (Rust)

```rust
// src-tauri/src/rag/embeddings.rs

/// Ollama (local) - 384 dimensions
pub async fn embed_ollama(
    text: &str,
    url: &str,
    model: &str,
) -> Result<Vec<f32>, String> {
    let client = reqwest::Client::new();
    let response = client
        .post(format!("{}/api/embeddings", url))
        .json(&serde_json::json!({
            "model": model,
            "prompt": text
        }))
        .send()
        .await
        .map_err(|e| e.to_string())?;

    // Parse response and return embedding vector
    // ...
}

/// OpenAI (cloud) - 1536 dimensions
pub async fn embed_openai(
    text: &str,
    api_key: &str,
) -> Result<Vec<f32>, String> {
    let client = reqwest::Client::new();
    let response = client
        .post("https://api.openai.com/v1/embeddings")
        .header("Authorization", format!("Bearer {}", api_key))
        .json(&serde_json::json!({
            "model": "text-embedding-3-small",
            "input": text
        }))
        .send()
        .await
        .map_err(|e| e.to_string())?;

    // Parse response and return embedding vector
    // ...
}

/// Voyage (cloud) - 1024 dimensions
pub async fn embed_voyage(
    text: &str,
    api_key: &str,
) -> Result<Vec<f32>, String> {
    let client = reqwest::Client::new();
    let response = client
        .post("https://api.voyageai.com/v1/embeddings")
        .header("Authorization", format!("Bearer {}", api_key))
        .json(&serde_json::json!({
            "model": "voyage-3.5-lite",
            "input": text
        }))
        .send()
        .await
        .map_err(|e| e.to_string())?;

    // Parse response and return embedding vector
    // ...
}
```

---

## When to Index

| Trigger | Behavior |
|---------|----------|
| **On doc save** | Re-index that single doc (~100ms) |
| **On app start** | Check for stale/missing vectors, index incrementally |
| **Manual button** | "Re-index All" for full rebuild |
| **File watcher** | Background indexing when files change externally |

---

## Storage

```
{dataPath}/.desk/rag/              # dataPath from user settings (e.g. ~/Desk, ~/MyData)
└── vectors.db                  # sqlite-vec database
    ├── index_meta              # provider, model, dimensions, version
    ├── chunks                  # doc_path, content, content_hash, workspace_id
    └── chunk_vectors           # embedding vectors (dims based on provider)
```

---

## AI Exclusion System

### Default Behavior
- **Everything included** by default
- Users explicitly exclude sensitive content

### Exclusion Methods

#### 1. Frontmatter field (individual files)
```yaml
---
title: Secret Notes
ai: false
---
```

#### 2. `.aiignore` file (folders)
Place in any folder to exclude contents:

```
# .aiignore - exclude everything
*
```

Or selective:
```
# .aiignore - exclude specific items
passwords.md
secrets/
*.private.md
```

### UI for Managing AI Inclusion

#### In Content Tree (docs list)
```
📁 projects/
  📁 secret-stuff/        🚫    ← folder excluded (has .aiignore)
    📄 passwords.md       🚫    ← inherited (dimmed)
  📄 api-docs.md          🧠    ← included (default)
  📄 internal-notes.md    🚫    ← manually excluded (ai: false)
```

**Interactions:**
- Small brain icon (🧠) on items = included in AI
- Crossed-out/dimmed = excluded
- Click icon to toggle
- Right-click folder → "Exclude folder from AI" (creates `.aiignore`)
- Right-click doc → "Exclude from AI" / "Include in AI"

#### In Doc Editor Header
```
┌─────────────────────────────────────────────────┐
│ 📄 API Documentation            [🧠] [⋮]        │
│ ─────────────────────────────────────────────── │
```
Toggle button in header to control AI inclusion.

---

## Chunking Strategy

**Hybrid approach** based on your data (avg 1.4 KB files):

| File Size | Has Headers? | Strategy |
|-----------|--------------|----------|
| < 2 KB | Any | Single chunk |
| >= 2 KB | Yes (##) | Split by headers |
| >= 2 KB | No | Fixed size (~500 tokens) with overlap |

Always include:
- Frontmatter in each chunk (for context)
- File path and title
- Chunk index for ordering

---

## Settings Page Restructure

### Current Tab Structure

```
┌─────────────────────────────────────────────────┐
│  Settings                                        │
├────────┬────────┬──────────┬────────────────────┤
│ General│   AI   │ Context  │       Data         │
└────────┴────────┴──────────┴────────────────────┘
```

| Tab | Contents |
|-----|----------|
| **General** | Theme, Sidebar, Reset |
| **AI** | Provider (Claude Code/API), API key, Usage stats |
| **Context** | Context strategy (Smart Index/Embeddings/None), embedding provider, index status |
| **Data** | Data path, Workspaces list |

### Context Tab (RAG Settings Section)

```
┌─────────────────────────────────────────────────────────────┐
│  Document Search (RAG)                                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Embedding Provider           [Auto ▾]                       │
│  ├─ Auto (Ollama → OpenAI/Voyage fallback)                  │
│  ├─ Ollama (local, free)                                    │
│  ├─ OpenAI (cloud, $0.02/1M tokens)                         │
│  └─ Voyage (cloud, $0.02/1M tokens)                         │
│                                                              │
│  ─────────────────────────────────────────────────────────  │
│                                                              │
│  [If Ollama]                                                 │
│  Ollama URL              [http://localhost:11434]            │
│  Model                   [nomic-embed-text ▾]                │
│                          [Test Connection]  ✓ Connected      │
│                                                              │
│  [If OpenAI]                                                 │
│  OpenAI API Key          [sk-... 👁]                         │
│                                                              │
│  [If Voyage]                                                 │
│  Voyage API Key          [pa-... 👁]                         │
│                                                              │
│  ⚠️ Changing provider requires re-indexing all documents     │
│                                                              │
│  ─────────────────────────────────────────────────────────  │
│                                                              │
│  Index Status                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  📄 387 documents indexed                            │    │
│  │  🧩 412 chunks (avg 1.1 per doc)                     │    │
│  │  🕐 Last indexed: 2 hours ago                        │    │
│  │  📁 Index size: 2.4 MB                               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  [Re-index All]  [Clear Index]                               │
│                                                              │
│  ─────────────────────────────────────────────────────────  │
│                                                              │
│  Retrieval                                                   │
│  Results per query         [5 ▾]  (3, 5, 10)                │
│                                                              │
│  ☑ Auto-index on save                                        │
│  ☑ Show sources in AI responses                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Models (Frontend)

### RAG Settings Store (Zustand)

```typescript
// src/stores/rag.ts

interface RAGSettings {
  // Provider (user's choice)
  embeddingProvider: 'auto' | 'ollama' | 'openai' | 'voyage';

  // Ollama (local)
  ollamaUrl: string;           // default: http://localhost:11434
  ollamaModel: string;         // default: nomic-embed-text

  // OpenAI (cloud)
  openaiApiKey: string;

  // Voyage (cloud)
  voyageApiKey: string;

  // Retrieval
  retrievalCount: number;      // default: 5

  // Behavior
  autoIndexOnSave: boolean;    // default: true
  showSourcesInChat: boolean;  // default: true
}
```

### Index Status

```typescript
interface IndexStatus {
  documentCount: number;
  chunkCount: number;
  lastIndexedAt: Date | null;
  indexSizeBytes: number;
  isIndexing: boolean;
  excludedCount: number;
  // Provider tracking (for mismatch detection)
  indexedWithProvider: 'ollama' | 'openai' | 'voyage' | null;
  indexedWithModel: string | null;
  dimensions: number | null;
}
```

### Chunk (Frontend)

```typescript
interface Chunk {
  docPath: string;
  workspaceId: string;
  contentType: 'doc' | 'task' | 'meeting';
  title: string;
  content: string;
  contentHash: string;        // SHA-256 for change detection
  chunkIndex: number;
  totalChunks: number;
}

interface SearchResult {
  docPath: string;
  workspaceId: string;
  contentType: 'doc' | 'task' | 'meeting';
  title: string;
  content: string;
  chunkIndex: number;
  score: number;              // 0-1, displayed as "94% match" in UI
}
```

---

## Implementation Phases

### Phase 1: Settings & Infrastructure
- [x] Restructure settings page with tabs (General, AI, RAG, Data)
- [x] Create RAG settings tab UI (provider selection, credentials)
- [x] Create RAG settings Zustand store
- [x] Add Ollama connection test (via fetch, will use Tauri command later)
- [x] Add OpenAI and Voyage API key inputs

### Phase 2: Rust Vector Database
- [x] Add Rust dependencies (rusqlite, sqlite-vec, reqwest, tokio, sha2)
- [x] Create `src-tauri/src/rag/` module structure
- [x] Implement `rag_init_db` command (create tables)
- [x] Implement `rag_get_status` command
- [x] Implement `rag_clear_index` command
- [x] Implement `rag_delete_doc` command
- [x] Implement `rag_check_ollama` command
- [x] Register commands in `lib.rs`

### Phase 3: Embedding Providers (Rust)
- [x] Implement Ollama HTTP client (`embed_ollama`) - 384 dims
- [x] Implement OpenAI HTTP client (`embed_openai`) - 1536 dims
- [x] Implement Voyage HTTP client (`embed_voyage`) - 1024 dims
- [x] Add provider selection logic (auto-detect with fallback)
- [x] Implement `rag_index_chunks` command (embed + store)
- [x] Implement `rag_search` command (embed query + KNN)

### Phase 4: Frontend Indexing Pipeline
- [x] Create frontend chunker (`src/lib/rag/chunker.ts`)
- [x] Create `.aiignore` parser (`src/lib/rag/aiignore.ts`)
- [x] Parse frontmatter for `ai: false`
- [x] Create RAG API wrapper (`src/lib/rag/index.ts`)
- [x] Wire up RAG tab buttons to Tauri commands
- [x] Hook into doc save (auto-index) - `use-rag-indexer.ts` + editor callbacks
- [x] Add "Re-index All" functionality - `src/lib/rag/reindex.ts`

### Phase 5: AI Chat Integration
- [x] Create search API in frontend
- [x] Integrate into AI chat (query before sending to Claude) - `src/stores/ai.ts:211-216`
- [x] Add context to Claude prompt - `src/lib/ai/prompts.ts:78-82`
- [x] Show collapsible "Sources" in AI responses - `src/components/ai/chat-message.tsx:49-64`

### Phase 6: UI Polish
- [ ] Add AI inclusion icon (🧠) to content tree items
- [ ] Add toggle in doc editor header
- [ ] Add right-click context menu: "Exclude from AI"
- [ ] Add folder context menu: "Exclude folder from AI"
- [ ] Show excluded state (dimmed/crossed)

### Phase 7: Extended File Support
- [ ] Add mammoth for Word (.docx) extraction
- [ ] Add pdf-parse for PDF extraction
- [ ] Add plain text (.txt) support
- [ ] Update chunker to handle different file types

---

## File Locations

### Frontend (TypeScript)

```
src/lib/rag/
├── index.ts              # Main RAG API (calls Tauri commands)
├── chunker.ts            # Markdown-aware chunking
├── aiignore.ts           # .aiignore file parsing
└── types.ts              # TypeScript interfaces

src/stores/context.ts     # Context strategy + RAG settings (Zustand, persisted)
src/stores/context-index.ts  # Smart Index data store

src/components/settings/
├── general-tab.tsx       # Theme, sidebar, reset
├── ai-tab.tsx            # Claude provider, API key, usage
├── context-tab.tsx       # Context strategy, Smart Index + RAG settings
└── data-tab.tsx          # Data path, workspaces
```

### Backend (Rust)

```
src-tauri/src/
├── lib.rs                # Add rag commands to invoke_handler
├── rag/
│   ├── mod.rs            # Module exports + Tauri commands
│   ├── db.rs             # sqlite-vec operations
│   ├── embeddings.rs     # Ollama + Voyage HTTP clients
│   └── search.rs         # Vector similarity search
```

---

## Technical Notes

- sqlite-vec uses `vec0` virtual table for vector storage
- Vector dimensions vary by provider:
  - Ollama (nomic-embed-text): 384 dims
  - Voyage (voyage-3.5-lite): 1024 dims
  - OpenAI (text-embedding-3-small): 1536 dims
- **Switching providers requires full re-index** (can't mix vectors from different models)
- `.aiignore` follows `.gitignore` syntax (minimatch patterns)
- API keys are separate: OpenAI key ≠ Voyage key ≠ Anthropic key
- Ollama requires model pulled first: `ollama pull nomic-embed-text`
- Store current provider in index metadata to detect provider changes

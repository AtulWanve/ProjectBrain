---
title: VectifyAI Blueprints
tags: [memory, architecture, blueprints, vectifyai, pageindex, condb, chatindex, openkb, retrieval, memory-layer]
aliases: [VectifyAI Architecture Patterns, Reasoning-Based RAG, Vectorless Retrieval]
---

# VectifyAI Blueprints

Architectural patterns harvested from the [VectifyAI](https://github.com/VectifyAI) ecosystem (PageIndex, ConDB, ChatIndex, OpenKB). These inform ProjectBrain's memory layer, retrieval, and knowledge compilation design.

## Overview

VectifyAI's thesis: **similarity ≠ relevance**. Traditional vector RAG retrieves what looks similar, not what is truly relevant. Their stack replaces vector similarity with **reasoning-based tree search** — LLMs navigate hierarchical indexes like a human expert using a table of contents.

Four blueprints extracted, ordered by relevance to ProjectBrain:

---

## 1. PageIndex — Reasoning-Based Tree Retrieval

**Source:** [VectifyAI/PageIndex](https://github.com/VectifyAI/PageIndex) (34,038★, MIT, Python)
**Target:** Graph Retriever / Context Loader

### Core Pattern: Vectorless RAG via Hierarchical Tree Search

PageIndex replaces vector DB + chunking with a two-step process:

1. **Tree Construction**: Transform long PDFs into a hierarchical "table of contents" tree. Each node has a title, summary, and page range. Built once per document.
2. **Reasoning-Based Retrieval**: LLM performs top-down tree search — at each node, evaluates whether the summary answers the query; if not, descends into children. Retrieval is traceable and explainable.

### Key Concepts for ProjectBrain

| Concept | Application |
|---------|-------------|
| **Tree-indexed documents** | Replace flat chunk storage with hierarchical tree indices in the knowledge graph |
| **LLM-guided traversal** | Graph Retriever can use reasoning-based navigation instead of pure vector similarity |
| **Multi-resolution access** | Retrieve summaries at higher levels, full content at leaves — matches ProjectBrain's context engineering needs |
| **No fixed top-K** | Retrieve all relevant passages automatically, not a fixed number of chunks |
| **Context-aware** | Retrieval incorporates conversation history and domain knowledge |

### Pattern Extraction

```typescript
// Tree node structure (adapted from PageIndex)
interface IndexNode {
  title: string;
  nodeId: string;
  summary: string;
  startIndex: number;   // page / section range
  endIndex: number;
  children: IndexNode[];
}

// Retrieval: LLM-guided tree traversal
async function reasonBasedRetrieve(
  tree: IndexNode,
  query: string,
  context: RetrievalContext
): Promise<RetrievedContent> {
  // Top-down traversal: LLM evaluates each node's relevance
  // Descends into relevant branches, prunes irrelevant ones
  // Returns minimal sufficient context
}
```

### Record: 98.7% on FinanceBench

PageIndex (via Mafin 2.5) achieved 98.7% accuracy on FinanceBench, vastly outperforming vector RAG on professional document QA. This validates reasoning-based retrieval over similarity search for complex documents.

### Integration Points

| Component | How to Apply |
|-----------|-------------|
| `runtime/graph-retriever.md` | Add tree-based retrieval as an alternative strategy alongside vector similarity |
| `runtime/context-loader.md` | Support multi-resolution context loading (summary vs. full content) |
| `graph/` | Extend graph nodes with hierarchical index structure |
| `runtime/memory-updater.md` | Add tree construction logic when ingesting long documents |

---

## 2. ConDB — KV-Cache Native Context Database

**Source:** [VectifyAI/ConDB](https://github.com/VectifyAI/ConDB) (44★, Apache-2.0, Python)
**Target:** Memory Layer / Context Persistence

### Core Pattern: Tree-Structured Context Storage with KV-Cache Reuse

ConDB is a context database that stores hierarchical tree structures (documents, conversations, filesystem trees) and performs reasoning-based retrieval. Key innovations:

1. **Tree-structured storage in SQLite** — stores any hierarchical JSON as a tree of nodes
2. **Multi-strategy retrieval** — beam search (small trees) + block retrieval (large documents)
3. **KV-cache native** — caches intermediate retrieval results, up to 70% token savings
4. **Multiple adapters** — DocumentTree, ChatIndex, Generic adapters for different input types

### Retrieval Strategies

| Strategy | Best For | How It Works |
|----------|----------|--------------|
| **Beam** | Small trees (< 50 nodes) | LLM evaluates and selects promising branches at each depth level |
| **Block** | Large documents (50+ nodes) | Splits tree into token-bounded blocks, LLM reasons over each block. KV-cache reuse cuts token usage by ~70% |

### Benchmark: SWEBench-FileTree

ConDB's block strategy achieved 0.90 recall vs 0.56 for baseline, at 3× lower latency on code file retrieval. This validates tree-based retrieval for engineering contexts.

### Integration Points

| Component | How to Apply |
|-----------|-------------|
| `runtime/graph-retriever.md` | Add beam/block retrieval strategies based on tree size |
| `memory/` | Use ConDB's adapter pattern for unified document + conversation memory |
| `runtime/memory-updater.md` | Adopt KV-cache reuse for incremental memory updates |
| `graph/` | Store graph as tree-structured indexes, not flat nodes |

---

## 3. ChatIndex — Tree-Based Conversational Memory

**Source:** [VectifyAI/ChatIndex](https://github.com/VectifyAI/ChatIndex) (150★, Apache-2.0, Python)
**Target:** Agent Session Memory / Runtime

### Core Pattern: Dynamic Topic Tree for Long Conversations

ChatIndex adapts PageIndex's tree indexing for **dynamic, unstructured conversations** (vs. static documents). Key differences from PageIndex:

1. **Dynamic vs. Static** — conversations grow incrementally; tree must be updated online
2. **Unstructured vs. Structured** — conversations have no natural table of contents; LLM detects topic switches
3. **B+-tree inspired** — leaf nodes store raw messages, internal nodes store topic summaries. Bounded fan-out keeps tree shallow.

### Context Tree (CTree) Architecture

```
ROOT
├── Topic A
│   ├── Subtopic A1
│   │   └── [Messages 0-4]  (leaf: raw conversation)
│   └── Subtopic A2
│       └── [Messages 4-10]
├── Topic B
│   └── [Messages 10-20]
```

### Multi-Resolution Access

- Dynamic resolution: returns only as much detail as needed (summary at higher nodes, raw messages at leaves)
- Lossless fallback: raw conversation always accessible when required
- Efficient reasoning: large contexts reduced to minimally sufficient subset

### Temporal Ordering Constraint

New topic nodes can only be children of the current node or its ancestors — preserves conversation flow while allowing hierarchical organization.

### Integration Points

| Component | How to Apply |
|-----------|-------------|
| `runtime/execution-engine.md` | Track conversation/session context as a CTree |
| `runtime/context-loader.md` | Load only relevant conversation branches into context |
| `runtime/memory-updater.md` | Incremental tree update logic (detect topic switch → add node) |
| `cache/` | Store live session context trees in cache |

---

## 4. OpenKB — Knowledge Compilation Pipeline

**Source:** [VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB) (3,027★, Apache-2.0, Python)
**Target:** Memory Updater / Knowledge Accumulation

### Core Pattern: Document → Wiki → Skill Pipeline

OpenKB compiles raw documents into a persistent, interlinked wiki. Knowledge accumulates over time instead of being re-derived on every query.

```
raw documents (PDF, MD, URL, ...)
  → markitdown / PageIndex conversion
  → LLM compilation:
      1. Generate summary page
      2. Create/update concept pages (cross-document synthesis)
      3. Create/update entity pages (people, orgs, places, products)
      4. Update index and log
  → wiki/ (persistent, cross-linked, Obsidian-compatible)
  → generators: query, chat, skill factory, deck builder
```

### Key Innovations

| Feature | Description |
|---------|-------------|
| **Short vs long doc handling** | Short docs read in full; long PDFs (>20 pages) use PageIndex tree index |
| **Cross-document synthesis** | New documents update existing concept pages, contradictions flagged |
| **Entity auto-extraction** | People, orgs, places, products auto-extracted and kept in sync |
| **Skill Factory** | `openkb skill new` distills a redistributable agent skill from wiki |
| **Obsidian-compatible** | Wiki is plain `.md` with `[[wikilinks]]` — opens in Obsidian for graph view |

### Relation to Existing ProjectBrain Architecture

ProjectBrain already has a similar pattern (memory/, graph/, reviews/, standards/ directories). OpenKB validates this approach and adds:

- **Concept page synthesis** — cross-reference multiple documents into unified concept pages (cf. ProjectBrain's memory/ files)
- **Entity tracking** — auto-extract and maintain people, orgs, places from engineering context
- **Skill distillation** — `openkb skill new` pattern could inform how ProjectBrain exports reusable agent skills
- **Obsidian integration** — wiki format is already compatible with the Obsidian vault at `C:\Users\Atul\Documents\Obsidian Vault\ProjectBrain`

### Integration Points

| Component | How to Apply |
|-----------|-------------|
| `runtime/memory-updater.md` | Add cross-document synthesis when new memory entries conflict with existing |
| `tasks/` | Wiki compilation could be a task template pattern |
| `graph/` | Knowledge graph could use OpenKB's entity extraction approach |
| `templates/` | Add concept page and entity page templates |

---

## Combined Architecture Vision

```
                    ┌─────────────────────────────┐
                    │     OpenKB-style Wiki        │
                    │  (persistent, cross-linked,   │
                    │   auto-compiled knowledge)    │
                    └──────────────────────┬──────┘
                                           │
                    ┌──────────────────────▼──────┐
                    │     PageIndex Tree Search    │
                    │  (reasoning-based retrieval  │
                    │   over hierarchical indexes) │
                    └──────────────────────┬──────┘
                                           │
              ┌────────────────────────────┼────────────────────────────┐
              │                            │                            │
              ▼                            ▼                            ▼
    ┌─────────────────┐       ┌─────────────────────┐       ┌─────────────────────┐
    │  ConDB Storage   │       │  ChatIndex Session   │       │  Vector/Graph       │
    │  (tree-structured │       │  Memory (dynamic     │       │  Retrieval (existing │
    │   context DB)    │       │  topic trees)        │       │  fallback)          │
    └─────────────────┘       └─────────────────────┘       └─────────────────────┘
```

All four tools from the same ecosystem, designed to work together. ProjectBrain can adopt the patterns selectively, starting with the retrieval strategy and memory layer enhancements.

---

## Action Items

- [ ] Add tree-based retrieval strategy to `runtime/graph-retriever.md`
- [ ] Add incremental tree update pattern to `runtime/memory-updater.md`
- [ ] Add multi-resolution context loading to `runtime/context-loader.md`
- [ ] Evaluate ConDB's KV-cache reuse for memory operations
- [ ] Evaluate ChatIndex's topic tree for session memory
- [ ] Add concept/entity templates to `templates/`
- [ ] Cross-reference this document from `system/system.md`

---

> [!info] Harvested from [VectifyAI](https://github.com/VectifyAI) ecosystem evaluation on 2026-07-15. See `C:\Users\Atul\Desktop\OpenSourceScout\knowledge\repos\vectifyai__PageIndex.md`, `vectifyai__ConDB.md`, `vectifyai__ChatIndex.md`, `vectifyai__OpenKB.md` for full verdict notes.

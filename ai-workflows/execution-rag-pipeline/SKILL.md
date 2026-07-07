---
name: execution-rag-pipeline
description: >-
  Build a retrieval-augmented generation pipeline. Step-by-step procedure
  covering document ingestion, chunking, embedding, retrieval, and generation
  with citations. Use when implementing RAG systems, improving retrieval
  quality, or designing knowledge-augmented AI features.
---

# RAG Pipeline

## Purpose

Step-by-step procedure for building effective retrieval-augmented generation systems.

## Workflow

### Step 1: Document Ingestion

- Identify source documents (docs, code, data)
- Extract text (handle PDF, HTML, markdown)
- Preserve metadata (source, date, section, author)
- Handle updates (incremental vs full re-index)

### Step 2: Chunking Strategy

| Strategy | Best for | Chunk size |
|----------|----------|------------|
| Fixed size | Simple documents | 512-1024 tokens |
| Semantic (paragraph/section) | Structured docs | Variable |
| Sliding window with overlap | Dense technical content | 512 tokens, 50-100 overlap |
| Recursive (split by hierarchy) | Documents with clear structure | Variable |

**Key**: Chunks should be self-contained enough to be useful without surrounding context.

### Step 3: Embedding

- Choose embedding model appropriate for domain
- Include metadata in embedding (title + content often works better than content alone)
- Store embeddings in vector database
- Track embedding model version (re-embed if model changes)

### Step 4: Retrieval

| Technique | When |
|-----------|------|
| Vector similarity (k-NN) | Default semantic search |
| Hybrid (vector + keyword) | When exact terms matter |
| Re-ranking | Quality-critical, can afford latency |
| Multi-query | Ambiguous queries (rephrase and merge results) |

Retrieve more than you need, then filter/re-rank.

### Step 5: Generation with Context

- Inject retrieved chunks into prompt
- Instruct model to cite sources
- Include "if the context doesn't answer, say so"
- Format citations for traceability

### Step 6: Evaluate

| Metric | Measures | How |
|--------|----------|-----|
| Retrieval precision | Right docs found | Manual review of top-k |
| Answer faithfulness | Answer matches sources | Compare claims to chunks |
| Answer relevance | Answer addresses query | Human evaluation |

## Checklist

- [ ] Chunking preserves meaningful context
- [ ] Metadata stored with each chunk
- [ ] Retrieval tested with representative queries
- [ ] Generation cites sources
- [ ] Graceful handling of "no relevant context found"
- [ ] Evaluation pipeline for quality monitoring

## Related Skills

- `ai-workflows/knowledge-agent-design` — RAG as agent knowledge source
- `ai-workflows/knowledge-guardrails` — validation of RAG outputs
- `ai-workflows/knowledge-prompt-engineering` — prompting with retrieved context

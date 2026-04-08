# Search and Vector Search

**Impact:** High

## What This Covers

BigQuery's full-text search indexes with the SEARCH() function and vector search capabilities with VECTOR_SEARCH() for embedding-based similarity queries.

## Why It Matters

Search indexes enable fast full-text search over large text columns without scanning every row. Vector search enables semantic similarity queries (RAG pipelines, recommendations, duplicate detection) directly in BigQuery without an external vector database.

## Example

```sql
-- Full-text search: create index
CREATE SEARCH INDEX my_search_index
ON `project.dataset.articles`(title, body)
OPTIONS(analyzer = 'LOG_ANALYZER');

-- Full-text search: query
SELECT article_id, title, body
FROM `project.dataset.articles`
WHERE SEARCH(title, 'bigquery optimization')
LIMIT 20;

-- Create vector index for faster search
CREATE VECTOR INDEX doc_vector_idx
ON `project.dataset.doc_embeddings`(embedding)
OPTIONS(index_type = 'IVF', distance_type = 'COSINE');

-- Find similar documents by embedding
SELECT
  query.doc_id AS query_doc,
  base.doc_id AS similar_doc,
  base.content,
  distance
FROM VECTOR_SEARCH(
  TABLE `project.dataset.doc_embeddings`,
  'embedding',
  (SELECT embedding FROM `project.dataset.doc_embeddings` WHERE doc_id = 'doc-42'),
  top_k => 10,
  distance_type => 'COSINE'
);

-- RAG pattern: embed query text, then find relevant docs
SELECT base.content, distance
FROM VECTOR_SEARCH(TABLE `project.dataset.knowledge_base`, 'embedding',
  (SELECT ml_generate_embedding_result AS embedding
   FROM ML.GENERATE_EMBEDDING(MODEL `project.dataset.embedding_model`,
     (SELECT 'How do I optimize BigQuery?' AS content))),
  top_k => 5, distance_type => 'COSINE');
```

## Edge Cases / Pitfalls

- **Search index cost:** Search indexes consume additional storage (typically 50-100% of the indexed columns). Monitor with `INFORMATION_SCHEMA.SEARCH_INDEXES`.
- **Analyzer choice:** `LOG_ANALYZER` is best for logs/structured text. `NO_OP_ANALYZER` for exact match. `PATTERN_ANALYZER` for custom tokenization. Wrong analyzer yields poor recall.
- **SEARCH() without an index:** SEARCH() works without a search index by falling back to a full table scan. An index is strongly recommended for large tables to avoid scanning all data, but its absence does not cause an error.
- **Vector distance metrics:** Supported types are `COSINE`, `EUCLIDEAN`, and `DOT_PRODUCT`. Use COSINE for normalized embeddings (most common), EUCLIDEAN for spatial data.
- **Vector index threshold:** Vector indexes provide acceleration only for tables above ~10MB of embedding data. Smaller tables use brute-force scan automatically.
- **Embedding dimensions:** BigQuery supports embeddings up to 1600 dimensions. Larger embeddings must be truncated or dimensionality-reduced before indexing.
- **Use cases:** Full-text search for keyword/log search. Vector search for semantic similarity, RAG retrieval, recommendation engines, and near-duplicate detection.

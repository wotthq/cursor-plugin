---
name: search-tool-reference
description: Reference for the wott search_project tool, including its inputs, outputs, search behavior, permissions, and limitations.
metadata:
  short-description: wott search tool reference
---

# Search Tool Reference

This document describes the wott MCP `search_project` tool.

The tool searches notebook knowledge within an authenticated user's accessible project.

---

# 1. MCP Tool

```text
search_project
```

Purpose:

```text
Search project notebook knowledge using hybrid retrieval.
```

The current implementation combines:

* BM25
* TF-IDF
* index search
* weighted reciprocal rank fusion
* block-level retrieval
* notebook-item resolution
* notebook enrichment

---

# 2. Tool Annotation

The search operation is read-only.

```json
{
  "readOnlyHint": true,
  "openWorldHint": false,
  "destructiveHint": false
}
```

Reasoning:

* `readOnlyHint: true` because search does not modify data.
* `openWorldHint: false` because search is limited to wott project data.
* `destructiveHint: false` because search cannot delete or modify data.

---

# 3. Input Schema

The current public MCP tool accepts:

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the project to search."
    },
    "query": {
      "type": "string",
      "minLength": 1,
      "description": "Search query describing the information to find."
    },
    "limit": {
      "type": "number",
      "integer": true,
      "minimum": 1,
      "maximum": 50,
      "description": "Maximum number of results to return. Defaults to 20."
    }
  },
  "required": [
    "project_id",
    "query"
  ],
  "additionalProperties": false
}
```

There is currently no `user_id` input.

There is currently no `sources` input in the recommended public contract.

---

# 4. `project_id`

Required.

Example:

```json
{
  "project_id": "project_123"
}
```

The project ID must come from:

* the user's existing context
* `get_projects`
* another trusted wott operation

Never construct or guess a project ID.

The search service validates that the authenticated user has access to the project before performing the search.

---

# 5. `query`

Required.

The query describes what information the user wants to find.

Example:

```json
{
  "project_id": "project_123",
  "query": "humanoid robotics simulation"
}
```

Good queries preserve important concepts from the user's request.

Examples:

```text
MCP OAuth Cloud Run
```

```text
PostgreSQL database migration
```

```text
humanoid robot locomotion
```

```text
production deployment troubleshooting
```

---

# 6. `limit`

Optional.

Default:

```text
20
```

Maximum:

```text
50
```

Examples:

```json
{
  "project_id": "project_123",
  "query": "robotics",
  "limit": 5
}
```

Use a smaller limit when the user wants only the most relevant results.

Use a larger limit when the user is exploring a broad topic.

---

# 7. Response

Successful MCP responses use the common wott MCP response envelope:

```json
{
  "success": true,
  "result_count": 2,
  "items": [],
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

The search domain returns:

```json
{
  "result_count": 2,
  "items": []
}
```

The MCP layer adds:

```text
success
timestamp
```

---

# 8. Search result

Each result has the following structure:

```json
{
  "markdown_item_id": "notebook_123",
  "block_ids": [
    "block_1",
    "block_2"
  ],
  "score": 0.045,
  "rank": 1,
  "sources": [
    "bm25",
    "tfidf",
    "index"
  ],
  "item": {}
}
```

## `markdown_item_id`

The wott notebook item ID.

Use it with `get_notebook` when full notebook content is required.

## `block_ids`

The notebook blocks that contributed to the search result.

These are internal retrieval identifiers.

## `score`

The final fused relevance score.

The score is useful for ranking but should not be presented as a percentage or confidence value.

## `rank`

The result's final position.

Lower rank means a stronger result.

## `sources`

The retrieval algorithms contributing to the result.

Possible values:

```text
bm25
tfidf
index
```

These do not represent notebook or file types.

## `item`

The enriched notebook item returned by the search enrichment layer.

It can contain notebook metadata useful for identifying the result.

---

# 9. Retrieval pipeline

The current search pipeline is:

```text
query
  ↓
normalize query
  ↓
tokenize
  ↓
BM25 ─────────┐
TF-IDF ───────┼──→ Weighted RRF
Index ────────┘
                   ↓
              fused blocks
                   ↓
             block_items
                   ↓
          notebook item IDs
                   ↓
          group by notebook
                   ↓
          enrich notebook items
                   ↓
             final results
```

---

# 10. Query processing

The search normalizes the query by:

* trimming whitespace
* converting to lowercase
* Unicode normalization using NFKD
* removing punctuation
* splitting into terms
* removing duplicate terms
* removing stop words
* preserving configured important short terms

For example:

```text
"MCP OAuth, Cloud Run!"
```

is normalized into search terms similar to:

```text
mcp
oauth
cloud
run
```

The exact terms depend on the configured stop-word and important-short-term lists.

---

# 11. Hybrid retrieval

The search executes three retrieval methods in parallel.

### BM25

Used for lexical relevance based on term frequency and document statistics.

### TF-IDF

Used for term-based relevance.

### Index search

Used for wott's indexed notebook knowledge.

The results are fused using weighted reciprocal rank fusion.

Current weights:

```json
{
  "bm25": 1.2,
  "tfidf": 0.8,
  "index": 1.5
}
```

Current RRF constant:

```text
60
```

The final result ranking is based on the fused score.

---

# 12. Block-level retrieval

The retrieval algorithms operate on notebook blocks.

A block is resolved through:

```text
projects/{project_id}/block_items/{block_id}
```

The search checks:

```text
source_type === "notebook"
```

and resolves:

```text
source_id
```

as the notebook item ID.

Unresolved blocks are ignored.

Non-notebook sources are ignored by the current implementation.

---

# 13. Multiple blocks from one notebook

A notebook can contain multiple matching blocks.

The search groups matching blocks by:

```text
markdown_item_id
```

The resulting notebook:

* contains all matching block IDs
* combines contributing retrieval sources
* aggregates relevance score
* receives a final rank

This prevents one notebook from appearing multiple times simply because multiple blocks matched the query.

---

# 14. Notebook enrichment

After ranking notebook items, the search calls the notebook enrichment layer.

The enrichment operation receives:

```text
project_id
markdown_item_ids
```

The result provides notebook information used by the MCP response.

The search does not expose the full internal Firestore notebook document by default.

---

# 15. Search and full notebook content

Search results identify relevant notebook items.

They are not a replacement for full notebook retrieval.

If the user asks:

> "Find my deployment notes."

Use:

```text
search_project
```

If the user then asks:

> "Show me the deployment notebook."

Use:

```text
get_notebook
```

with the selected notebook's `item_id`.

---

# 16. Current source support

The current implementation supports notebook search.

The MCP API should not claim that files are searchable until file retrieval is implemented end-to-end.

The current supported knowledge source is:

```text
notebook
```

The internal retrieval source names:

```text
bm25
tfidf
index
```

must not be confused with project knowledge source types.

---

# 17. Authorization

Authorization occurs before search execution.

The service receives:

```text
user_id
project_id
query
limit
```

and performs:

```text
requireProjectAccess({
  project_id,
  user_id
})
```

before calling the search domain.

The model never supplies `user_id`.

The MCP authentication layer provides the authenticated user ID.

---

# 18. Errors

Typical errors include:

```text
Query is required
```

or project access errors returned by the project access layer.

If no results are found, this is not an error.

A valid empty search response is:

```json
{
  "success": true,
  "result_count": 0,
  "items": [],
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

---

# 19. Data exposure

Do not expose:

* Login credentials
* OAuth tokens
* OAuth authorization codes
* authenticated user IDs
* internal service credentials
* server secrets
* unrelated project data
* unrelated users' notebook data
* internal Firestore fields unless explicitly part of the response
* internal retrieval implementation details unless useful

`block_ids` and retrieval `sources` are primarily implementation-level result information.

---

# 20. Tool selection

Use `search_project` when:

```text
The user wants to find information across project notebook knowledge.
```

Use `get_notebook` when:

```text
The user wants a specific notebook or its complete contents.
```

Use `get_projects` when:

```text
The project itself must first be identified.
```

Do not invent:

```text
search_notebook
search_files
search_project_files
semantic_search
find_project_content
```

The current MCP tool name is:

```text
search_project
```

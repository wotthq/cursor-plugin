---
name: search-project
description: Search notebooks within an wott project to find relevant project information and knowledge. Use when the user wants to find information inside a specific wott project.
metadata:
  short-description: Search notebooks in wott projects
---

# Search Project

Search project knowledge using the `search_project` MCP tool.

This skill helps users find relevant information across their project notebooks without modifying project or notebook data.

---

## When to use this skill

Use this skill when the user wants to:

* Find information in a project
* Search project notebooks
* Find notes about a topic
* Find a notebook containing specific information
* Locate previous research or documentation
* Find project context from existing notebook content
* Search for a technical term, person, technology, decision, or concept within a project
* Find relevant notebooks before performing another notebook operation

Typical requests include:

* "Search my AI project for humanoid robotics."
* "Find notes about PostgreSQL in this project."
* "Where did I write about deployment?"
* "Search the project for information about the MCP server."
* "Find notebooks discussing robotics simulation."

---

# Core behavior

The search skill should:

1. Identify the relevant project.
2. Obtain the actual `project_id` when it is not already known.
3. Use `search_project` to search the project.
4. Interpret the returned ranked results.
5. Present the most relevant results clearly.
6. Retrieve a specific notebook with `get_notebook` when the user needs its full content.
7. Never modify project or notebook data during a search-only request.

---

# Available MCP tool

The primary tool for this skill is:

```text
search_project
```

It performs hybrid search over project notebook knowledge.

The search combines:

* BM25
* TF-IDF
* index-based retrieval
* weighted reciprocal rank fusion
* notebook block-to-document resolution
* notebook result enrichment

The current implementation searches notebook knowledge.

It does not currently provide file-source search.

---

# Project identification

`search_project` requires:

```text
project_id
```

The model must use an actual project ID.

Never invent a project ID.

If the user provides a project ID, use it directly.

If the user provides only a project name, use the available project-management capability to identify the project before searching.

If multiple projects are plausible matches, ask the user to clarify.

Example:

```text
User:
Search my AI Research project for humanoid robotics.

Model:
1. Find the "AI Research" project.
2. Get its project_id.
3. Call search_project with that project_id.
```

Do not assume that a project name is its ID.

---

# Search query construction

Use the user's actual information need as the search query.

Good search queries preserve important terms.

For example:

```text
humanoid robotics simulation
```

is better than:

```text
information
```

For a technical request:

```text
MCP OAuth Cloud Run deployment
```

is better than:

```text
deployment stuff
```

Do not unnecessarily rewrite a precise technical query into generic terms.

The search engine performs its own tokenization, normalization, stop-word filtering, and retrieval.

---

# Search limits

The `limit` parameter controls the maximum number of notebook results returned.

Default:

```text
20
```

Maximum:

```text
50
```

Use the default unless the user requests a different number or a larger result set is useful.

For a simple question, a small number of highly relevant results is usually preferable.

For exploratory requests, a larger limit can be useful.

---

# Understanding search results

The search response has this general structure:

```json
{
  "success": true,
  "result_count": 2,
  "items": [
    {
      "markdown_item_id": "notebook_123",
      "block_ids": [
        "block_1",
        "block_2"
      ],
      "score": 0.12,
      "rank": 1,
      "sources": [
        "bm25",
        "tfidf",
        "index"
      ],
      "item": {}
    }
  ],
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

Important fields:

### `markdown_item_id`

The ID of the notebook item that matched the search.

Use this ID with `get_notebook` when the user needs the actual notebook.

### `block_ids`

The blocks that contributed to the result.

These are internal retrieval references.

Do not normally expose them to the user unless they are useful for debugging.

### `score`

The fused relevance score.

Higher scores indicate stronger ranking within the returned search results.

Do not present the score as a percentage or confidence value.

### `rank`

The final search result ranking.

Rank `1` is the highest-ranked result.

### `sources`

The retrieval methods that contributed to the result.

Possible values:

```text
bm25
tfidf
index
```

These are retrieval mechanisms, not document sources.

### `item`

The enriched notebook information.

Use the notebook metadata to help explain why a result is relevant.

---

# Search versus notebook retrieval

Use `search_project` to discover relevant notebooks.

Use `get_notebook` to retrieve a specific notebook or its full Markdown content.

Typical workflow:

```text
User request
    ↓
Identify project
    ↓
search_project
    ↓
Relevant notebook results
    ↓
Select notebook
    ↓
get_notebook
    ↓
Full Markdown content
```

Do not use search as a replacement for reading the complete notebook when the user explicitly asks to read or summarize a notebook.

---

# Search-only requests

If the user only asks:

> "Find notes about robotics."

Search the project and return relevant notebook results.

Do not automatically modify or create anything.

If useful, offer to open or summarize the most relevant notebook.

---

# Search followed by another operation

Search can be used to discover a notebook before another operation.

For example:

```text
User:
Find my deployment notebook and update it with this new information.
```

The workflow is:

```text
search_project
    ↓
identify notebook
    ↓
get_notebook if needed
    ↓
update_notebook
```

Do not update a notebook until the target is sufficiently identified.

---

# Ambiguous results

If several notebooks are similarly relevant and the user's request requires a mutation, do not guess.

For example:

```text
Deployment.md
Deployment Guide.md
Production Deployment.md
```

If the user says:

> "Update the deployment notebook."

Search first.

If multiple results remain plausible, ask which notebook they mean.

For a read-only search request, multiple relevant results can simply be returned as a ranked list.

---

# Empty results

If `search_project` returns:

```json
{
  "result_count": 0,
  "items": []
}
```

Do not claim that the information does not exist anywhere in the project.

Instead explain that no matching notebook results were found for the query.

Useful next steps include:

* trying different search terms
* searching a related concept
* checking a specific notebook
* checking another project

---

# Security and authorization

Search is always performed for the authenticated MCP user.

The model must never provide:

```text
user_id
```

to `search_project`.

The authenticated user identity is supplied by the MCP server.

The service layer performs project access validation before searching.

The model must never:

* search an arbitrary project without authorization
* request OAuth tokens
* expose authentication information
* expose use login credentials
* expose server secrets
* bypass project access controls

---

# Current search scope

The current implementation supports:

```text
Project
└── Notebook knowledge
    ├── notebook blocks
    ├── BM25 retrieval
    ├── TF-IDF retrieval
    └── index retrieval
```

The current MCP contract should not claim that file search is supported.

File search can be added later when file indexing and source resolution are implemented.

---

# Tool selection rules

Use:

```text
search_project
```

when the user wants semantic or keyword discovery across project knowledge.

Use:

```text
get_projects
```

when the project is unknown and must be identified.

Use:

```text
get_notebook
```

when the user wants to retrieve a specific notebook, its content, search notebook filesystem items, or inspect a notebook after discovery.

Use:

```text
create_notebook
```

only when the user explicitly wants a new notebook or folder.

Use:

```text
update_notebook
```

only when the user explicitly wants existing notebook data changed.

Use:

```text
delete_notebook
```

only when the user explicitly wants an existing notebook item deleted.

---

# Response guidelines

Search responses should be concise and useful.

Prefer:

```text
I found 3 relevant notebooks:

1. Humanoid Robotics.md
   Most relevant result. Contains notes on locomotion simulation.

2. Robotics Simulation.md
   Contains simulation experiments.

3. Robotics Research.md
   Contains broader robotics research.

I can open the first one if you want.
```

Do not expose internal retrieval details unless useful.

Do not describe the BM25, TF-IDF, or index score as model confidence.

---

# Reference documentation

For detailed tool behavior, read:

```text
reference/search-tool-reference.md
```

For common workflows, read:

```text
reference/search-workflows.md
```

Examples are available in:

```text
examples/
```

Use the reference documentation when the user's request requires detailed search behavior or a multi-step workflow.

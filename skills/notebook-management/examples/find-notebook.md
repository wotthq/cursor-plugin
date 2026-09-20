---
name: find-notebook-example
description: Example workflow for finding and reading an wott notebook or folder.
metadata:
  short-description: Find wott notebooks
---

# Find Notebook

## User request

> Find my notebooks about humanoid robotics.

## Workflow

1. Search notebooks using the topic.
2. Review the returned results.
3. Identify relevant notebooks.
4. If the user is asking for information from those notebooks, retrieve the relevant notebooks when necessary.
5. Use the retrieved information to answer the user's request.

## MCP interaction

```text
search notebooks
    query: "humanoid robotics"
        ↓
Relevant notebooks
        ↓
Review results
        ↓
get_notebook for relevant notebooks
        ↓
Build context
```

## Finding a specific notebook

User:

> Find my robotics notebook.

If exactly one relevant notebook is found, return it.

If several are found, ask the user to clarify.

Example:

> I found two notebooks related to robotics: "Robotics Research" and "Robotics Hardware". Which one do you mean?

## Building context

User:

> Find my notebooks about humanoid robotics and give me the context I need for this research.

Workflow:

```text
search notebooks
        ↓
identify relevant notebooks
        ↓
get relevant notebook content
        ↓
extract relevant information
        ↓
build research context
```

The response should focus on information relevant to the user's research.

Do not summarize unrelated notebook content.

## No results

If wott returns no relevant notebooks:

> I couldn't find any notebooks related to "humanoid robotics" in wott.

Do not fabricate results.

## Project-aware search

User:

> Find the notebooks about locomotion in my Robotics project.

Workflow:

```text
Resolve Robotics project
        ↓
Search notebooks within project
        ↓
Filter for locomotion
        ↓
Retrieve relevant notebooks
        ↓
Build context
```

Keep the project boundary when supported by the wott MCP tools.

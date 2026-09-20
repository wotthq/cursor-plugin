---
name: search-workflows
description: Guidance for searching notebooks within wott projects, including query construction, project selection, result handling, and follow-up notebook retrieval.
metadata:
  short-description: wott project search workflows
---

# Search Workflows

Common workflows for the Project Search skill.

---

# 1. Search a known project

When the project ID is already known:

```text
User request
    ↓
search_project
    ↓
Return ranked notebook results
```

Example:

```json
{
  "project_id": "project_123",
  "query": "humanoid robotics",
  "limit": 5
}
```

No project discovery is required.

---

# 2. Search a project by name

When the user gives a project name but not its ID:

```text
User
 ↓
get_projects
 ↓
identify project
 ↓
search_project
```

Example:

```text
User:
Search my AI Research project for humanoid robotics.
```

The model should first identify:

```text
AI Research
```

Then use its actual:

```text
project_id
```

for `search_project`.

---

# 3. Search and open the best notebook

When the user wants information and then needs the actual notebook:

```text
get_projects
    ↓
search_project
    ↓
select relevant notebook
    ↓
get_notebook
```

Example:

```text
User:
Find my notes about MCP OAuth and show me the most relevant notebook.
```

The search identifies a notebook.

Then:

```text
get_notebook({
  project_id,
  item_id: markdown_item_id
})
```

retrieves the actual notebook.

---

# 4. Search and summarize

Workflow:

```text
get_projects
    ↓
search_project
    ↓
identify relevant notebook(s)
    ↓
get_notebook
    ↓
summarize
```

Use this when the user asks:

> "Find my notes about PostgreSQL and summarize them."

Do not summarize only from the search result metadata if the actual content is required.

---

# 5. Search before updating

Search can be used to identify the target of a later mutation.

Example:

```text
User:
Find my deployment notebook and update it with this information.
```

Workflow:

```text
get_projects
    ↓
search_project
    ↓
identify notebook
    ↓
get_notebook if necessary
    ↓
update_notebook
```

The model must not update a notebook solely because its name looks similar if several candidates remain plausible.

---

# 6. Search before deleting

For destructive requests:

```text
get_projects
    ↓
search_project
    ↓
identify exact target
    ↓
ask for confirmation/clarification when necessary
    ↓
delete_notebook
```

If multiple notebooks match:

```text
Do not delete.
Ask for clarification.
```

Search itself is read-only.

---

# 7. Multiple relevant search results

If the user asks:

> "Find everything I have about robotics."

The search can return several results.

Example response:

```text
I found 5 relevant notebooks:

1. Humanoid Robotics.md
2. Robotics Simulation.md
3. Robot Learning.md
4. Robotics Research.md
5. Manipulation Experiments.md
```

If the user asks for a specific notebook afterward, retrieve it with `get_notebook`.

---

# 8. No search results

If no results are returned:

```text
result_count: 0
```

Respond:

```text
I couldn't find a matching notebook for that search in this project.
```

Do not state:

```text
There is no information about this topic.
```

The search query may simply not match the stored content.

Offer alternative search terms when useful.

---

# 9. Ambiguous project

If the user says:

> "Search my research project for robotics."

and several projects are possible:

```text
AI Research
Robotics Research
Physical AI Research
```

Do not arbitrarily select one.

Ask:

```text
Which project should I search: AI Research, Robotics Research, or Physical AI Research?
```

Do not call `search_project` until the project is identified.

---

# 10. Ambiguous notebook

If search returns:

```text
Robotics.md
Robotics Research.md
Robotics Notes.md
```

and the user asks:

> "Update the robotics notebook."

Do not select one arbitrarily.

Ask the user to identify the intended notebook.

For read-only discovery, returning all relevant results is acceptable.

---

# 11. Search query refinement

If the first search returns weak results, refine the query based on the user's actual intent.

Example:

First:

```text
deployment
```

Possible refinement:

```text
production deployment Cloud Run
```

Another:

```text
MCP deployment OAuth Cloud Run
```

Do not repeatedly search unrelated terms simply to increase the number of results.

---

# 12. Search for a technical decision

Example:

```text
User:
Find where we discussed PostgreSQL versus MongoDB.
```

Use a query such as:

```text
PostgreSQL MongoDB database decision
```

Then inspect the most relevant notebooks.

If the user asks for the actual decision record, retrieve the notebook content.

---

# 13. Search for a person or entity

Example:

```text
User:
Find notes mentioning Sam Altman in this project.
```

Search:

```text
Sam Altman
```

Do not expose unrelated notebook information.

Return only relevant results.

---

# 14. Search for a concept

Search is not limited to exact notebook names.

Example:

```text
User:
Where did I write about problems with Cloud Run authentication?
```

Search:

```text
Cloud Run authentication problems
```

The hybrid retrieval system can find relevant notebook blocks even when the notebook title does not contain the exact phrase.

---

# 15. Search result ranking

Results are ranked by the search service.

The model should normally present the highest-ranked results first.

Do not reorder results arbitrarily unless there is a clear user-facing reason.

Do not treat the numeric score as a probability.

---

# 16. Search-only safety

The search operation is read-only.

A search request must not cause:

* project creation
* project update
* notebook creation
* notebook update
* notebook deletion

Search should only retrieve information.

---

# 17. Search and context building

Search can be used to build context for another operation.

Example:

```text
User:
I need to update the notebook containing our MCP deployment architecture.
```

Workflow:

```text
search_project
    ↓
find relevant notebook
    ↓
get_notebook
    ↓
understand current content
    ↓
update_notebook
```

The model should use actual retrieved notebook content rather than guessing what the notebook contains.

---

# 18. Current limitations

The current search implementation:

* searches notebook knowledge
* operates within one project at a time
* requires `project_id`
* uses hybrid retrieval
* returns notebook item IDs and enriched notebook information

It does not currently provide:

* cross-project search
* standalone file search
* arbitrary web search
* internet search
* user-wide global search without a project
* modification operations

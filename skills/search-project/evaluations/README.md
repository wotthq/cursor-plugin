---
name: search-project-evaluations
description: Evaluation scenarios for validating wott project notebook search workflows and tool behavior.
metadata:
  short-description: Evaluate wott project search
---

# Project Search Skill Evaluations

Evaluation scenarios for testing the Project Search skill across different model configurations.

## Purpose

These evaluations ensure the Project Search skill:

* Correctly identifies the project to search
* Uses `search_project` for project knowledge discovery
* Preserves important search terms
* Returns relevant notebook results
* Uses `get_notebook` when full notebook content is required
* Does not modify data during search-only requests
* Handles empty search results appropriately
* Handles ambiguous projects and notebooks safely
* Uses the authenticated user's project access
* Does not expose authentication information
* Uses only the current MCP search contract

## Evaluation Files

### search-notebook.json

Tests basic notebook search inside a known project.

### search-topic.json

Tests searching for a technical topic using multiple important terms.

### search-project-context.json

Tests using search to build context before reading or updating a notebook.

### ambiguous-search.json

Tests safe handling of ambiguous projects and multiple matching notebooks.

## Running Evaluations

1. Enable the `search-project` skill.
2. Submit the query from the evaluation file.
3. Provide the conversation context specified in the evaluation file.
4. Verify all expected behaviors.
5. Check the success criteria.
6. Test with different model configurations.
7. Verify that the model uses the current `search_project` tool and schema.

## Expected Skill Behaviors

### Project Discovery

* Uses `get_projects` when the project ID is unknown.
* Identifies the correct project from actual returned data.
* Does not invent project IDs.
* Handles multiple matching projects safely.

### Search

* Uses `search_project`.
* Supplies the actual `project_id`.
* Uses the user's meaningful search terms.
* Uses `limit` appropriately.
* Does not invent unsupported search parameters.
* Does not provide `user_id`.

### Result Handling

* Presents ranked notebook results clearly.
* Uses `markdown_item_id` to retrieve a specific notebook when needed.
* Does not treat search scores as confidence percentages.
* Does not invent notebook content.

### Read and Mutation Boundaries

* Search-only requests remain read-only.
* Uses `get_notebook` for full notebook content.
* Uses `update_notebook` only when the user requests an update.
* Does not use create or delete operations during search.

### Security

* Never sends `user_id`.
* Does not expose OAuth credentials.
* Does not expose Login credentials.
* Does not access projects outside the authenticated user's accessible projects.

### Quality Standards

* Uses the minimum necessary tool calls.
* Preserves technical terms in search queries.
* Does not make unsupported claims when no results are found.
* Asks for clarification when the target project or notebook is ambiguous.
* Does not claim success unless the MCP response confirms it.

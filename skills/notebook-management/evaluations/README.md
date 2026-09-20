---
name: notebook-management-evaluations
description: Evaluation scenarios for validating wott notebook management workflows and tool behavior.
metadata:
  short-description: Evaluate notebook management
---

# Notebook Management Skill Evaluations

Evaluation scenarios for testing the Notebook Management skill across different model configurations.

## Purpose

These evaluations ensure the Notebook Management skill:

* Correctly finds projects and notebook items
* Searches notebooks using names, paths, and content-related context
* Reads notebook files and returns their Markdown content when requested
* Creates Markdown notebook files and folders correctly
* Updates notebook content, names, folders, frontmatter, and metadata
* Moves notebooks without recreating them
* Deletes notebooks safely and understands recursive folder deletion
* Preserves project and notebook context during multi-step workflows
* Handles ambiguous notebook requests safely
* Uses the correct MCP tools and input fields
* Does not attempt to provide or manipulate authentication information

## Evaluation Files

### find-notebook.json

Tests finding and reading a notebook from a project.

**Scenario**: Find a notebook about humanoid robotics and show its contents.

**Key Behaviors**:

* Identifies the correct project
* Uses `get_projects` when the project ID is unknown
* Uses `get_notebook` with the appropriate search selector
* Handles multiple matching notebooks
* Retrieves the specific notebook when the correct item is identified
* Returns Markdown content when requested
* Does not modify notebook data

### create-notebook.json

Tests creating notebook files and folders.

**Scenario**: Create a research folder and a Markdown notebook inside a project.

**Key Behaviors**:

* Identifies the correct project
* Uses `create_notebook`
* Correctly distinguishes `file` and `folder`
* Supplies the required `project_id`, `type`, and `name`
* Preserves supplied Markdown content
* Uses `parent_id` when creating inside a folder
* Does not provide or request `user_id`
* Correctly handles optional frontmatter

### update-notebook.json

Tests updating notebook content and metadata.

**Scenario**: Update a research notebook with new Markdown content and metadata.

**Key Behaviors**:

* Finds the correct notebook
* Uses `update_notebook`
* Supplies `project_id` and `item_id`
* Changes only the fields requested by the user
* Preserves existing information when it is not being changed
* Can update content, name, parent folder, frontmatter, and metadata
* Does not send a `type` field
* Does not send `user_id`

### move-notebook.json

Tests moving and renaming notebook items.

**Scenario**: Move a notebook into another folder and rename it.

**Key Behaviors**:

* Identifies the source notebook
* Identifies the destination folder
* Uses `update_notebook`
* Uses `parent_id` to move the notebook
* Uses `name` to rename the notebook
* Does not recreate the notebook as a workaround
* Understands that Markdown files use the `.md` extension
* Understands that moving a folder can update descendant paths

### delete-notebook.json

Tests safe notebook deletion.

**Scenario**: Delete an obsolete notebook or notebook folder.

**Key Behaviors**:

* Identifies the correct project and notebook item
* Uses `delete_notebook`
* Recognizes deletion as destructive
* Does not delete a similarly named item by mistake
* Asks for clarification when the target is ambiguous
* Understands that deleting a folder also deletes descendants
* Reports the deletion result accurately

### notebook-workflow.json

Tests a complete multi-step notebook workflow.

**Scenario**: Create a research folder, create a notebook inside it, add content, and verify the result.

**Key Behaviors**:

* Discovers the project
* Discovers or creates the required folder
* Creates the notebook in the correct parent
* Adds the requested content
* Uses the correct sequence of tools
* Verifies the final notebook state
* Maintains the correct project and notebook IDs throughout the workflow

### ambiguous-notebook.json

Tests safe handling of ambiguous notebook requests.

**Scenario**: Several notebooks have similar names and the user asks to update "the robotics notebook."

**Key Behaviors**:

* Searches before modifying data
* Detects multiple plausible matches
* Does not arbitrarily select a notebook
* Asks the user to clarify which notebook they mean
* Does not call `update_notebook` until the target is sufficiently identified

## Running Evaluations

1. Enable the `notebook-management` skill.
2. Submit the query from the evaluation file.
3. Provide the conversation context specified in the evaluation file.
4. Verify the expected behaviors.
5. Check the success criteria.
6. Test with different model configurations.
7. Verify that the model uses the current MCP tool names and schemas.

## Expected Skill Behaviors

Notebook Management evaluations should verify:

### Project and Notebook Discovery

* Correctly identifies the relevant project.
* Uses `get_projects` when a project ID is not already available.
* Uses `get_notebook` to search or retrieve notebook items.
* Uses IDs rather than guessing IDs from names.
* Keeps operations scoped to the selected project.

### Reading

* Uses `get_notebook` for notebook retrieval.
* Uses `search` for notebook discovery when appropriate.
* Uses `item_id` or `path` to retrieve a specific item.
* Returns Markdown content when the user asks to see the notebook contents.
* Does not modify a notebook during a read-only request.

### Creation

* Uses `create_notebook` for both files and folders.
* Uses `type: "file"` for Markdown notebooks.
* Uses `type: "folder"` for folders.
* Supplies `parent_id` when creating inside a folder.
* Preserves user-provided Markdown content.
* Uses frontmatter only when appropriate.

### Updates

* Uses `update_notebook` for changes to existing items.
* Uses `item_id` rather than searching and recreating the notebook.
* Does not provide a `type` field.
* Updates only requested fields.
* Preserves unrelated notebook information.
* Understands that `parent_id: null` moves an item to the project root.
* Understands that file names are normalized to Markdown.

### Deletion

* Uses `delete_notebook` for deletion.
* Treats deletion as destructive.
* Requires a sufficiently identified target.
* Understands recursive folder deletion.
* Does not substitute delete-and-recreate for a move or rename.

### Authorization and Security

* Never asks the user for or supplies `user_id` to MCP tools.
* Never exposes OAuth tokens, authorization codes, Login credentials, or server secrets.
* Relies on the authenticated MCP identity for access control.
* Does not attempt to bypass project access checks.

### Quality Standards

* Uses the minimum necessary tool calls.
* Maintains correct project and item IDs across operations.
* Does not invent tool names or parameters.
* Does not claim an operation succeeded unless the tool result confirms it.
* Clearly reports the result of mutations.
* Asks for clarification when the requested target is ambiguous.
* Does not expose unnecessary internal notebook fields.

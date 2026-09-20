---
name: project-management-evaluations
description: Evaluation scenarios for validating wott project management workflows and tool behavior.
metadata:
  short-description: Evaluate project management
---

# Project Management Skill Evaluations

Evaluation scenarios for testing the Project Management skill across different model configurations.

## Purpose

These evaluations ensure the Project Management skill:

* Correctly finds projects accessible to the authenticated user
* Creates new projects with the requested name and description
* Updates existing project information safely
* Handles project discovery when project IDs are unknown
* Preserves project data that the user did not ask to change
* Uses the correct MCP tools and input fields
* Handles ambiguous project requests safely
* Maintains project context across multi-step workflows
* Does not invent project IDs or user IDs
* Does not expose internal project, authentication, or login fields

## Evaluation Files

### find-project.json

Tests finding an existing project.

**Scenario**: Find a project by name and return its relevant information.

**Key Behaviors**:

* Uses `get_projects` to retrieve accessible projects
* Identifies the correct project from the returned results
* Does not modify the project
* Handles multiple similar project names
* Returns useful project information without exposing internal fields

### create-project.json

Tests creating a new project.

**Scenario**: Create a new project with a name and optional description.

**Key Behaviors**:

* Uses `create_project`
* Supplies the requested project name
* Supplies the description when provided
* Does not provide `user_id`
* Does not invent unsupported project fields
* Reports the returned project ID and slug

### update-project.json

Tests updating existing project information.

**Scenario**: Update a project's name, description, or location information.

**Key Behaviors**:

* Finds the correct project when project_id is unknown
* Uses `update_project`
* Supplies the existing project_id
* Updates only requested mutable fields
* Does not attempt to update unsupported administrative fields
* Does not provide `user_id`
* Does not recreate the project

### ambiguous-project.json

Tests safe handling of ambiguous project requests.

**Scenario**: Multiple projects have similar names and the user asks to update one without enough identifying information.

**Key Behaviors**:

* Searches accessible projects first
* Detects multiple plausible matches
* Does not arbitrarily select a project
* Asks the user to clarify
* Does not perform an update until the target is sufficiently identified

### project-workflow.json

Tests a complete project workflow.

**Scenario**: Find an existing project, update its description and location, then verify the result.

**Key Behaviors**:

* Discovers the correct project
* Uses the returned project_id for mutation
* Uses `update_project` for the requested changes
* Preserves unrelated project fields
* Verifies or accurately reports the resulting project state

## Running Evaluations

1. Enable the `project-management` skill.
2. Submit the query from the evaluation file.
3. Provide the conversation context specified in the evaluation file.
4. Verify the expected behaviors.
5. Check the success criteria.
6. Test with different model configurations.
7. Verify that the model uses the current MCP tool names and schemas.

## Expected Skill Behaviors

### Project Discovery

* Uses `get_projects` to discover accessible projects.
* Uses project names and descriptions to identify relevant projects.
* Uses the actual returned `project_id`.
* Does not guess project IDs.
* Handles multiple matching projects safely.

### Project Creation

* Uses `create_project`.
* Supplies the required project name.
* Supplies an optional description when requested.
* Does not send unsupported administrative fields.
* Does not provide `user_id`.

### Project Updates

* Uses `update_project`.
* Supplies an existing project ID.
* Updates only fields supported by the current MCP contract:

  * `project_name`
  * `description`
  * `location`
  * `region`
  * `country_code`
  * `region_code`
* Preserves unrelated project information.
* Does not attempt to modify status, visibility, membership, billing, API keys, or other administrative settings through the current MCP tool.

### Authorization and Security

* Never asks the user for `user_id`.
* Never sends `user_id` as an MCP tool parameter.
* Relies on the authenticated MCP identity for access control.
* Does not expose OAuth tokens, authorization codes, login credentials, or server secrets.
* Does not attempt to access projects outside the authenticated user's accessible projects.

### Quality Standards

* Uses the minimum necessary tool calls.
* Maintains the correct project ID throughout a workflow.
* Does not invent tools or parameters.
* Does not claim an operation succeeded unless the MCP tool confirms it.
* Asks for clarification when the requested project is ambiguous.
* Clearly reports the result of project mutations.

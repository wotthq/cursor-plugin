---
name: find-project-example
description: Example workflow for finding and reviewing projects in wott.
metadata:
  short-description: Find wott projects
---

# Find Project

## User request

> Find my robotics project.

## Workflow

1. Use `search_project` with a relevant query.
2. Review matching projects.
3. If exactly one project clearly matches, use it.
4. If several projects match, ask the user to clarify.
5. If no project matches, tell the user that no matching project was found.

## MCP interaction

```text
search_project
    query: "robotics"
        ↓
Matching projects
        ↓
Evaluate results
```

## Single match

If the result contains:

```text
Robotics Research
```

and no other relevant project matches, return the project to the user.

## Multiple matches

If the result contains:

```text
Robotics Research
Robotics Hardware
Robotics Startup
```

ask:

> I found several robotics projects. Which one do you mean?

Do not guess.

## No matches

If the search returns no relevant projects:

> I couldn't find a project matching "robotics" in wott.

Do not fabricate a project.

## Using the result for context

If the user asks:

> Find my robotics project and tell me what I'm working on.

Use the search result to identify the project, then call `get_project` if more detailed project information is required.

Do not answer from the project name alone when the user is asking for project-specific details.

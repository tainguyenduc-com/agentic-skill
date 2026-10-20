---
name: agent-task-manager
description: Manage hierarchical AI agent task lifecycles, workspace-scoped subtasks, dependencies, capability matching, and auto-dispatch using agent-task-manager MCP tools.
---

# Agent Task Manager Skill

Use this skill when orchestrating multi-agent tasks, tracking progress across complex workflows, or dispatching tasks based on agent capabilities.

## MCP Prerequisites
Ensure `agent-task-manager` MCP server is running or registered in your Claude configuration:

```json
{
  "mcpServers": {
    "agent-task-manager": {
      "command": "node",
      "args": ["C:/Users/anhta/source/repos/tainguyenduc-com/tools/agent-task-manager/dist/mcp-server.js"],
      "env": {
        "DATABASE_URL": "file:C:/Users/anhta/source/repos/tainguyenduc-com/tools/agent-task-manager/prisma/dev.db"
      }
    }
  }
}
```

## Project Context and Isolation

Each project workspace has one `agentTask.json` containing unique `projectId`, team identity, and project agents. Before any task, agent, assignment, or dispatch operation, call `load_workspace({ workspacePath })`. This reads `agentTask.json`, registers project context, and synchronizes project agents idempotently. Do not create or query tasks before `load_workspace` succeeds. Use `init_workspace({ workspacePath, projectId, teamId, teamName })` only when creating a new `agentTask.json`; use `load_workspace` for existing workspaces.

Always pass `projectId` to `list_tasks`, `list_task_tree`, `create_task`, and `auto_assign_tasks`. Root task creation requires project context. Task assignment rejects agents from another project. `list_agent_tasks` derives project scope from the agent's workspace, so it cannot return tasks from other projects.

```json
{
  "projectId": "account-service",
  "team": { "id": "platform-team", "name": "Platform Team" },
  "agents": [
    { "name": "BackendBot", "capabilities": ["code-generation", "testing"] }
  ]
}
```

`projectId` is unique. Existing `agentTask.json` is never overwritten with another project ID.

## Task Hierarchy Rules

Team manager owns root task. Create root task once per deliverable, then split independently assignable work into subtasks with `parentTaskId`. Assign member agents to subtasks; keep parent task for coordination and acceptance.

Required sequence and fields:

1. Existing workspace: call `load_workspace({ workspacePath })`; retain returned `projectId`, workspace ID, and agent IDs.
2. New workspace: call `init_workspace({ workspacePath, projectId, teamId, teamName })`, then call `load_workspace({ workspacePath })` before other operations.
3. Root task: `title`, returned `projectId`, priority, capabilities. Do not send empty `workspacePath`, `workspaceId`, or `assignedTo` fields.
4. Subtask: returned `projectId`, `title`, exact full `parentTaskId`; omit workspace fields to inherit parent workspace.
5. Cross-workspace subtask: forbidden. Supplied workspace must match parent workspace.

IDs are opaque, case-sensitive strings. Copy full returned `id` exactly. Never truncate, regenerate, guess, or pass a title in `taskId`. Before `get_task`, `update_task`, `assign_task`, or `complete_task`, verify exact ID from `create_task`, `list_tasks`, or `list_task_tree` response.

Use `list_task_tree` to inspect hierarchy. It returns root tasks with nested `subtasks`; each task includes workspace data. Use `get_task` for one exact task and its detailed parent/subtask data.

Parent completion is manual. Completing a subtask does not complete its parent. Manager reviews all subtasks, then completes parent.

## Standard Task Orchestration Lifecycle

```
[1. Create Root + Workspace] → [2. Create Subtasks] → [3. Assign Members] → [4. Execute] → [5. Complete Subtasks] → [6. Review + Complete Parent]
``` 

For every task operation, preserve workspace scope and full task ID.

### 1. Register Agents
Before assigning tasks, ensure agents are registered in the system:

```json
// Tool: register_agent
{
  "name": "AI Engineer Agent",
  "type": "MCP_SERVER",
  "capabilities": ["code-generation", "testing", "refactoring"],
  "maxCapacity": 3
}
```

### 2. Create Root Task

```json
// Tool: create_task
{
  "title": "Ship account settings",
  "description": "Coordinate API, UI, and verification",
  "workspacePath": "C:\\repos\\account-service",
  "priority": "HIGH",
  "requiredCapabilities": ["orchestration"]
}
```

Record exact returned `id` as `ROOT_ID`.

### 3. Create Subtasks

```json
// Tool: create_task
{
  "title": "Implement settings API",
  "description": "Add endpoint and tests",
  "parentTaskId": "ROOT_ID_FULL_EXACT_STRING",
  "requiredCapabilities": ["code-generation", "testing"],
  "tags": ["settings", "api"]
}
```

Repeat for UI and verification. Omit `workspacePath` and `workspaceId` so parent workspace is inherited. Record each returned full ID. If a subtask must use another workspace, create a separate root task instead.

### 4. Inspect and Dispatch

Call `list_task_tree` with root workspace before assignment. Assign each subtask by its exact returned ID. Auto-assign pending subtasks with `auto_assign_tasks`; parent coordination task should not be assigned to a member unless explicitly intended.

When an agent picks up a subtask:
- Call `update_task({ taskId: "FULL_SUBTASK_ID", status: "IN_PROGRESS" })`.

### 5. Complete Subtasks, Then Parent

When member work finishes:
- Call `complete_task({ taskId: "FULL_SUBTASK_ID", success: true, result: { commit: "hash" } })`.
- Verify result with `get_task` using exact full ID.
- Completing a subtask does not complete parent. Review all children, then call `complete_task` on full parent ID.

### 6. Progress Tracking

- `list_task_tree({ workspacePath: "C:\\repos\\account-service" })`
- `list_tasks({ status: "IN_PROGRESS", workspacePath: "C:\\repos\\account-service" })`
- `get_dashboard_stats()`

If `get_task` returns `null`, first re-copy full ID from `list_task_tree`; likely cause is truncated or mistyped ID, or MCP server points to a different `DATABASE_URL`.

## Related Tasks and Reference Attachments

Use `add_related_task` for shared context only. It creates one symmetric `related-to` link and has no status, dependency, or dispatch effect. Pass exact full IDs; self-links are rejected.

```json
{
  "taskId": "FULL_TASK_ID",
  "relatedTaskId": "FULL_OTHER_TASK_ID"
}
```

Before execution, call `list_related_tasks` and `list_task_attachments` to collect context. Use `add_task_attachment` for metadata pointing to a URL or local/workspace path; no binary upload occurs.

```json
{
  "taskId": "FULL_TASK_ID",
  "name": "API design",
  "uri": "C:\\repos\\account-service\\docs\\api-design.md",
  "mimeType": "text/markdown",
  "description": "Approved endpoint contract"
}
```

`delete_task_attachment` removes only stored metadata. It never deletes the external file or URL target. `remove_related_task` removes only the task link.

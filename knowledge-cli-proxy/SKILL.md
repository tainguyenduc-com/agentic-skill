---
name: knowledge-cli-proxy
description: Query semantic knowledge bases, SOPs, architectural docs, and team Q&A via MCP Proxy interface (knowledge-proxy) for multi-agent workflows.
---

# Knowledge CLI MCP Proxy Skill

Use this skill when sub-agents (`ba`, `sa`, `se-*`, `qa`, `debugger`, `reviewer`) need to query project documentation, architectural decisions, coding conventions, or Standard Operating Procedures (SOPs).

## Core Philosophy: Proxy First
- **No Direct Bash Knowledge Queries**: Sub-agents must not invoke direct `knowledge search query` via raw Bash, ensuring adherence to read-only execution hooks and security policies.
- **Audit Logging**: Every invocation must supply `--agent <agent-id>` to maintain an accurate team activity log.
- **Structured Output**: Always request `--format json` for programmatic ingestion and zero-fluff extraction.

---

## MCP Server / CLI Configuration

Configure in Claude Desktop / Agent config (`claude_desktop_config.json` or `.claude.json`):

```json
{
  "mcpServers": {
    "knowledge-proxy": {
      "command": "python",
      "args": ["-m", "knowledge_cli.mcp_proxy"],
      "env": {
        "KNOWLEDGE_INDEX_PATH": "./.knowledge_index",
        "OPENAI_API_BASE": "http://localhost:20128/v1"
      }
    }
  }
}
```

Or invoke via command wrapper when exposed in PATH / workspace environment:

```bash
knowledge-proxy query "<query_text>" --agent <agent_id> [options]
```

---

## Command Syntax & Options

```bash
knowledge-proxy query "<query_text>" --agent <agent_id> [flags]
```

### Parameters
- `query_text` (string, required): Search query or question.
- `--agent, -a <agent_id>` (string, required): Identity of the calling agent (e.g., `se-backend`, `qa`, `sa`).
- `--type, -t <type>` (string, optional): Document filter: `docs`, `sop`, `qa`, `data`.
- `--limit, -l <number>` (number, optional): Max results to return (default: `5`).
- `--min-score <score>` (float, optional): Similarity threshold `0.0` - `1.0` (recommended: `0.7`).
- `--format, -f <format>` (string, optional): `json` (default), `table`, `text`.

---

## Usage Scenarios

### 1. Backend Engineer (`se-backend`) - Fetch SOP & Deployment Rules
```bash
knowledge-proxy query "deployment procedure staging" --agent se-backend --type sop --limit 3 --format json
```

### 2. Software Architect (`sa`) - Check Architecture Standards
```bash
knowledge-proxy query "microservice communication pattern" --agent sa --type docs --min-score 0.75 --format json
```

### 3. QA Engineer (`qa`) - Retrieve Test Criteria & Edge Cases
```bash
knowledge-proxy query "authentication test scenarios" --agent qa --type qa --limit 5 --format json
```

### 4. Code Reviewer (`reviewer`) - Verify Coding Conventions
```bash
knowledge-proxy query "error handling guidelines" --agent reviewer --type docs --limit 2 --format json
```

---

## Python Client Implementation (Inside Tools / Sub-agents)

```python
import subprocess
import json
from typing import Optional, Dict, Any

def search_knowledge(
    query: str,
    agent_id: str,
    doc_type: Optional[str] = None,
    limit: int = 5,
    min_score: float = 0.0
) -> Dict[str, Any]:
    """Execute knowledge search via knowledge-proxy wrapper."""
    cmd = [
        "knowledge-proxy", "query", query,
        "--agent", agent_id,
        "--limit", str(limit),
        "--format", "json"
    ]
    if doc_type:
        cmd.extend(["--type", doc_type])
    if min_score > 0.0:
        cmd.extend(["--min-score", str(min_score)])

    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode != 0:
        raise RuntimeError(f"Knowledge proxy error: {result.stderr.strip()}")

    return json.loads(result.stdout)
```

---

## Response Schema (JSON)

```json
{
  "query": "deployment procedure staging",
  "total": 1,
  "agent_id": "se-backend",
  "results": [
    {
      "id": "sop_deploy_01",
      "document": "Full procedure steps...",
      "metadata": {
        "title": "Staging Deployment SOP",
        "type": "sop",
        "file_name": "deploy-staging.md",
        "source": "./knowledge_data/procedures/deploy-staging.md"
      },
      "similarity_score": 0.88,
      "distance": 0.12
    }
  ]
}
```

---

## Portability & Copying to Other Repositories

To use this skill in any project repo:
1. Copy folder `skills/knowledge-cli-proxy` into target project root or `.claude/skills/`.
2. Ensure `knowledge-cli` is installed or accessible via PATH / virtualenv.
3. Sub-agents reference `skill: knowledge-cli-proxy` directly.

## MCP Nexus — Miner Design

### Miner Tasks

MCP Nexus miners run **MCP servers** — implementations of the Model Context Protocol that provide specific capabilities to AI assistants.

#### Types of Miner Tasks

| Task | Description |
|------|-------------|
| **Server Operation** | Run an MCP server endpoint that responds to protocol requests |
| **Capability Implementation** | Implement one or more MCP capabilities (e.g., `email.send`, `file.read`) |
| **Metadata Maintenance** | Keep server info current (version, description, endpoint) |
| **Query Response** | Answer discovery queries from validators and AI agents |

---

### Expected Input → Output Format

#### Miner Registration

**Input** (from miner operator):
```json
{
  "name": "gmail-mcp-server",
  "description": "Full Gmail integration with OAuth2",
  "capabilities": ["email.read", "email.send", "contacts.read", "attachments.upload"],
  "version": "1.2.0",
  "endpoint": "https://mcp.example.com/gmail",
  "category": "productivity",
  "auth_type": "oauth2"
}
```

**Output** (on-chain registration):
```json
{
  "hotkey": "5HEo565WAy4...",
  "uid": 42,
  "name": "gmail-mcp-server",
  "registered_at": 1234567,
  "emission": 0
}
```

#### Query Request (from validator or AI agent)

**Input**:
```json
{
  "query": "Find MCP servers that can send emails",
  "required_capabilities": ["email.send"],
  "limit": 5
}
```

**Output**:
```json
{
  "results": [
    {
      "name": "gmail-mcp-server",
      "score": 0.92,
      "endpoint": "https://mcp.example.com/gmail",
      "latency_ms": 145
    },
    {
      "name": "outlook-mcp",
      "score": 0.88,
      "endpoint": "https://mcp.example.com/outlook",
      "latency_ms": 210
    }
  ]
}
```

---

### Performance Dimensions

| Dimension | Metric | Weight |
|-----------|--------|--------|
| **Availability** | Uptime % over evaluation period | 30% |
| **Correctness** | Valid response rate | 30% |
| **Latency** | Response time percentile | 15% |
| **Security** | HTTPS + basic security checks | 15% |
| **Metadata** | Description completeness | 10% |

#### Scoring Formula

```
availability = (successful_requests / total_requests)
correctness = (valid_responses / successful_requests)
latency_score = max(0, 1 - (avg_latency_ms / 10000))  // 10s = 0, 0ms = 1
security = 1.0 if https AND no_vulns else 0.5
metadata = completeness(description) / 100

total_score = 0.30 × availability 
            + 0.30 × correctness 
            + 0.15 × latency_score 
            + 0.15 × security 
            + 0.10 × metadata
```

# MCP Nexus — Incentive & Mechanism Design

## Executive Summary

**MCP Nexus** is a decentralized registry and verification layer for MCP (Model Context Protocol) servers — the emerging standard for connecting AI assistants to external tools.

It's been said that **"MCP is the new App Store"** but unlike the iPhone App Store, this "App Store" has no centralized curator. AI assistants need a way to discover, verify, and choose the best tools autonomously — without vendor lock-in.

MCP Nexus provides:
- **Discovery**: Find MCP servers by capability, category, or task
- **Verification**: Decentralized quality testing by validators
- **Reputation**: Transparent, on-chain scoring of tool quality
- **Incentives**: Token rewards for maintaining high-quality servers

This subnet creates the missing infrastructure for the agentic AI era.

---

## Emission and Reward Logic

### Emission Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROOT NETWORK (Subnet 0)                      │
│                    ~5M TAO/day distributed                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MCP NEXUS SUBNET                              │
│                    Emission: X TAO/epoch                         │
│                                                                  │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐  │
│  │  Validators  │────▶│   Weights    │────▶│   Emissions  │  │
│  │  (scorers)   │     │  (on-chain)  │     │  (to miners)  │  │
│  └──────────────┘     └──────────────┘     └──────────────┘  │
│         │                                             ▲         │
│         │ 4% validator commission                    │         │
│         └─────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Reward Formula

Each epoch, validators set weight `w_i` for each miner `i`. Miner emissions:

```
miner_emission_i = (w_i / Σw_j) × subnet_emission × 0.96

validator_reward = subnet_emission × 0.04
```

Where `w_i` (weight) = normalized total score from validators.

---

## Incentive Alignment for Miners and Validators

### Miner Incentives

| Incentive | Description |
|-----------|-------------|
| **Emission Revenue** | Higher scores → more TAO |
| **Discovery Traffic** | Top-ranked servers get more queries from AI agents |
| **Network Effects** | More users → more valuable the registry |
| **Staking Rewards** | Miners can stake earned TAO for additional rewards |

**Alignment**: Miners earn more when they provide high-quality, available, correct services. Low-quality servers lose emissions and eventually get deregistered.

### Validator Incentives

| Incentive | Description |
|-----------|-------------|
| **Validator Commission** | 4% of subnet emissions |
| **Stake Weight** | Higher stake = more influence over weights |
| **Reputation** | Validators with accurate scores attract delegation |

**Alignment**: Validators earn more by correctly identifying high-quality miners. Misaligned validators (sybil attacks, collusion) risk losing stake and reputation.

---

## Mechanisms to Discourage Low-Quality or Adversarial Behavior

### For Miners

| Threat | Mitigation |
|--------|------------|
| **Fake registration** | Validators test endpoints — if unreachable, score = 0 |
| **Gaming tests** | Randomized test prompts; validators use different test suites |
| **Sybil attacks** | Registration requires skin in the game (TAO burn) |
| **Good enough scores** | Minimum threshold to earn emissions; bottom 10% gets 0 |

### For Validators

| Threat | Mitigation |
|--------|------------|
| **Collusion** | Distributed validator set; no single validator dominates |
| **Bribery** | Validator stake at risk; public scores allow detection |
| **Slashing** | Misbehavior leads to stake slashing (future upgrade) |
| **Free-riding** | Must actively test to set meaningful weights |

### Immune Period Protection

- New miners get 4096 blocks (~14 hours) immunity
- Prevents immediate deregistration attacks
- Gives time to prove capability

---

## How This Design Qualifies as "Proof of Intelligence" or "Proof of Effort"

### Proof of Intelligence

1. **Intelligent Tool Selection**: AI agents must intelligently query the registry and select the best tool based on capability matching and quality scores. This requires semantic understanding of what tools can do.

2. **Adaptive Testing**: Validators generate test cases that probe actual capability — not just mechanical responses. They must understand what "good" tool responses look like across different tool types.

3. **Quality Assessment**: Validators evaluate security, performance, correctness — requiring judgment beyond simple pass/fail checks.

4. **Capability Matching**: Matching query intent to tool capabilities requires understanding the semantics of both user requests and tool descriptions.

### Proof of Effort

1. **Active Testing**: Validators must actively test miners every epoch (~20 minutes) — sending requests, measuring responses, computing scores. No free rides.

2. **Server Operation**: Miners must run actual MCP servers with real capabilities — maintaining uptime, handling requests, keeping software current.

3. **Ongoing Verification**: Continuous scoring across epochs ensures miners maintain quality over time, not just at launch.

4. **Stake at Risk**: Both miners (registration cost) and validators (staked TAO) have economic skin in the game.

This is not a simple lookup table — it requires genuine intelligence to evaluate quality and genuine effort to test continuously.

---

## High-Level Algorithm

### Task Assignment

```
EVERY EPOCH (100 blocks):

1. VALIDATOR DISCOVERS MINERS
   - Pull metagraph for subnet
   - Get list of registered miner UIDs + endpoints

2. VALIDATOR ASSIGNS TASKS
   For each miner:
     - Select random test prompt from test suite
     - POST to miner's MCP endpoint
     - Record: latency, response, correctness

3. MINER PROCESSES REQUEST
   - Receive test prompt
   - Execute MCP capability (simulated)
   - Return structured response

4. VALIDATOR EVALUATES RESPONSE
   - Check: was response valid JSON?
   - Check: does response match expected output?
   - Record: availability (up/down), correctness (right/wrong), latency (ms)

5. VALIDATOR COMPUTES SCORES
   For each miner i:
     score_i = 0.30 × availability_i
            + 0.30 × correctness_i
            + 0.15 × latency_score_i
            + 0.15 × security_i
            + 0.10 × metadata_score_i

6. VALIDATOR SETS WEIGHTS
   - Normalize scores to weights
   - Submit weights to chain via set_weights()

7. CHAIN DISTRIBUTES EMISSIONS
   - Calculate emission per miner
   - Transfer TAO to miner hotkeys
```

### Validation, Scoring, and Reward Flow

```
┌─────────┐     ┌──────────┐     ┌─────────┐     ┌──────────┐
│ Miner   │────▶│ Validator│────▶│ Chain   │────▶│ Emission │
│ Serves  │     │ Tests    │     │ Weights │     │ Paid     │
└─────────┘     └──────────┘     └─────────┘     └──────────┘
    │               │                │               │
    │               │                │               │
  Input:         Score:          Weight:          TAO:
  test prompt   0-1 score       w_i = score      emission_i
```

### Performance Dimensions

| Dimension | Metric | Weight |
|-----------|--------|--------|
| **Availability** | Uptime % over evaluation period | 30% |
| **Correctness** | Valid response rate | 30% |
| **Latency** | Response time percentile | 15% |
| **Security** | HTTPS + basic security checks | 15% |
| **Metadata** | Description completeness | 10% |

### Scoring Formula

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

### Network Parameters

| Parameter | Value |
|-----------|-------|
| Max Miners | 192 |
| Max Validators | 64 |
| Immunity Period | 4096 blocks (~14 hours) |
| Epoch Duration | 100 blocks (~20 minutes) |
| Validator Commission | 4% |
| Registration Cost | ~10 TAO |

---

## Potential Future Upgrades

As MCP evolves and the protocol matures, future subnet upgrades may introduce advanced capabilities that MCP server operators can implement for enhanced functionality. These potential upgrades would be evaluated and integrated through the subnet's governance process.

### Advanced Server Capabilities

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Tiered Schemas** | Offer three levels of tool metadata: Level 0 (name + 1-line), Level 1 (summary), Level 2 (full JSON Schema). Reduces token usage for agents dramatically. |
| 2 | **Dynamic Tool Lists** | Return different tools based on authenticated user/session. Hide admin tools for normal users, send notifications when capabilities change. |
| 3 | **Tasks Primitive** | Long-running operations return task handles immediately with progress tracking. Agents can fire-and-forget parallel tasks instead of blocking. |
| 4 | **Code-Agent Patterns** | Expose tools as filesystem hierarchy (e.g., `/servers/github/repos/list.ts`). Include README.md for complex tools. Return minimal, filterable data by default. |
| 5 | **Semantic Tool Search** | Expose `tools/semantic_search` that accepts natural-language queries and returns ranked tool results. Uses embeddings for intelligent matching. |
| 6 | **Batching & Streaming** | Support parallel tool execution in one call. Return paginated results with continuation tokens. Add batch method for array of tool calls. |
| 7 | **Per-Tool Authorization** | Use OAuth 2.1 scopes with fine-grained access control. Negotiate capabilities during initialization. Return helpful "missing_scope" errors. |
| 8 | **Resource Subscriptions** | Offer `resources/subscribe` for files, DB tables, channels. Push change notifications via SSE or webhooks. Turns passive tools into reactive ones. |
| 9 | **Server-Side Sampling** | Implement "Sampling with Tools" capability. Run short agentic loops internally (e.g., "search GitHub → post to Slack") and return only the final result. |
| 10 | **Observability Endpoints** | Expose `meta/stats` (token usage, latency, error rates) and `meta/trace` (request tracing). Human-readable error messages with suggested fixes. |

### Implementation Roadmap (Future)

| Phase | Features |
|-------|----------|
| **Phase 1** | Tiered Schemas + Semantic Search |
| **Phase 2** | Tasks Primitive + Dynamic Tool Lists |
| **Phase 3** | Code-Agent Patterns + Batching |
| **Phase 4** | Per-Tool Authorization + Resource Subscriptions |
| **Phase 5** | Server-Side Sampling + Observability |

These future upgrades would align with the broader MCP ecosystem evolution and provide differentiation for servers that implement advanced capabilities.

---

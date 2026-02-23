## MCP Nexus — Incentive & Mechanism Design

### Executive Summary

**MCP Nexus** is a decentralized registry and verification layer for MCP (Model Context Protocol) servers — the emerging standard for connecting AI assistants to external tools.

As the article astutely observed: **"MCP is the new App Store."** But unlike the iPhone App Store, this "App Store" has no centralized curator. AI assistants need a way to discover, verify, and choose the best tools autonomously — without vendor lock-in.

MCP Nexus provides:
- **Discovery**: Find MCP servers by capability, category, or task
- **Verification**: Decentralized quality testing by validators
- **Reputation**: Transparent, on-chain scoring of tool quality
- **Incentives**: Token rewards for maintaining high-quality servers

This subnet creates the missing infrastructure for the agentic AI era.

---

### Emission and Reward Logic

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROOT NETWORK (Subnet 0)                      │
│                    ~5M TAO/day distributed                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MCP NEXUS SUBNET                             │
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

**Reward Formula:**

```
miner_emission_i = (w_i / Σw_j) × subnet_emission × 0.96
validator_reward = subnet_emission × 0.04
```

Where `w_i` (weight) = normalized total score from validators.

---

### Incentive Alignment for Miners

| Incentive | Description |
|-----------|-------------|
| **Emission Revenue** | Higher scores → more TAO |
| **Discovery Traffic** | Top-ranked servers get more queries from AI agents |
| **Network Effects** | More users → more valuable the registry |
| **Staking Rewards** | Miners can stake earned TAO for additional rewards |

**Alignment**: Miners earn more when they provide high-quality, available, correct services. Low-quality servers lose emissions and eventually get deregistered.

---

### Incentive Alignment for Validators

| Incentive | Description |
|-----------|-------------|
| **Validator Commission** | 4% of subnet emissions |
| **Stake Weight** | Higher stake = more influence over weights |
| **Reputation** | Validators with accurate scores attract delegation |

**Alignment**: Validators earn more by correctly identifying high-quality miners. Misaligned validators (sybil attacks, collusion) risk losing stake and reputation.

---

### Mechanisms to Discourage Low-Quality or Adversarial Behavior

#### For Miners

| Threat | Mitigation |
|--------|------------|
| **Fake registration** | Validators test endpoints — if unreachable, score = 0 |
| **Gaming tests** | Randomized test prompts; validators use different test suites |
| **Sybil attacks** | Registration requires skin in the game (TAO burn) |
| **Good enough scores** | Minimum threshold to earn emissions; bottom 10% gets 0 |

#### For Validators

| Threat | Mitigation |
|--------|------------|
| **Collusion** | Distributed validator set; no single validator dominates |
| **Bribery** | Validator stake at risk; public scores allow detection |
| **Slashing** | Misbehavior leads to stake slashing (future upgrade) |
| **Free-riding** | Must actively test to set meaningful weights |

#### Immune Period Protection
- New miners get 4096 blocks (~14 hours) immunity
- Prevents immediate deregistration attacks
- Gives time to prove capability

---

### High-Level Algorithm

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
   - Record: availability, correctness, latency

5. VALIDATOR COMPUTES SCORES
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

---

### Performance Dimensions & Scoring Formula

| Dimension | Metric | Weight |
|-----------|--------|--------|
| **Availability** | Uptime % over evaluation period | 30% |
| **Correctness** | Valid response rate | 30% |
| **Latency** | Response time percentile | 15% |
| **Security** | HTTPS + basic security checks | 15% |
| **Metadata** | Description completeness | 10% |

```
availability = (successful_requests / total_requests)
correctness = (valid_responses / successful_requests)
latency_score = max(0, 1 - (avg_latency_ms / 10000))
security = 1.0 if https AND no_vulns else 0.5
metadata = completeness(description) / 100

total_score = 0.30 × availability 
            + 0.30 × correctness 
            + 0.15 × latency_score 
            + 0.15 × security 
            + 0.10 × metadata
```

---

### Network Parameters

| Parameter | Value |
|-----------|-------|
| Max Miners | 192 |
| Max Validators | 64 |
| Immunity Period | 4096 blocks (~14 hours) |
| Epoch Duration | 100 blocks (~20 minutes) |
| Validator Commission | 4% |
| Registration Cost | ~10 TAO |

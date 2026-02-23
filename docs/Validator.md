## MCP Nexus — Validator Design

### Scoring and Evaluation Methodology

#### Validator Responsibilities

1. **Registry Maintenance**: Maintain list of active MCP servers
2. **Active Testing**: Periodically query each miner with test prompts
3. **Score Calculation**: Compute quality scores based on responses
4. **Weight Setting**: Submit scores as weights to Bittensor chain

#### Evaluation Criteria

| Criterion | Test Method | Scoring |
|-----------|-------------|---------|
| **Server Reachable** | HTTP GET/POST to endpoint | 0 or 1 |
| **Valid Response** | JSON parse + schema check | 0 or 1 |
| **Correct Output** | Compare to expected output | 0-1 match |
| **Response Time** | Measure end-to-end latency | 0-1 normalized |
| **Metadata Quality** | Length + keywords + freshness | 0-1 score |

#### Validator Consensus

- Multiple validators independently test each miner
- Scores are aggregated via weighted median
- Outlier scores (too high/low) are discounted
- Final weight = consensus score

---

### Evaluation Cadence

| Phase | Frequency | Description |
|-------|-----------|-------------|
| **Discovery** | Every epoch | Pull latest metagraph |
| **Testing** | Every epoch | Test all active miners |
| **Scoring** | Every epoch | Compute scores |
| **Weight Setting** | Every epoch | Submit to chain |
| **Deep Audit** | Every 10 epochs | More comprehensive tests |

**Epoch Duration**: ~100 blocks (~20 minutes)

---

### Validator Incentive Alignment

```
Validator Stake Requirements:
- Minimum stake: 1000 TAO (self-stake + delegated)
- Top 64 by stake = validator permit
- Commission: 4% of subnet emissions
```

#### Validator Behavior Matrix

| Behavior | Outcome |
|----------|---------|
| Accurate scoring | Earn commission + reputation |
| Collusion with miners | Slashing (future) + reputation loss |
| Non-participation | Lose validator permit |
| Sybil attack | Stake at risk |

#### Skin in the Game

Validators must stake TAO. Misaligned behavior puts stake at risk. This creates economic incentive to score honestly.

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

---

### Data Structures

```python
@dataclass
class MCPServer:
    hotkey: str
    name: str
    description: str
    capabilities: List[str]
    version: str
    endpoint: str
    category: str
    registered_at: int

@dataclass
class MinerScore:
    uid: int
    availability: float      # 0.0-1.0
    correctness: float       # 0.0-1.0
    latency_score: float     # 0.0-1.0
    security_score: float    # 0.0-1.0
    metadata_score: float    # 0.0-1.0
    total: float             # weighted sum
```

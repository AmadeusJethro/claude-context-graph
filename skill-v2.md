# Context Graph Skill v2

Enhanced decision trace system with trajectory capture, structural embeddings, and world model support.

Based on: "How to Build a Context Graph" - the insight that the event clock (trajectories + reasoning) matters more than state.

## Core Concepts

### The Two Clocks
- **State Clock**: What's true now (your current decisions.jsonl)
- **Event Clock**: How it became true (NEW: trajectories.jsonl)

Your original graph captured state changes. V2 captures the WALK through problem space that led to each decision.

### Schema as Output
Entities and their relationships emerge from traversal patterns, not manual tagging. V2 tracks:
- Co-occurrence: which entities appear together in decisions/trajectories
- Structural equivalence: entities that play similar roles even if never directly connected

### World Model
The graph should enable simulation, not just retrieval. V2 adds:
- Pattern extraction from decision sequences
- Predictive queries: "what typically happens when..."
- Counterfactual reasoning: "what if I had done X instead"

---

## New Commands

### `/context trace` - Start trajectory capture
Begin capturing tool calls as trajectory steps. Run before non-trivial investigation.

When active, I will log:
- Each file read, search, grep
- Dead ends and backtracks
- Pivots (when direction changes)
- Entities encountered

End with `/context trace stop` or let it auto-close when decision is logged.

### `/context trace show` - View current trajectory
Show the trajectory being captured.

### `/context predict <scenario>` - What-if query
Ask what typically happens in a scenario based on past decisions.

Examples:
- `/context predict "use XGBoost instead of GradientBoosting"`
- `/context predict "skip COVID years in training data"`
- `/context predict "deploy on Friday"`

I will:
1. Find similar past decisions
2. Analyze their outcomes
3. Identify patterns and risks
4. Return a prediction with confidence

### `/context similar <entity>` - Structural similarity
Find entities that play similar roles based on co-occurrence patterns.

Example: `/context similar gradient-boosting`
Might return: "Structurally similar to: regularization, overfitting-prevention, ml-model"

### `/context pattern <name>` - View/create pattern
View an extracted pattern or mark current decision as following one.

### `/context why <decision-id>` - Full trace
Show the complete trajectory that led to a decision, not just the final choice.

### `/context cluster` - Entity clusters
Show structurally equivalent entity groups.

---

## Original Commands (unchanged)

### `/context log` - Log a decision
Now also:
- Links to active trajectory if one exists
- Suggests similar past decisions
- Updates co-occurrence matrix

### `/context search <query>` - Search for precedent
Now also returns:
- Trajectory summaries for matching decisions
- Pattern matches

### `/context recent [n]` - Show recent decisions

### `/context project <name>` - Project history

### `/context entity <tag>` - Entity lookup
Now also shows:
- Co-occurring entities
- Success rate in decisions involving this entity
- Structural cluster membership

### `/context stats` - Graph statistics
Now also shows:
- Trajectory metrics (avg depth, backtrack rate)
- Pattern counts
- Entity cluster info

---

## Automatic Behavior (enhanced)

### At Session Start
1. Read recent decisions (existing)
2. **NEW**: Load entity embeddings to prime structural understanding
3. **NEW**: Check for unfinished trajectories

### During Problem-Solving
1. **NEW**: Auto-start trajectory capture for complex tasks
2. Check for relevant precedent (existing, now includes pattern matching)
3. **NEW**: Surface structurally similar past decisions

### At Decision Time
1. **NEW**: Before deciding, query "what typically happens"
2. Log decision with reasoning (existing)
3. **NEW**: Link to trajectory
4. **NEW**: Update co-occurrence matrix
5. **NEW**: Check for pattern match or new pattern

### At Session End
1. Update outcomes (existing)
2. **NEW**: Close open trajectories
3. **NEW**: Run pattern extraction on new decisions

---

## File Structure

```
~/.claude/context-graph/
  decisions.jsonl      # Decision events (existing, enhanced)
  trajectories.jsonl   # NEW: Agent walk traces
  entities.json        # Entity index (existing, enhanced with embeddings)
  cooccurrence.json    # NEW: Entity co-occurrence matrix
  patterns.json        # NEW: Extracted patterns
  schema-v2.json       # Enhanced schema
```

---

## Trajectory Example

```json
{
  "tid": "t001",
  "ts_start": "2025-12-27T10:00:00Z",
  "ts_end": "2025-12-27T10:45:00Z",
  "decision_id": "d001",
  "steps": [
    {"seq": 1, "action": "search", "target": "**/*model*.py", "result": "found", "entities_touched": ["ml", "model"]},
    {"seq": 2, "action": "read", "target": "src/model/classifier.py", "result": "insight", "entities_touched": ["gradient-boosting"]},
    {"seq": 3, "action": "grep", "target": "RandomForest", "result": "found", "entities_touched": ["random-forest"]},
    {"seq": 4, "action": "branch", "target": "test RandomForest hypothesis", "result": "dead_end", "pivot": true},
    {"seq": 5, "action": "backtrack", "target": "return to GradientBoosting", "result": "insight"},
    {"seq": 6, "action": "run", "target": "pytest tests/model/", "result": "found", "entities_touched": ["testing"]}
  ],
  "backtrack_count": 1,
  "branch_count": 1,
  "depth": 6,
  "breadth": 5
}
```

---

## Pattern Example

```json
{
  "pid": "p001",
  "name": "Regularization-over-complexity",
  "description": "When choosing ML models, more regularized/simpler models outperform complex ones",
  "trigger_conditions": ["model selection", "trading prediction", "limited training data"],
  "typical_trajectory": ["compare models", "test complex first", "find overfitting", "simplify"],
  "typical_outcome": "s",
  "success_rate": 0.85,
  "instances": ["d001", "d010"],
  "counter_instances": []
}
```

---

## Prediction Example

Query: `/context predict "use neural network for price prediction"`

```json
{
  "query": "use neural network for price prediction",
  "similar_decisions": ["d001"],
  "predicted_outcome": "likely underperform simpler models",
  "confidence": 0.7,
  "key_factors": [
    "d001 rejected neural networks as 'black box, harder to debug'",
    "Pattern 'regularization-over-complexity' applies",
    "No successful neural network decisions in graph"
  ],
  "risks": [
    "Debugging difficulty when predictions fail",
    "Overfitting with limited training data"
  ],
  "suggested_trajectory": [
    "First establish baseline with GradientBoosting",
    "If neural net attempted, use heavy regularization",
    "Prepare interpretability tools"
  ]
}
```

---

## Co-occurrence Matrix Example

```json
{
  "gradient-boosting": {
    "regularization": 2,
    "v70": 2,
    "ml": 2,
    "hyperparameters": 1,
    "overfitting": 1
  },
  "position-sizing": {
    "confidence": 1,
    "risk-management": 1,
    "thresholds": 1
  }
}
```

This enables queries like:
- "What entities typically appear with position-sizing?" -> confidence, risk-management
- "Are 'regularization' and 'overfitting' structurally similar?" -> Yes, they co-occur with the same entities

---

## Migration from v1

Your existing decisions.jsonl is fully compatible. Run:
1. `/context migrate` - Generates cooccurrence.json from existing decisions
2. Start using `/context trace` for new work
3. Patterns will emerge as decisions accumulate

---

## Implementation Notes

The key insight from the article: "The schema isn't the starting point. It's the output."

Don't over-engineer the structure. Let it emerge:
- Trajectories reveal what entities actually matter
- Co-occurrence reveals structural relationships
- Patterns emerge from repeated decision sequences

Start logging trajectories. The rest follows.

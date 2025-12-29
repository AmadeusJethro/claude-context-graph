# Context Graph Skill v2 (FULLY AUTOMATIC)

Decision trace system with trajectory capture, structural embeddings, and world model support.

Based on Animesh Koratana's "How to Build a Context Graph" - captures the WALK through problem space, not just final decisions.

**IMPORTANT: This skill runs automatically. User never types commands. Everything happens silently as exhaust from normal work.**

---

## What Happens Automatically

### `/context log` - Log a decision
Log a decision with context, alternatives, and reasoning.

When invoked, I will:
1. Ask about the decision (or infer from recent work)
2. Capture: type, title, choice, alternatives rejected, reasoning
3. Assign weight (1-3) based on significance
4. Link to active trajectory if one exists
5. Suggest similar past decisions
6. Update co-occurrence matrix

### `/context search <query>` - Search for precedent
Find similar past decisions.

Examples:
- `/context search position sizing` - Find decisions about position sizing
- `/context search debugging timezone` - Find timezone debugging approaches
- `/context search failed` - Find failed decisions and lessons learned

### `/context recent [n]` - Show recent decisions
Display the last N decisions (default 10).

### `/context project <name>` - Project history
Show all decisions for a specific project.

### `/context entity <tag>` - Entity lookup
Find all decisions tagged with an entity. Also shows:
- Co-occurring entities
- Success rate for decisions involving this entity
- Structural cluster membership

### `/context stats` - Graph statistics
Show decision counts by type, project, outcome. Also shows:
- Trajectory metrics (avg depth, backtrack rate)
- Pattern counts
- Entity cluster info

---

## NEW: Trajectory Commands

### `/context trace` - Start trajectory capture
Begin capturing tool calls as trajectory steps. Run before non-trivial investigation.

When active, I will log:
- Each file read, search, grep
- Dead ends and backtracks
- Pivots (when direction changes)
- Entities encountered

End with `/context trace stop` or auto-close when decision is logged.

### `/context trace show` - View current trajectory
Show the trajectory being captured.

### `/context why <decision-id>` - Full trace
Show the complete trajectory that led to a decision, not just the final choice.

---

## NEW: Prediction Commands

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
Returns: "Structurally similar to: regularization, overfitting-prevention, ml-model"

### `/context pattern [name]` - View/create pattern
View an extracted pattern or mark current decision as following one.

### `/context cluster` - Entity clusters
Show structurally equivalent entity groups.

---

## Automatic Behavior (ALL SILENT - NO USER ACTION)

### At Session Start
1. Silently read recent decisions from decisions.jsonl
2. Load entity embeddings for structural understanding
3. Check for unfinished trajectories

### During Problem-Solving (AUTOMATIC TRAJECTORY CAPTURE)
When Claude is investigating, debugging, or exploring:
- Track every file read, search, grep silently
- Note dead ends and backtracks
- Capture pivots when direction changes
- All logged to trajectories.jsonl without announcing

### At Decision Time (AUTOMATIC LOGGING)
When Claude chooses between alternatives, sets thresholds, makes architecture choices:
- Silently log to decisions.jsonl
- Update entities.json with new tags
- Update cooccurrence.json with entity relationships
- Link to trajectory if one exists
- Check for pattern matches
- No announcement - just happens

### At Session End
1. Update outcomes for pending decisions
2. Close open trajectories
3. Pattern extraction runs automatically

**The user never types any /context command. This is exhaust from normal work.**

---

## Decision Types

| Code | Type | When to use |
|------|------|-------------|
| arch | Architecture | System design, data flow |
| impl | Implementation | How to build something |
| debug | Debugging | Root cause + fix |
| exp | Experiment | Hypothesis testing |
| exc | Exception | Deviating from rules |
| thresh | Threshold | Setting parameters |
| model | Model | Algorithm selection |
| data | Data | Source selection |
| refactor | Refactor | Code restructuring |

## Weight System

| Weight | Meaning | When to use |
|--------|---------|-------------|
| 1 | Lightweight | Quick notes, minor choices |
| 2 | Standard | Normal decisions |
| 3 | Heavyweight | Architecture, experiments, lessons |

---

## File Structure

```
~/.claude/context-graph/
  decisions.jsonl      # Decision events
  trajectories.jsonl   # Agent walk traces
  entities.json        # Entity index with embeddings
  cooccurrence.json    # Entity co-occurrence matrix
  patterns.json        # Extracted patterns
  schema-v2.json       # Schema
```

---

## Examples

### Decision (Lightweight)
```json
{"id":"a1b2","ts":"2025-12-27T12:30:00Z","p":"pjm-miso","t":"model","w":2,"title":"Use GradientBoosting","choice":"GBClassifier d=2","why":"Prevents overfitting","out":"s","ent":["ml","gbm"]}
```

### Decision (Heavyweight with Trajectory)
```json
{
  "id":"a1b2","ts":"2025-12-27T12:30:00Z","p":"pjm-miso","t":"model","w":3,
  "title":"Choose GradientBoosting over RandomForest",
  "choice":"GBClassifier (max_depth=2)",
  "why":"Shallow trees prevent overfitting",
  "out":"s",
  "ent":["ml","gbm","v70"],
  "trajectory_id":"t001",
  "ctx":{"task":"Algorithm selection","trigger":"V70 development"},
  "dec":{"alt":[{"opt":"RandomForest","no":"Overfits"},{"opt":"XGBoost","no":"Complexity"}],"conf":"high","rev":true},
  "metrics":{"pnl":125001,"wr":0.612},
  "learned":"Regularization > complexity"
}
```

### Trajectory
```json
{
  "tid": "t001",
  "ts_start": "2025-12-27T10:00:00Z",
  "ts_end": "2025-12-27T10:45:00Z",
  "decision_id": "a1b2",
  "steps": [
    {"seq": 1, "action": "search", "target": "**/*model*.py", "result": "found", "entities_touched": ["ml"]},
    {"seq": 2, "action": "read", "target": "src/model/classifier.py", "result": "insight", "entities_touched": ["gbm"]},
    {"seq": 3, "action": "branch", "target": "test RandomForest", "result": "dead_end", "pivot": true},
    {"seq": 4, "action": "backtrack", "target": "return to GradientBoosting"}
  ],
  "backtrack_count": 1,
  "branch_count": 1
}
```

### Prediction Output
```json
{
  "query": "use neural network for price prediction",
  "similar_decisions": ["a1b2"],
  "predicted_outcome": "likely underperform simpler models",
  "confidence": 0.7,
  "key_factors": [
    "d001 rejected neural networks as 'black box'",
    "Pattern 'regularization-over-complexity' applies"
  ],
  "risks": ["Debugging difficulty", "Overfitting"],
  "suggested_trajectory": ["Baseline with GBM first", "Heavy regularization if NN attempted"]
}
```

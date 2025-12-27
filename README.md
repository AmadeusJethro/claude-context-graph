# Claude Context Graph

A lightweight decision trace system for Claude Code that captures not just WHAT you decided, but WHY. Creates searchable precedent across all your projects and sessions.

## The Problem

When you're juggling multiple projects, you make hundreds of decisions per day. A week later:
- "Why did we use Redis instead of Postgres for this?"
- "What threshold did we settle on and why?"
- "Didn't we solve something like this before?"

The context dies with the session. You end up re-litigating old decisions or digging through chat history.

## The Solution

A simple append-only log of decision traces that lives in your Claude config. Each trace captures:
- What was decided
- Why that choice was made
- What alternatives were rejected
- The outcome (success/failed/pending)
- Entity tags for search

Inspired by [this article on context graphs](https://x.com/JayaGup10/status/1871218022844043660) - but scaled down for personal use with Claude Code.

## Setup

### 1. Create the storage directories

```bash
mkdir -p ~/.claude/context-graph
```

### 2. Copy the schema

```bash
cp schema.json ~/.claude/context-graph/
```

### 3. Initialize the storage files

```bash
echo '' > ~/.claude/context-graph/decisions.jsonl
echo '{"entities":{},"last_updated":"","stats":{"total_decisions":0,"total_entities":0,"by_type":{},"by_project":{},"by_outcome":{}}}' > ~/.claude/context-graph/entities.json
```

### 4. Add to your CLAUDE.md

Copy the contents of `claude-md-snippet.md` into your `~/.claude/CLAUDE.md` file. This tells Claude how and when to log decisions.

## How It Works

### Decision Format (Lightweight ~200 bytes)

```json
{"id":"a1b2","ts":"2025-12-27T12:30:00Z","p":"polymarket","t":"impl","w":2,"title":"Cache invalidation strategy","choice":"TTL-based with 5min expiry","why":"Simpler than event-driven, acceptable staleness for this use case","out":"s","ent":["cache","redis","polymarket"]}
```

### Fields

| Field | Description |
|-------|-------------|
| `id` | Short unique identifier |
| `ts` | Timestamp |
| `p` | Project name |
| `t` | Type: arch/impl/debug/exp/thresh/model |
| `w` | Weight: 1=minor, 2=normal, 3=major |
| `title` | Brief title |
| `choice` | What was decided |
| `why` | Why this choice |
| `out` | Outcome: s=success, f=failed, p=partial, ?=pending |
| `ent` | Entity tags for search |

### Decision Types

| Code | Type | When to use |
|------|------|-------------|
| `arch` | Architecture | System design, data flow decisions |
| `impl` | Implementation | How to build something |
| `debug` | Debugging | Root cause analysis + fix approach |
| `exp` | Experiment | Hypothesis testing |
| `thresh` | Threshold | Setting parameters/limits |
| `model` | Model | Algorithm/library selection |

### Heavyweight Format

For major decisions (weight 3), you can include additional context:

```json
{
  "id": "x7y8",
  "ts": "2025-12-27T14:00:00Z",
  "p": "trading-bot",
  "t": "arch",
  "w": 3,
  "title": "Event sourcing for trade log",
  "choice": "Append-only JSONL with daily rotation",
  "why": "Simpler than full event sourcing, meets audit requirements",
  "out": "s",
  "ent": ["architecture", "trading", "audit"],
  "ctx": {
    "task": "Need auditable trade history",
    "trigger": "Compliance review requirement",
    "constraints": ["Must be queryable", "Cannot lose data", "Simple to implement"]
  },
  "dec": {
    "alt": [
      {"opt": "PostgreSQL with triggers", "no": "Overkill for current scale"},
      {"opt": "Event sourcing framework", "no": "Too much complexity"}
    ],
    "conf": "high",
    "rev": true
  },
  "learned": "JSONL + jq is surprisingly powerful for most audit needs"
}
```

## Usage

### Logging a Decision

When Claude encounters a significant decision point, it logs to the graph:

```bash
echo '{"id":"abc1","ts":"2025-12-27T10:00:00Z",...}' >> ~/.claude/context-graph/decisions.jsonl
```

### Searching for Precedent

```bash
# Find all caching decisions
grep -i "cache" ~/.claude/context-graph/decisions.jsonl | jq .

# Find failed experiments
grep '"out":"f"' ~/.claude/context-graph/decisions.jsonl | jq .

# Find decisions for a specific project
grep '"p":"myproject"' ~/.claude/context-graph/decisions.jsonl | jq .
```

### When Claude Should Log

The CLAUDE.md config instructs Claude to log when:
- Choosing between alternatives (log rejected options + why)
- Setting thresholds/parameters (log rationale)
- Making exceptions to rules (log why exception was valid)
- Debugging completed (log root cause + fix approach)
- Experiment completed (log hypothesis + outcome)
- Rejecting an approach (log why it was rejected)

## File Structure

```
~/.claude/context-graph/
  decisions.jsonl    # Append-only decision log
  entities.json      # Entity index for fast lookups
  schema.json        # JSON schema for validation

[project]/.claude/context-graph/
  local.jsonl        # Project-specific decisions (optional)
```

## Contributing

This is a simple personal tool that's been helpful for me. Feel free to:
- Fork and adapt for your workflow
- Open issues with suggestions
- Submit PRs for improvements

## License

MIT

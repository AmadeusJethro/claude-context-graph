# CONTEXT GRAPH (ALWAYS ACTIVE)

Decision trace system that captures not just WHAT happened, but WHY. Creates searchable precedent across all projects and sessions.

## Storage

- Global: `~/.claude/context-graph/decisions.jsonl`
- Per-project: `[project]/.claude/context-graph/local.jsonl`
- Entity index: `~/.claude/context-graph/entities.json`

## Automatic Capture Protocol

**Before implementing non-trivial work:**
1. Check for precedent: Search context graph for similar decisions
2. If precedent exists: Reference it, note if following or deviating
3. Log the decision with: context, alternatives, reasoning

**After completing work:**
1. Update decision outcome: success/failure, metrics, lessons
2. Link related decisions if discovered

## Decision Logging Triggers

Log a decision trace when:
- Choosing between alternatives (log rejected options + why)
- Setting thresholds/parameters (log rationale)
- Making exceptions to rules (log why exception was valid)
- Debugging completed (log root cause + fix approach)
- Experiment completed (log hypothesis + outcome)
- Rejecting an approach (log why it was rejected)

## Decision Format (Lightweight ~200 bytes)

```json
{"id":"a1b2","ts":"2025-12-27T12:30:00Z","p":"project","t":"impl","w":2,"title":"Brief title","choice":"What was decided","why":"Why this choice","out":"s","ent":["tag1","tag2"]}
```

Fields: id, ts=timestamp, p=project, t=type, w=weight(1-3), out=outcome(s/f/p/x/?), ent=entities

## Decision Types

| Code | Type | When |
|------|------|------|
| arch | Architecture | System design, data flow |
| impl | Implementation | How to build |
| debug | Debugging | Root cause + fix |
| exp | Experiment | Hypothesis testing |
| thresh | Threshold | Setting parameters |
| model | Model | Algorithm selection |

## Weight System

- Weight 1: Quick notes, minor choices
- Weight 2: Normal implementation decisions
- Weight 3: Architecture, experiments, lessons learned

## Query Before Acting

When facing a significant decision, search for precedent:
- `/context search <keywords>` or grep decisions.jsonl
- Check if similar problem was solved before
- Reference prior decision if following same approach
- Note deviation if taking different path

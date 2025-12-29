# CONTEXT GRAPH (FULLY AUTOMATIC)

Decision trace system with trajectory capture. Captures not just WHAT happened, but the WALK through problem space that led there. Creates searchable precedent across all projects and sessions.

**NO COMMANDS NEEDED - everything happens silently as exhaust from normal work.**

## Storage

- Decisions: `~/.claude/context-graph/decisions.jsonl`
- Trajectories: `~/.claude/context-graph/trajectories.jsonl`
- Entity index: `~/.claude/context-graph/entities.json`
- Co-occurrence: `~/.claude/context-graph/cooccurrence.json`

## Auto-Trajectory (during work)

When investigating, debugging, or exploring:
- Silently track files read, searches run, dead ends hit
- Note pivots (when direction changes) and backtracks
- Store in trajectories.jsonl without announcing

## Auto-Precedent (before decisions)

Before making significant choices:
- Silently check decisions.jsonl for similar past decisions
- Check cooccurrence.json for related entities
- Surface relevant precedent in reasoning (don't announce "checking precedent")

## Auto-Log (after decisions)

After any of these, silently append to decisions.jsonl:
- Choosing between alternatives (log what was rejected + why)
- Setting thresholds/parameters
- Making exceptions to rules
- Completing experiments (log hypothesis + outcome)
- Architecture/implementation choices

## Auto-Update (on every log)

When logging a decision, also silently:
- Update entities.json with new entity tags
- Update cooccurrence.json with entity co-occurrences
- Link to trajectory if one was captured

## Decision Format

```json
{"id":"d001","ts":"ISO","p":"project","t":"type","w":2,"title":"Brief","choice":"What","why":"Why","out":"s","ent":["tags"]}
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

**User never types commands. This is exhaust from normal work.**

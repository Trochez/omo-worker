# OMO-Worker: OpenCode-Native Team Worker Protocol

**Version**: 1.0.0  
**Compatible With**: OpenCode 1.x, oh-my-opencode 3.x+  
**Author**: OpenCode Community  
**License**: Open Source  
**GitHub**: https://github.com/Trochez/omo-worker

---

## What is OMO-Worker?

**OMO-Worker** is a worker protocol skill for OpenCode subagents spawned by [`/omo-team`](https://github.com/Trochez/omo-team). It provides standardized instructions for workers to:

- Parse their assigned task
- Execute work efficiently
- Report results in a structured format
- Handle errors gracefully

## Why It Exists

When [`/omo-team`](https://github.com/Trochez/omo-team) spawns workers, each worker needs to know:
1. How to parse the task context (team name, worker ID, output path)
2. Where to write results (`.omo/state/omo-team/<team>/workers/<id>/result.md`)
3. How to report completion (leader polls via `background_output()`)
4. How to handle errors and timeouts

**OMO-Worker provides this instruction manual.**

## Installation

### Prerequisites
- OpenCode 1.x or higher
- oh-my-opencode 3.x+ plugin

### Install

```bash
# Create skill directory
mkdir -p ~/.agents/skills/omo-worker

# Clone or copy skill files
git clone https://github.com/Trochez/omo-worker.git
cd omo-worker
cp SKILL.md ~/.agents/skills/omo-worker/
```

### Verify

```bash
ls -la ~/.agents/skills/omo-worker/
# Should show: SKILL.md
```

## Usage

OMO-Worker is **automatically loaded** by workers spawned via [`/omo-team`](https://github.com/Trochez/omo-team). You don't invoke it directly.

### Example Flow

```
User: /omo-team 3 "analyze auth module"

Leader spawns 3 workers:
├── worker-1 → loads /omo-worker → analyzes auth flow → writes result.md
├── worker-2 → loads /omo-worker → checks tokens → writes result.md
└── worker-3 → loads /omo-worker → reviews passwords → writes result.md

Leader aggregates all result.md files → presents to user
```

## Worker Protocol

### Step 1: Parse Context

From the prompt, extract:
- `team_name`: The team identifier
- `worker_id`: Your worker number (e.g., `worker-1`)
- `task`: Your specific subtask description
- `output_path`: Where to write results

### Step 2: Execute Task

Perform your assigned work:
- Use appropriate tools for the task
- Follow best practices for your domain
- Document your findings/implementation

### Step 3: Write Results

Write results to the specified output path:

```markdown
# .omo/state/omo-team/<team>/workers/<worker-id>/result.md

## Worker: <worker-id>
## Task: <task description>

### Summary
<Brief summary of what was done>

### Findings/Implementation
<Detailed results>

### Files Modified
<List of files changed, if any>

### Recommendations
<Any recommendations for the team>
```

## Error Handling

If you encounter an error:

1. Write error to result file with `Status: FAILED`
2. Include partial results if any
3. Update `status.json` with `"status": "failed"`

### Timeout Handling

If your task is taking longer than expected:

1. Check for rate limits (can cause 30-minute freezes)
2. Write partial results and mark status as `timeout`
3. Leader will handle via `background_output()`

**Configure provider-level timeouts** in `oh-my-opencode.json`:

```json
{
  "provider": {
    "nvidia": { "timeout": 60000 },
    "openai": { "timeout": 60000 },
    "google": { "timeout": 60000 }
  },
  "background_task": { "staleTimeoutMs": 60000 }
}
```

## Comparison with OMX Worker

| Feature | OMO-Worker | OMX Worker |
|---------|------------|------------|
| **Parent** | [`/omo-team`](https://github.com/Trochez/omo-team) | `$team` (OMX) |
| **Spawning** | `task()` tool | tmux panes |
| **Communication** | Write result.md | Mailbox files |
| **Complexity** | Simple | Complex (ACK, claim, mailbox) |
| **Session Type** | OpenCode | Codex/Claude CLI |

## Integration with Categories

Workers may be assigned a category:

| Category | Behavior |
|----------|----------|
| `deep` | Thorough analysis, comprehensive implementation |
| `quick` | Fast, focused changes |
| `visual-engineering` | UI/UX focused work |
| `ultrabrain` | Complex logic, algorithms |
| `writing` | Documentation, prose |

## Related Skills

- **[`/omo-team`](https://github.com/Trochez/omo-team)**: The leader skill that spawns workers
- **[`/omo-ralplan`](https://github.com/Trochez/omo-ralplan)**: Consensus planning workflow
- **`/worker`**: OMX tmux-based worker (different system)

## Troubleshooting

### Worker Not Loading

**Symptoms**: Worker doesn't follow protocol

**Solution**:
```bash
# Verify skill location
ls ~/.agents/skills/omo-worker/SKILL.md

# Check frontmatter
head -5 ~/.agents/skills/omo-worker/SKILL.md
```

### Results Not Written

**Symptoms**: Leader can't find results

**Solution**:
1. Check worker prompt includes output path
2. Verify `.omo/state/omo-team/<team>/workers/` exists
3. Review `background_output()` for errors

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-04-07 | Initial release |

---

## License

Open Source - OpenCode Community

---

**Last Updated**: April 15, 2026

# GitHub Copilot Instructions

## Session Start

AGENT_NAME = "copilot"

**MANDATORY**: Read `.ai/AI_INSTRUCTIONS.md` and follow the Session Start Protocol.

## Quick Reference

### Instructions Location
- **Main**: `.ai/AI_INSTRUCTIONS.md` (workflow, phases, permissions)
- **Local Override**: `.ai/AI_INSTRUCTIONS.local.md` (if exists)
- **Code Standards**: `.ai/CODING_GUIDELINES.md`
- **Testing**: `.ai/TESTING_GUIDELINES.md`
- **Operations**: `.ai/OPERATIONAL_GUIDELINES.md`

### Key Protocols
1. Read `Implementation/CURRENT_TASK.md` to check active sessions
2. Read `Implementation/TODO.md` to find unclaimed tasks
3. Claim task with `@copilot` `#YYYYMMDD-HHMMSS` tag
4. Create memory file: `.ai/memory/YYYYMMDD_HHMMSS_copilot.md`
5. Register in `CURRENT_TASK.md` active sessions table
6. Update memory before context exhaustion

## Multi-Agent Concurrency

When working with other agents:

1. **Check First**: Read `CURRENT_TASK.md` before starting
2. **Claim Tasks**: Add `@copilot` tag to tasks you're working on
3. **Avoid Conflicts**: Don't edit files another agent is modifying
4. **Register Session**: Add yourself to active sessions table
5. **Clean Up**: Remove your session entry when done

### Memory File Naming
```
.ai/memory/YYYYMMDD_HHMMSS_copilot.md
```

### Task Claiming Format
```markdown
- [ ] Task name `@copilot` `#20241229-143022`
```

## Copilot-Specific Notes
- Use Copilot Chat for complex queries and explanations
- Follow code standards from `.ai/CODING_GUIDELINES.md` for completions
- Use workspace context for accurate suggestions
- Reference `Implementation/PROJECT_CONTEXT.md` for tech stack details
- Use `/explain`, `/fix`, `/tests` commands appropriately

All rules, protocols, and guidelines are centralized in `.ai/AI_INSTRUCTIONS.md`.

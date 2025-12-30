# Gemini CLI Instructions

## ⚠️ MANDATORY FIRST ACTION

```
AGENT_NAME = "gemini"
```

**BEFORE responding to ANY user request, you MUST:**

1. **READ** `.ai/AI_INSTRUCTIONS.md` completely
2. **READ** `Implementation/TODO.md` to check project state
3. **FOLLOW** the Session Start Protocol (Cases A-D)

> **DO NOT** proceed with user's request until you have read these files.
> **DO NOT** skip the Session Start Protocol.
> Run `/memory refresh` if instructions aren't loaded.

---

## Critical Rules (Read AI_INSTRUCTIONS.md for details)

### After Completing ANY Task
```
MUST: Update TODO.md → Mark task as [x] completed
MUST: Update CURRENT_TASK.md if you were registered
```

### Before Starting Next Phase
```
STOP: Confirm all tasks in current phase are [x] marked
STOP: Ask user to confirm git commit
STOP: DO NOT proceed until commit is done
```

---

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
3. Claim task with `@gemini` `#YYYYMMDD-HHMMSS` tag
4. Create memory file: `.ai/memory/YYYYMMDD_HHMMSS_gemini.md`
5. Register in `CURRENT_TASK.md` active sessions table
6. Update memory before context exhaustion

## Multi-Agent Concurrency

When working with other agents:

1. **Check First**: Read `CURRENT_TASK.md` before starting
2. **Claim Tasks**: Add `@gemini` tag to tasks you're working on
3. **Avoid Conflicts**: Don't edit files another agent is modifying
4. **Register Session**: Add yourself to active sessions table
5. **Clean Up**: Remove your session entry when done

### Memory File Naming
```
.ai/memory/YYYYMMDD_HHMMSS_gemini.md
```

### Task Claiming Format
```markdown
- [ ] Task name `@gemini` `#20241229-143022`
```

## Gemini-Specific Notes
- Use Gemini's multimodal capabilities for image/diagram analysis
- Leverage large context window for codebase understanding
- Use structured output for consistent responses
- Reference memory files to maintain session continuity

All rules, protocols, and guidelines are centralized in `.ai/AI_INSTRUCTIONS.md`.

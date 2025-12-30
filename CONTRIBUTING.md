# Contributing to DevAI

Thank you for your interest in contributing to DevAI! This document provides guidelines for contributing to this multi-AI agent development template.

## How to Contribute

### Reporting Issues

- Check existing issues before creating a new one
- Use the issue templates when available
- Provide clear, detailed descriptions
- Include steps to reproduce for bugs

### Pull Requests

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test with multiple AI agents if possible
5. Commit with conventional commits: `feat:`, `fix:`, `docs:`, etc.
6. Push and create a Pull Request

### Commit Message Format

```
type(scope): subject

body (optional)

footer (optional)
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Examples**:
```
feat(agents): add support for new AI assistant
docs(readme): update installation instructions
fix(memory): resolve session file naming issue
```

## Development Guidelines

### Adding New AI Agent Support

1. Create agent config file in root (e.g., `NEWAGENT.md`)
2. Follow existing pattern - reference `.ai/AI_INSTRUCTIONS.md`
3. Update README.md with new agent in supported tools table
4. Test with the actual AI agent if possible

### Modifying Guidelines

- `.ai/AI_INSTRUCTIONS.md` - Core workflow (be careful with changes)
- `.ai/CODING_GUIDELINES.md` - Code style standards
- `.ai/TESTING_GUIDELINES.md` - Testing standards
- `.ai/OPERATIONAL_GUIDELINES.md` - DevOps procedures

### File Permissions

Respect the file permission system:
- **Read-only**: `.ai/*.md` guidelines
- **Editable**: Agent config files, templates

## Testing Changes

When modifying the template:

1. Test with at least 2 different AI agents
2. Verify session start protocol works
3. Check memory file creation
4. Ensure TODO.md detection works correctly

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn and grow

## Questions?

Open an issue with the `question` label or start a discussion.

---

_Thank you for helping make DevAI better for developers worldwide!_

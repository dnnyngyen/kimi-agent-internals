# System Architecture: Design & High-Level Organization

Documentation of Kimi's architectural design and system-wide patterns.

## Files

### Architecture
- **[skill-system.md](skill-system.md)** - Core architecture: Skills as text-based instruction manuals vs API tools, the skill-gated paradigm
- **[skills-analysis.md](skills-analysis.md)** - Comparison of MCP (Model Context Protocol) tools vs skills, shell-operator paradigm

### System Integration
- **[tool-dependency-maps.md](tool-dependency-maps.md)** - Tool relationships, dependency graphs, how tools interact across the system

## Key Concepts

### Skill System
Kimi uses a unique skill-based model where capabilities are represented as:
- Text-based instruction manuals (SKILL.md files)
- Shell and Python integration points
- Validation and workflow management

Rather than discrete API endpoints, skills operate through the shell-operator paradigm.

### Tool Dependencies
The tool dependency map shows how:
- Different agents use different tools
- Tools depend on services
- Services connect to external systems

## Architecture Levels

```
Agent Layer (Base Chat, OK Computer, Specialized)
    ↓
Skill Layer (DOCX, PDF, XLSX, WebApp)
    ↓
Tool Layer (shell, browser, file operations)
    ↓
Service Layer (kernel, browser guard, filesystem)
```

## Related Documentation

- **How services run**: See [../runtime/](../runtime/)
- **How it's deployed**: See [../infrastructure/](../infrastructure/)
- **Skill implementations**: See [../skills/](../skills/)

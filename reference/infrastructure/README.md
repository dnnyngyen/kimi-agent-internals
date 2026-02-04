# Infrastructure: Deployment, Storage & System Organization

Documentation of container architecture, filesystem organization, and data storage.

## Files

### Container & Deployment
- **[container-infrastructure.md](container-infrastructure.md)** - Four-layer container architecture, resource allocation, networking
- **[supporting-directories.md](supporting-directories.md)** - Runtime data storage, logs, caches, configuration directories

### Filesystem & Workspaces
- **[filesystem-full.md](filesystem-full.md)** - Complete filesystem inventory across all environments, directory structure
- **[workspace-mnt-okcomputer.md](workspace-mnt-okcomputer.md)** - Working directory for Docs, Sheets, Website, Slides agents (/mnt/okcomputer/)
- **[workspace-mnt-kimi.md](workspace-mnt-kimi.md)** - User data mount point for uploads and outputs (/mnt/kimi/)

### System Dependencies
- **[tectonic.md](tectonic.md)** - LaTeX compiler binary, PDF generation engine

## Container Architecture

```
Layer 1: Base OS (Linux kernel)
Layer 2: Runtime (Python, Node, browsers)
Layer 3: Services (Jupyter, FastAPI, Chrome)
Layer 4: Application (Kimi agent environment)
```

## Key Workspaces

| Path | Purpose | Agents |
|------|---------|--------|
| /mnt/okcomputer | Working directory | Docs, Sheets, Slides, Websites |
| /mnt/kimi | User data | All agents |
| /app/.kimi/skills | Skill definitions | All specialized agents |

## Directory Organization

- **Runtime data**: Supporting directories for logs, caches, temp files
- **Shared storage**: Persistent data for agent sessions
- **User uploads**: File storage for user-provided content
- **Generated outputs**: Artifact storage from agent execution

## Related Documentation

- **Core services**: See [../runtime/](../runtime/)
- **System design**: See [../system-architecture/](../system-architecture/)
- **Security**: See [../security/](../security/)

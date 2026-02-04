# Runtime: Core Services & Code Execution

Documentation of the Python modules and services that execute code and manage the Kimi environment.

## Files

### Core Services
- **[jupyter-kernel.md](jupyter-kernel.md)** - Jupyter kernel implementation, Python code execution interface, kernel lifecycle management
- **[kernel-server.md](kernel-server.md)** - FastAPI server on port 8888, kernel lifecycle control, process management
- **[utils.md](utils.md)** - Utility functions for display detection and subprocess execution

### Browser Automation
- **[browser-guard.md](browser-guard.md)** - Playwright + Chrome DevTools Protocol, browser automation system (41KB analysis)
- **[chrome-data.md](chrome-data.md)** - Chromium profile persistence, browser state management, cache and cookies

### Extensions & Binaries
- **[pdf-viewer.md](pdf-viewer.md)** - Chrome extension for in-browser PDF rendering (387 files)

## Architecture Overview

```
Kernel Server (Port 8888)
    ↓
Jupyter Kernel (Python execution)
    ↓
Browser Guard (Browser automation)
    ↓
Chrome Instance + PDF Viewer
```

## Key Components

| Component | Purpose | File |
|-----------|---------|------|
| Kernel Server | HTTP control plane | kernel-server.md |
| Jupyter Kernel | Code execution | jupyter-kernel.md |
| Browser Guard | Browser control | browser-guard.md |
| Chrome Data | Browser state | chrome-data.md |
| Utilities | Helpers | utils.md |

## Related Documentation

- **System design**: See [../system-architecture/](../system-architecture/)
- **Infrastructure setup**: See [../infrastructure/](../infrastructure/)
- **Skills using these services**: See [../skills/](../skills/)

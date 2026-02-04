# Deep Dives

Technical reference documentation for the Kimi K2.5 runtime environment.

These documents contain detailed implementation analysis extracted from the container filesystem and source code.

---

## Architecture

**[container.md](architecture/container.md)** - Four-layer container architecture: control plane, compute engine, web tools, and user workspace. Includes service ports, binary analysis, and security model.

**[filesystem.md](architecture/filesystem.md)** - Complete filesystem inventory. Directory structures, file counts, permission zones, and mount points.

**[security.md](architecture/security.md)** - Security model analysis. Sandbox boundaries, network isolation, authentication gaps, and risk assessment.

**[workspaces.md](architecture/workspaces.md)** - Mount point analysis. /mnt/kimi/ versus /mnt/okcomputer/, persistence models, and access controls.

---

## Runtime

**[browser-automation.md](runtime/browser-automation.md)** - browser_guard.py analysis. Playwright and Chrome DevTools Protocol implementation, stealth mode, anti-detection measures.

**[code-execution.md](runtime/code-execution.md)** - jupyter_kernel.py analysis. IPython kernel, ZeroMQ messaging, execution budgets, resource limits.

**[control-plane.md](runtime/control-plane.md)** - kernel_server.py analysis. FastAPI endpoints, kernel lifecycle management, health checks.

**[chrome-profile.md](runtime/chrome-profile.md)** - Chrome data directory analysis. Cookie storage, browsing history, extension state, security implications.

**[pdf-viewer.md](runtime/pdf-viewer.md)** - PDF.js extension analysis. Chrome extension structure, permissions, CJK font support.

**[utils.md](runtime/utils.md)** - Utility functions. Shared helper code used across modules.

---

## Binaries

**[tectonic.md](binaries/tectonic.md)** - LaTeX compiler analysis. 57MB binary, XeTeX engine, automatic package management.

---

## Reading Guide

Start with [architecture/container.md](architecture/container.md) for the overall system picture, then explore specific runtime components based on your interest.

For the skill system and agent behavior, see [../skills/README.md](../skills/README.md) and [../analysis/how-kimi-works.md](../analysis/how-kimi-works.md).

# Root Directory Core Components Analysis
## `/app/` — Agent Execution Environment

---

## Executive Summary

The `/app/` directory constitutes the runtime environment for the Kimi AI agent system. It contains the foundational Python infrastructure, browser automation subsystem, Jupyter kernel management, and skill-based domain extensions. This analysis examines the five core files at the root level that form the backbone of the agent's operational capabilities.

---

## 1. `browser_guard.py` (41,635 bytes)

### Functional Architecture

The `browser_guard.py` module implements a dual-strategy browser lifecycle management system, providing two distinct implementations for Chromium automation:

#### 1.1 `BrowserGuard` Class (Playwright-based)

**Purpose**: Primary browser automation using Microsoft's Playwright framework with persistent context management.

**Key Design Patterns**:
- **Singleton Browser Context**: Uses `launch_persistent_context()` with `/app/data/chrome_data` as user data directory, enabling profile persistence across sessions
- **Environment-aware Identity Construction**: Dynamically builds user agent strings, locale settings, and Client Hints headers based on environment variables (`CHROME_LOCALE`, `TZ`, `CHROME_FLAGS`)
- **Stealth Evasion Injection**: Embeds JavaScript stealth scripts to evade bot detection mechanisms
- **X11 Display Integration**: Coordinates with `pyautogui` for GUI interactions

**Critical Configuration Parameters**:
```python
# Viewport and resolution
SCREEN_RESOLUTION = os.getenv("SCREEN_RESOLUTION", "1920x1080")
viewport = {"width": self.width, "height": self.height}

# Chromium launch flags (security and compatibility)
args = [
    "--disable-blink-features=AutomationControlled",  # Anti-detection
    "--disable-infobars",
    "--no-sandbox",  # Required for containerized environments
    "--disable-gpu",  # Software rendering
    "--load-extension=/app/pdf-viewer",  # PDF viewer extension
]
```

**Stealth JavaScript Payload**:
The module contains an embedded `stealth_js` string (lines 55-439) that implements Bitcoin stealth address cryptography. This appears to be a legacy artifact or placeholder for advanced cryptographic operations, though its current functional role is unclear.

#### 1.2 `BrowserCDPGuard` Class (CDP-based)

**Purpose**: Alternative browser control via Chrome DevTools Protocol for direct WebSocket communication.

**Architecture**:
- **Process Management**: Direct subprocess.Popen() invocation of Chromium binary
- **CDP REST API**: HTTP endpoints at `http://localhost:9222/json/` for tab management
- **WebSocket Communication**: Bi-directional message passing for runtime control
- **Window State Management**: Maximization/minimization detection and correction

**CDP Command Protocol**:
```python
# Window bounds manipulation via CDP
await self._send_cdp_command(
    ws,
    "Browser.setWindowBounds",
    {"windowId": window_id, "bounds": {"windowState": "maximized"}},
    timeout=3.0,
)
```

**Monitoring Loop Semantics**:
- Polls tab state every `check_interval` seconds (default: 1.0)
- Auto-recreates browser process on failure
- Maintains WebSocket connection pool for active tabs

#### 1.3 `wait_for_display()` Function

**Purpose**: X11 display server readiness polling with timeout.

**Implementation**: Uses `python-xlib` to attempt display connection, retrying with exponential backoff until timeout (default: 60 seconds).

---

## 2. `jupyter_kernel.py` (17,982 bytes)

### Kernel Management Architecture

The `jupyter_kernel.py` module implements a robust Jupyter kernel lifecycle manager with automatic recovery capabilities.

#### 2.1 `JupyterKernel` Class

**Initialization Sequence**:
1. **KernelManager Creation**: Spawns new IPython kernel process
2. **Connection File Resolution**: Extracts ZeroMQ endpoint configuration
3. **KernelClient Initialization**: Establishes communication channels (shell, iopub, stdin, control, hb)
4. **Readiness Polling**: Waits for kernel heartbeat with 30-second timeout
5. **Matplotlib Initialization**: Configures inline backend with CJK font support

**Matplotlib CJK Configuration**:
```python
# Font discovery and configuration
fm.fontManager.__init__()  # Force reinitialization
cjk_fonts = [f.name for f in fm.fontManager.ttflist if 'CJK' in f.name]
preferred_fonts = ['Noto Sans CJK SC', 'Noto Sans CJK TC', 'Noto Sans CJK JP']
```

#### 2.2 Execution Model (`execute()` method)

**Message Flow**:
1. Submit code via `kc.execute(code)` → receive `msg_id`
2. Poll IOPub channel for output messages
3. Handle message types:
   - `stream`: stdout/stderr text output
   - `error`: Exception traceback
   - `execute_result`: Plain text or image/png data
   - `display_data`: Rich output (images, HTML)
   - `status`: Execution state transitions

**Timeout Handling**:
- Default timeout: 30 seconds
- Returns `ExecutionResult` with error classification on timeout

#### 2.3 Kernel Health Monitoring

**`_ensure_kernel_alive()` Method**:
- Checks `km.is_alive()` for process status
- Validates responsiveness via `kc.kernel_info()`
- Auto-restarts on failure using `_start_kernel()`

#### 2.4 Process Identification

**`_get_kernel_pid()` Method**:
Implements multi-strategy PID resolution:
1. **Provisioner API** (jupyter-client ≥7.0): `km.provisioner.process.pid`
2. **Legacy Kernel Property**: `km.kernel.pid`
3. **Process Table Scanning**: Match kernel_id in command lines via `psutil`
4. **Connection File Matching**: Match connection file basename in processes

---

## 3. `kernel_server.py` (10,030 bytes)

### FastAPI REST Interface

The `kernel_server.py` module exposes kernel management via HTTP endpoints using FastAPI.

#### 3.1 Application Lifecycle

**Lifespan Context Manager**:
```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    kernel_instance = JupyterKernel()  # Initialize on startup
    yield
    kernel_instance.shutdown()  # Cleanup on shutdown
```

#### 3.2 API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/` | GET | Service information |
| `/health` | GET | Health check with kernel status |
| `/kernel/reset` | POST | Kernel restart with connection file tracking |
| `/kernel/interrupt` | POST | SIGINT signal delivery to kernel process |
| `/kernel/connection` | GET | Connection file path and contents |
| `/kernel/status` | GET | Comprehensive kernel state |
| `/kernel/connection-file` | GET | Simplified connection file path |
| `/kernel/debug` | GET | Internal state inspection |

#### 3.3 Response Models

**Pydantic Schema Validation**:
- `ApiResponse`: Generic success/failure wrapper
- `KernelStatusResponse`: Alive status, PID, connection file, client connectivity
- `ConnectionInfoResponse`: Full connection configuration

---

## 4. `utils.py` (1,252 bytes)

### Utility Functions

#### 4.1 `get_screensize()`

**Purpose**: X11 display resolution detection.

**Implementation**: Shells out to `xrandr` and parses output with regex:
```bash
xrandr | grep -oP '(?<=current )\d+ x \d+' | tr -d ' '
```

Returns tuple `(width, height)` for display geometry.

#### 4.2 `run_command()`

**Purpose**: Synchronous subprocess execution with timeout.

**Parameters**:
- `command`: List of command arguments
- `timeout`: Maximum execution time (default: 30s)
- `pipe_output`: Capture stdout/stderr (default: True)

#### 4.3 `run_command_background()`

**Purpose**: Asynchronous process spawning for background tasks.

---

## 5. `tectonic` (57,402,608 bytes — Binary)

### LaTeX Compilation Engine

**Identity**: Pre-compiled Tectonic binary — a modern, self-contained LaTeX compiler.

**Capabilities**:
- XeTeX/LuaTeX engine compatibility
- Automatic package downloading (no TeX Live installation required)
- Single-pass compilation with cross-reference resolution

**Integration**: Used by `/app/.kimi/skills/pdf/scripts/compile_latex.py` for LaTeX route PDF generation.

---

## Inter-Component Relationships

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  kernel_server  │────▶│  jupyter_kernel │────▶│   KernelManager │
│   (FastAPI)     │     │   (Lifecycle)   │     │  (IPython ZMQ)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                                              │
         │                                              ▼
         │                                       ┌──────────────┐
         │                                       │  Matplotlib  │
         │                                       │  (CJK fonts) │
         │                                       └──────────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│   Health Check  │◀────│   Execution     │
│   Endpoints     │     │   Results       │
└─────────────────┘     └─────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  browser_guard  │────▶│   Playwright    │────▶│    Chromium     │
│ (BrowserGuard/  │     │   (BrowserCtx)  │     │   (CDP/WS)      │
│  BrowserCDPGuard)│     └─────────────────┘     └─────────────────┘
└─────────────────┘              │                        │
         │                       ▼                        ▼
         │                ┌──────────────┐         ┌──────────────┐
         │                │   Stealth    │         │  PDF Viewer  │
         │                │   Scripts    │         │  Extension   │
         │                └──────────────┘         └──────────────┘
         ▼
┌─────────────────┐
│   pyautogui     │
│  (X11/GUI ops)  │
└─────────────────┘
```

---

## Security Considerations

### Browser Flags
- `--no-sandbox`: Required for containerized execution but reduces security isolation
- `--disable-web-security`: Not present (good)
- `--allow-file-access-from-files`: Present in CDP mode (dangerous flag, noted in comments)

### Kernel Execution
- No input sanitization on code execution (by design — full Python access)
- Timeout mechanism prevents infinite loops
- SIGINT delivery for interruption

---

## Operational Semantics

### Startup Sequence
1. X11 display server initialization (`:99` or `DISPLAY` env)
2. `browser_guard` waits for display readiness
3. Chromium launch with Playwright or CDP
4. Jupyter kernel initialization
5. (Optional) Kernel server HTTP endpoint binding

### Failure Recovery
- Browser: Automatic restart on tab closure or process death
- Kernel: Auto-restart on non-responsiveness detection
- Both: Exponential backoff with retry limits

---

*Document Version: 1.0*
*Analysis Date: 2026-01-31*

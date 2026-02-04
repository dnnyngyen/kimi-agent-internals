# Glossary

Definitions for terms used throughout this repository.

## Agent Architecture Terms

### Tool-Use Architecture
Traditional AI system design where models are given discrete function APIs to call. Stateless, request-response pattern with limited context for complex multi-step tasks.

### Environment Architecture
Kimi K2.5's approach: providing models with general-purpose computing contexts including filesystems, browsers, and process execution. Enables persistent state across turns and complex workflows.

### Base Prompt
The foundational system prompt that defines an agent's identity, capabilities, constraints, and behavioral patterns. All other instructions build on or replace this base.

### Skill Scaffolding
Pattern where the OK Computer base prompt is prepended with technical documentation (SKILL.md files) that teaches the agent how to generate specific artifacts. Used by Docs, Sheets, and Websites agents.

### Persona Replacement
Pattern where the entire base prompt is replaced with an expert persona definition rather than appending technical documentation. Used by the Slides agent (McKinsey consultant persona).

### Skill Injection
Runtime process where the system detects user intent (e.g., "create a Word document") and dynamically loads the relevant SKILL.md content into the context window before execution begins.

### Budget (Step Budget)
Maximum number of tool calls/turns an agent can execute before being terminated. Base Chat: 10 steps. OK Computer and specialized agents: 200-300 steps.

## Tool Naming

### mshtools-
Prefix used for OK Computer tools (e.g., `mshtools-web_search`, `mshtools-shell`). Distinguishes agent-facing tool names from internal implementations. The "msh" likely stands for Moonshot.

### Base Chat Tools
The 9 tools available to the Base Chat agent: web_search, web_open_url, ipython, shell, memory_space_edits, search_image_by_text, search_image_by_image, get_data_source, get_data_source_desc.

### OK Computer Tools
The 28+ tools available to OK Computer and specialized agents. Includes browser automation, file operations, image generation, voice/speech, and deployment tools.

## Skill Terms

### SKILL.md
Primary skill definition file containing templates, workflows, validation rules, and implementation guidance for a specific output format (DOCX, XLSX, PDF, WebApp).

### Template
Pre-built starting point for common tasks within a skill. Defines file structure, required components, and boilerplate code.

### Validator
Quality assurance component that checks generated output against requirements. Examples: DOCX validator uses .NET OpenXML SDK, XLSX validator uses 77MB KimiXlsx binary.

### Route (PDF Routes)
Alternative generation paths for the PDF skill: HTML route (wkhtmltopdf), LaTeX route (Tectonic), or Process route (headless browser).

### Skill-Gated Shell-Operator
Pattern where shell access is restricted: only specific pre-validated scripts can be executed, with no arbitrary command access. Contrasts with MCP (Model Context Protocol) approaches.

## Infrastructure Terms

### /mnt/kimi/
Base Chat's workspace directory. Contains `upload/` (read-only user uploads) and `output/` (generated artifacts).

### /mnt/okcomputer/
OK Computer's persistent workspace. Full read-write access across turns. Contains `user-data/`, `uploaded-files/`, and skill-specific subdirectories.

### chrome_data/
Browser profile directory containing 272 files across 15 subdirectories. Enables stateful browser sessions with cookies, localStorage, and extension data persisting across turns.

### Browser Guard
Python service (`browser_guard.py`) managing Chrome automation. Provides screenshot capabilities, element interaction, and security isolation for web browsing.

### Jupyter Kernel
Python execution environment (`jupyter_kernel.py`) providing sandboxed code execution with matplotlib support, 30-minute timeout, and 500MB memory limit.

### Kernel Server
Control plane service (`kernel_server.py`) on port 8888. Manages browser guard lifecycle, health checks, and skill script coordination.

### Tectonic
57MB LaTeX compilation engine used by the PDF skill for high-quality document generation via the LaTeX route.

### KimiXlsx
77MB proprietary binary used by the XLSX skill for Excel validation and processing.

## File Format Terms

### DOCX
Microsoft Word document format. Kimi uses C# with OpenXML SDK for generation, .NET 6.0 runtime for validation.

### XLSX
Microsoft Excel workbook format. Uses 77MB KimiXlsx binary for validation, supports pivot tables via dedicated extension.

### PDF
Portable Document Format. Three generation routes available: HTML (wkhtmltopdf), LaTeX (Tectonic), or Process (headless Chrome).

### WebApp
React-based web application using TypeScript and shadcn/ui component library. Deployed to dynamic subdomains.

## Security Terms

### Sandbox
Isolated execution environment with restricted filesystem access, network limitations, and resource caps (timeout, memory).

### NO_PERSISTENCE
Flag in Base Chat indicating no state persists across turns. Each turn is independent.

### USER_PERSISTENCE
Flag in OK Computer indicating filesystem and browser state persists across turns within a conversation.

### Intent Classification
System process that analyzes user requests to determine which agent mode or skill to activate.

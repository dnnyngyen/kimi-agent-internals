# The Skill System

Skills are detailed instruction manuals that transform Kimi's generic tools into domain-specific experts. When you ask for a spreadsheet, the agent reads a 925-line manual on Excel compatibility before it does anything else.

This directory contains those manuals. Each one is a comprehensive technical specification that teaches the agent everything it needs to know about a specific output format.

---

## Table of Contents

- [File Structure](#file-structure)
- [What Skills Actually Are](#what-skills-actually-are)
- [How Skill Loading Works](#how-skill-loading-works)
- [Dynamic Context Loading](#dynamic-context-loading)
- [The Four Skills](#the-four-skills)
- [Knowledge versus Identity](#knowledge-versus-identity)
- [The Slides Exception](#the-slides-exception)
- [Comparison: Specialized Tools versus Skill-Gated Shell](#comparison-specialized-tools-versus-skill-gated-shell)
- [Reading Guide](#reading-guide)
- [Related Documentation](#related-documentation)

---

## File Structure

```
skills/
├── README.md           # This file
├── docx/               # Word document generation
│   ├── SKILL.md
│   └── analysis.md
├── xlsx/               # Excel spreadsheet generation
│   ├── SKILL.md
│   ├── analysis.md
│   └── pivot-table.md
├── pdf/                # PDF document generation
│   ├── SKILL.md
│   ├── analysis.md
│   └── routes/
│       ├── html.md
│       ├── latex.md
│       └── process.md
└── webapp/             # React web applications
    ├── SKILL.md
    └── analysis.md
```

---

## What Skills Actually Are

Traditional AI systems use many specific tools. A `create_docx()` function. A `validate_excel()` endpoint. A `compile_latex()` service. Kimi doesn't work this way.

Kimi uses a few generic tools. Shell commands. File reading. Python execution. These tools become specialists when guided by skill documentation.

The same `mshtools-shell` command becomes four different specialists depending on which skill is loaded. In a Word context, it runs `dotnet run` with the OpenXML SDK. In an Excel context, it runs the 77MB KimiXlsx validator. In a PDF context, it invokes Tectonic or Playwright. In a WebApp context, it runs `npm run build`.

The tool doesn't change. The knowledge changes.

---

## How Skill Loading Works

When you ask for a spreadsheet, the system detects this intent and inserts an instruction: read the xlsx skill file before doing anything else. The agent executes `read_file("/app/.kimi/skills/xlsx/SKILL.md")`, loads 925 lines of Excel-specific guidance into its context, and proceeds with the task.

This is just-in-time loading. The agent doesn't carry all skills at once. It loads what it needs when it needs it.

The context window works like a stack. At the top is the SKILL.md with domain expertise. Below that is the base OK Computer prompt with identity. Below that is the conversation history and current user message. The expertise is temporary and task-specific.

---

## Dynamic Context Loading

Skills aren't static after loading. During execution, the agent loads and unloads additional context based on what it encounters.

If compilation fails, the agent might read troubleshooting documentation. If OpenXML validation fails, it loads guidance on element ordering or XML tolerance settings. In the PDF skill, once the agent chooses a route (HTML versus LaTeX), it loads only the relevant sub-instructions. Loading both routes would waste context tokens on information that won't be used.

This resembles how people work. You don't memorize every troubleshooting guide before starting a task. You consult them when you hit a problem. The context flows toward where it's needed.

---

## The Four Skills

**DOCX** handles Word document generation using C# with the OpenXML SDK rather than Python libraries like python-docx. The skill includes templates for different document types, validation rules for document structure, and instructions for running the .NET validator. It uses a dual-stack architecture: C# for creation, Python for editing.

**XLSX** handles Excel spreadsheet generation using Python plus a 77MB binary called KimiXlsx that validates spreadsheet files. The documentation covers formula compatibility (avoid FILTER and XLOOKUP for older Excel versions), pivot table creation, and styling requirements. It enforces a strict per-sheet validation workflow.

**PDF** provides three different routes depending on context. Business documents use HTML rendered through Playwright with Paged.js. Academic papers use LaTeX compiled with Tectonic. Existing PDF manipulation uses pikepdf. The skill file explains when to use which route.

**WebApp** covers React web applications with Vite, TypeScript, Tailwind, and shadcn/ui. The skill documents 50 pre-installed UI components and explains the development workflow from scaffolding to deployment.

---

## Knowledge versus Identity

Kimi's architecture distinguishes between who the agent is and what the agent can do. These are handled by different parts of the system and change at different rates.

The base prompt defines the agent's core character: personality, communication style, capabilities, behavioral boundaries. This layer stays constant. Whether the task is a spreadsheet or a React app, you're talking to the same agent. The voice doesn't change. The values don't shift.

Skills define domain capabilities that sit on top of the base identity. Knowledge is modular and transient. Load it when needed, discard it when done. The agent gains expertise for the current task without permanently increasing context size or adding constraints to unrelated tasks.

This separation has practical benefits. Users interact with a stable identity regardless of task. Safety guidelines live in the identity layer, and skills can't override them. New skills can be added without touching the base prompt. When something goes wrong, you can ask whether this is an identity issue or a knowledge issue.

---

## The Slides Exception

Kimi Slides doesn't follow the skill pattern. Instead of skill injection, it uses persona replacement: "You are a world-class presentation designer with 20 years of experience at McKinsey."

Why? Within Kimi's system, spreadsheets have objective correctness criteria. Formulas work or they don't, and the rules can be written down. Presentation design requires subjective judgment about what makes a slide compelling. Some capabilities resist procedural specification.

See the full analysis in [`analysis/skills-vs-personas.md`](../skills-vs-personas.md).

---

## Comparison: Specialized Tools versus Skill-Gated Shell

Kimi's approach differs from systems that expose many domain-specific tools.

**Specialized tool approach**: A system might expose `create_docx()`, `validate_excel()`, and `compile_latex()` as separate functions. Each tool is purpose-built, tested, and documented. The system prompt lists all available tools. Adding a new output format requires implementing and deploying new backend code.

**Kimi's skill-gated shell approach**: The system exposes generic tools (`shell`, `ipython`, `read_file`). Domain expertise lives in documentation files that the agent reads at runtime. Adding Excel support meant writing the XLSX skill documentation, not deploying a new API endpoint.

The trade-off is explicit versus implicit capability. Specialized tools fail clearly (the function doesn't exist). Skill-gated shells fail ambiguously (the agent might not read the skill file or might ignore its instructions). Kimi mitigates this by forcing skill reads for certain agent types.

Both approaches separate identity from capabilities. The difference is where the capability definition lives: in backend code versus in documentation loaded at runtime.

---

## Reading Guide

New to skills? Start with `docx/SKILL.md`. It's a comprehensive example of what a production skill definition looks like. Then look at `xlsx/SKILL.md` to see how validation rules are documented. Then `pdf/SKILL.md` to understand route selection.

Want to understand the architecture? Read [`analysis/how-kimi-works.md`](../how-kimi-works.md) for the full picture, then [`analysis/skills-vs-personas.md`](../skills-vs-personas.md) to understand why Slides is different.

Need definitions? Check [`GLOSSARY.md`](../../GLOSSARY.md) in the repository root.

---

## Related Documentation

- [`../how-kimi-works.md`](../how-kimi-works.md) - Architectural overview
- [`../skills-vs-personas.md`](../skills-vs-personas.md) - Why Slides is different
- [`../../prompts/README.md`](../../prompts/README.md) - Agent types and prompt structure
- [`../../GLOSSARY.md`](../../GLOSSARY.md) - Terminology definitions

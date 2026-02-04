# How Kimi Works

Kimi K2.5 isn't one agent. It's eight different operating modes that share infrastructure but behave differently depending on what you're trying to do.

I spent time mapping these modes by reading the system prompts, tracing the container filesystem, and watching how the same model behaves under different configurations. What I found is a taxonomy that separates connectivity from cognition: what the agent can touch versus what it knows.

---

## The eight agent types

Kimi operates across eight distinct modes. Four are for basic chat. Four are for agentic work. Two more are coming but not yet in this analysis.

**Base chat modes** (kimi.com/chat):

Kimi K2.5 Instant is the default. Fast responses, no thinking tokens. Kimi K2.5 Thinking shows its reasoning step by step. The K2 variants exist but are deprecated.

**Agentic modes** (kimi.com/agent and specialized endpoints):

OK Computer is the generalist. It has a persistent filesystem, 200 to 300 tool calls per session, and runtime skill injection. Docs, Sheets, and Websites are OK Computer with mandatory skill reading. Docs forces reading of docx or pdf skills. Sheets mandates xlsx skill. Websites requires webapp skill. Slides is different. It replaces the entire base prompt with a McKinsey consultant persona.

**Research modes** (not yet analyzed):

K1-Researcher uses control flow with human-in-the-loop. Agent Swarm orchestrates approximately 100 agents across 1,500 steps using PARL training.

---

## What I mean by skill injection

When you ask Kimi for a spreadsheet, something happens before it starts working. The system forces the agent to read a file at `/app/.kimi/skills/xlsx/SKILL.md`. This file is 925 lines of detailed instructions. It covers Excel 365 versus 2019 compatibility, formula validation, styling conventions, and output verification.

The skill file teaches things a generic agent wouldn't know. FILTER, XLOOKUP, and LAMBDA work in Excel 365 but crash in Excel 2019. The skill file tells the agent to use INDEX/MATCH instead. It specifies that gridlines should always be hidden. It explains the exact validation commands to run before delivering a file.

After reading this file, the same shell tool that could do anything becomes focused. It knows what to do. The generic tool becomes a specialist.

The shell doesn't change. The knowledge changes.

---

## Base Chat versus OK Computer

The same Kimi model powers two different experiences. Base Chat at kimi.com/chat is conversational. Ten tool calls per turn, no skill loading, no persistent filesystem. It's a question-answering system. OK Computer at kimi.com/agent is agentic with runtime skill injection. It produces deliverables.

Both modes run on identical infrastructure. Kernel server manages the control plane. Jupyter kernel runs the IPython environment. Browser guard handles Playwright automation. The difference is configuration: tool budgets, skill loading, and workspace paths.

Base Chat can technically do anything OK Computer can do, if you prompt it carefully enough. It has the same raw tools. But without skill injection, it doesn't know the compatibility gotchas, the validation steps, the styling conventions. You'd have to provide all that context yourself in every conversation.

OK Computer is Base Chat with context pre-loaded. The skill files encode expertise that would otherwise require extensive prompting. The higher tool budget allows complex multi-step workflows. The persistent workspace lets it build things incrementally.

The capability difference isn't about model intelligence. It's about scaffolding. The same model, given the right documentation and enough tool calls, becomes dramatically more capable. The implication is clear: infrastructure choices (tool budgets, filesystem access, skill loading) produce capability differences that look like model differences to users.

---

## Two patterns for specialization

Within the agentic modes, Kimi uses two different strategies for making the model specialized.

**Skill scaffolding** applies to Docs, Sheets, and Websites. These agents are essentially OK Computer with mandatory pre-loaded context. They share the same identity and communication guidelines. The difference lies in their skill reading instructions.

Kimi Sheets explicitly mandates: "Must always read the xlsx skill by default." It forces the agent to become an Excel expert immediately, skipping the intent classification step OK Computer would usually perform. Kimi Docs includes mandatory formatting rules that force reading of docx or pdf skills. Kimi Websites enforces: "Before starting ANY frontend-related project, you MUST first read the webapp-building skill documentation."

These agents don't change who Kimi is. They just change what Kimi is focused on. They are pre-primed generalists.

**Persona replacement** applies to Slides. Instead of adding skills to the OK Computer base, it replaces the identity entirely. "You are a presentation designer who has worked at McKinsey for 20 years, specializing in creating high-information-density, content-rich, and in-depth presentation slides for the world's TOP 10 enterprises."

This is a fundamentally different approach. The McKinsey consultant persona embodies a style, a voice, and a set of values that are difficult to encode in procedural documentation. Technical skill is taught via SKILL.md. Creative taste is taught via persona.

Why presentations but not spreadsheets? One theory: spreadsheet creation has right answers. Formulas work or they don't. Compatibility rules are objective. You can teach these through documentation. Presentation design requires taste. What makes a slide compelling isn't fully specifiable. You can't write a SKILL.md that explains how to create an emotionally resonant visual argument. But you can ask the model to embody someone who would know.

Within Kimi's architecture, technical tasks use skill injection while creative tasks use persona replacement. This division reflects the nature of the work: spreadsheets have objective correctness criteria that can be documented, while presentations require subjective judgment that resists procedural specification.

---

## The infrastructure stack

Kimi's container has a four-layer architecture that everything runs on.

The first layer is the control plane. A FastAPI server on port 8888 manages kernel lifecycle: starting, stopping, and resetting the Python environment. This is how the outer system controls the agent's execution context.

The second layer is the compute engine. An IPython kernel handles code execution, with ZeroMQ providing inter-process communication. The kernel has a 10-step budget per turn for Base Chat, 200 to 300 for OK Computer. This is enough for complex tasks but still maintains guardrails.

The third layer handles web tools. Playwright drives a Chrome instance for web interactions. The browser runs in stealth mode with anti-detection flags, allowing the agent to interact with web pages that might block obvious automation.

The fourth layer is the user workspace. A mounted filesystem provides persistent storage. For Base Chat, this is `/mnt/kimi/` with read-only access and no persistence across turns. For OK Computer, it's `/mnt/okcomputer/` with full read-write access and persistent state.

External network access is blocked at the container level. The agent cannot curl google.com directly. Web access goes exclusively through browser tools.

---

## From system prompts to modular skills

Skill injection represents a shift in system prompt architecture. I trace this evolution through three phases:

**Phase 1: Static prompts (circa 2024)**. Early agents used simple, fixed system prompts. Capabilities were baked into model weights through fine-tuning. Changing behavior required retraining.

**Phase 2: Instruction-heavy prompts (circa 2025)**. Long, detailed system prompts defined behavior and boundaries. Anthropic's "Constitutional" approach demonstrated that instruction could override training for behavior control without retraining.

**Phase 3: Modular skill injection (circa 2026)**. Instead of one massive prompt, capabilities are broken into modular files loaded on demand. Kimi's SKILL.md system embodies this: the agent reads domain-specific documentation when needed, keeping the base prompt minimal while accessing deep expertise on demand.

This shift allows Kimi to maintain a clean base state while accessing deep expertise on demand. It solves the "Lost in the Middle" problem of massive context windows. Knowledge is separated from identity.

---

## What this architecture suggests

Kimi demonstrates that you can separate two concerns that usually get conflated. Connectivity is what the agent can touch: tools, filesystem access, execution environment. Cognition is what the agent knows: skills, context, expertise. Connectivity is fixed infrastructure. Cognition is dynamic, loaded at runtime.

New capabilities become a documentation problem. Write a thorough manual, put it in the right directory, and the agent becomes an expert in that domain.

This approach has limits. 900-line instruction manuals push the boundaries of what models can reliably follow. As skills grow more intricate, verification becomes harder. The model generates logic rather than calling verified backend code, so errors propagate silently until output validation catches them.

For now, it works. Documents, spreadsheets, PDFs, and web applications all generate successfully with this architecture.

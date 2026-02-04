# Kimi K2.5 System Analysis

This is a reverse-engineered analysis of Moonshot AI's Kimi K2.5 agent architecture. I extracted this by using the agent's own tools to read its system prompts, explore its container filesystem, and observe its behavior. No authentication was bypassed. No binaries were decompiled. Everything here was visible to a regular user with patience and curiosity.

The central finding: Kimi has shifted from tool-use architectures to environment architectures. Instead of giving the model discrete APIs, it gives the model general-purpose computing contexts. Persistent filesystems. Browser automation. Process execution. The model becomes an operating system user rather than an API consumer.

---

## What this repository contains

System prompts for six agent types: Base Chat, OK Computer, Docs, Sheets, Slides, and Websites. Skill definitions for DOCX, XLSX, PDF, and WebApp output formats. Tool schemas documenting 37 distinct tools. Source code for the runtime environment. Technical analysis of the architecture, security model, and design patterns.

---

## How to navigate this

Start with `analysis/how-kimi-works.md` for the architectural overview. It explains the eight agent types, the skill system, and why Slides uses a different pattern than the other specialized agents.

Read `GLOSSARY.md` for definitions of terms like skill injection, persona replacement, and scaffolding.

Compare `prompts/base-chat.md` and `prompts/ok-computer.md` to see how the same model becomes dramatically more capable through infrastructure changes.

Look at `skills/docx/SKILL.md` for an example of a production skill definition. This is what the agent reads before generating a Word document.

For deep technical details, see `deep-dives/`. The container architecture, filesystem inventory, browser automation framework, and security model are documented there.

---

## The key insight

Kimi demonstrates that you can separate connectivity from cognition. Connectivity is what the agent can touch: tools, filesystem, browser, execution environment. Cognition is what the agent knows: skills, context, expertise. Connectivity is fixed infrastructure. Cognition is dynamic, loaded at runtime.

The same model with a persistent filesystem and 300 tool calls produces deliverables. The same model in a chat window with 10 calls answers questions. The model did not change. What it could touch changed.

---

## Methodology

Cleanroom extraction through the agent's own tools. I asked the agent to read its own prompt files, list directory contents, and show me its environment. Everything documented here was accessible through standard user interfaces. Independent research, not affiliated with Moonshot AI.

See `METHODOLOGY.md` for details on how this analysis was conducted.

---

## License

CC BY 4.0. See `LICENSE` for details.

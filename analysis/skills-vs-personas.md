# Skills versus Personas

Kimi Slides doesn't work like the other specialized agents. Docs, Sheets, and Websites use skill scaffolding: they take the OK Computer base prompt and add technical documentation. Slides replaces the entire identity with a McKinsey consultant persona. This is a fundamentally different approach to specialization.

I wanted to understand why. What makes presentations different from spreadsheets? Why would you use persona replacement for one and skill scaffolding for the other?

---

## The two patterns

**Skill scaffolding** is what Docs, Sheets, and Websites use. The pattern is: OK Computer identity plus mandatory skill reading. These agents share the same personality and communication style. The difference is what they know, not who they are.

Kimi Sheets forces reading of the xlsx skill. Kimi Docs requires reading docx or pdf skills. Kimi Websites mandates reading the webapp-building skill. These are pre-primed generalists. They don't change who Kimi is. They just change what Kimi is focused on.

**Persona replacement** is what Slides uses. The pattern is: replace OK Computer entirely with a different character. "You are a presentation designer who has worked at McKinsey for 20 years, specializing in creating high-information-density, content-rich, and in-depth presentation slides."

The McKinsey persona embodies a style, a voice, and a set of values. High-density layouts. Clean structure. Consultant-grade aesthetics. These are difficult to encode in procedural documentation.

---

## Why two patterns?

The nature of the task determines which pattern works better.

Spreadsheets and documents have objective correctness criteria. Formulas work or they don't. Compatibility rules are verifiable. Validation is binary: pass or fail. You can fully document these in a SKILL.md. Spreadsheet skills can be taught through instruction.

Presentations have subjective quality criteria. Aesthetic choices are not binary. Information density is a judgment call. Audience impact cannot be validated procedurally. You cannot fully specify what makes a slide compelling in documentation. Design taste cannot be taught through instruction.

Technical tasks get skill scaffolding. Creative tasks get persona replacement.

---

## What persona provides

The McKinsey consultant persona provides four things that SKILL.md cannot.

First, aesthetic direction. The McKinsey style is distinctive: clean, hierarchical, data-driven. The persona knows this implicitly.

Second, workflow authority. The agent presents itself as an expert, not a tool. It follows a rigid workflow. It communicates authoritatively.

Third, design philosophy. The persona knows how to balance density, clarity, and impact. This is consultant-grade judgment.

Fourth, communication standards. The tone and structure align with consultant expectations. The agent speaks differently than it does in OK Computer mode.

---

## The three-phase workflow

Slides follows a different workflow than other agents. While Docs and Sheets follow a technical-then-validate flow, Slides follows design-then-structure-then-render.

Phase one is design analysis. The McKinsey consultant analyzes requirements and audience from a consultant perspective. It determines information density strategy. It chooses visual direction, color system, and typography. The output is a design document in Markdown. This phase has no SKILL.md equivalent. The agent must think like a consultant, not follow procedure.

Phase two is content planning. The persona guides what counts as valuable research and how to structure content. The agent generates an outline interactively using the generate_slides_outline tool. The user confirms or modifies the structure before rendering begins. This collaborative approval prevents wasted rendering when the persona-driven outline doesn't match user intent.

Phase three is rendering. This phase is technical and procedural: generate HTML, apply CSS from the design system, convert to PPTX using slides_generator, deliver. This part is similar to other agents' rendering phases.

---

## Why no SKILL.md for Slides?

The absence of `/app/.kimi/skills/slides/SKILL.md` is not a gap. It is the expected outcome of the persona strategy.

Creating a SKILL.md for presentation design would require encoding subjective taste as procedures. You would need to specify the McKinsey aesthetic as rules. You would need to turn creative judgment into checklists. This is fundamentally impossible.

Where Docs agent has: "Read 32KB of technical procedures from docx/SKILL.md."

Slides agent has: "Be a McKinsey consultant with 20 years of presentation experience."

The second approach distributes expertise through character, not documentation.

---

## Key differences

Docs, Sheets, and Websites use skill injection. Their base identity stays OK Computer. Their knowledge source is SKILL.md files. Their validation is binary: pass or fail. Their first step is reading the skill file. Their correctness criteria are procedural rules. They use automatic generation without user approval.

Slides uses persona replacement. Its base identity becomes the McKinsey consultant. Its knowledge source is character perspective. Its validation is qualitative: compelling or not compelling. Its first step is thinking like a consultant. Its correctness criteria are aesthetic judgment. It uses interactive approval with outline confirmation.

---

## Architectural implications

The Slides agent demonstrates persona replacement as a specialization strategy. This is not a limitation. Slides doesn't have a SKILL.md because the task doesn't admit procedural specification.

It is deliberate architecture: using a McKinsey persona instead of technical documentation.

It uses different validation: qualitative user approval replaces binary checking.

It requires a collaborative process: interactive outline approval is built in.

The theoretical insight is that creative tasks require character embodiment, not skill injection.

Where Docs says "follow the DOCX rules," Slides says "think like a consultant." The absence of SKILL.md is not a gap. It is the signature of this different paradigm.

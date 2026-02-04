# Analysis

Research findings and insights from analyzing the Kimi K2.5 agent system.

---

## Documents

### [how-kimi-works.md](how-kimi-works.md)

Comprehensive overview of Kimi's architecture, agent taxonomy, and design patterns.

**Contains**:

1. The 8 agent types and their relationships
2. Environment vs. Tool-Use architecture comparison
3. Base Chat vs. OK Computer analysis
4. The skill system and how it works
5. Four-layer container architecture
6. Historical evolution of agent architecture

**Start here** if you are new to this repository.

---

### [skills-vs-personas.md](skills-vs-personas.md)

Analysis of the two specialization patterns: Skill Scaffolding versus Persona Replacement.

**Contains**:

1. Why Docs, Sheets, and Web use skill injection
2. Why Slides uses persona replacement
3. Technical vs. creative task distinction
4. The McKinsey consultant persona analysis
5. Three-phase workflow for presentation design

**Read this** to understand why Slides is architecturally different from other agents.

---

## Key Insights Summary

**1. Infrastructure is Intelligence**

The same model with persistent filesystem and 300 tool calls is "smarter" than the same model in a chat window with 10 calls.

**2. Context is Specialization**

You do not need fine-tuning to create a "Sheets Agent". You just need to force-feed it the right documentation.

**3. Persona is for Taste**

Technical tasks get skill documentation. Creative tasks get expert personas.

**4. Skills vs. Personas**

Skill scaffolding works for objective correctness. Persona replacement works for subjective excellence.

Skill scaffolding uses procedural documentation. Persona replacement uses character embodiment.

Skill scaffolding validates with binary pass or fail. Persona replacement validates with qualitative compelling assessment.

Docs, Sheets, and Websites use skill scaffolding. Slides uses persona replacement.

---

## Related Documentation

- [../GLOSSARY.md](../GLOSSARY.md) - Definitions of terms used in these analyses
- [../skills/README.md](../skills/README.md) - The skill system explained
- [../prompts/README.md](../prompts/README.md) - Agent hierarchy and prompt structure

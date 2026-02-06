# Analysis

Research findings and insights from analyzing the Kimi K2.5 agent system.

---

## Table of Contents

- [Documents](#documents)
- [Key Insights Summary](#key-insights-summary)
- [Related Documentation](#related-documentation)

---

## Documents

| Document | Description | Start Here If... |
|----------|-------------|------------------|
| [`how-kimi-works.md`](how-kimi-works.md) | Comprehensive overview of Kimi's architecture, agent taxonomy, and design patterns | You are new to this repository |
| [`skills-vs-personas.md`](skills-vs-personas.md) | Analysis of Skill Scaffolding versus Persona Replacement patterns | You want to understand why Slides is architecturally different |

---

### how-kimi-works.md

Comprehensive overview of Kimi's architecture, agent taxonomy, and design patterns.

**Contains**:

1. The 8 agent types and their relationships
2. Environment vs. Tool-Use architecture comparison
3. Base Chat vs. OK Computer analysis
4. The skill system and how it works
5. Four-layer container architecture
6. Historical evolution of agent architecture

---

### skills-vs-personas.md

Analysis of the two specialization patterns: Skill Scaffolding versus Persona Replacement.

**Contains**:

1. Why Docs, Sheets, and Web use skill injection
2. Why Slides uses persona replacement
3. Technical vs. creative task distinction
4. The McKinsey consultant persona analysis
5. Three-phase workflow for presentation design

---

## Key Insights Summary

**1. Context is Specialization**

You do not need fine-tuning to create a "Sheets Agent". You just need to force-feed it the right documentation.

**2. Persona is for Taste**

Technical tasks get skill documentation. Creative tasks get expert personas.

**3. Skills vs. Personas**

Skill scaffolding works for objective correctness. Persona replacement works for subjective excellence.

Skill scaffolding uses procedural documentation. Persona replacement uses character embodiment.

Skill scaffolding validates with binary pass or fail. Persona replacement validates with qualitative compelling assessment.

Docs, Sheets, and Websites use skill scaffolding. Slides uses persona replacement.

---

## Related Documentation

- [`../GLOSSARY.md`](../GLOSSARY.md) - Definitions of terms used in these analyses
- [`./skills/README.md`](./skills/README.md) - The skill system explained
- [`../prompts/README.md`](../prompts/README.md) - Agent hierarchy and prompt structure

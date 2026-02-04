# Skills: Deep-Dives by Capability

Comprehensive analysis of each Kimi skill including architecture, workflow, templates, and implementation.

## Skills Overview

Kimi provides four specialized skills for different output types:

### [docx/](docx/) - Word Document Generation
**Files**: 5 documents

Deep dive into Word document generation using:
- C# OpenXML SDK (template creation)
- Python openpyxl wrapper (document editing)
- .NET validator (schema compliance)

**Read**: [docx/docx-skill-analysis.md](docx/docx-skill-analysis.md)

### [pdf/](pdf/) - PDF Processing & Generation
**Files**: 2 documents

PDF generation with multiple routes:
- HTML + Paged.js for complex layouts
- LaTeX + Tectonic for mathematical content
- pikepdf for PDF manipulation

**Read**: [pdf/pdf-skill-analysis.md](pdf/pdf-skill-analysis.md)

### [xlsx/](xlsx/) - Excel Spreadsheet Generation
**Files**: 2 documents

Excel generation with validation:
- Python openpyxl for spreadsheet creation
- KimiXlsx binary for validation
- Pivot table support

**Read**: [xlsx/xlsx-skill-analysis.md](xlsx/xlsx-skill-analysis.md)

### [webapp/](webapp/) - React Web Applications
**Files**: 3 documents

Web application generation:
- React + TypeScript + Vite environment
- shadcn/ui components (50+ UI elements)
- Tailwind CSS styling

**Read**: [webapp/webapp-skill-analysis.md](webapp/webapp-skill-analysis.md)

## File Organization per Skill

Each skill subdirectory contains:
- `*-skill-analysis.md` - Technical deep-dive and architecture
- `*-skill-workflow.md` - How the skill integrates with agents
- `*-templates-analysis.md` (if applicable) - Template systems
- `*-scripts-analysis.md` (if applicable) - Helper scripts
- `*-validator-analysis.md` (if applicable) - Validation systems

## Quick Navigation

| Need | Go To |
|------|-------|
| Understand how a skill works | Read `*-skill-analysis.md` in that skill's folder |
| Learn the workflow | Read `*-skill-workflow.md` |
| See templates/examples | Read `*-templates-analysis.md` |
| Understand validation | Read `*-validator-analysis.md` or `*-scripts-analysis.md` |

## Related Documentation

- **How agents use skills**: See [../system-architecture/skill-system.md](../system-architecture/skill-system.md)
- **Runtime services**: See [../runtime/](../runtime/)
- **Skill definitions**: See [../../agents/skills/](../../agents/skills/)

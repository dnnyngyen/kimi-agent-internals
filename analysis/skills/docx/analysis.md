# DOCX Skill Analysis

Comprehensive analysis of the Word document generation skill.

---

## Overview

The DOCX skill handles Word document generation through an unusual dual-stack architecture. Unlike typical Python-based document generation, it uses C# with OpenXML SDK for creation and Python with lxml for editing.

This separation comes from different needs. Creation benefits from type-safe SDK usage with automatic package structure management. Editing requires direct XML manipulation for surgical precision.

---

## Dual-Stack Architecture

### Creation Path (C#)

```bash
./scripts/docx build [output.docx]
```

**Pipeline**:

1. `dotnet build` - Compilation
2. `dotnet run -- <output_path>` - Generation
3. `fix_element_order.py` - XML sequence correction
4. `validator/` - OpenXML schema validation
5. `validate_docx.py` - Business rule validation
6. `pandoc` - Content verification

**Path conventions**:

- Working directory: `/tmp/docx-work/`
- Output directory: `/mnt/okcomputer/output/`
- Skill directory: `/app/.kimi/skills/docx/`

### Editing Path (Python)

```python
from scripts.docx_lib.editing import (
    DocxContext,
    add_comment, reply_comment, resolve_comment,
    insert_paragraph, insert_text, propose_deletion,
    enable_track_changes
)

with DocxContext("input.docx", "output.docx") as ctx:
    add_comment(ctx, "Target text", "Comment content")
```

---

## C# Templates

The skill includes templates for different document types.

**KimiDocx.csproj** is the .NET project configuration. It is 15 lines.

**Program.cs** is the entry point stub that gets overwritten. It is 7 lines.

**Example.cs** is a complete English document example. It spans approximately 1,100 lines.

**CJKExample.cs** is the Chinese, Japanese, and Korean document example. It spans approximately 1,000 lines.

### Key Technical Patterns

**Element Ordering Rules** (critical for OpenXML compliance):

```csharp
// sectPr requires sequence: headerRef → footerRef → pgSz → pgMar
// Tables require: tblPr → tblGrid → tr (grid is mandatory)

// Column width consistency enforced:
table.Append(new TableGrid(
    new GridColumn { Width = "3600" },
    new GridColumn { Width = "5400" }
));
// Must match:
new TableCellWidth { Width = "3600", Type = TableWidthUnitValues.Dxa }
```

**Floating Background Image**:

```csharp
new DW.Anchor(...) {
    BehindDoc = true,  // Key: behind text
    Locked = false,
    LayoutInCell = true,
    AllowOverlap = true
}
```

### CJK (Chinese/Japanese/Korean) Handling

**The Compilation Problem**:

```csharp
// ❌ WRONG - Chinese quotes break C# compilation
new Text("请点击"确定"按钮")  // CS1003: Syntax error

// ✓ CORRECT - Unicode escapes
new Text("请点击\u201c确定\u201d按钮")
```

Unicode escapes for CJK punctuation:

- Left double quote " is `\u201c`
- Right double quote " is `\u201d`
- Left single quote ' is `\u2018`
- Right single quote ' is `\u2019`

**Verbatim String Trap**:

```csharp
// ❌ WRONG - @"" verbatim, \u NOT escaped
string text = @"她说\u201c你好\u201d";  // Literal: 她说\u201c你好\u201d

// ✓ CORRECT - regular string
string text = "她说\u201c你好\u201d";   // Output: 她说"你好"
```

---

## Validation Pipeline

### .NET Validator

The validator is a .NET 6.0+ application using the OpenXML SDK. The runtime consists of an ELF binary (~72 KB) with dependencies totaling approximately 6.9 MB.

**Configuration Files** (in repo):
- `Validator.deps.json` - NuGet dependency manifest
- `Validator.runtimeconfig.json` - .NET runtime configuration

**Runtime Dependencies** (not in repo):
- `DocumentFormat.OpenXml.dll` (6.3 MB)
- `DocumentFormat.OpenXml.Framework.dll` (469 KB)
- `System.IO.Packaging.dll` (142 KB)

### Validation Checks

**OpenXML Schema**:

- Package structure (`[Content_Types].xml`, `_rels/`)
- XML well-formedness
- Schema compliance
- Relationship integrity

**Business Rules**:

- `tblGrid` presence in tables
- Column width consistency
- `sectPr` ordering
- Style definitions exist

### Exit Codes

Exit code 0 means validation passed.

Exit code 1 means schema validation failed.

Exit code 2 means business rule violation.

Exit code 3 means file not found.

Exit code 4 means parse error.

---

## Tool Interaction Flow

### Creation Workflow

```
User Request
    ↓
read_file(SKILL.md)         # Load instructions
    ↓
read_file(Example.cs)       # Load template
    ↓
shell: init                 # Setup environment
    ↓
ipython: Edit Program.cs    # Customize template
    ↓
shell: build output.docx    # Compile & validate
    ↓
read_file(output.docx)      # Verify with pandoc
    ↓
Deliver
```

### Critical Path Timing

1. Read Skill takes approximately 200 milliseconds for SKILL.md plus Example.cs
2. IPython Edit takes approximately 500 milliseconds to generate Program.cs
3. Shell Build takes approximately 2000 milliseconds for dotnet build, run, and validate
4. Verify takes approximately 300 milliseconds for read_file with pandoc

Total time is approximately 3 seconds per document, dominated by C# compilation.

---

## Visual Design System

Color palettes for different document styles:

**Morandi** uses soft muted tones for artistic and editorial documents.

**Earth tones** uses brown, olive, and natural colors for environmental and organic themes.

**Nordic** uses cool gray and misty blue for minimalist and tech documents.

**Wabi-sabi** uses gray, raw wood, and zen tones for traditional aesthetics.

**French elegance** uses off-white and dusty pink for luxury and feminine themes.

**Industrial** uses charcoal, rust, and concrete for manufacturing documents.

**Academic** uses navy, burgundy, and ivory for research and education.

Cover backgrounds generate via HTML and CSS through Playwright:

- Device scale factor 2 (794 by 1123 pixels becomes 1588 by 2246 pixels)
- Floating anchor with `BehindDoc=true`
- Center whitespace for title placement

---

## Python Library Structure

```
scripts/
├── docx_skill.py          # Main skill implementation
├── docx_lib/
│   ├── docx.py            # Core operations
│   ├── docx2python.py     # DOCX to Python conversion
│   ├── tables.py          # Table processing
│   ├── text.py            # Text extraction
│   ├── images.py          # Image handling
│   ├── styles.py          # Style management
│   └── editing/           # Document editing subsystem
│       ├── comments.py
│       ├── context.py
│       ├── helpers.py
│       └── revisions.py
```

**Dependencies**:

- `python-docx>=0.8.11`
- `lxml>=4.6.3`
- `Pillow>=8.0.0`

---

## Key Insights

### 1. Language Bifurcation

Shell orchestrates compiled binaries like the C# toolchain. IPython handles dynamic scripting like Python lxml for XML surgery.

### 2. Validation as Gatekeeper

Every shell `build` command includes mandatory validation. The skill cannot produce output without passing OpenXML schema validation via binary, business rule validation via Python, and pandoc content verification.

### 3. Template-Driven Code Generation

IPython modifies C# template files like Program.cs, which then compile via shell and dotnet. This is meta-programming. Python writes C# that writes docx.

### 4. Documentation-as-Capability

There is no backend document service. The agent reads SKILL.md with 32KB of instructions. The agent generates code in C# or Python. The agent executes via shell and ipython. Validation ensures correctness.

The shell and ipython are not just tools. They are the runtime environment for the skill's documented procedures.

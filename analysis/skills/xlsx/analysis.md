# XLSX Skill Analysis

Comprehensive analysis of the Excel spreadsheet generation skill.

---

## Overview

The XLSX skill handles Excel spreadsheet generation through a validation-centric workflow. Unlike the DOCX skill which uses multiple tools and languages, XLSX relies primarily on Python plus openpyxl for creation, with a 77MB binary called KimiXlsx handling validation, PivotTable creation, and formula checking.

---

## Technology Stack

**Creation** uses Python plus openpyxl plus pandas for spreadsheet structure, data, formulas, and styling.

**Validation** uses the KimiXlsx CLI, a 77MB binary, for formula checking, schema validation, and PivotTables.

---

## The KimiXlsx Binary

At 77,001,601 bytes, KimiXlsx is the largest binary in the Kimi system.

### Binary Details

```bash
$ file KimiXlsx
KimiXlsx: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV),
          dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,
          for GNU/Linux 2.6.32, BuildID[sha1]=771f85eb041bea21c911f46511d0b9727b0b6f68,
          stripped
```

The binary is an ELF 64-bit LSB PIE executable for x86-64 architecture. It is 77 MB and stripped. PIE is enabled.

**Size breakdown**:

- OpenXML SDK: ~30 MB
- PivotTable Engine: ~20 MB
- Chart Engine: ~15 MB
- Validation Logic: ~10 MB
- Runtime: ~2 MB

### Commands

**recheck** detects formula errors like #VALUE! and #DIV/0!. Exit code 0 means pass. Exit code 1 means fail.

**reference-check** validates formula references for out-of-range or header inclusion errors. Exit code 0 means pass. Exit code 1 means fail.

**validate** checks OpenXML schema and forbidden functions. Exit code 0 means pass. Non-zero means fail.

**pivot** creates PivotTable with chart. Exit code varies.

**chart-verify** confirms charts have data. Exit code 0 means pass. Exit code 1 means fail.

**inspect** performs structure analysis with JSON output. Exit code 0 means pass.

---

## Validation Workflow

### Per-Sheet Validation (Zero Tolerance)

```
For each sheet:
    1. PLAN → Design structure, formulas, references
    2. CREATE → Write data, formulas, styling (ipython)
    3. SAVE → wb.save()
    4. CHECK → recheck + reference-check → Fix until 0 errors
    5. NEXT → Proceed only when current sheet passes

After ALL sheets:
    6. VALIDATE → validate command → Exit code 0 required
    7. DELIVER → Only validated files
```

**Key principle**: Validation is interleaved with creation, not just post-processing. Shell commands act as circuit breakers between sheets.

### Standard Workflow

```
Read SKILL.md (925 lines)
    ↓
Plan sheets (Cover, Data, Analysis, Pivot)
    ↓
Sheet 1 (Cover):
    ipython: Create cover styling
    ipython: Save
    shell: recheck → 0 errors?
    shell: reference-check → 0 errors?
    Yes → Next sheet
    ↓
Sheet 2 (Data):
    ipython: Add data + formulas
    ipython: Add charts
    ipython: Save
    shell: recheck → 0?
    shell: reference-check → 0?
    Yes → Next sheet
    ↓
...
Final Validation:
    shell: validate → Exit code 0?
    Yes → Deliver
```

---

## Formula Constraints

### Forbidden Functions (Excel 365 Only)

These functions work in Excel 365 but crash in Excel 2019 and earlier:

**FILTER()** should be replaced with AutoFilter or SUMIF and COUNTIF.

**UNIQUE()** should be replaced with Remove Duplicates or COUNTIF helper.

**XLOOKUP()** should be replaced with INDEX plus MATCH.

**XMATCH()** should be replaced with MATCH.

**LET()** should be replaced with helper cells.

**LAMBDA()** should be replaced with named ranges.

**SORT() and SORTBY()** should be replaced with manual sorting.

The `validate` command specifically checks for these and rejects files that use them.

### Formula Mandate

The skill requires Excel formulas, not calculated values:

```python
# CORRECT - Formula
ws['C2'] = '=A2+B2'

# FORBIDDEN - Static value
result = value_a + value_b
ws['C2'] = result  # Violates skill principles
```

Benefits:

- Enables user modification in Excel
- Self-updating when referenced data changes
- Professional spreadsheet standards

---

## PivotTable Creation Protocol

PivotTables have strict workflow constraints.

### Correct Order

```
openpyxl creates base.xlsx (all sheets except pivot)
    ↓
KimiXlsx pivot command (adds PivotTable)
    ↓
validate
    ↓
DELIVER (do NOT modify again!)
```

### Wrong Order (Corrupts File)

```
pivot creates pivot.xlsx
    ↓
openpyxl opens to add Cover sheet  ← CORRUPTS pivotCache!
    ↓
File broken
```

**Critical rule**: The pivot binary modifies Excel files in ways openpyxl does not understand, corrupting the pivotCache if subsequently opened.

### Pivot Command

```bash
/app/.kimi/skills/xlsx/scripts/KimiXlsx pivot \
    data.xlsx output.xlsx \
    --source "Sales!A1:F100" \
    --rows "Category" \
    --values "Revenue:sum" \
    --location "Summary!A3" \
    --chart "bar"
```

---

## IPython Usage Patterns

### Workbook Construction

```python
from openpyxl import Workbook
from openpyxl.styles import PatternFill, Font, Border, Alignment
from openpyxl.chart import BarChart, Reference
from openpyxl.formatting.rule import DataBarRule

# Create
wb = Workbook()
ws = wb.active
ws.title = "Data"

# Data with formulas
ws['C2'] = '=A2+B2'                        # Sum
ws['D2'] = '=C2/B2*100'                    # Percentage
ws['E2'] = '=SUM(A2:A100)'                 # Aggregation

# Styling
ws['A1'].font = Font(bold=True, color="FFFFFF")
ws['A1'].fill = PatternFill(start_color="333333", fill_type="solid")

# Conditional formatting
ws.conditional_formatting.add('C2:C100',
    DataBarRule(start_type='min', end_type='max', color='4A90D9'))

# Charts
chart = BarChart()
chart.add_data(Reference(ws, min_col=2, min_row=1, max_row=4))
ws.add_chart(chart, "E2")

wb.save('/mnt/okcomputer/output/data.xlsx')
```

### Cover Page Creation

```python
# Cover sheet (always first)
cover = wb.create_sheet("Cover", 0)
cover.sheet_view.showGridLines = False

# Merged title
cover.merge_cells('B2:G2')
cover['B2'] = "Report Title"
cover['B2'].font = Font(size=20, bold=True)

# Metrics table with cross-sheet references
cover['B5'] = "Key Metrics"
cover['B6'] = "Total Revenue"
cover['C6'] = '=Data!E100'  # Formula linking to data sheet
```

### Cross-Sheet References

```python
# VLOOKUP pattern
ws['D2'] = '=IFERROR(VLOOKUP(A2,Data!$G$2:$I$50,3,FALSE),"N/A")'

# Cross-sheet formula
summary['B2'] = '=SUM(Data!C2:C100)'
```

---

## Visual Design System

### Minimalist Monochrome (Default)

**Base colors**: White (#FFFFFF), Black (#000000), Grey shades

**Accent**: Blue only (#0066CC, #4A90D9, #E6F0FA)

**Forbidden**: All other colors

### Professional Finance

**Background**: #ECF0F1 (light gray)

**Header**: #1F4E79 (dark blue)

**Regional convention**: China uses Red for Up and Green for Down. International uses Green for Up and Red for Down.

---

## Key Insights

### 1. Validation-First Design

Unlike DOCX where validation is a final step, XLSX enforces per-sheet validation. Shell commands are invoked multiple times during creation.

### 2. Binary as "Secret Sauce"

The 77MB KimiXlsx binary parses Excel OpenXML natively. It detects formula errors openpyxl cannot catch. It validates Excel 365 versus 2019 compatibility. It creates PivotTables via pure OpenXML SDK.

### 3. Formula-Centric Philosophy

Formulas over values enable user modification. They are self-updating when data changes. They follow professional spreadsheet standards.

### 4. Python-Only Creation (Except Pivot)

Unlike DOCX which uses C# generation, XLSX uses Python and openpyxl for everything except PivotTable creation. This is because openpyxl handles styling and formulas well. There is no compilation step for faster iteration. PivotTable cache is too complex for openpyxl.

### 5. Validation-as-Gatekeeper

The agent has freedom to write any Python and openpyxl code. The binary validator enforces correctness. Per-sheet validation prevents error accumulation. Formula errors are blocking, not warnings.

---

## Security Assessment

**Network access**: None. The binary operates offline for validation.

**File system**: Read-only for input, write for output.

**Execution**: Sandboxed.

**Stripped**: Yes, making it harder to reverse engineer.

The binary is completely offline. There are no package downloads or external service contacts.

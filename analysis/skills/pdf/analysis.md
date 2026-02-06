# PDF Skill Analysis

Comprehensive analysis of the PDF generation and processing skill.

---

## Overview

The PDF skill handles PDF creation and processing through a multi-route architecture. Unlike other skills that use a single toolchain, PDF generation requires different approaches depending on document type.

**HTML Route** is best for business documents and reports. It uses Playwright plus Paged.js.

**LaTeX Route** is best for academic papers and citations. It uses the Tectonic compiler.

**Process Route** is for existing PDF manipulation. It uses pikepdf plus pdfplumber.

**Convert Route** is for Office format conversion. It uses LibreOffice.

---

## Multi-Route Architecture

```
PDF Request
    ↓
Route Selection:
├── Business report with complex layout → HTML Route
├── Academic paper with citations → LaTeX Route
├── Modify existing PDF → Process Route
└── Convert Office document → Convert Route
```

---

## HTML Route (Primary)

The HTML route is the default and most complex. It uses `html_to_pdf.js`, a roughly 600 line Node.js script orchestrating the conversion.

### Conversion Pipeline

```
Input HTML
    ↓
Load & Inject Playwright ──→ Custom CSS (optional)
    ↓
Detect Counters ──→ Warn if CSS counters used
Detect Mermaid ──→ Wait for SVG rendering
Detect KaTeX ──→ Trigger math rendering
    ↓
Fix Counters ──→ Burn in counter values
Fix Tall Elements ──→ Remove page-break-inside
    ↓
Inject Paged.js ──→ Local bundle or CDN
    ↓
Wait for Pages ──→ Poll until count stable
    ↓
Detect Overflow ──→ Warn if elements overflow
Get Statistics ──→ Word count, figures, tables
Detect Anomalies ──→ Blank/low-content pages
    ↓
Export PDF (scale=1.5 workaround)
```

### Critical Workarounds

**CSS Counter Detection**:

Paged.js reorders DOM during pagination, breaking CSS counters. The script detects counter usage and warns users.

```javascript
// Detect counter-reset, counter-increment, content: counter()
const counterWarnings = await page.evaluate(() => {
    const warnings = [];
    const styles = document.querySelectorAll('style');
    styles.forEach((style, idx) => {
        const css = style.textContent || '';
        if (/content\s*:\s*[^;]*counter\s*\(/i.test(css)) {
            // ... warn
        }
    });
    return warnings;
});
```

**Counter Fix (Burn-in)**:

Before Paged.js runs, the script "burns in" counter values as data attributes.

```javascript
const counterFixCount = await page.evaluate(() => {
    let fixedCount = 0;
    const olLists = document.querySelectorAll('ol');
    olLists.forEach(ol => {
        const liItems = ol.querySelectorAll(':scope > li');
        const hasCustomNumbering = computedStyle.listStyleType === 'none';
        if (hasCustomNumbering) {
            ol.setAttribute('data-counter-fixed', 'true');
            liItems.forEach((li, index) => {
                li.setAttribute('data-counter', index + 1);
                fixedCount++;
            });
        }
    });
    return fixedCount;
});
```

**Mermaid Rendering Wait**:

The script waits for Mermaid diagrams to render before pagination.

```javascript
const hasMermaid = await page.evaluate(() => {
    return document.querySelectorAll('.mermaid').length > 0;
});

if (hasMermaid) {
    await page.waitForFunction(() => {
        const mermaids = document.querySelectorAll('.mermaid');
        for (const m of mermaids) {
            if (!m.querySelector('svg') && !m.getAttribute('data-processed')) {
                return false;
            }
        }
        return true;
    }, { timeout: 30000 });
    await delay(2000);  // Stabilization time
}
```

**Pagination Stability Detection**:

The script waits for the page count to stabilize.

```javascript
let lastPageCount = 0;
let stableCount = 0;
const maxWaitTime = 120000;  // 2 minutes max

while (stableCount < 3) {
    if (Date.now() - startTime > maxWaitTime) {
        console.log('Warning: Pagination timeout');
        break;
    }
    await delay(1000);
    const currentPageCount = await page.evaluate(() =>
        document.querySelectorAll('.pagedjs_page').length
    );
    if (currentPageCount === lastPageCount) {
        stableCount++;
    } else {
        stableCount = 0;
        lastPageCount = currentPageCount;
        console.log(`  Pagination in progress: ${currentPageCount} pages...`);
    }
}
```

**PDF Export Scale Workaround**:

The script uses `scale: 1.5` to fix a page filling issue.

```javascript
await page.pdf({
    path: outputPath,
    format: 'A4',
    printBackground: true,
    preferCSSPageSize: true,
    tagged: true,
    scale: 1.5  // Critical workaround
});
```

### Critical Constraints

**Paged.js Loading**: The conversion script injects automatically. Do NOT duplicate.

**JS Chart Libraries**: PROHIBITED. ECharts, Chart.js, D3.js, and Plotly cause layout conflicts.

**Data Charts**: Use img tags with pre-generated matplotlib PNG.

**Mermaid Layout**: Default to `flowchart LR`. TB risks overflow.

**Mermaid Nodes**: Maximum 12 total, maximum 3 subgraphs, maximum 5 nodes per subgraph.

**CSS Counters**: PROHIBITED. Paged.js DOM reordering breaks counters.

### Cover Design Requirements

Full-bleed covers require all three CSS rules:

```css
body { margin: 0; padding: 0; }                    /* Reset browser default */
@page :first { margin: 0; }                        /* Remove page margin */
.cover { margin: 0; position: relative;            /* Element positioning */
         width: 210mm; height: 297mm; overflow: hidden; }
```

---

## LaTeX Route

The LaTeX route uses the Tectonic compiler, a 57.4MB binary and modern LaTeX engine that auto-downloads packages.

### Compilation

```bash
python3 /app/.kimi/skills/pdf/scripts/compile_latex.py main.tex --runs 2
```

The wrapper provides log filtering to remove noise, multiple compilation passes, PDF statistics extraction, and error and warning categorization.

### Log Filtering

```python
FILTER_PATTERNS = [
    r'^note: "version 2" Tectonic command-line interface activated',
    r'^note: Running TeX',
    r'^note: Rerunning TeX because',
    r'^note: Running xdvipdfmx',
    r'^note: downloading ',
    r'^note: Skipped writing .* intermediate files',
]
```

### Issue Categorization

**Errors** matching `^error:` are critical.

**Warnings** matching `^warning:` are medium priority.

**Layout** issues like `Overfull/Underfull \[hv]box` are medium priority.

**Fonts** issues matching `Font shape` or `Missing character` are low priority.

### Package Loading Order

```latex
% Required base packages
\usepackage{graphicx, xcolor, geometry, amsmath}

% ... other packages ...

% hyperref MUST be loaded LAST
\usepackage[colorlinks=true, linkcolor=blue, citecolor=darkgray,
            urlcolor=blue, bookmarks=true, unicode=true]{hyperref}
```

---

## Process Route

The Process route uses pikepdf and pdfplumber for existing PDF manipulation.

```bash
pdf.py <command> <subcommand> [options]
```

### Commands

**form** with subcommands `info` and `fill` handles form field inspection and filling.

**extract** with subcommands `text`, `table`, and `image` handles content extraction.

**pages** with subcommands `merge`, `split`, `rotate`, and `crop` handles page operations.

**meta** with subcommands `get` and `set` handles metadata operations.

**convert** handles format conversion.

### Output Format

```json
// Success
{"status": "success", "data": {...}}

// Error
{"status": "error", "error": "ErrorType", "message": "...", "hint": "..."}
```

### Exit Codes

Exit code 0 means success.

Exit code 1 means argument error.

Exit code 2 means file not found.

Exit code 3 means PDF parse error.

Exit code 4 means operation failed.

---

## Paged.js Library

The skill includes a bundled copy of Paged.js (922KB, version 0.4.3) for offline operation.

### Why Bundled?

1. **Offline Operation**: No CDN dependency
2. **Version Stability**: Fixed, tested version
3. **Performance**: Local file loads faster than network
4. **Reliability**: Avoids CDN availability issues

### Key Features

- `@page` rule processing
- Page break control (`page-break-before`, `page-break-after`, `page-break-inside`)
- Running headers and footers (`@top-center`, `@bottom-center`)
- Named pages (`page: cover`)
- Page number counters (`counter(page)`)

---

## Script Inventory

**html_to_pdf.js** is roughly 600 lines of Node.js code for HTML to PDF conversion.

**paged.polyfill.js** is a 922KB library containing the Paged.js pagination library.

**compile_latex.py** is roughly 350 lines of Python for LaTeX compilation wrapper.

**pdf.py** is roughly 260 lines of Python for the unified CLI entry point.

**cmd_extract.py** is roughly 150 lines of Python for text, table, and image extraction.

**cmd_pages.py** is roughly 150 lines of Python for merge, split, rotate, and crop operations.

**cmd_form.py** is roughly 120 lines of Python for form filling.

**cmd_meta.py** is roughly 80 lines of Python for metadata operations.

**cmd_convert.py** is roughly 100 lines of Python for format conversion.

**browser_helper.js** is roughly 100 lines of Node.js for Playwright utilities.

---

## Output Statistics

```
========================
PDF Information
========================
File: document.pdf
Pages: 12
Size: 245.6 KB
Words: ~3,456
Figures/Tables: 5 figures / 3 tables
Path: /mnt/okcomputer/output/document.pdf

========================
Anomaly Detection
========================
  P8: [Low content] 45 words (avg 320, only 14%)
```

---

## Key Insights

### 1. Route Selection is Critical

Different document types require different tools. The skill provides intelligent routing based on content analysis.

### 2. HTML Route Complexity

The HTML route is the most complex due to browser-based pagination via Paged.js, multiple rendering engines for Mermaid and KaTeX, workarounds for CSS limitations, and real-time overflow detection.

### 3. LaTeX for Academic Precision

Tectonic provides automatic package management, superior math typesetting, citation and bibliography support, and academic formatting standards.

### 4. Process Route for Manipulation

pikepdf and pdfplumber enable form field operations, content extraction, page manipulation, and metadata handling.

### 5. Bundled Dependencies

The skill bundles key dependencies like Paged.js and Tectonic to ensure offline operation, version stability, and consistent output.

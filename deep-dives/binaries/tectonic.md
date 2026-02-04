# Tectonic LaTeX Compiler

Tectonic is a modern, self-contained LaTeX compiler based on the XeTeX engine. At 57,402,608 bytes, this binary provides complete LaTeX compilation capabilities without requiring a full TeX Live installation. It automatically downloads required packages on first use.

---

## Binary Analysis

```bash
$ file /app/tectonic
/app/tectonic: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV),
               dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,
               BuildID[sha1]=b14ef99db5d130541d5788c55c3d4faea1cbd00e,
               for GNU/Linux 3.2.0, with debug_info, not stripped
```

The binary is an ELF 64-bit LSB Position Independent Executable for x86-64 architecture. It is dynamically linked with debug info present. The size is 57.4 MB. PIE is enabled for ASLR protection.

Size breakdown:
- XeTeX Engine: ~20 MB
- Package Index: ~5 MB
- Font Data: ~15 MB
- Support Libraries: ~17 MB

---

## Core Features

Tectonic runs on the XeTeX engine which is Unicode-aware. It manages packages through automatic download on first use. It supports system fonts, OpenType, and TrueType formats. Output goes to PDF via xdvipdfmx. Cross-references resolve automatically.

Tectonic is modern XeTeX. It understands Unicode natively and can use system fonts directly without LaTeX font configuration. This makes it significantly easier to use with non-English languages and custom fonts.

---

## Compilation Workflow

```
.tex Input
    │
    ▼
XeTeX Engine
    │
    ├──► Parse LaTeX
    ├──► Resolve packages (download if needed)
    ├──► Process cross-references
    └──► Generate .xdv (extended DVI)
    │
    ▼
xdvipdfmx
    │
    ▼
PDF Output
```

Tectonic runs XeTeX to generate an extended DVI file. Then xdvipdfmx converts that to PDF. The process handles package downloads automatically. If you use a package not present locally, Tectonic fetches it from CTAN.

---

## PDF Skill Integration

The PDF skill uses Tectonic for the LaTeX route:

```
PDF Skill
    │
    ├── HTML Route ──► Playwright + Paged.js
    │
    ├── LaTeX Route ──► compile_latex.py
    │                    │
    │                    └── tectonic
    │                         │
    │                         └── PDF Output
    │
    └── Process Route ──► pikepdf + pdfplumber
```

The `compile_latex.py` wrapper script provides log filtering to remove noise, multiple compilation passes, PDF statistics extraction, and error and warning categorization.

Usage:
```bash
# Single compilation
python3 /app/.kimi/skills/pdf/scripts/compile_latex.py main.tex

# Multiple passes (for cross-references)
python3 /app/.kimi/skills/pdf/scripts/compile_latex.py main.tex --runs 2

# With bibliography
python3 /app/.kimi/skills/pdf/scripts/compile_latex.py main.tex --runs 3
```

---

## Package Management

Tectonic maintains a local package cache at `~/.cache/Tectonic/`. It downloads packages from CTAN on first use. It caches them for subsequent compilations. No manual TeX Live installation is required.

This is the key advantage over traditional LaTeX distributions. TeX Live requires a 4+ GB installation upfront. Tectonic starts at 57 MB and fetches only what you need.

---

## Log Filtering

The compile_latex.py wrapper filters verbose Tectonic output:

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

Issue categorization works as follows. Errors matching `^error:` are critical. Warnings matching `^warning:` are medium priority. Layout issues like `Overfull/Underfull \[hv]box` are medium priority. Font issues matching `Font shape` or `Missing character` are low priority.

---

## PDF Statistics

The wrapper extracts PDF metadata after compilation:

```python
def extract_pdf_info(pdf_path):
    from pypdf import PdfReader
    reader = PdfReader(pdf_path)

    num_pages = len(reader.pages)
    text = ''.join(page.extract_text() for page in reader.pages)
    word_count = len([w for w in text.split() if w.strip()])

    # Count images
    image_count = 0
    for page in reader.pages:
        if '/XObject' in page['/Resources']:
            xobjects = page['/Resources']['/XObject'].get_object()
            for obj in xobjects:
                if xobjects[obj]['/Subtype'] == '/Image':
                    image_count += 1

    return num_pages, word_count, image_count
```

---

## Comparison with TeX Live

Tectonic is 57 MB versus TeX Live at 4+ GB. Tectonic is a single binary. TeX Live requires complex installation. Tectonic manages packages automatically. TeX Live uses manual management through tlmgr. Tectonic has high portability. TeX Live has low portability. Tectonic works offline after first download. TeX Live works fully offline immediately.

Tectonic trades offline capability for size and simplicity. After first use with a given package set, it is fully offline capable. The initial download requirement means it is not suitable for fully air-gapped environments.

---

## Security Analysis

Tectonic requires network access to download packages from CTAN. It reads .tex files and writes .pdf files. Execution is sandboxed by the OS. PIE is enabled for ASLR protection. Debug info is present since the binary is not stripped.

The network access is the main security consideration. Tectonic downloads and executes LaTeX package code from the internet. In the Kimi container, this is sandboxed and acceptable. In high-security environments, you would want to pre-populate the cache.

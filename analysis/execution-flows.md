# Execution Flows and Tool Chains

This document maps how tools chain together in different execution scenarios.

---

## High-Level Dependency Graph

```
User Request
    │
    ├─► Intent Classification (System Layer)
    │
    ├─► Base Chat Path ───────────────────────────────────────────────┐
    │   ├─► web_search ──┐                                            │
    │   ├─► web_open_url │                                            │
    │   ├─► ipython ─────┼─► External Data/Computation                │
    │   ├─► memory_space─┤                                            │
    │   └─► datasources ─┘                                            │
    │                                                                 │
    └─► OK Computer Path ─────────────────────────────────────────────┤
        │                                                             │
        ├─► Skill Detection ──┐                                       │
        │   ├─► docx ─────────┼─► read_file(SKILL.md)                 │
        │   ├─► xlsx ─────────┤                                       │
        │   ├─► pdf ──────────┤                                       │
        │   └─► webapp ───────┘                                       │
        │                                                             │
        ├─► Tool Selection                                            │
        │   ├─► read_file ◄─── Required first for skills              │
        │   ├─► todo_read ──── Session state check                    │
        │   └─► todo_write ─── Task planning                          │
        │                                                             │
        ├─► Data Acquisition                                          │
        │   ├─► web_search                                            │
        │   ├─► get_data_source                                       │
        │   ├─► search_image_by_text                                  │
        │   └─► search_image_by_image                                 │
        │                                                             │
        ├─► Browser Automation (if needed)                            │
        │   ├─► browser_visit ◄─── Entry point                        │
        │   ├─► browser_click                                         │
        │   ├─► browser_find                                          │
        │   ├─► browser_input                                         │
        │   ├─► browser_scroll_*                                      │
        │   ├─► browser_screenshot                                    │
        │   └─► browser_state                                         │
        │                                                             │
        ├─► Content Generation                                        │
        │   ├─► ipython ◄─────── Primary computation                  │
        │   ├─► write_file                                            │
        │   ├─► edit_file                                             │
        │   └─► shell ◄─────── Build orchestration                    │
        │                                                             │
        ├─► Media Generation                                          │
        │   ├─► generate_image                                        │
        │   ├─► generate_speech                                       │
        │   └─► generate_sound_effects                                │
        │                                                             │
        ├─► Asset Extraction (Web Replication)                        │
        │   ├─► screenshot_web_full_page                              │
        │   ├─► find_asset_bbox                                       │
        │   └─► crop_and_replicate_assets_in_image                    │
        │                                                             │
        └─► Delivery                                                  │
            ├─► deploy_website                                        │
            ├─► slides_generator                                      │
            └─► KIMI_REF tags (automatic)                             │
```

---

## Skill-Specific Tool Chains

### DOCX Skill Tool Chain

```
read_file(SKILL.md) ─► read_file(Example.cs)
    │
    ▼
shell: ./docx init ──► Setup environment
    │
    ▼
ipython: Generate Program.cs ──► Meta-programming
    │
    ▼
shell: ./docx build ──► Compilation pipeline:
    ├─► dotnet build
    ├─► dotnet run
    ├─► python fix_element_order.py
    ├─► ./validator/Validator
    ├─► python validate_docx.py
    └─► pandoc verification
    │
    ▼
read_file(output.docx) ──► Verification
    │
    ▼
KIMI_REF output.docx
```

### XLSX Skill Tool Chain

```
read_file(SKILL.md)
    │
    ▼
Per-Sheet Loop:
    ├─► ipython: openpyxl creation
    ├─► ipython: wb.save()
    ├─► shell: KimiXlsx recheck ──┐
    ├─► shell: reference-check ───┤ Validation gates
    └─► Error? ──► Fix & retry ◄──┘
    │
    ▼
[If PivotTable needed]:
    ├─► shell: KimiXlsx pivot (MUST be last)
    │
    ▼
shell: KimiXlsx validate ──► Exit code 0 required
    │
    ▼
KIMI_REF output.xlsx
```

### PDF Skill Tool Chain (HTML Route)

```
read_file(SKILL.md) ─► read_file(routes/html.md)
    │
    ▼
ipython: Generate HTML + CSS
    ├─► matplotlib charts (if needed)
    └─► KaTeX math (if needed)
    │
    ▼
write_file: /tmp/input.html
    │
    ▼
shell: pdf.sh html input.html ──► Playwright + Paged.js
    ├─► Mermaid rendering (if present)
    ├─► Pagination stability detection
    └─► PDF export (scale: 1.5)
    │
    ▼
KIMI_REF output.pdf
```

### PDF Skill Tool Chain (LaTeX Route)

```
read_file(SKILL.md) ─► read_file(routes/latex.md)
    │
    ▼
ipython: Generate .tex source
    ├─► Document structure
    ├─► Math formulas
    └─► Bibliography (.bib)
    │
    ▼
write_file: /tmp/main.tex
    │
    ▼
shell: compile_latex.py main.tex --runs 2
    ├─► tectonic run 1 (generate aux)
    └─► tectonic run 2 (resolve references)
    │
    ▼
KIMI_REF output.pdf
```

### WebApp Skill Tool Chain

```
read_file(SKILL.md)
    │
    ▼
shell: init-webapp.sh "Title"
    ├─► Copy 73-file template
    └─► npm install (26,082 files)
    │
    ▼
ipython: Component architecture planning
    │
    ▼
write_file: src/components/*.tsx
    ├─► React components
    ├─► TypeScript types
    └─► shadcn/ui usage
    │
    ▼
shell: npm run build ──► Vite bundler:
    ├─► Tree-shaking
    ├─► Code splitting
    ├─► Asset optimization
    └─► dist/ generation
    │
    ▼
deploy_website(dist)
    │
    ▼
Public URL returned
```

---

## Tool Synergy Patterns

### Pattern 1: Data Pipeline

```
get_data_source ──► ipython (pandas analysis) ──► write_file (CSV)
     │                                          │
     └─► Source citation                        └─► KIMI_REF
```

### Pattern 2: Web Research

```
web_search ──► browser_visit ──► browser_scroll ──► browser_screenshot
    │                                                  │
    └─► Initial discovery                              └─► Visual verification
```

### Pattern 3: Asset Extraction

```
screenshot_web_full_page ──► find_asset_bbox ──► crop_and_replicate
        │                          │                    │
        └─► Full page capture      └─► Detection        └─► Extraction
```

### Pattern 4: Media Production

```
generate_image ──► ipython (Pillow processing) ──► write_file
    │                                               │
    └─► Base image                                  └─► Final asset
```

### Pattern 5: Document Assembly

```
read_file (multiple) ──► ipython (analysis) ──► Skill-specific generation
        │                                         │
        └─► Source material                       └─► shell/ipython pipeline
                                                        │
                                                        ▼
                                                   KIMI_REF
```

---

## Key Insights

1. **Skill Loading is Always First**: Every specialized workflow starts with `read_file(SKILL.md)`

2. **Validation Gates**: XLSX skill enforces per-sheet validation; DOCX validates after build

3. **Build Orchestration**: Complex skills (DOCX, WebApp) use shell as build system, not just command executor

4. **Route Selection**: PDF skill branches early (HTML vs LaTeX) with different tool chains

5. **Meta-Programming**: DOCX skill uses IPython to generate C# that generates documents

6. **External Toolchains**: WebApp skill manages 26K+ node_modules files through npm/vite

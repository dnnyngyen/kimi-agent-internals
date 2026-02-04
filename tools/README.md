# Tools

Tool schemas and documentation for the Kimi K2.5 agent system.

---

## Overview

Kimi provides different tool sets depending on the agent mode. Base Chat gets 9 tools. OK Computer gets 28. The difference is not just quantity. It is capability.

Base Chat can search the web, run Python, and access a read-only filesystem. OK Computer adds browser automation, image generation, voice synthesis, and full filesystem access. The same underlying model becomes dramatically more powerful through tooling alone.

---

## Base Chat Tools (9)

**Web and Search**

`web_search` finds information across the internet. `web_open_url` visits specific pages. These two tools handle most information retrieval tasks.

`search_image_by_text` and `search_image_by_image` handle visual search. The first finds images matching a description. The second does reverse image search.

**Code and Data**

`ipython` executes Python code in a sandboxed environment. This is the primary computation engine.

`shell` runs bash commands against a read-only filesystem. Base Chat cannot write files. It can only inspect what exists.

`get_data_source` and `get_data_source_desc` connect to configured data sources. These require prior setup in the Kimi interface.

**Memory**

`memory_space_edits` manages the memory space. This is persistent storage for facts and context across conversations.

---

## OK Computer Tools (28)

**Browser Automation (8 tools)**

OK Computer adds full browser control through Playwright. `browser_visit` navigates to URLs. `browser_click`, `browser_input`, and `browser_find` interact with page elements. `browser_scroll_up` and `browser_scroll_down` handle navigation. `browser_screenshot` captures visual state. `browser_state` returns the accessibility tree for parsing.

These tools enable complex web workflows. The agent can log into sites, fill forms, extract data, and navigate multi-page applications.

**File Operations (3 tools)**

`read_file`, `write_file`, and `edit_file` provide full filesystem access. Unlike Base Chat, OK Computer can create and modify files. This enables document generation, code writing, and data transformation pipelines.

**Image and Media (6 tools)**

`generate_image` creates images from text descriptions. `crop_and_replicate_assets_in_image` extracts visual assets from screenshots. `find_asset_bbox` locates specific elements within images for extraction.

`generate_speech` converts text to spoken audio. `generate_sound_effects` creates ambient sounds. `get_available_voices` lists voice options for speech generation.

**Search and Data (4 tools)**

The same web search and image search tools from Base Chat appear here. `get_data_source` provides access to configured data sources.

**Utilities (5 tools)**

`ipython` and `shell` work like Base Chat, but with full filesystem access. `todo_read` and `todo_write` manage task lists for complex multi-step workflows. `screenshot_web_full_page` captures entire web pages including content below the fold.

**Deployment (1 tool)**

`deploy_website` publishes static sites to a public URL. This takes a built web application and makes it accessible on the internet.

**Slides (1 tool)**

`slides_generator` converts HTML presentations to PPTX format. This is the final step in the Slides agent workflow.

---

## The Universal Shell Pattern

All skills use the same generic tools. The `shell` tool that validates Excel files is identical to the `shell` tool that compiles LaTeX documents. What changes is the knowledge loaded into the context window.

When processing a spreadsheet request, the agent reads the XLSX skill documentation. This 925-line manual teaches compatibility rules, validation procedures, and styling conventions. The generic shell tool becomes an Excel specialist through context, not code.

See [../skills/README.md](../skills/README.md) for how skills transform generic tools into domain experts.

---

## Directory Structure

```
tools/
├── base-chat/          # 9 Base Chat tool docs
│   ├── web_search.md
│   ├── ipython.md
│   └── ...
├── ok-computer/        # 28 OK Computer tool docs
│   ├── browser_visit.md
│   ├── browser_click.md
│   ├── shell.md
│   └── ...
├── base-chat.json      # Base Chat tools schema
└── ok-computer.json    # OK Computer tools schema
```

---

## Tool Prefix Note

The "mshtools-" prefix in the original files stands for "Moonshot Tools". In this repository, we have removed the prefix for cleaner filenames while preserving the original tool names in the documentation.

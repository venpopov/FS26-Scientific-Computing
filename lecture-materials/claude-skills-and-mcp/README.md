# Week 2 — Skills & MCP Servers

Files in this folder:

- `LECTURE_PLAN.md` — timing, exercises, instructor cheat-sheet
- `slides.md` — Marp-compatible slide deck (`<!-- _class: exercise -->` and `<div class="gap">` mark exercise / open-discussion slides)

## Rendering the slides

**Option 1 — VS Code:** install the *Marp for VS Code* extension, open `slides.md`, hit the preview / export button.

**Option 2 — CLI:**

```bash
npx -y @marp-team/marp-cli slides.md -o slides.pdf
# or HTML:
npx -y @marp-team/marp-cli slides.md -o slides.html
# or live preview while editing:
npx -y @marp-team/marp-cli -s .
```

## Quick edits before class

- Slide *Resources* — drop in the class repo URL once you have one.
- Exercise A — pick which sample PDFs/CSVs go in the demo repo.
- Exercise B — if your students don't have `uvx`, swap `uvx mcp-server-time` for `npx -y @modelcontextprotocol/server-everything` or remove that row.

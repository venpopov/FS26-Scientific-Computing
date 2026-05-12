# Week 2 — Extending Claude Code: Skills & MCP Servers

**Audience:** Programming students. Last week: intro to Claude Code (chat, edits, basic tools).
**This week:** extending Claude with *skills* (instructions on demand) and *MCP servers* (external tools/data).
**Total time:** ~90 min (30 lecture + 30 exercises + 20 build-your-own + 10 wrap).

---

## Learning goals

By the end students can:

1. Explain the difference between a **skill** (a prompt-shaped instruction file) and an **MCP server** (a process exposing tools/resources).
2. Install a public skill, see it auto-trigger, and invoke it explicitly with `/skill-name`.
3. Read a `SKILL.md` and predict whether Claude will load it for a given task.
4. Write a minimal skill themselves (with Claude's help via `skill-creator`).
5. Add an MCP server to Claude Code and call one of its tools.
6. Decide which mechanism (skill vs MCP) suits a given problem.

---

## Pre-class checklist (send the night before)

- Claude Code installed and working (from last week).
- A free GitHub account (only needed for one optional MCP).
- `node` ≥ 20 and `uvx` or `npx` available (`npx --version` works = good).
- Clone the demo repo (a tiny throwaway repo with sample PDFs, CSVs, and a junk SQLite DB) — instructor provides URL.

---

## Lecture flow

| Time   | Block                                  | Mode                |
| ------ | -------------------------------------- | ------------------- |
| 0–10   | Recap + motivation: "why extend?"      | Slides + discussion |
| 10–25  | Skills: what, where, when, how         | Slides              |
| 25–45  | **Exercise A** — install + drive-test  | Hands-on            |
| 45–55  | Build-your-own skill with skill-creator | Live demo + hands-on |
| 55–75  | MCP servers: the protocol, mental model | Slides              |
| 75–85  | **Exercise B** — wire up an MCP        | Hands-on            |
| 85–90  | Wrap, when-to-use-which, homework      | Slides              |

---

## Exercise A — Install and drive-test a public skill

Use Anthropic's official skills repo: `https://github.com/anthropics/skills`.

**Recommended starter skills (all read-only, no secrets, no network):**

1. **`pdf`** — extract text and tables from a PDF. Safe: pure file processing.
2. **`docx`** / **`xlsx`** / **`pptx`** — read/write Office docs. Safe: local files only.
3. **`mcp-builder`** — meta-skill that teaches Claude to scaffold an MCP server. Safe: only writes files.

**Install path:** copy the chosen skill folder into `~/.claude/skills/<skill-name>/` so `SKILL.md` lives at `~/.claude/skills/<skill-name>/SKILL.md`.

**Drive-test tasks (give one to each student, or pairs):**

- Hand Claude `sample.pdf` and ask: "summarize page 3" → watch the `pdf` skill auto-trigger.
- Ask explicitly: `/pdf extract all tables from quarterly.pdf as CSV`.
- Inspect: ask Claude `which skills are loaded right now? which one would you use for X?`

**Discussion prompt:** open the skill's `SKILL.md`. What's in the frontmatter? What's in the body? When did Claude load the body?

---

## Build-your-own skill (live demo + 10-min student attempt)

Have students run:

```
/skill-creator make a skill called "uzh-citation" that, given a DOI, fetches the citation
in APA format and inserts it into the open document
```

Walk through the result: name, description, body, any allowed-tools. Discuss why the description must include trigger phrases like "DOI", "citation", "APA" — Claude only sees the description at trigger time.

---

## Exercise B — Wire up an MCP server

**Recommended starters (all stdio, no auth, no external accounts):**

| Server                                          | What it gives Claude                | Risk    |
| ----------------------------------------------- | ----------------------------------- | ------- |
| `@modelcontextprotocol/server-filesystem`       | Read/write files in a chosen dir    | Low (sandboxed to path) |
| `@modelcontextprotocol/server-fetch`            | HTTP GETs                            | Low |
| `@modelcontextprotocol/server-time`             | Timezone-aware now()                 | None |
| `@modelcontextprotocol/server-sqlite`           | Query a local SQLite DB              | Low |
| `context7` (already installed for many)         | Live library/API docs lookup         | None |

**Skip in class** (need auth or have wider blast radius): GitHub MCP, Slack MCP, Gmail MCP, computer-use.

**Install pattern:**

```
claude mcp add fs -- npx -y @modelcontextprotocol/server-filesystem /tmp/student-sandbox
claude mcp add time -- uvx mcp-server-time
```

Then in a new Claude Code session, ask: "list files in the sandbox" or "what's the current time in Tokyo?" and watch the tools light up.

**Discussion:** run `/mcp` to see the server status. Open `~/.claude.json` or `.mcp.json` and look at the actual config. What changes if the server dies?

---

## When to use which

| Need                                            | Reach for     |
| ----------------------------------------------- | ------------- |
| "Whenever the task looks like X, remember Y"    | **Skill**     |
| "Give Claude a new ability that requires code"  | **MCP**       |
| "Codify a workflow / checklist / output style"  | **Skill**     |
| "Access an external system (DB, API, app)"      | **MCP**       |
| "Bundle a domain expert prompt"                 | **Skill**     |

Quick mantra: **skills are knowledge; MCPs are capabilities.**

---

## Homework

1. Write one skill that automates something you actually did this week (≤ 30 lines of body).
2. Add one MCP server you'd use beyond class (e.g., GitHub MCP with a read-only PAT) and answer in a paragraph: what new tasks does it unlock?
3. Submit both via PR to your fork of the class repo.

---

## Instructor cheat-sheet — likely student questions

- **"Why didn't my skill trigger?"** → The description didn't match the task vocabulary, or another skill matched better. Show: `cat ~/.claude/skills/<name>/SKILL.md` and revise the description.
- **"Where's the skill's body until I use it?"** → Disk. Only the *name + description* enter context up front; the body loads when invoked. (Progressive disclosure.)
- **"Why don't I see my MCP tools?"** → Run `/mcp`. The server may have failed to start (bad path, missing binary, wrong syntax in the JSON).
- **"Can I share a skill with a teammate?"** → Yes: commit it under `.claude/skills/` in a repo, or publish as part of a plugin.
- **"Skill vs. CLAUDE.md?"** → CLAUDE.md is always loaded; skills load on demand. Use CLAUDE.md for *always-true* facts, skills for *sometimes-relevant* procedures.

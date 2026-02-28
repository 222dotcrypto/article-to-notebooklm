# Article → NotebookLM

Claude Code skill that loads web articles into Google NotebookLM and generates study materials — summaries, podcasts, slide decks, flashcards, quizzes, and mind maps.

## Features

- **Any URL** — articles, blog posts, documentation pages
- **Paywall fallback** — if direct URL fails, extracts text via WebFetch and loads as text source
- **Smart grouping** — multiple articles on the same topic go into one notebook
- **7 artifact types** — Briefing Doc, Study Guide, audio podcast, slide deck, flashcards, quiz, mind map
- **Downloads** — all artifacts saved locally to `~/article-to-notebooklm/downloads/`

## Prerequisites

- [NotebookLM MCP server](https://github.com/jxnl/notebooklm-mcp) — configured and authenticated
- Claude Code

## Installation

### Git clone (recommended)

```bash
# Global (all projects)
git clone https://github.com/222dotcrypto/article-to-notebooklm.git ~/.claude/skills/article-to-notebooklm

# Project-local
git clone https://github.com/222dotcrypto/article-to-notebooklm.git .claude/skills/article-to-notebooklm
```

### Manual

Copy `SKILL.md` to `~/.claude/skills/article-to-notebooklm/`.

## Usage

Trigger the skill in Claude Code by:

- Pasting an article URL — the skill activates automatically (non-YouTube links)
- Typing: `"analyze this article"`, `"load article into notebooklm"`
- Running: `/article-to-notebooklm <url>`

### What happens

1. Checks NotebookLM auth (auto-login if needed)
2. Adds URL as a source (falls back to text extraction if needed)
3. Creates a NotebookLM notebook
4. Asks what to generate: summary, study guide, podcast, slides, flashcards, quiz, or mind map
5. Downloads generated artifacts locally

## How It Works

```
Article URL → NotebookLM MCP (source_add as URL)
           → [fallback: WebFetch → source_add as text]
           → Studio (generate chosen artifacts)
           → Download locally
```

## File Structure

```
article-to-notebooklm/
├── SKILL.md        # Skill instructions (agent SOP)
└── README.md
```

## License

MIT

# ax-skill

A [Claude Code](https://claude.ai/code) skill that makes Claude build websites optimized for AI agents and LLMs from the start — not as an afterthought.

When active, Claude applies all AEO (Agent Experience Optimization) and GEO (Generative Engine Optimization) best practices throughout development: generating the right discovery files, structured data, semantic HTML, HTTP headers, and sitemap automatically, for any stack.

## What it covers

- `/llms.txt` and `/llms-full.txt` — machine-readable site summaries
- `/.well-known/agent.json` — A2A protocol agent card
- `robots.txt` — explicit rules for 30+ AI crawlers
- `/.well-known/security.txt` and `/.well-known/ai.txt`
- JSON-LD structured data (`WebSite`, `Organization`, `WebPage`, `BreadcrumbList`)
- Meta tags + Open Graph
- Security + AI discovery HTTP headers
- Sitemap generation
- OpenAPI / MCP endpoint guidance

Stack-aware patterns for **Next.js (App Router)**, **Astro**, and **Vercel**.

Validates with [ax-audit](https://github.com/lucioduran/ax-audit). Target: all checks ≥ 80, overall grade A.

## Install

**Project-level** (applies only to this project):

```bash
mkdir -p .claude/commands
curl -o .claude/commands/ax.md https://raw.githubusercontent.com/lucioduran/ax-skill/main/ax.md
```

**Global** (applies to all your projects):

```bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/ax.md https://raw.githubusercontent.com/lucioduran/ax-skill/main/ax.md
```

Or clone and symlink:

```bash
git clone https://github.com/lucioduran/ax-skill ~/.claude/ax-skill
ln -s ~/.claude/ax-skill/ax.md ~/.claude/commands/ax.md
```

## Usage

In any Claude Code session, type:

```
/ax
```

Claude enters AX mode and applies all standards to the current project. Works best when used at the start of a project, but can be applied to existing projects to fill gaps.

## Validate your score

```bash
npx ax-audit@latest https://yoursite.com
```

## Related

- [ax-audit](https://github.com/lucioduran/ax-audit) — the CLI that audits AEO/GEO compliance

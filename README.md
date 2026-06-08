# ax-skill

A [Claude Code](https://claude.ai/code) skill that makes Claude build websites fully optimized for AI agents and LLMs from the start, not as an afterthought.

When active, Claude applies all AEO (Agent Experience Optimization) and GEO (Generative Engine Optimization) best practices throughout development: generating discovery files, structured data, semantic HTML, HTTP headers, AI meta tags, and sitemap automatically, for any stack. It covers every signal in the [ax-audit](https://github.com/lucioduran/ax-audit) scoring system.

## What it covers

All 14 ax-audit checks:

- `/llms.txt` and `/llms-full.txt` — machine-readable site summaries with quality standards
- `/.well-known/agent.json` — A2A protocol agent card with actionable skills
- `robots.txt` — explicit rules for 30+ AI crawlers across training, search, and fetching buckets
- `/.well-known/security.txt` and `/.well-known/ai.txt`
- `/.well-known/openapi.json` — API discoverability spec
- `/.well-known/mcp.json` — Model Context Protocol tool integration
- JSON-LD structured data — `@graph` philosophy, `sameAs` grounding, Article, FAQPage, HowTo, Product schemas
- HTTP headers — security headers, Link header, CORS on well-known resources, HSTS preload
- HTML rendering — SPA trap detection, semantic landmarks, server-rendered content requirements
- SEO basics — title/description length, canonical, `<html lang>`, hreflang with x-default
- Meta tags — `ai:summary`, `ai:content_type`, `ai:author`, `ai:api`, `ai:agent_card`, `rel="alternate"`, `rel="me"`, OpenGraph, Twitter Card
- TLS/HTTPS — 301 redirect, HSTS preload path
- Sitemap — priority strategy, dynamic generation
- Well-known AI files — `ai.txt`, `agents.json`, `ai-plugin.json`, `nlweb.json`

Stack-aware patterns for **Next.js (App Router)**, **Astro**, and **Vercel**.

## Install

**Global** (applies to all your projects):

```bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/ax.md https://raw.githubusercontent.com/lucioduran/ax-skill/main/ax.md
```

**Project-level** (applies only to this project):

```bash
mkdir -p .claude/commands
curl -o .claude/commands/ax.md https://raw.githubusercontent.com/lucioduran/ax-skill/main/ax.md
```

Or clone and symlink:

```bash
git clone https://github.com/lucioduran/ax-skill ~/.claude/ax-skill
ln -s ~/.claude/ax-skill/ax.md ~/.claude/commands/ax.md
```

## Usage

In any Claude Code session:

```
/ax
```

Claude enters AX mode and applies all standards to the current project. Works best at the start of a project. Can also be applied to existing projects to fill gaps.

## Validate your score

```bash
npx ax-audit@latest https://yoursite.com
```

Target: every check scores 80 or above. Overall grade A (90+).

## CI/CD integration

```bash
# Fail the build on any check below 80
npx ax-audit@latest https://yoursite.com --format json | \
  node -e "const r=require('/dev/stdin');const f=r.results.filter(x=>x.score<80);if(f.length){console.error(f.map(x=>x.id+':'+x.score).join(', '));process.exit(1)}"

# Fail on regression from baseline
npx ax-audit@latest https://yoursite.com --baseline .ax-baseline.json --fail-on-regression
```

## Related

- [ax-audit](https://github.com/lucioduran/ax-audit) — the CLI that audits AEO/GEO compliance

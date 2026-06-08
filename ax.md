# AX: Agent & LLM Web Optimization

You are now in **AX mode**. Every web project you work on must be fully optimized for AI agents and LLMs. Apply these standards throughout development, not as a post-build checklist, but as the default way you build.

**Writing rule:** Never use em dashes (—) in any content you generate, in any file, comment, or prose. Use a colon, comma, or period instead.

## Core Principle

Websites have two consumers: humans and machines. Both get a perfect experience. In AX mode, AI agents (crawlers, search bots, reasoning models, autonomous agents) are first-class users. Every architectural decision you make accounts for machine readability.

---

## Mandatory Files

Generate these files early in development and keep them accurate as the project evolves. Replace every placeholder with real content from the actual project.

### 1. `/llms.txt`

Required format per the [llmstxt.org](https://llmstxt.org) spec:

```markdown
# Site Name

> One clear paragraph describing what this site does, written for an AI agent to understand at a glance. Be specific about the domain, purpose, and audience.

## About
- [About Us](https://example.com/about): Description of what this page contains
- [Team](https://example.com/team): Who works here

## Documentation
- [Docs](https://example.com/docs): Full documentation index
- [API Reference](https://example.com/api): API endpoints and usage

## Key Pages
- [Page Name](https://example.com/page): What an agent would find here
```

Rules:
- First line MUST be `# Site Name` (H1)
- Second non-blank line MUST be a blockquote `> description`
- Sections organized with `##` headings
- Every link needs a meaningful description after the colon
- Content-Type: `text/plain` or `text/markdown`
- Minimum 200 characters

### 2. `/llms-full.txt`

Same format as `llms.txt` but comprehensive: include all pages, full API documentation, complete site structure. This is what agents use when they need the full picture.

### 3. `/.well-known/agent.json`

[A2A (Agent-to-Agent) protocol](https://google.github.io/A2A/) card:

```json
{
  "name": "Site Name",
  "description": "What this site/service does and what agents can accomplish with it",
  "url": "https://example.com",
  "protocolVersion": "0.2.0",
  "skills": [
    {
      "id": "search",
      "description": "Search the site content by keyword or topic"
    }
  ],
  "capabilities": {
    "streaming": false,
    "pushNotifications": false
  },
  "authentication": {
    "schemes": ["none"]
  },
  "documentationUrl": "https://example.com/docs"
}
```

Rules:
- `name`, `description`, `url`, `skills` are required fields
- `url` MUST match the site's origin exactly
- Each skill needs both `id` and `description`
- Content-Type: `application/json`

### 4. `/robots.txt`

Declare explicit rules for all major AI crawlers. Default is allow unless there are specific reasons to restrict:

```
User-agent: *
Allow: /

# AI Training
User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Claude-Web
Allow: /

User-agent: Anthropic-AI
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: CCBot
Allow: /

User-agent: Bytespider
Allow: /

User-agent: Meta-ExternalAgent
Allow: /

User-agent: Meta-ExternalFetcher
Allow: /

User-agent: Cohere-AI
Allow: /

User-agent: Applebot-Extended
Allow: /

User-agent: AI2Bot
Allow: /

User-agent: DeepSeek-AI
Allow: /

User-agent: MistralAI-User
Allow: /

# AI Search & Answer Engines
User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: Claude-SearchBot
Allow: /

User-agent: Claude-User
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Perplexity-User
Allow: /

User-agent: DuckAssistBot
Allow: /

User-agent: GeminiBot
Allow: /

User-agent: KagiBot
Allow: /

# AI Fetching Agents
User-agent: FirecrawlAgent
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Rules:
- MUST declare explicit rules for at minimum: `GPTBot`, `ClaudeBot`, `ChatGPT-User`, `Claude-SearchBot`, `Google-Extended`, `PerplexityBot`, `OAI-SearchBot`, `CCBot`
- Must include `Sitemap:` directive
- Content-Type: `text/plain`

### 5. `/.well-known/security.txt`

```
Contact: mailto:security@example.com
Expires: 2027-12-31T23:59:00.000Z
Preferred-Languages: en
Canonical: https://example.com/.well-known/security.txt
Policy: https://example.com/security-policy
```

Rules:
- `Contact` and `Expires` are required
- `Expires` must be a future ISO 8601 datetime
- Keep `Canonical` pointing to `/.well-known/security.txt`

### 6. `/.well-known/ai.txt`

Opt-in/opt-out for AI training ([Spawning AI](https://site.spawning.ai/spawning-ai-txt) format):

```
# AI Training Policy
Allow: GPTBot
Allow: ClaudeBot
Allow: Google-Extended
Allow: CCBot
Allow: PerplexityBot

Policy: https://example.com/ai-policy
Contact: mailto:ai@example.com
```

### 7. `/sitemap.xml`

Standard XML sitemap. Every public URL must be listed. For dynamic sites, generate programmatically using the stack patterns below.

---

## Structured Data

Every page needs JSON-LD in `<head>`. Use `@graph` to bundle multiple entities. The validator at [schema.org](https://validator.schema.org/) must pass without errors.

### Root layout / homepage

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebSite",
      "@id": "https://example.com/#website",
      "url": "https://example.com",
      "name": "Site Name",
      "description": "What the site does",
      "publisher": { "@id": "https://example.com/#organization" }
    },
    {
      "@type": "Organization",
      "@id": "https://example.com/#organization",
      "name": "Organization Name",
      "url": "https://example.com",
      "logo": "https://example.com/logo.png",
      "sameAs": ["https://twitter.com/handle", "https://github.com/org"]
    }
  ]
}
```

### Per-page additions

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebPage",
      "@id": "https://example.com/page/#webpage",
      "url": "https://example.com/page/",
      "name": "Page Title",
      "description": "Page description",
      "isPartOf": { "@id": "https://example.com/#website" }
    },
    {
      "@type": "BreadcrumbList",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
        { "@type": "ListItem", "position": 2, "name": "Page", "item": "https://example.com/page" }
      ]
    }
  ]
}
```

---

## Meta Tags

Every page in `<head>`:

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Page Title | Site Name</title>
<meta name="description" content="Specific page description, 120–160 characters" />
<link rel="canonical" href="https://example.com/page/" />

<meta property="og:title" content="Page Title" />
<meta property="og:description" content="Page description" />
<meta property="og:type" content="website" />
<meta property="og:url" content="https://example.com/page/" />
<meta property="og:image" content="https://example.com/og-image.png" />
<meta property="og:site_name" content="Site Name" />

<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Page Title" />
<meta name="twitter:description" content="Page description" />
<meta name="twitter:image" content="https://example.com/og-image.png" />
```

Rules:
- `og:image` must exist and be 1200×630px minimum
- Every page needs unique `title` and `description`
- `canonical` must be present on every page
- Description: 120–160 characters

---

## HTTP Headers

### Security headers (required)

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'
```

### AI discovery headers

```
Link: </llms.txt>; rel="ai-content-policy"
Link: </.well-known/agent.json>; rel="agent-card"
```

---

## Stack Patterns

Detect the stack from `package.json`, config files, and framework dependencies, then apply the right patterns.

### Next.js (App Router)

**`next.config.ts`:**

```ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
          {
            key: 'Link',
            value: '</llms.txt>; rel="ai-content-policy", </.well-known/agent.json>; rel="agent-card"',
          },
        ],
      },
    ];
  },
};

export default nextConfig;
```

**`app/robots.ts`:**

```ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  const base = 'https://example.com';
  const aiCrawlers = [
    'GPTBot', 'ClaudeBot', 'Claude-Web', 'Anthropic-AI',
    'Google-Extended', 'CCBot', 'OAI-SearchBot', 'ChatGPT-User',
    'Claude-SearchBot', 'Claude-User', 'PerplexityBot', 'GeminiBot',
    'FirecrawlAgent', 'Bytespider', 'Cohere-AI', 'DeepSeek-AI',
  ];

  return {
    rules: [
      { userAgent: '*', allow: '/' },
      ...aiCrawlers.map((userAgent) => ({ userAgent, allow: '/' })),
    ],
    sitemap: `${base}/sitemap.xml`,
  };
}
```

**`app/sitemap.ts`:**

```ts
import type { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  const base = 'https://example.com';
  return [
    { url: base, lastModified: new Date(), changeFrequency: 'monthly', priority: 1 },
    // add all routes dynamically
  ];
}
```

**`app/layout.tsx` metadata:**

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: { default: 'Site Name', template: '%s | Site Name' },
  description: 'Site description optimized for AI agents and users',
  metadataBase: new URL('https://example.com'),
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://example.com',
    siteName: 'Site Name',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'Site Name' }],
  },
  twitter: { card: 'summary_large_image' },
  robots: { index: true, follow: true },
};
```

**`app/llms.txt/route.ts`:**

```ts
import { NextResponse } from 'next/server';

const content = `# Site Name

> Description of what this site does for AI agents.

## About
- [About](https://example.com/about): What the site is

## Documentation
- [Docs](https://example.com/docs): Technical documentation
`;

export function GET() {
  return new NextResponse(content, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
```

Static files go in `public/`:
- `public/llms-full.txt`
- `public/.well-known/agent.json`
- `public/.well-known/security.txt`
- `public/.well-known/ai.txt`

### Astro

Static files go in `public/`:
- `public/llms.txt`
- `public/llms-full.txt`
- `public/robots.txt`
- `public/.well-known/agent.json`
- `public/.well-known/security.txt`
- `public/.well-known/ai.txt`

Dynamic `llms.txt` via `src/pages/llms.txt.ts`:

```ts
export async function GET() {
  const content = `# Site Name\n\n> Description.\n\n## Pages\n- [Home](https://example.com): Home page`;
  return new Response(content, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
```

Sitemap via `src/pages/sitemap.xml.ts`:

```ts
export async function GET() {
  const pages = ['', '/about', '/docs'];
  const base = 'https://example.com';
  const xml = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${pages.map((p) => `  <url><loc>${base}${p}</loc><changefreq>monthly</changefreq></url>`).join('\n')}
</urlset>`;
  return new Response(xml, { headers: { 'Content-Type': 'application/xml' } });
}
```

Headers via `astro.config.mjs` (dev only; use `vercel.json` or hosting config for production):

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://example.com',
});
```

### Vercel (`vercel.json`)

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Strict-Transport-Security", "value": "max-age=31536000; includeSubDomains" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
        { "key": "Link", "value": "</llms.txt>; rel=\"ai-content-policy\", </.well-known/agent.json>; rel=\"agent-card\"" }
      ]
    }
  ]
}
```

---

## HTML & Content

**Semantic structure:** Use correct HTML5 semantic elements: `<main>`, `<article>`, `<section>`, `<aside>`, `<nav>`, `<header>`, `<footer>`. Never use `<div>` where a semantic element fits.

**Headings:** One `<h1>` per page. Hierarchical `<h2>` → `<h3>`, never skip levels. Headings describe content, not decoration.

**Content first:** Critical content in initial HTML, not JavaScript-rendered. Many agents and crawlers don't execute JS.

**Links:** `<a>` elements must have descriptive text. Never "click here" or "read more" without context.

**Images:** Every `<img>` needs meaningful `alt` text.

**Language:** `<html lang="en">` (or correct locale).

---

## API & MCP

If the project exposes an API, add:
- `openapi.json` or `openapi.yaml` at `/openapi.json`
- Required fields: `info.title`, `info.description`, `info.version`, full `paths`
- Link from `llms.txt` and `agent.json`

If the project supports MCP (Model Context Protocol):
- Expose endpoint at `/mcp` or `/.well-known/mcp.json`
- List it in `agent.json` under `capabilities`

---

## Process

When starting or working on a web project in AX mode:

1. **Detect stack**: read `package.json`, framework config files, dependencies
2. **Generate mandatory files**: create all files from the Mandatory Files section using real site content, not placeholders
3. **Add structured data**: JSON-LD for every layout and page type
4. **Configure headers**: security + discovery headers via the detected stack's config
5. **Write semantic HTML**: follow Content guidelines for every component
6. **Keep files current**: when adding routes or features, update `llms.txt`, `sitemap.xml`, and `agent.json` skills array

---

## Validation

After setup or any significant feature, validate with:

```bash
npx ax-audit@latest <site-url>
```

Target: all checks ≥ 80, overall grade A (≥ 90). Fix regressions before marking the task done.

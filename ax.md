---
name: ax
description: The gold standard for building websites that AI agents, LLMs, and autonomous crawlers can fully read, understand, and act on. Covers every signal in the ax-audit scoring system and the reasoning behind each one.
---

# AX: Agent & LLM Web Optimization

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to help you build for the agent web. My knowledge covers every signal that separates sites AI cites from sites AI ignores. Run `npx ax-audit@latest <url>` at any point to measure where you stand.

Do not provide any other information until the user asks a question.

---

You are an engineer who builds for two consumers simultaneously: humans and machines. In a world where AI agents browse, summarize, cite, and interact with websites on behalf of users, the sites that win are the ones machines can fully understand. You treat agent-readability as a first-class requirement from line one, not a checklist you add before launch.

**Writing rule:** Never use em dashes (—) in any content, prose, code comments, or generated files. Use a colon, comma, or period instead.

---

## Core Philosophy

### The citation economy

Every time a user asks ChatGPT, Claude, or Perplexity something your site covers, an agent is running a pipeline: retrieve candidate sources, rank them by signal quality, synthesize an answer, cite the winners. Your site's AEO signals determine whether you land in that citation set or get skipped.

Traditional SEO optimized for Googlebot. AEO optimizes for reasoning models, retrieval pipelines, and autonomous agents. The audience shifted. The signals shifted. Most of the web has not caught up.

This is not a future concern. It is happening today on every query.

### Two consumers, one codebase

A website has exactly two types of consumers:

- **Humans** who use a browser, see CSS, run JavaScript, and navigate with deliberate intent
- **Machines** that issue raw HTTP requests, parse structured signals, and build representations without rendering anything

Both need a perfect experience. A site that looks great in Chrome but returns an empty shell to a non-JS crawler is half-built. A site with a beautiful `llms.txt` but no server-rendered content gives agents an incomplete picture.

Every decision in AX mode accounts for both. They are not in conflict. Server-rendered HTML is better for both. Semantic markup serves both. Clear headings and descriptions help humans navigating and machines parsing equally.

### Explicit beats implicit everywhere

AI agents do not infer. They read what you declare. A `robots.txt` with only a wildcard rule forces agents to guess. An `agent.json` with an empty skills array gives nothing to work with. JSON-LD with one entity type when three are relevant leaves the knowledge graph incomplete.

Every place where you could explicitly declare something, you should. The cost is one file or one field. The benefit is that agents act on your site with confidence instead of ambiguity.

### The compounding effect

> "All those unseen details combine to produce something that's just stunning, like a thousand barely audible voices all singing in tune." — Paul Graham

No single AEO signal wins citations. The combination of excellent `llms.txt`, explicit robots.txt rules, server-rendered content, complete structured data, correct headers, and AI meta tags creates a signal profile that agents recognize as reliable. Each check you pass increases that profile. Missing a few makes you look like every other unmaintained site.

---

## What an Agent Actually Experiences

This is the most important section to internalize. Before any rule makes sense, understand what actually happens when an agent arrives at a site.

### A site that fails

GPTBot arrives at a typical SPA. Here is the exact sequence:

1. Sends `GET /` with `User-Agent: GPTBot`
2. Reads response headers. No `Strict-Transport-Security`. No `Link` header. No security signal.
3. Reads HTML body. Finds `<div id="root"></div>` and a 400KB JavaScript bundle. Zero visible text. The page does not exist to this agent.
4. Tries `/robots.txt`. Finds `User-agent: * / Allow: /`. No explicit GPTBot rule. Infers access.
5. Tries `/llms.txt`. Gets 404.
6. Tries `/.well-known/agent.json`. Gets 404.
7. Tries `/.well-known/mcp.json`. Gets 404.
8. Has visited the site and learned almost nothing. No content. No capabilities. No identity.

The site is invisible to agents regardless of how good its design is.

### A site that wins

GPTBot arrives at an AX-optimized site:

1. Sends `GET /` with `User-Agent: GPTBot`
2. Reads response headers. Sees `Strict-Transport-Security`. Sees `Link: </llms.txt>; rel="ai-content-policy", </.well-known/agent.json>; rel="agent-card"`. Knows exactly where to look.
3. Reads HTML body. Finds `<main>` with 900 words of server-rendered content. One `<h1>` declaring the page topic. JSON-LD in `<head>` with WebSite, Organization, and BreadcrumbList entities connected via `@id`.
4. Reads `<meta name="ai:summary" content="...">` and `<link rel="alternate" href="/llms.txt">`.
5. Fetches `/llms.txt`. Gets a structured description of the entire site with every major section linked and described.
6. Fetches `/.well-known/agent.json`. Gets a complete A2A card with specific skills it can act on.
7. Fetches `/robots.txt`. Finds explicit `GPTBot / Allow: /` rule alongside 28 other AI crawlers.

Five HTTP requests. Complete picture. High-confidence citation candidate.

---

## The AEO Priority Framework

Not all checks are equal. Build in this order when resources are limited:

| Check | Weight | What it unlocks |
| --- | --- | --- |
| llms.txt | 11 | Primary structured entry point for every LLM reading your site |
| robots.txt | 11 | Controls whether AI crawlers can reach you at all |
| HTML Rendering | 9 | Most crawlers don't execute JS. Content not in HTML doesn't exist |
| Structured Data | 9 | Machine vocabulary for your entities and content types |
| HTTP Headers | 9 | Security signals, AI discovery pointers, CORS for well-known resources |
| agent.json (A2A) | 7 | Agent-to-agent protocol card, capability declaration |
| MCP | 7 | Direct tool integration for reasoning models |
| SEO Basics | 7 | Title, description, canonical, lang: identity signals every agent uses |
| security.txt | 6 | Trust signal for automated systems |
| Meta Tags | 6 | AI meta tags, rel="alternate", OpenGraph, Twitter Card |
| OpenAPI | 6 | API discoverability for agents that call endpoints |
| TLS/HTTPS | 5 | Baseline requirement. No secure connection means no agent trust |
| Sitemap | 4 | Complete crawl surface map |
| Well-Known AI | 3 | Emerging consent and capability signals |

Build the top six first. Excellent `llms.txt` and `robots.txt` with server-rendered HTML will outperform perfect structured data on a JS-only shell every time.

---

## Quick Start: First 30 Minutes on a New Project

Before writing any feature code, do this. These actions take 30 minutes and cover the highest-weight checks.

```bash
# 1. Audit what you're starting from
npx ax-audit@latest https://your-staging-url.com

# 2. Create the file structure
mkdir -p public/.well-known
touch public/robots.txt
touch public/llms.txt
touch public/llms-full.txt
touch public/.well-known/agent.json
touch public/.well-known/security.txt
touch public/.well-known/ai.txt
```

Then fill each file with real content using the specs below. 30 minutes. Check passes for robots.txt, llms.txt, agent.json, security.txt, ai.txt, and well-known-ai are all green.

---

## llms.txt: The Agent's First Impression

### What it actually does

`/llms.txt` is the file AI agents read to understand your site before crawling it. It is a README written for a reasoning model: it tells the agent what the site is, what it contains, and where to look for specific things. When an LLM is deciding whether your site is a relevant source for a query, the quality of `llms.txt` directly affects that decision.

The spec is minimal by design: Markdown, starting with an H1, followed by a blockquote description, organized with section headings and links. The minimalism is intentional. Agents need structured, scannable, machine-readable text, not HTML.

### Good vs great

| Bad | Good |
| --- | --- |
| `> We make software for businesses.` | `> Real-time analytics for e-commerce teams. Tracks conversion funnels, cohort retention, and revenue attribution across Shopify, WooCommerce, and custom storefronts. Used by 2,000+ stores processing $50M+ monthly GMV.` |
| `- [Home](https://example.com)` | `- [Dashboard Overview](https://example.com/docs/dashboard): How to read the main analytics dashboard, including funnel visualization and cohort comparison views` |
| `- [API](https://example.com/api)` | `- [REST API Reference](https://example.com/api): Query analytics data programmatically. Covers authentication, rate limits, and all 23 endpoints with request/response examples.` |

The difference is specificity. The agent reading the good version knows exactly what the site is for, who uses it, and what it will find on each page before fetching a single URL.

### The description blockquote test

The blockquote is the most important field. Apply this test: could this description apply to 100 other sites, or only yours? If it could apply to 100 others, rewrite it. A good description answers: what does this site do, who is it for, what is the scale or scope, what makes it different?

Never write "Welcome to our website" or "We help businesses grow." Write what you would tell a journalist in the first sentence of a pitch email.

### The complete spec

```markdown
# Site Name

> Specific description: domain, audience, scope, differentiator. This is what LLMs read to decide relevance.

## Core Product
- [Feature Name](https://example.com/feature): What an agent will find and can learn here

## Documentation
- [Getting Started](https://example.com/docs/start): First steps, setup, and initial configuration
- [API Reference](https://example.com/api): All endpoints, authentication, rate limits, and examples

## About
- [About](https://example.com/about): Organization background and mission
- [Pricing](https://example.com/pricing): Plans with feature and limit comparisons
```

**Rules ax-audit enforces:**
- First line must be `# Site Name` (H1, no exceptions)
- Second non-blank line must be `> description` (blockquote, not paragraph)
- At least one `##` section heading
- At least one Markdown link `[text](url)`
- Minimum 100 characters (aim for 500+)
- Content-Type must be `text/plain` or `text/markdown`

### Implementation

**Next.js App Router** (dynamic, so content stays current):

```ts
// app/llms.txt/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const content = `# Site Name

> Specific description of what this site does, who it serves, and what makes it useful.

## Core Product
- [Feature](https://example.com/feature): What agents will find here

## Documentation
- [Docs](https://example.com/docs): Technical documentation and guides
- [API](https://example.com/api): API reference with all endpoints
`;

  return new NextResponse(content, {
    headers: {
      'Content-Type': 'text/plain; charset=utf-8',
      'X-Robots-Tag': 'noindex',
    },
  });
}
```

The `X-Robots-Tag: noindex` keeps this file out of Google search results while remaining fully accessible to AI agents that fetch it directly.

**Astro:**

```ts
// src/pages/llms.txt.ts
export async function GET() {
  const content = `# Site Name\n\n> Description.\n\n## Pages\n- [Home](https://example.com): Home`;
  return new Response(content, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
```

### llms-full.txt: go comprehensive

`/llms-full.txt` is the expanded version. Include every route, all API endpoint documentation, every major feature documented in full. Some agents, when they have permission to read deeply, prefer the full version. Treat it as the machine-readable index of your entire site.

---

## robots.txt: The Gatekeeper

### The wildcard trap

Most sites have `User-agent: * / Allow: /` and consider it done. This is inadequate for two reasons.

First, it gives AI crawlers no explicit signal of intent. Some crawlers prefer explicit permission over inferring from the wildcard. Second, any future `Disallow` rule under the wildcard group accidentally blocks crawlers you did not intend to block. Explicit rules are immune to this.

The wildcard is a fallback. Explicit rules are a declaration. Declare your intent for every major AI agent.

### The three buckets

| Bucket | Purpose | Examples |
| --- | --- | --- |
| Training | Fetch content to build model training datasets | GPTBot, ClaudeBot, Google-Extended, CCBot, Bytespider |
| Search & answer | Fetch live content to answer user queries | OAI-SearchBot, ChatGPT-User, PerplexityBot, GeminiBot |
| Fetching agents | On-demand retrieval for user or automated workflows | FirecrawlAgent, Bingbot |

You need explicit rules for all three. Blocking training crawlers affects whether your content ends up in future models. Blocking search crawlers affects whether AI answer engines cite you for live queries. Both have consequences.

### The complete robots.txt

```
User-agent: *
Allow: /

# Training crawlers
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

User-agent: cohere-training-data-crawler
Allow: /

User-agent: Applebot-Extended
Allow: /

User-agent: Amazonbot
Allow: /

User-agent: AI2Bot
Allow: /

User-agent: AI2Bot-Dolma
Allow: /

User-agent: DeepSeek-AI
Allow: /

User-agent: MistralAI-User
Allow: /

User-agent: Diffbot
Allow: /

# Search and answer engines
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

User-agent: Google-CloudVertexBot
Allow: /

User-agent: KagiBot
Allow: /

User-agent: YouBot
Allow: /

User-agent: PhindBot
Allow: /

# Fetching agents
User-agent: FirecrawlAgent
Allow: /

Sitemap: https://example.com/sitemap.xml
```

**Next.js App Router:**

```ts
// app/robots.ts
import type { MetadataRoute } from 'next';

const AI_CRAWLERS = [
  // Training
  'GPTBot', 'ClaudeBot', 'Claude-Web', 'Anthropic-AI',
  'Google-Extended', 'CCBot', 'Bytespider',
  'Meta-ExternalAgent', 'Meta-ExternalFetcher',
  'Cohere-AI', 'cohere-training-data-crawler',
  'Applebot-Extended', 'Amazonbot', 'AI2Bot', 'AI2Bot-Dolma',
  'DeepSeek-AI', 'MistralAI-User', 'Diffbot',
  // Search and answer
  'OAI-SearchBot', 'ChatGPT-User', 'Claude-SearchBot', 'Claude-User',
  'PerplexityBot', 'Perplexity-User', 'DuckAssistBot',
  'GeminiBot', 'Google-CloudVertexBot', 'KagiBot', 'YouBot', 'PhindBot',
  // Fetching
  'FirecrawlAgent',
];

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      { userAgent: '*', allow: '/' },
      ...AI_CRAWLERS.map((userAgent) => ({ userAgent, allow: '/' })),
    ],
    sitemap: 'https://example.com/sitemap.xml',
  };
}
```

The minimum required core (ax-audit fails without these eight): `GPTBot`, `ClaudeBot`, `ChatGPT-User`, `Claude-SearchBot`, `Google-Extended`, `PerplexityBot`, `OAI-SearchBot`, `CCBot`.

---

## agent.json: Your A2A Card

### What A2A actually is

The Agent-to-Agent protocol is Google's open standard for how AI agents discover and communicate with web services. `/.well-known/agent.json` is your agent card: it declares what your site can do, how to authenticate, and what skills agents can invoke.

AI orchestration systems read `agent.json` to decide whether to include your site as a tool in their workflow. A well-formed agent card gets you into agent toolchains. A missing one means agents skip you or must infer capabilities from raw HTML.

### Skills that get used vs skills that get ignored

The `skills` array is the entire point of `agent.json`. Vague skills are not useful. Specific skills get invoked.

| Bad skill | Good skill |
| --- | --- |
| `{ "id": "browse", "description": "Browse the site" }` | `{ "id": "search-docs", "description": "Search technical documentation by keyword, feature name, or API endpoint. Returns ranked results with page titles, descriptions, and direct URLs." }` |
| `{ "id": "api", "description": "Use the API" }` | `{ "id": "get-pricing", "description": "Retrieve current pricing plans with per-plan feature lists, usage limits, and billing cycle details." }` |

Each skill description should answer: what specific query or task would an agent use this skill for?

### The URL origin requirement

The `url` field must match the site's origin exactly. If you audit `https://example.com` but `agent.json` has `"url": "https://www.example.com"`, ax-audit flags it as a mismatch. Agents use the URL field to confirm they are communicating with the authoritative agent for a given domain.

### CORS: the silent failure

`/.well-known/agent.json` must return `Access-Control-Allow-Origin: *`. Without it, browser-based AI agents get a CORS error. The file can exist and be perfectly valid JSON and still be completely inaccessible. ax-audit checks for this. It is the most commonly missed configuration.

```json
{
  "name": "Site Name",
  "description": "What agents can accomplish with this site. Be specific about capabilities.",
  "url": "https://example.com",
  "protocolVersion": "0.2.0",
  "skills": [
    {
      "id": "search",
      "description": "Search site content by keyword or topic. Returns titles, descriptions, and URLs of matching pages."
    },
    {
      "id": "get-docs",
      "description": "Retrieve technical documentation for a specific feature, API endpoint, or integration."
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

Serve this as `public/.well-known/agent.json` with:

```
Content-Type: application/json
Access-Control-Allow-Origin: *
```

---

## Structured Data: The Knowledge Graph

### Why JSON-LD beats everything

Microdata and RDFa embed schema.org markup in HTML, coupling it tightly to your markup structure. JSON-LD lives in a `<script>` tag in `<head>`, independent of layout. When your HTML structure changes, your structured data is untouched. When agents parse your page, JSON-LD is immediately available without walking the DOM.

Always use JSON-LD. Never use microdata.

### The @graph philosophy

The `@graph` array lets you declare multiple entities in a single block and link them by `@id`. Without `@graph`, your Organization is disconnected from your WebSite. Your WebPage does not know it belongs to your site. Entities are islands. With `@graph`, they form a connected knowledge graph that LLMs can traverse and reason over.

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
      "sameAs": [
        "https://twitter.com/handle",
        "https://linkedin.com/company/name",
        "https://github.com/org",
        "https://en.wikipedia.org/wiki/Organization_Name"
      ]
    }
  ]
}
```

### The sameAs property and LLM grounding

`sameAs` is how you connect your entities to the broader knowledge graph that LLMs were trained on. When your Organization's `sameAs` includes the Wikipedia, Wikidata, and LinkedIn URLs, LLMs can ground your entity against their pre-existing knowledge of your company. They no longer have to treat you as an unknown entity.

Include `sameAs` for every real-world entity: organizations, people, locations, products. Link to Wikipedia, Wikidata (`https://www.wikidata.org/wiki/Q...`), Crunchbase, LinkedIn, and any authoritative source where the entity has a profile.

### Per-page structured data

Every page needs a WebPage entity and a BreadcrumbList. The root layout handles WebSite and Organization. Individual pages extend the graph:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebPage",
      "@id": "https://example.com/docs/setup/#webpage",
      "url": "https://example.com/docs/setup/",
      "name": "Setup Guide",
      "description": "How to install and configure the product",
      "isPartOf": { "@id": "https://example.com/#website" },
      "datePublished": "2024-01-15",
      "dateModified": "2025-03-10"
    },
    {
      "@type": "BreadcrumbList",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
        { "@type": "ListItem", "position": 2, "name": "Docs", "item": "https://example.com/docs" },
        { "@type": "ListItem", "position": 3, "name": "Setup", "item": "https://example.com/docs/setup" }
      ]
    }
  ]
}
```

`datePublished` and `dateModified` are temporal signals. LLMs use them to assess content freshness. A page with `dateModified` from last month ranks higher for freshness than one with no date. Always include both.

### Content-type specific schemas

Beyond WebPage, apply the correct schema type for the content:

| Content type | Schema type | Key fields |
| --- | --- | --- |
| Blog post, article | `Article` | `headline`, `author`, `datePublished`, `dateModified`, `image` |
| Documentation page | `TechArticle` | `headline`, `proficiencyLevel`, `dependencies` |
| FAQ page | `FAQPage` | `mainEntity` array of `Question` with `acceptedAnswer` |
| How-to guide | `HowTo` | `name`, `step` array with `HowToStep` |
| Product page | `Product` | `name`, `description`, `offers`, `aggregateRating` |
| Person profile | `Person` | `name`, `jobTitle`, `affiliation`, `sameAs` |
| Event | `Event` | `name`, `startDate`, `location`, `organizer` |

**Article example:**

```json
{
  "@type": "Article",
  "@id": "https://example.com/blog/post/#article",
  "headline": "How to Configure Zero-Downtime Deployments",
  "description": "Step-by-step guide for configuring rolling deployments with health checks",
  "datePublished": "2025-01-10",
  "dateModified": "2025-05-20",
  "author": {
    "@type": "Person",
    "@id": "https://example.com/team/alice/#person",
    "name": "Alice Chen",
    "url": "https://example.com/team/alice"
  },
  "publisher": { "@id": "https://example.com/#organization" },
  "image": "https://example.com/blog/post/cover.png",
  "isPartOf": { "@id": "https://example.com/#website" }
}
```

**FAQPage example** (extremely high citation value for AI answer engines):

```json
{
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How does billing work?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "You are billed monthly based on the plan you select. The Starter plan is $29/month for up to 10,000 events. Upgrade or downgrade at any time."
      }
    }
  ]
}
```

FAQPage structured data is one of the highest-signal content types for AI answer engines. When a user asks a question that matches one of your FAQ entries, the agent has a directly usable answer. Use it on pricing pages, support pages, and product pages.

### Validation is not optional

Every JSON-LD block must pass [validator.schema.org](https://validator.schema.org/) without errors. A syntax error silently invalidates the entire block. ax-audit checks for invalid JSON. ax-audit does not catch schema violations. Use both.

**Next.js JSON-LD injection:**

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@graph': [
      {
        '@type': 'WebSite',
        '@id': 'https://example.com/#website',
        url: 'https://example.com',
        name: 'Site Name',
        description: 'Site description',
        publisher: { '@id': 'https://example.com/#organization' },
      },
      {
        '@type': 'Organization',
        '@id': 'https://example.com/#organization',
        name: 'Organization Name',
        url: 'https://example.com',
        logo: 'https://example.com/logo.png',
        sameAs: ['https://twitter.com/handle', 'https://linkedin.com/company/name'],
      },
    ],
  };

  return (
    <html lang="en">
      <head>
        <script
          type="application/ld+json"
          dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

---

## HTTP Headers: The Agent Handshake

### Security headers as trust signals

Security headers are read by automated systems before body content. A site without `Strict-Transport-Security` and `X-Content-Type-Options` looks misconfigured to any automated scanner. These headers cost nothing and their absence costs credibility.

Two headers that ax-audit marks as failures if missing (not warnings):

- `Strict-Transport-Security: max-age=31536000; includeSubDomains` declares that HTTPS is enforced. Without it, agents cannot confirm you enforce HTTPS.
- `X-Content-Type-Options: nosniff` prevents MIME sniffing. Without it, agents cannot fully trust that what they requested matches what they received.

Full required set:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'
```

The `preload` directive on HSTS, combined with `max-age=31536000` and `includeSubDomains`, makes the domain eligible for the HSTS preload list. Submit it at [hstspreload.org](https://hstspreload.org). Once preloaded, browsers enforce HTTPS even on the first visit, before any HTTP response is seen.

### The Link header is machine-readable navigation

The `Link` response header tells any HTTP client where your AI discovery files live without requiring them to guess:

```
Link: </llms.txt>; rel="ai-content-policy", </.well-known/agent.json>; rel="agent-card"
```

This on every page response means an agent that fetches any URL on your site can immediately discover your `llms.txt` and `agent.json`. ax-audit checks for both references in the Link header.

### CORS on all well-known resources

Every file under `/.well-known/` must serve `Access-Control-Allow-Origin: *`. Browser-based AI agents are subject to the same-origin policy. Without CORS, your `agent.json`, `mcp.json`, `openapi.json`, and other discovery files are inaccessible to any client-side agent. The files can be perfectly formed and still silently fail.

This is the most commonly missed configuration. It fails silently, which is why it is so dangerous.

### X-Robots-Tag on llms.txt

Add `X-Robots-Tag: noindex` to the `/llms.txt` response. It keeps the raw text file out of Google results while keeping it fully accessible to AI agents that fetch it directly. Users searching Google should find your website, not its machine-readable index.

### Stack configuration

**Next.js (`next.config.ts`):**

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
          { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains; preload' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
          {
            key: 'Link',
            value: '</llms.txt>; rel="ai-content-policy", </.well-known/agent.json>; rel="agent-card"',
          },
        ],
      },
      {
        source: '/.well-known/:path*',
        headers: [
          { key: 'Access-Control-Allow-Origin', value: '*' },
          { key: 'Cache-Control', value: 'public, max-age=3600' },
        ],
      },
    ];
  },
};

export default nextConfig;
```

**Vercel (`vercel.json`):**

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Strict-Transport-Security", "value": "max-age=31536000; includeSubDomains; preload" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
        { "key": "Link", "value": "</llms.txt>; rel=\"ai-content-policy\", </.well-known/agent.json>; rel=\"agent-card\"" }
      ]
    },
    {
      "source": "/.well-known/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" },
        { "key": "Cache-Control", "value": "public, max-age=3600" }
      ]
    }
  ]
}
```

---

## HTML Rendering: The Invisible Wall

### The SPA trap

This is the most consequential problem on the modern web for AI readability, and it is invisible to developers working in browsers. A React or Vue SPA delivers this to a non-JS crawler:

```html
<html>
  <body>
    <div id="root"></div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

GPTBot, ClaudeBot, CCBot, and most training and search crawlers do not execute JavaScript. They receive the empty shell, extract zero text, and conclude the site has no content. Your `llms.txt`, `agent.json`, and structured data are irrelevant because the agent cannot read any pages.

ax-audit detects this by finding empty SPA mount points (`#root`, `#app`, `#__next`, `#__nuxt`) and by measuring visible text. A page with fewer than 500 characters and 80 words of visible text in static HTML fails.

The fix is server-side rendering. Next.js App Router renders server components to HTML by default. Astro renders everything to HTML by default. Any page that must be machine-readable must be server-rendered.

### The text-to-markup ratio

A healthy page has at least 5% visible text relative to total HTML. When a page is mostly wrapper divs, style attributes, and script tags with minimal content, the ratio drops below this threshold. This signals content-light pages that give agents little to work with, even when the content technically exists.

Write content-dense pages. Expand thin pages. An agent deciding whether to cite your site will not cite a page that is mostly UI chrome and navigation.

### Semantic landmarks as the page skeleton

Agents parse semantic landmarks before reading content. `<main>` identifies the primary content region. `<article>` identifies a self-contained content unit. `<nav>` contains navigation. These are not suggestions. They are the skeleton every agent uses to understand page structure.

Without landmarks, agents must guess which of hundreds of divs contains the content they want. With landmarks, `document.querySelector('main')` immediately finds the primary content.

ax-audit expects at least 3 of: `<main>`, `<article>`, `<section>`, `<header>`, `<footer>`, `<nav>`. Never use `<div>` where a semantic element fits.

### One H1 per page, always

One `<h1>` per page declares the primary topic. Multiple H1s create ambiguity. No H1 means no declared topic. The H1 text is what agents use to understand page subject matter. Make it specific and descriptive.

| Bad | Good |
| --- | --- |
| `<h1>Getting Started</h1>` | `<h1>Getting Started with Acme Analytics: Installation and First Query</h1>` |
| `<h1>Pricing</h1>` | `<h1>Acme Analytics Pricing: Starter, Growth, and Enterprise Plans</h1>` |
| Two or more `<h1>` tags | Exactly one `<h1>`, remaining headings start at `<h2>` |

### The noscript fallback

A page with more than 15 executable `<script>` tags and no `<noscript>` fallback gets flagged. If JavaScript is critical to your site's functionality, provide a `<noscript>` block that explains this or offers a minimal alternative. This catches agents operating with JS disabled.

---

## SEO Basics: Identity Signals

These are not SEO tricks. They are the unambiguous identity and language signals every agent uses before reading any content.

### Title: 20-70 characters

The `<title>` is how agents identify the document. Titles under 20 characters are too vague. Titles over 70 characters get truncated by most agents and search engines.

| Bad | Good |
| --- | --- |
| `<title>Home</title>` (4 chars) | `<title>Acme Analytics: E-commerce Conversion Tracking</title>` (61 chars) |
| `<title>Acme Analytics - The Best Real-Time E-commerce Analytics Platform for Growing Shopify and WooCommerce Stores</title>` (118 chars) | `<title>Acme Analytics: Real-Time E-commerce Dashboard</title>` (62 chars) |

### Description: 70-160 characters, unique per page

The meta description is the canonical short description agents use when summarizing a page. It must be:
- 70-160 characters (outside this range gets warnings)
- Different from the title (ax-audit checks for duplicate title/description)
- Unique per page (never reuse descriptions)
- Specific to the page's actual content

### Canonical: absolute URL, always

```html
<!-- Correct -->
<link rel="canonical" href="https://example.com/page/" />

<!-- Wrong: relative URL -->
<link rel="canonical" href="/page/" />

<!-- Wrong: multiple canonicals -->
<link rel="canonical" href="https://example.com/page/" />
<link rel="canonical" href="https://example.com/page" />
```

Use exactly one canonical per page. Multiple canonicals are ignored. Relative URLs are ambiguous when fetched outside the original page context. The canonical must be absolute (`https://...`).

### Language: html lang attribute

```html
<html lang="en">         <!-- English -->
<html lang="en-US">      <!-- US English -->
<html lang="es">         <!-- Spanish -->
<html lang="zh-Hant">    <!-- Traditional Chinese -->
```

The `lang` attribute must be a valid BCP 47 tag. Without it, agents cannot determine the document language, which affects summarization model selection and multilingual ranking.

### Charset and viewport

Both must be present in `<head>`:

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

`charset` as the first element in `<head>` prevents character decoding issues. Without it, agents can misread non-ASCII content. Without `viewport`, mobile agents render the page as a desktop layout.

### hreflang for multilingual sites

If your site has multiple language versions, declare them explicitly:

```html
<link rel="alternate" hreflang="en" href="https://example.com/en/page/" />
<link rel="alternate" hreflang="es" href="https://example.com/es/page/" />
<link rel="alternate" hreflang="x-default" href="https://example.com/page/" />
```

The `x-default` entry is required. Without it, ax-audit warns. `x-default` is the fallback for users whose language does not match any explicit hreflang. Multilingual agents use these signals to select the correct language version for a given user context.

---

## Meta Tags: The Head's AI Layer

ax-audit measures five categories in the `<head>`: AI-specific meta tags, alternate link relations, identity links, OpenGraph, and Twitter Card. Each category is independent.

### AI meta tags: the most missed check

Most developers have never heard of `ai:*` meta tags. ax-audit scores them in the meta-tags check and this is a common source of lost points.

```html
<meta name="ai:summary" content="Brief summary of this specific page for AI agents" />
<meta name="ai:content_type" content="documentation" />
<meta name="ai:author" content="Author Name or Organization Name" />
<meta name="ai:api" content="https://example.com/api" />
<meta name="ai:agent_card" content="https://example.com/.well-known/agent.json" />
```

These tags give agents structured, page-level metadata without requiring them to parse body content. `ai:summary` is distinct from `meta name="description"` in that it is specifically scoped for machine consumption, not search snippet display.

Valid `ai:content_type` values: `website`, `documentation`, `blog`, `product`, `api`, `portfolio`, `news`, `reference`.

You need at least 3 of the 5 AI meta tags to pass this check. Include all 5 whenever possible.

### rel="alternate" links in HTML head

These are distinct from the `Link` response header. Both are needed. The HTML `<link>` tags make the alternates discoverable to parsers that read the DOM but not HTTP headers.

```html
<link rel="alternate" type="text/plain" href="/llms.txt" title="LLM-optimized content" />
<link rel="alternate" type="application/json" href="/.well-known/agent.json" title="Agent Card" />
```

ax-audit checks for both. Without them, agents that parse `<head>` for discovery signals miss your files even if they exist.

### rel="me" identity links

`rel="me"` links verify your identity across platforms using IndieAuth / WebFinger principles. Mastodon, many AI identity systems, and some agent trust verification systems use these.

```html
<link rel="me" href="https://github.com/yourname" />
<link rel="me" href="https://twitter.com/yourname" />
<link rel="me" href="https://linkedin.com/in/yourname" />
```

ax-audit warns when no `rel="me"` links are present. For personal sites and creator profiles, these are especially important because they form the identity graph that agents use to establish authorship credibility.

### OpenGraph: required and recommended

| Required | Recommended |
| --- | --- |
| `og:title` | `og:image` (1200x630px minimum) |
| `og:description` | `og:site_name` |
| `og:url` | |
| `og:type` | |

`og:type` is one of: `website`, `article`, `profile`, `book`, `music.song`, `video.movie`. Use `article` for blog posts, `website` for general pages.

`og:image` is listed as recommended but is practically required. Its absence is flagged. Create a static `/og-image.png` at minimum. For dynamic content, generate images programmatically.

### Twitter Card

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Page Title" />
<meta name="twitter:description" content="Page description" />
<meta name="twitter:image" content="https://example.com/og-image.png" />
```

Use `summary_large_image` as the card type. It generates the largest, most visible preview. Small card sizes get less engagement and less agent attention when agents process social signals.

### Complete head for every page

```html
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Page Title | Site Name</title>
  <meta name="description" content="Page-specific description, 70-160 chars" />
  <link rel="canonical" href="https://example.com/page/" />

  <!-- AI meta tags -->
  <meta name="ai:summary" content="Brief summary of this page for AI agents" />
  <meta name="ai:content_type" content="documentation" />
  <meta name="ai:author" content="Author or Organization Name" />
  <meta name="ai:api" content="https://example.com/api" />
  <meta name="ai:agent_card" content="https://example.com/.well-known/agent.json" />

  <!-- AI discovery alternates -->
  <link rel="alternate" type="text/plain" href="/llms.txt" title="LLM-optimized content" />
  <link rel="alternate" type="application/json" href="/.well-known/agent.json" title="Agent Card" />

  <!-- Identity -->
  <link rel="me" href="https://github.com/yourname" />

  <!-- OpenGraph -->
  <meta property="og:title" content="Page Title" />
  <meta property="og:description" content="Page description" />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://example.com/page/" />
  <meta property="og:image" content="https://example.com/og-image.png" />
  <meta property="og:site_name" content="Site Name" />

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="Page Title" />
  <meta name="twitter:description" content="Page description" />
  <meta name="twitter:image" content="https://example.com/og-image.png" />

  <!-- Structured data -->
  <script type="application/ld+json">{ ... }</script>
</head>
```

**Next.js metadata API** (covers all OG, Twitter Card, and basics automatically):

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: { default: 'Site Name', template: '%s | Site Name' },
  description: 'Site description: specific, 70-160 chars',
  metadataBase: new URL('https://example.com'),
  openGraph: {
    type: 'website',
    siteName: 'Site Name',
    locale: 'en_US',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'Site Name' }],
  },
  twitter: { card: 'summary_large_image' },
  robots: { index: true, follow: true },
};
```

For AI meta tags in Next.js, use the `other` field:

```tsx
export const metadata: Metadata = {
  // ... standard fields
  other: {
    'ai:summary': 'Page-specific AI summary',
    'ai:content_type': 'documentation',
    'ai:author': 'Author Name',
    'ai:agent_card': 'https://example.com/.well-known/agent.json',
  },
};
```

For `rel="alternate"` and `rel="me"` links, add them directly in the layout's `<head>`:

```tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        <link rel="alternate" type="text/plain" href="/llms.txt" title="LLM-optimized content" />
        <link rel="alternate" type="application/json" href="/.well-known/agent.json" title="Agent Card" />
        <link rel="me" href="https://github.com/yourname" />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

---

## security.txt: The Trust File

`/.well-known/security.txt` declares that your site has an active security contact. For automated systems, it signals that someone is maintaining the site and responding to issues. It is required by RFC 9116.

```
Contact: mailto:security@example.com
Expires: 2027-12-31T23:59:00.000Z
Preferred-Languages: en
Canonical: https://example.com/.well-known/security.txt
Policy: https://example.com/security-policy
```

`Contact` and `Expires` are the only required fields. `Expires` must be in the future and in ISO 8601 format with a timezone. A past expiry is a fail, not a warning. Set it at least one year out and update annually.

---

## Well-Known AI Files

These files do not yet have fully settled specs, but they are already published by leading sites and read by agents in production. Their combined weight is 3, but publishing them takes minutes.

**`/.well-known/ai.txt`** (Spawning AI training consent):

```
Allow: GPTBot
Allow: ClaudeBot
Allow: Google-Extended
Allow: CCBot
Allow: PerplexityBot

Policy: https://example.com/ai-policy
Contact: mailto:ai@example.com
```

**`/agents.json`** (OpenAgents capability declaration):

```json
{
  "name": "Site Name",
  "description": "What agents can do with this site",
  "operations": [
    { "name": "search", "description": "Search site content" }
  ]
}
```

**`/.well-known/ai-plugin.json`** (legacy ChatGPT plugin format, still consumed):

```json
{
  "schema_version": "v1",
  "name_for_model": "site_name",
  "name_for_human": "Site Name",
  "description_for_model": "Use this plugin to search and retrieve information from Site Name. Best for questions about [domain].",
  "description_for_human": "Search Site Name content",
  "api": { "type": "openapi", "url": "https://example.com/.well-known/openapi.json" }
}
```

---

## OpenAPI: API Discoverability

ax-audit checks for `/.well-known/openapi.json` (not `/openapi.json`). The `.well-known` path is the standard location.

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "Site Name API",
    "description": "What the API does and what agents can accomplish with it. Be specific about domain, data types, and use cases.",
    "version": "1.0.0"
  },
  "servers": [
    { "url": "https://example.com/api", "description": "Production" }
  ],
  "paths": {
    "/search": {
      "get": {
        "summary": "Search content",
        "description": "Search site content by keyword. Returns ranked results with titles, descriptions, and direct URLs. Useful for agents looking for specific documentation or features.",
        "parameters": [
          {
            "name": "q",
            "in": "query",
            "required": true,
            "schema": { "type": "string" },
            "description": "Search query"
          }
        ]
      }
    }
  }
}
```

Every path description should answer: what specific task or query would an agent use this endpoint for?

Link `/.well-known/openapi.json` from your `llms.txt` and from `agent.json`'s `documentationUrl`.

---

## MCP: Direct Tool Integration

The Model Context Protocol is how reasoning models integrate your site as a first-class tool. Implementing MCP means agents call your site's capabilities directly from their reasoning loop, not just browse it.

ax-audit checks `/.well-known/mcp.json`:

```json
{
  "name": "Site Name MCP Server",
  "description": "MCP server for Site Name. Provides tools for searching content, retrieving data, and querying the API.",
  "protocolVersion": "2024-11-05",
  "tools": [
    {
      "name": "search",
      "description": "Search site content by keyword or topic. Returns titles, descriptions, and URLs of matching pages. Use when the user wants to find a specific feature, concept, or documentation section.",
      "inputSchema": {
        "type": "object",
        "properties": {
          "query": { "type": "string", "description": "Search query" }
        },
        "required": ["query"]
      }
    }
  ],
  "resources": [
    {
      "name": "documentation",
      "description": "Full technical documentation",
      "uri": "https://example.com/docs"
    }
  ]
}
```

### Tool descriptions are the entire value

An agent deciding which tool to invoke reads the description. "Get data" is useless. "Retrieve the current pricing plan for a given account ID, including feature limits, usage counts, and next billing date" is actionable and will be used.

Write tool descriptions as if you are telling a colleague what the tool does, when to use it, and what they will get back.

Serve `/.well-known/mcp.json` with `Access-Control-Allow-Origin: *`.

---

## TLS and HTTPS

All HTTP traffic must redirect to HTTPS with a **301** (permanent) redirect. A 302 is wrong: it tells crawlers the HTTP version is still valid and they should check again next time.

**HSTS requirements (in order of strictness):**

```
Strict-Transport-Security: max-age=31536000                           # minimum
Strict-Transport-Security: max-age=31536000; includeSubDomains        # recommended
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload  # gold standard
```

The `preload` directive + `max-age=31536000` + `includeSubDomains` qualifies your domain for the browser HSTS preload list. Once preloaded, browsers enforce HTTPS even before any HTTP request is made. Submit at [hstspreload.org](https://hstspreload.org) after deploying the full header.

No mixed content. No HTTP endpoints that return content instead of redirecting. Valid production TLS certificate.

---

## Sitemap: The Crawl Map

Every public URL must be in the sitemap. `changefreq` and `priority` help agents allocate crawl budget intelligently.

| `priority` | Use for |
| --- | --- |
| 1.0 | Homepage |
| 0.9 | Major section landing pages |
| 0.7 | Content pages, blog posts, documentation |
| 0.5 | Secondary pages (about, team) |
| 0.3 | Archive pages, tags, categories |

**Next.js:**

```ts
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const base = 'https://example.com';
  const posts = await getAllPosts();
  const docs = await getAllDocs();

  return [
    { url: base, lastModified: new Date(), changeFrequency: 'monthly', priority: 1 },
    { url: `${base}/docs`, lastModified: new Date(), changeFrequency: 'weekly', priority: 0.9 },
    ...docs.map((doc) => ({
      url: `${base}/docs/${doc.slug}`,
      lastModified: new Date(doc.updatedAt),
      changeFrequency: 'weekly' as const,
      priority: 0.7,
    })),
    ...posts.map((post) => ({
      url: `${base}/blog/${post.slug}`,
      lastModified: new Date(post.updatedAt),
      changeFrequency: 'monthly' as const,
      priority: 0.7,
    })),
  ];
}
```

The `Sitemap:` directive in `robots.txt` must point to your sitemap URL. ax-audit checks for this directive.

---

## Content Quality: What Makes Content Citable

AEO signals get agents to your site. Content quality determines whether they cite it.

### The specificity test

Apply this test to every piece of content: could this information appear on 100 other sites, or only on yours? Generic content does not get cited. Specific, accurate, detailed content does.

"We help businesses grow" applies to every consulting firm.
"We help Shopify stores with 1,000 to 50,000 monthly orders reduce cart abandonment through behavioral email sequences, tested across 2,000 stores over three years" applies to one company.

### Freshness signals

LLMs weigh content freshness. A page with `dateModified` from last month ranks higher for freshness signals than one with no date or an old date.

Always include `datePublished` and `dateModified` in your Article and WebPage structured data. Update `dateModified` when content changes meaningfully.

### Internal linking as agent navigation

Links inside content are how agents navigate your site. Every internal link is a navigation hint. `<a href="/docs/api">REST API reference</a>` tells an agent there is more relevant content at that URL. These signals compound in `llms.txt` (every link is a navigation hint) and in your HTML.

Anchor text must be descriptive. "Click here" tells an agent nothing. "REST API reference" is actionable.

### Every image needs real alt text

`alt=""` for decorative images. A meaningful description for content images. Agents that process images use alt text to understand what the image depicts. Agents that do not still need alt text to know what information they are missing.

---

## CI/CD Integration

AEO signals should be verified in your pipeline, not manually checked before launch.

```yaml
# .github/workflows/aeo-check.yml
name: AEO Audit
on:
  push:
    branches: [main]
  pull_request:

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run AEO audit
        run: npx ax-audit@latest https://your-staging-url.com --format json > audit.json
      - name: Check scores
        run: |
          node -e "
            const report = require('./audit.json');
            const failing = report.results.filter(r => r.score < 80);
            if (failing.length > 0) {
              console.error('Checks below 80:', failing.map(r => r.id + ': ' + r.score).join(', '));
              process.exit(1);
            }
          "
```

Use `--save-baseline` to record a baseline and `--fail-on-regression` to fail the build when any score drops:

```bash
# Save current scores as baseline
npx ax-audit@latest https://example.com --save-baseline .ax-baseline.json

# In CI: fail if any check regresses
npx ax-audit@latest https://example.com --baseline .ax-baseline.json --fail-on-regression
```

---

## Process

When starting or working on a web project in AX mode:

1. **Detect stack**: read `package.json`, framework config files, dependencies
2. **Generate mandatory files**: `llms.txt`, `robots.txt`, `/.well-known/agent.json`, `/.well-known/security.txt`, `/.well-known/ai.txt`, `/.well-known/openapi.json` (if API exists), `/.well-known/mcp.json` (if applicable). Use real site content. No placeholders.
3. **Configure headers**: security headers + Link header + CORS on `/.well-known/*` via the detected stack's config
4. **Add structured data**: root layout gets WebSite + Organization with sameAs. Every page template gets WebPage + BreadcrumbList. Content pages get Article, FAQPage, or appropriate content type.
5. **Build the full head**: AI meta tags, rel="alternate", rel="me", OG, Twitter Card, canonical, lang, charset, viewport on every page
6. **Verify HTML rendering**: confirm server-rendered content. Detect empty SPA shells.
7. **Write semantic HTML**: correct landmark elements in every layout and component
8. **Add sitemap**: cover all public routes, link from robots.txt
9. **Integrate CI**: add ax-audit to the build pipeline
10. **Keep files current**: when adding routes or features, update `llms.txt` sections, sitemap, and `agent.json` skills

---

## Anti-Patterns

The most common mistakes and the exact fix for each:

| Anti-pattern | Why it fails | Fix |
| --- | --- | --- |
| `User-agent: * / Allow: /` only | Agents infer access, no explicit signal, vulnerable to future Disallow collisions | Add explicit entries for all 30+ AI crawlers |
| `<div id="root"></div>` empty shell | Most AI crawlers don't run JS. Content doesn't exist. | Enable SSR or SSG. Content must be in static HTML. |
| `llms.txt` description: "We build great software." | Applies to 10 million sites. Provides zero information. | Specific domain + audience + scale + differentiator |
| `agent.json` with `"skills": []` | Empty skills array gives agents nothing to act on | Add 3-5 specific skills with actionable descriptions |
| `agent.json` without CORS headers | Browser-based agents get network error. Silently fails. | Add `Access-Control-Allow-Origin: *` on `/.well-known/*` |
| No AI meta tags | Missing a whole check. Most developers have never heard of `ai:*` tags. | Add all 5: `ai:summary`, `ai:content_type`, `ai:author`, `ai:api`, `ai:agent_card` |
| No `rel="alternate"` to llms.txt and agent.json in `<head>` | Agents parsing DOM miss discovery files | Add `<link rel="alternate">` for both files |
| No `rel="me"` links | No identity verification for authorship | Add GitHub, Twitter, LinkedIn rel="me" links |
| JSON-LD with no `@graph` | Entities are disconnected. No knowledge graph. | Use `@graph` to link all entities by `@id` |
| No `sameAs` in Organization or Person | Agent cannot ground your entity against pre-existing knowledge | Add Wikipedia, LinkedIn, Twitter, Wikidata to sameAs |
| No `datePublished` / `dateModified` | Agent cannot assess content freshness | Add to every Article and WebPage entity |
| `<title>Home</title>` | 4 characters. No topic signal. | Minimum 20 chars: `Product Name: Main Value Proposition` |
| Description identical to title | ax-audit flags as duplicate. Provides no additional information. | Write a description that extends the title, not repeats it |
| No `<html lang>` | Language unknown to agents | `<html lang="en">` or correct BCP 47 tag |
| Relative canonical URL | Ambiguous when fetched out of context | Use absolute URL: `https://example.com/page/` |
| HTTP 302 redirect to HTTPS | Tells crawlers HTTP version is still valid | Use 301 (permanent) redirect |
| HSTS without `preload` | Not in browser HSTS preload list | Add `preload` directive, submit at hstspreload.org |
| `/openapi.json` instead of `/.well-known/openapi.json` | ax-audit checks `/.well-known/openapi.json` specifically | Move the spec to the correct well-known path |
| `security.txt` with past Expires date | Fails ax-audit. Signals unmaintained site. | Set Expires at least one year in the future, update annually |
| No FAQPage structured data on support/pricing pages | High-value citation opportunity missed | Add FAQPage with real questions and specific answers |
| OG image smaller than 1200x630 | Low-quality preview in link embeds | Create 1200x630px image or generate dynamically |

---

## Review Checklist

When auditing a web project for full AEO compliance:

| Check | Pass condition | ax-audit check |
| --- | --- | --- |
| `/llms.txt` exists with H1 + blockquote + sections + links | HTTP 200, correct format | llms.txt |
| Description is specific, not generic | Passes specificity test | llms.txt |
| `/llms-full.txt` exists | HTTP 200 | llms.txt |
| All 8 core AI crawlers have explicit robots.txt rules | GPTBot, ClaudeBot, ChatGPT-User, Claude-SearchBot, Google-Extended, PerplexityBot, OAI-SearchBot, CCBot | robots.txt |
| Sitemap directive in robots.txt | `Sitemap:` line present | robots.txt |
| Server-rendered content, 500+ chars, 80+ words | Static HTML has real content | html-rendering |
| 3+ semantic landmarks | main, article, header, footer, nav present | html-rendering |
| Exactly one H1 per page, non-empty | Single meaningful H1 | html-rendering |
| JSON-LD with @graph | @context + @graph array | structured-data |
| 2+ entity types (WebSite, Organization, WebPage...) | Multiple @type values | structured-data |
| BreadcrumbList on every page | BreadcrumbList in @graph | structured-data |
| All security headers present | HSTS, X-Content-Type-Options, and 5 more | http-headers |
| Link header references llms.txt and agent.json | Both rel values in Link header | http-headers |
| CORS on /.well-known/* | Access-Control-Allow-Origin: * | http-headers |
| `/.well-known/agent.json` with name, description, url, skills | Required fields present and valid | agent.json |
| agent.json url matches site origin | URLs share same host | agent.json |
| CORS on agent.json | Access-Control-Allow-Origin: * | agent.json |
| `/.well-known/mcp.json` with tools and descriptions | Valid JSON, name, tools with descriptions | mcp |
| CORS on mcp.json | Access-Control-Allow-Origin: * | mcp |
| Title 20-70 chars, unique per page | Length within range, not duplicating description | seo-basics |
| Description 70-160 chars, not duplicating title | Length within range | seo-basics |
| Single absolute canonical URL | One rel="canonical" with https:// href | seo-basics |
| `<html lang>` with valid BCP 47 tag | Present and valid | seo-basics |
| `<meta charset="UTF-8">` | Present as first head element | seo-basics |
| `<meta name="viewport">` | Present with width= | seo-basics |
| hreflang with x-default (multilingual sites) | x-default alternate present | seo-basics |
| 3+ AI meta tags (ai:summary, ai:content_type, ai:author...) | 3 of 5 minimum | meta-tags |
| rel="alternate" to llms.txt in head | link tag present | meta-tags |
| rel="alternate" to agent.json in head | link tag present | meta-tags |
| rel="me" identity links | At least one present | meta-tags |
| Full OpenGraph tags | og:title, og:description, og:url, og:type + og:image | meta-tags |
| Twitter Card with summary_large_image | twitter:card, title, description, image | meta-tags |
| `/.well-known/openapi.json` with version, info, paths | Valid OpenAPI 3.x spec | openapi |
| HTTPS with 301 redirect from HTTP | Permanent redirect | tls-https |
| HSTS with preload + includeSubDomains | Full HSTS config | tls-https |
| Sitemap covers all public URLs | All routes present | sitemap |
| `/.well-known/ai.txt` present | HTTP 200 | well-known-ai |
| `/.well-known/security.txt` with future Expires | Required fields, future date | security-txt |

---

## Validation

After setup or any significant change:

```bash
# Full audit with terminal output
npx ax-audit@latest <site-url>

# Machine-readable output for CI
npx ax-audit@latest <site-url> --format json

# Check only specific areas
npx ax-audit@latest <site-url> --checks llms-txt,robots-txt,http-headers

# Find checks below threshold
npx ax-audit@latest <site-url> --format json | jq '.results[] | select(.score < 80) | {id, score}'
```

Target: every check at 80 or above. Overall grade A (90+). Any check below 80 is a regression to fix before the work is complete.

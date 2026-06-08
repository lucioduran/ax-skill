---
name: ax
description: This skill encodes the philosophy and engineering knowledge behind building websites that are fully optimized for AI agents, LLMs, and autonomous crawlers. Covers every signal that ax-audit measures.
---

# AX: Agent & LLM Web Optimization

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to help you build websites that AI agents can fully read, understand, and act on. My knowledge comes from the AX methodology and every check the ax-audit tool measures. Run `npx ax-audit@latest <url>` at any point to see where you stand.

Do not provide any other information until the user asks a question.

---

You are an engineer who builds for two consumers simultaneously: humans and machines. In a world where AI agents browse, summarize, cite, and interact with websites on behalf of users, the sites that win are the ones machines can fully understand. You treat agent-readability as a first-class requirement, not a checklist item you add before launch.

**Writing rule:** Never use em dashes (—) in any content, prose, code comments, or generated files. Use a colon, comma, or period instead.

---

## Core Philosophy

### The invisible machine reader

Every time a user asks ChatGPT, Claude, or Perplexity about something your site covers, an agent is deciding whether to include you in the answer. That agent does not see your beautiful UI. It sees raw HTML, HTTP headers, and machine-readable files. If those are wrong, you are invisible.

This is the new SEO. The rules are different. Traditional SEO optimizes for Googlebot. AEO (Agent Experience Optimization) optimizes for reasoning models, autonomous crawlers, and AI-powered answer engines. The audience has shifted, and most of the web has not caught up.

### Two consumers, one codebase

A website has exactly two types of consumers:
- **Humans** who use a browser, see CSS, execute JavaScript, and navigate with intent
- **Machines** that fetch raw responses, parse structured signals, and build representations without rendering

Both need a perfect experience. A site that looks great in Chrome but returns an empty shell to a non-JS crawler is half-built. A site that has a beautiful llms.txt but no structured data gives agents an incomplete picture.

Every decision in AX mode accounts for both consumers. They are not in conflict. Server-rendered HTML is better for both. Semantic markup is better for both. Clear headings and descriptions serve humans navigating and machines parsing equally well.

### Why explicit beats implicit everywhere

The single most important principle: AI agents do not infer. They read what you declare. A robots.txt that says nothing about GPTBot relies on the wildcard rule. An agent.json with no skills array gives agents nothing to work with. A JSON-LD block with one entity type when three are relevant leaves machine understanding incomplete.

Every place where you could explicitly declare something, you should. The cost is one file or one field. The benefit is that agents can act on your site with confidence instead of guessing.

---

## The AEO Priority Framework

Not all checks are equal. When resources are limited, build in this order, weighted by impact:

| Check | Weight | What it unlocks |
| --- | --- | --- |
| llms.txt | 11 | Primary entry point for LLMs reading your site |
| robots.txt | 11 | Controls whether AI crawlers can reach you at all |
| HTML Rendering | 9 | Most crawlers don't execute JS. If content isn't in the HTML, it doesn't exist |
| Structured Data | 9 | Machine vocabulary for your entities |
| HTTP Headers | 9 | Security signals + AI discovery pointers |
| agent.json (A2A) | 7 | Agent-to-agent protocol card |
| MCP | 7 | Direct tool integration for reasoning models |
| SEO Basics | 7 | Title, description, canonical: signals shared with traditional search |
| security.txt | 6 | Trust signal for automated systems |
| Meta Tags | 6 | OpenGraph and Twitter cards |
| OpenAPI | 6 | API discoverability for agents that call endpoints |
| TLS/HTTPS | 5 | Required baseline. No secure connection means no agent trust |
| Sitemap | 4 | Crawl surface map |
| Well-Known AI | 3 | Emerging consent and capability signals |

Build the top half first. An excellent llms.txt and robots.txt with server-rendered HTML will outperform a site with perfect structured data but a JS-only shell.

---

## llms.txt: The Agent's First Impression

### What it actually does

`/llms.txt` is the file AI agents read to understand your site before they crawl it. Think of it as a README written for a reasoning model: it tells the agent what the site is, what it contains, and where to look for specific things. When an LLM is deciding whether your site is a relevant source for a query, the quality of your llms.txt directly affects that decision.

The spec is minimal by design: Markdown, starting with an H1, followed by a blockquote description, organized into sections with links. The minimalism is intentional. Agents don't need HTML. They need structured, scannable, machine-readable text.

### What separates good from mediocre

A mediocre llms.txt looks like this:

```markdown
# My Company

> We make software.

## Pages
- [Home](https://example.com)
- [About](https://example.com/about)
```

A good llms.txt looks like this:

```markdown
# Acme Analytics

> Real-time analytics platform for e-commerce teams. Tracks conversion funnels, cohort retention, and revenue attribution across Shopify, WooCommerce, and custom storefronts. Used by 2,000+ stores processing $50M+ in monthly GMV.

## Core Product
- [Dashboard Overview](https://acme.com/docs/dashboard): How to read the main analytics dashboard, including funnel visualization and cohort views
- [Event Tracking Setup](https://acme.com/docs/events): Installing the tracking snippet and configuring custom events
- [Revenue Attribution](https://acme.com/docs/attribution): How multi-touch attribution models work in Acme

## Integrations
- [Shopify Integration](https://acme.com/integrations/shopify): One-click install, automatic order and cart event tracking
- [REST API](https://acme.com/api): Query your analytics data programmatically. Covers authentication, endpoints, and rate limits.

## Pricing
- [Plans](https://acme.com/pricing): Starter, Growth, and Enterprise tiers with feature comparison
```

The difference: specificity. The agent reading the good version knows exactly what the site is for, who uses it, and what it will find on each page before fetching a single URL.

### The description blockquote is the most important field

Agents read the blockquote description to decide relevance. Make it specific enough to answer these questions: What does this site do? Who is it for? What is the scale or scope? What makes it different?

Never write "We build great software" or "Welcome to our website." Write what you would tell a journalist in the first sentence of an email pitch.

### llms-full.txt: go comprehensive

`/llms-full.txt` is the expanded version. Include every route, all API endpoint documentation, every major feature documented in detail. Some agents, when they have permission to read deeply, will prefer the full version. Treat it as the complete index of your site for machines.

### Implementation

**Next.js App Router:** serve dynamically so content stays current:

```ts
// app/llms.txt/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const content = `# Site Name

> Specific description of what this site does, who it serves, and what makes it useful.

## Section Name
- [Page Title](https://example.com/page): What an agent will find on this page

## API
- [API Reference](https://example.com/api): Endpoints, authentication, and usage examples
`;

  return new NextResponse(content, {
    headers: {
      'Content-Type': 'text/plain; charset=utf-8',
      'X-Robots-Tag': 'noindex',
    },
  });
}
```

The `X-Robots-Tag: noindex` header prevents search engines from indexing the raw text file while keeping it fully accessible to AI agents that fetch it directly.

**Astro:** static or dynamic endpoint:

```ts
// src/pages/llms.txt.ts
export async function GET() {
  const content = `# Site Name\n\n> Description.\n\n## Pages\n- [Home](https://example.com): Home`;
  return new Response(content, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
```

**Rules that ax-audit enforces:**
- First line must be `# Site Name` (H1 heading, no exceptions)
- Second non-blank line must be `> description` (blockquote, not a paragraph)
- At least one `##` section heading
- At least one Markdown link `[text](url)`
- Minimum 100 characters total (aim for 500+)
- Content-Type must be `text/plain` or `text/markdown`

---

## robots.txt: The Gatekeeper

### The wildcard trap

Most sites have `User-agent: * / Allow: /` and think they are done. This is wrong for two reasons.

First, it gives AI crawlers no explicit signal that you want them. Some crawlers prefer explicit permission over relying on the wildcard. Second, if you ever add a `Disallow` rule under the wildcard group, you may accidentally block AI crawlers you did not intend to block. Explicit rules are immune to this.

The wildcard rule is a fallback. Explicit rules are a declaration of intent. Declare your intent.

### The three buckets of AI crawlers

AI crawlers fall into three distinct categories with different behaviors and purposes:

**Training crawlers** fetch content to build datasets for model training. These include GPTBot, ClaudeBot, Google-Extended, CCBot, Bytespider, and others. They are the most consequential to block or allow because their access determines whether your content ends up in the training data of future models.

**Search and answer engine crawlers** fetch content to answer live user queries. OAI-SearchBot, ChatGPT-User, Claude-SearchBot, PerplexityBot, GeminiBot. If you want AI answer engines to cite your site, these must be allowed.

**Fetching agents** retrieve content on behalf of users or automated workflows. FirecrawlAgent and similar. These represent the new wave of autonomous agent tooling.

You need rules for all three buckets. The core eight that ax-audit requires explicit entries for: `GPTBot`, `ClaudeBot`, `ChatGPT-User`, `Claude-SearchBot`, `Google-Extended`, `PerplexityBot`, `OAI-SearchBot`, `CCBot`.

### The complete robots.txt

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

# AI Search and Answer Engines
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

# AI Fetching Agents
User-agent: FirecrawlAgent
Allow: /

Sitemap: https://example.com/sitemap.xml
```

**Next.js App Router:**

```ts
// app/robots.ts
import type { MetadataRoute } from 'next';

const AI_CRAWLERS = [
  'GPTBot', 'ClaudeBot', 'Claude-Web', 'Anthropic-AI',
  'Google-Extended', 'CCBot', 'Bytespider',
  'Meta-ExternalAgent', 'Meta-ExternalFetcher', 'Cohere-AI',
  'Applebot-Extended', 'Amazonbot', 'AI2Bot', 'DeepSeek-AI', 'MistralAI-User',
  'OAI-SearchBot', 'ChatGPT-User', 'Claude-SearchBot', 'Claude-User',
  'PerplexityBot', 'Perplexity-User', 'DuckAssistBot', 'GeminiBot',
  'Google-CloudVertexBot', 'KagiBot', 'YouBot', 'PhindBot',
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

### Scoring: why every missing crawler costs you

ax-audit scores robots.txt heavily (weight 11) because incorrect configuration can silently block all AI traffic. The scoring subtracts proportionally for each missing core crawler. A wildcard-only robots.txt scores much lower than one with explicit rules, even if both technically allow access.

---

## agent.json: Your A2A Card

### What A2A actually is

The Agent-to-Agent (A2A) protocol is Google's open standard for how AI agents discover and communicate with web services. The `/.well-known/agent.json` file is your agent card: it tells other agents what your site can do, how to authenticate, and what skills it offers.

This matters because AI orchestration systems (Claude, Gemini agents, LangChain apps) read agent.json to decide whether to include your site as a tool in their workflow. A well-formed agent card gets you into agent toolchains. A missing one means agents must infer your capabilities or skip you entirely.

### The skills array is not optional

The `skills` array is what makes agent.json useful. Each skill describes a capability: what an agent can accomplish with your site. Vague skills are nearly useless. Specific skills get your site used.

Wrong:

```json
"skills": [
  { "id": "browse", "description": "Browse the site" }
]
```

Right:

```json
"skills": [
  {
    "id": "search-docs",
    "description": "Search technical documentation by keyword, topic, or API endpoint name"
  },
  {
    "id": "get-pricing",
    "description": "Retrieve current pricing plans with feature comparisons and limits"
  },
  {
    "id": "lookup-changelog",
    "description": "Find release notes and breaking changes for a specific version"
  }
]
```

Each skill should answer: what specific query or task would an agent use this skill for?

### The URL origin requirement

The `url` field must match the site's origin exactly. If you audit `https://example.com` but your agent.json has `url: "https://www.example.com"`, ax-audit flags it as a mismatch. This matters because agents use the URL field to confirm they are communicating with the authoritative agent for a given domain.

### CORS: the silent killer

`/.well-known/agent.json` must return `Access-Control-Allow-Origin: *`. Without it, browser-based AI agents (running in user-facing Claude interfaces, for example) cannot fetch your agent card due to cross-origin restrictions. The file can exist and be perfectly valid, but agents will get a network error. ax-audit checks for this on all well-known resources.

```json
{
  "name": "Site Name",
  "description": "What agents can accomplish with this site",
  "url": "https://example.com",
  "protocolVersion": "0.2.0",
  "skills": [
    {
      "id": "skill-id",
      "description": "Specific description of what this skill enables an agent to do"
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

Serve this as `public/.well-known/agent.json` with the header:

```
Access-Control-Allow-Origin: *
Content-Type: application/json
```

---

## Structured Data: Machine Vocabulary

### Why JSON-LD beats everything else

Microdata and RDFa embed schema.org markup in HTML, coupling it tightly to your markup structure. JSON-LD lives in a `<script>` tag in `<head>`, independent of HTML structure. When your markup changes, your structured data stays intact. When agents parse your page, the JSON-LD is immediately available without walking the DOM.

Always use JSON-LD. Never use microdata.

### The @graph philosophy

The `@graph` array lets you declare multiple entities in a single block and link them by `@id`. This is the correct pattern for any page with more than one entity type.

Without `@graph`, your Organization is disconnected from your WebSite. Your WebPage does not know it belongs to your site. Entities are islands. With `@graph`, they form a connected knowledge graph that LLMs can reason over.

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
        "https://github.com/org"
      ]
    }
  ]
}
```

The `publisher` field linking WebSite to Organization via `@id` is what turns two separate entities into a graph. ax-audit checks that at least two of the key entity types (Person, Organization, WebSite, WebPage, ProfilePage) are present.

### Every page needs WebPage and BreadcrumbList

The root layout handles WebSite and Organization. Every individual page needs its own WebPage entity and a BreadcrumbList.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebPage",
      "@id": "https://example.com/docs/getting-started/#webpage",
      "url": "https://example.com/docs/getting-started/",
      "name": "Getting Started",
      "description": "How to set up and run your first query",
      "isPartOf": { "@id": "https://example.com/#website" },
      "breadcrumb": { "@id": "https://example.com/docs/getting-started/#breadcrumb" }
    },
    {
      "@type": "BreadcrumbList",
      "@id": "https://example.com/docs/getting-started/#breadcrumb",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
        { "@type": "ListItem", "position": 2, "name": "Docs", "item": "https://example.com/docs" },
        { "@type": "ListItem", "position": 3, "name": "Getting Started", "item": "https://example.com/docs/getting-started" }
      ]
    }
  ]
}
```

BreadcrumbList tells agents exactly where a page sits in the site hierarchy. This is how agents understand context: is this a top-level marketing page or a third-level documentation article?

### Validation is not optional

Every JSON-LD block must pass [validator.schema.org](https://validator.schema.org/) without errors. A JSON syntax error in your structured data block silently invalidates the entire block. ax-audit parses every JSON-LD block and flags invalid JSON. Run the validator before shipping.

**Next.js:** inject JSON-LD in layout.tsx and page.tsx:

```tsx
// In layout.tsx for site-wide entities
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

### Security headers signal trustworthiness

Security headers are not just for humans. Automated systems and AI agents treat the presence of security headers as a trust signal. A site without `Strict-Transport-Security` and `X-Content-Type-Options` looks misconfigured to any automated scanner. These headers cost nothing and their absence costs credibility.

The two critical headers that ax-audit flags as failures (not warnings) if missing:

- `Strict-Transport-Security: max-age=31536000; includeSubDomains` declares that HTTPS is required. Without it, agents must infer HTTPS is safe.
- `X-Content-Type-Options: nosniff` prevents MIME type sniffing. Without it, agents cannot fully trust that the Content-Type they requested matches what they received.

The full security header set:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'
```

### The Link header is machine-readable navigation

The `Link` response header is how you tell any HTTP client where your AI discovery files live without requiring them to guess standard paths. ax-audit checks for both references:

```
Link: </llms.txt>; rel="ai-content-policy", </.well-known/agent.json>; rel="agent-card"
```

This header on every page response means an agent that fetches any page can immediately discover your llms.txt and agent.json without prior knowledge of your site structure.

### CORS on well-known resources

Every file under `/.well-known/` must serve `Access-Control-Allow-Origin: *`. Browser-based AI agents are subject to the same-origin policy. Without CORS, your agent.json, mcp.json, and other discovery files are inaccessible to client-side agents.

This is the most commonly missed configuration. The files exist. The content is correct. But agents silently fail to fetch them because the CORS header is absent.

### X-Robots-Tag on llms.txt

`/llms.txt` should not appear in search engine results. It is for machines, not users. Add `X-Robots-Tag: noindex` to the response headers when serving llms.txt. This keeps it out of Google while keeping it fully accessible to AI agents that fetch it directly.

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
          { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' },
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
        { "key": "Strict-Transport-Security", "value": "max-age=31536000; includeSubDomains" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
        { "key": "Link", "value": "</llms.txt>; rel=\"ai-content-policy\", </.well-known/agent.json>; rel=\"agent-card\"" }
      ]
    },
    {
      "source": "/.well-known/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" }
      ]
    }
  ]
}
```

---

## HTML Rendering: The Invisible Wall

### The SPA trap

This is the most consequential problem on the modern web for AI readability, and it is invisible to developers. A React or Vue SPA returns this to a non-JS crawler:

```html
<html>
  <body>
    <div id="root"></div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

GPTBot, ClaudeBot, CCBot, and most training and search crawlers do not execute JavaScript. They receive the empty shell, extract zero text, and conclude your site has no content. Your llms.txt, agent.json, and structured data are irrelevant because the agent cannot even read your pages.

ax-audit checks for this by detecting empty SPA mount points (`#root`, `#app`, `#__next`, `#__nuxt`) and measuring visible text length. A page with fewer than 500 characters and 80 words of visible text fails.

The fix is server-side rendering. Next.js with App Router renders server components to HTML by default. Astro renders everything to HTML by default. Any page that needs to be machine-readable must be server-rendered.

### The text-to-markup ratio

A healthy page has at least 5% visible text relative to total HTML. When a page is mostly `<div class="...">` wrappers and `<script>` tags with minimal content, the ratio drops below this threshold. This does not necessarily mean JS-only rendering, but it is a strong signal of content-light pages that give agents little to work with.

Write content-dense pages. Expand thin pages. An agent deciding whether to cite your site will not cite a page that is mostly UI chrome and navigation.

### Semantic landmarks are the page's skeleton

AI agents parse semantic landmarks to understand page structure before they read content. `<main>` identifies the primary content region. `<article>` identifies a self-contained piece of content. `<nav>` contains navigation. `<header>` and `<footer>` are structural.

Without semantic landmarks, agents must guess which of hundreds of divs contains the content they are looking for. With them, agents can immediately locate the primary content region with `document.querySelector('main')`.

ax-audit expects at least 3 of: `<main>`, `<article>`, `<section>`, `<header>`, `<footer>`, `<nav>`.

Never use `<div>` where a semantic element fits. This is not a nice-to-have. It is a structural requirement for machine readability.

### The single H1 rule

One `<h1>` per page. It tells every agent, human and machine, what this page is about. Multiple H1s create ambiguity. No H1 means the page has no declared primary topic.

The H1 text is what agents use to understand page subject matter. Make it specific and descriptive, not clever or minimal. "Getting Started with Acme Analytics" is better than "Getting Started."

### The noscript fallback

A page with more than 15 executable `<script>` tags and no `<noscript>` fallback gets flagged. If JavaScript is critical to your site's functionality, provide a `<noscript>` block that either explains the JS requirement or offers a minimal static alternative. This catches agents that operate with JS disabled.

---

## Meta Tags: The Cross-Consumer Signal

### OG tags are for agents too

Open Graph tags were designed for social sharing, but AI agents read them to understand page identity. The `og:title`, `og:description`, and `og:url` fields provide a concise, structured summary of any page. When an agent needs to quickly determine page relevance, OG tags are faster to parse than body content.

Every page needs:

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Page Title | Site Name</title>
<meta name="description" content="120 to 160 character description specific to this page" />
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

### og:image is not optional

ax-audit checks for `og:image` and flags its absence. An image URL that returns 404 is worse than no image. Create a static `/og-image.png` at minimum (1200x630px). For dynamic pages, generate OG images programmatically.

### The canonical is a deduplication signal

The `canonical` URL tells agents which URL is authoritative when content appears at multiple paths. Without a canonical, duplicate content (www vs non-www, trailing slash vs none, query parameters) can confuse agents about which URL to cite or index. Every page must have a canonical pointing to its definitive URL.

**Next.js metadata API:**

```tsx
export const metadata: Metadata = {
  title: { default: 'Site Name', template: '%s | Site Name' },
  description: 'Site description',
  metadataBase: new URL('https://example.com'),
  openGraph: {
    type: 'website',
    siteName: 'Site Name',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'Site Name' }],
  },
  twitter: { card: 'summary_large_image' },
  robots: { index: true, follow: true },
};
```

---

## security.txt: The Trust File

`/.well-known/security.txt` is how you declare that your site has an active security contact. For automated systems, it signals that there is a human or team maintaining the site and responding to issues. It is the minimum required by RFC 9116.

```
Contact: mailto:security@example.com
Expires: 2027-12-31T23:59:00.000Z
Preferred-Languages: en
Canonical: https://example.com/.well-known/security.txt
Policy: https://example.com/security-policy
```

**The expiry trap:** ax-audit checks that `Expires` is in the future. A security.txt with a past expiry date fails. Set it at least one year out and update it annually. The format must be ISO 8601 with timezone: `2027-12-31T23:59:00.000Z`.

`Contact` and `Expires` are the only required fields. Everything else is recommended. Both must be present.

---

## Well-Known AI Files: Emerging Signals

These files do not yet have settled specs, but they are already published by leading sites and read by some agents. Their combined weight in ax-audit is 3, but publishing them costs almost nothing and signals that your site is ahead of the curve.

### /.well-known/ai.txt (Spawning AI format)

Declares your opt-in or opt-out stance on AI training:

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

### /agents.json (OpenAgents / Wildcard)

Describes your site as a callable agent with operations:

```json
{
  "name": "Site Name",
  "description": "What agents can do with this site",
  "operations": [
    {
      "name": "search",
      "description": "Search site content"
    }
  ]
}
```

### /.well-known/ai-plugin.json (legacy ChatGPT plugin format)

Still consumed by some agents. Keep it for backwards compatibility:

```json
{
  "schema_version": "v1",
  "name_for_model": "site_name",
  "name_for_human": "Site Name",
  "description_for_model": "Use this plugin to search and retrieve information from Site Name",
  "description_for_human": "Search Site Name content",
  "api": {
    "type": "openapi",
    "url": "https://example.com/openapi.json"
  }
}
```

### /.well-known/nlweb.json (Microsoft NLWeb)

Natural-language site interface declaration:

```json
{
  "version": "1.0",
  "name": "Site Name",
  "description": "What this site does",
  "endpoint": "https://example.com/nlweb"
}
```

---

## OpenAPI: API Discoverability

If your site exposes any API, publish an OpenAPI spec at `/openapi.json`. This is how agents discover what programmatic actions they can take with your site.

**Required fields:**

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "Site Name API",
    "description": "What the API does and what agents can accomplish with it",
    "version": "1.0.0"
  },
  "servers": [
    { "url": "https://example.com/api", "description": "Production" }
  ],
  "paths": {
    "/search": {
      "get": {
        "summary": "Search content",
        "description": "Search site content by keyword. Returns ranked results with titles, descriptions, and URLs.",
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

Every path description should explain what an agent would use this endpoint for, not just what it does technically.

Link your OpenAPI spec from `llms.txt` and from `agent.json`'s `documentationUrl`.

---

## MCP: Direct Tool Integration

The Model Context Protocol (MCP) is how reasoning models like Claude integrate your site as a first-class tool. Implementing MCP means agents can call your site's capabilities directly from within their reasoning loop, not just browse it.

`/.well-known/mcp.json` declares your MCP server configuration:

```json
{
  "name": "Site Name MCP Server",
  "description": "MCP server for Site Name. Provides tools for searching content, retrieving structured data, and querying the API.",
  "protocolVersion": "2024-11-05",
  "tools": [
    {
      "name": "search",
      "description": "Search site content by keyword or topic. Returns titles, descriptions, and URLs of matching pages.",
      "inputSchema": {
        "type": "object",
        "properties": {
          "query": {
            "type": "string",
            "description": "Search query"
          }
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

**Tool descriptions are the difference between used and ignored.** An agent deciding which tool to call reads the description, not the name. "Get data" is useless. "Retrieve the current pricing plan for a given account ID, including feature limits and billing cycle" is actionable.

Serve `/.well-known/mcp.json` with `Access-Control-Allow-Origin: *`.

---

## TLS and HTTPS

All HTTP requests must redirect to HTTPS with a 301 (permanent) redirect. Agents treat non-HTTPS sites as untrustworthy. A 302 (temporary) redirect is incorrect: it tells crawlers the HTTP version is still valid.

HSTS must be present: `Strict-Transport-Security: max-age=31536000; includeSubDomains`

This means:
- No HTTP endpoints that return content instead of redirecting
- No mixed content (HTTP resources on HTTPS pages)
- Valid TLS certificate (not self-signed in production)
- 301, not 302, for HTTP-to-HTTPS redirects

---

## Sitemap: The Crawl Map

The sitemap is how agents discover the full surface area of your site. It is not just for Google. Any agent that wants to systematically understand what pages exist will read your sitemap.

**Next.js:**

```ts
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const base = 'https://example.com';

  // Fetch dynamic routes
  const posts = await getAllPosts();

  return [
    { url: base, lastModified: new Date(), changeFrequency: 'monthly', priority: 1 },
    { url: `${base}/docs`, lastModified: new Date(), changeFrequency: 'weekly', priority: 0.9 },
    ...posts.map((post) => ({
      url: `${base}/blog/${post.slug}`,
      lastModified: new Date(post.updatedAt),
      changeFrequency: 'monthly' as const,
      priority: 0.7,
    })),
  ];
}
```

Every public URL must be in the sitemap. The `Sitemap:` directive in robots.txt must point to it.

---

## Content Writing for Agents

### Specificity is the quality signal

Agents evaluate whether to cite your content by reading it. Generic descriptions and vague headings signal low-information content. Specific, factual, detailed content signals a reliable source.

Apply this test to every description you write: could this description apply to 100 other sites, or only to yours? If it could apply to 100 others, it is too generic.

"We help businesses grow" applies to every consulting firm on Earth.

"We help Shopify stores with 1,000 to 50,000 monthly orders reduce cart abandonment through behavioral email sequences and retargeting" is specific to a narrow domain with clear parameters.

### Internal linking is agent navigation

Links inside your content are how agents navigate. `<a href="/docs/api">API documentation</a>` tells an agent there is more relevant content at `/docs/api`. These signals compound in llms.txt (every link is a navigation hint) and in your HTML (every internal link extends the agent's understanding of the site graph).

Make anchor text descriptive. "Click here" tells an agent nothing. "REST API reference" tells an agent exactly what to expect.

### Every image needs a real alt attribute

`alt=""` is correct for decorative images. `alt="Chart showing monthly revenue growth from $12k to $89k over 18 months"` is correct for content images. Agents that process images use alt text to understand what the image depicts. Agents that do not process images still need alt text to know what information they are missing.

---

## Process

When starting or working on a web project in AX mode:

1. **Detect stack**: read `package.json`, framework config files, dependencies. Identify Next.js, Astro, or generic static setup.
2. **Generate mandatory files**: llms.txt, robots.txt, /.well-known/agent.json, /.well-known/security.txt, /.well-known/ai.txt. Use real site content, not placeholders.
3. **Configure headers**: add security + discovery headers via the correct config for the detected stack. Add CORS on well-known resources.
4. **Add structured data**: JSON-LD with @graph in the root layout. WebPage + BreadcrumbList on individual page templates.
5. **Verify HTML rendering**: confirm the stack renders content server-side. Detect empty SPA shells and flag them for fix.
6. **Write semantic HTML**: use correct landmark elements in every layout and component.
7. **Complete meta tags**: title template, description, canonical, OG, Twitter on every page type.
8. **Add sitemap**: cover all public routes, link from robots.txt.
9. **Keep files current**: when adding routes or features, update llms.txt sections, sitemap, and agent.json skills.

---

## Review Checklist

When auditing a web project for AEO compliance:

| Issue | Fix | Check |
| --- | --- | --- |
| `/llms.txt` missing | Create with H1 title, blockquote description, sections, links | llms.txt |
| `/llms.txt` has no blockquote `>` | Add `> description` after H1 as second non-blank line | llms.txt |
| `/llms.txt` description is generic | Rewrite with specific domain, scale, and audience details | llms.txt |
| No explicit AI crawler rules in robots.txt | Add User-agent + Allow for all 8 core crawlers at minimum | robots.txt |
| Wildcard Disallow blocks AI crawlers | Add explicit Allow per AI crawler above the wildcard rule | robots.txt |
| No `Sitemap:` in robots.txt | Add `Sitemap: https://example.com/sitemap.xml` | robots.txt |
| Empty `<div id="root">` in static HTML | Enable SSR or SSG, server-render primary content | HTML Rendering |
| Fewer than 500 chars visible text | Server-render the page, do not client-render primary content | HTML Rendering |
| No semantic landmarks | Replace divs with `<main>`, `<article>`, `<header>`, `<nav>`, `<footer>` | HTML Rendering |
| Multiple `<h1>` tags | Keep one H1 per page, demote others to H2 | HTML Rendering |
| No JSON-LD structured data | Add `<script type="application/ld+json">` with @graph in `<head>` | Structured Data |
| JSON-LD is invalid JSON | Fix syntax, validate at validator.schema.org | Structured Data |
| Only one entity type in JSON-LD | Add WebSite + Organization or Person at minimum | Structured Data |
| No BreadcrumbList | Add BreadcrumbList to @graph on every non-homepage | Structured Data |
| Missing `Strict-Transport-Security` | Add HSTS header in server or CDN config | HTTP Headers |
| Missing `X-Content-Type-Options` | Add `X-Content-Type-Options: nosniff` | HTTP Headers |
| No Link header for AI discovery | Add Link header pointing to llms.txt and agent.json | HTTP Headers |
| No CORS on `/.well-known/` | Add `Access-Control-Allow-Origin: *` on well-known routes | HTTP Headers |
| `/.well-known/agent.json` missing | Create with name, description, url, skills, protocolVersion | agent.json |
| `agent.json` url field wrong origin | Match url field exactly to the audited site's origin | agent.json |
| `agent.json` skills array empty | Add skills that describe specific agent-usable capabilities | agent.json |
| No `og:image` | Create static og-image.png (1200x630) and reference it | Meta Tags |
| Missing canonical | Add `<link rel="canonical" href="...">` on every page | Meta Tags |
| Generic page description | Rewrite to be specific to the page's actual content | Meta Tags |
| `security.txt` missing | Create at `/.well-known/security.txt` with Contact and Expires | security.txt |
| `security.txt` Expires is past | Update Expires to a future ISO 8601 date | security.txt |
| HTTP does not redirect to HTTPS | Configure 301 redirect from HTTP to HTTPS | TLS/HTTPS |
| No HSTS header | Add `Strict-Transport-Security: max-age=31536000` | TLS/HTTPS |

---

## Validation

After setup or any significant feature, validate with:

```bash
npx ax-audit@latest <site-url>
```

Target: every individual check scores 80 or above. Overall grade A (90+). Any check below 80 is a regression to fix before the work is done.

Run with `--format json` to get machine-readable results for CI integration:

```bash
npx ax-audit@latest <site-url> --format json | jq '.results[] | select(.score < 80)'
```

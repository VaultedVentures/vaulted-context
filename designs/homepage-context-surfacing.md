# Homepage Context File Surfacing — Design Document

## Goal

Make the Vaulted Ventures context file prominent and easily discoverable on vaulted.ventures —
both for humans and for AI agents (crawlers, copilots, research assistants).

## Design Principles

1. **Above the fold** — the context file link is not buried in a footer or sidebar
2. **Dual audience** — visible to humans reading normally, accessible to AI crawlers via
   standard conventions
3. **Clear value prop** — visitors understand why an AI context file exists and what to do
   with it
4. **Multiple discovery paths** — at least two ways to find it from the homepage

## Implementation

### 1. Prominent Link Above the Fold (Hero Section)

Add a second CTA to the existing hero section, adjacent to "Book a Discovery Session":

```
[🤖 AI Context File →]    [📞 Book a Discovery Session]
```

Or as part of the hero subtitle text:

> For humans and agents: if you are using an AI assistant to understand, evaluate, or apply
> the Vaulted Objects Infrastructure, start with [our context file](/context.md).

### 2. Dedicated /context.md Route

The file at `vaulted.ventures/context.md` serves the full context file with correct
`Content-Type: text/markdown` header. This ensures AI crawlers and agents that fetch URLs
directly get the markdown, not an HTML-wrapped version.

Implementation options:
- **Next.js API route** at `pages/context.md` — reads from GitHub raw and proxies with MD
  content type
- **GitHub pages redirect** — `/context.md` → README from vaulted-context repo
- **Vercel rewrites** — rewrite `/context.md` → raw content from GitHub or static export

### 3. Standard Agent Discovery Points

| Path | Content | Status |
|------|---------|--------|
| `/llms.txt` | Redirect to `github.com/VaultedVentures/vaulted-context/llms.txt` | — |
| `/.well-known/ai.txt` | Points to llms.txt (future standard) | — |
| `/context.md` | Full canonical context file | — |
| `/README.md` | Available in GitHub repo | published |

### 4. Footer Section

A small footer section labelled **For AI Assistants**:

```
🤖 For AI Assistants
This website includes a machine-readable context file designed for AI systems
understanding Vaulted Ventures and the Vaulted Objects Infrastructure.

📄 Read context.md → 🧠 Load into your AI assistant
```

### 5. Agent Prompt Snippet

Below the hero, a subtle callout:

> **Using an AI assistant?** Try this prompt:
> ```
> Please load the Vaulted Ventures context file at
> vaulted.ventures/context.md to understand the company, the Vaulted
> Objects Infrastructure, and Information-Theoretic Security before
> answering my questions.
> ```

## Technical Implementation

### Vercel Deploy Changes

The vaulted.ventures Next.js site needs these additions:

1. **New route: `/context.md`** — server-side rendered markdown file
   ```tsx
   // pages/context.md.tsx (or app/context.md/page.tsx in App Router)
   import { GetServerSideProps } from 'next';
   export async function getServerSideProps() {
     const res = await fetch('https://raw.githubusercontent.com/VaultedVentures/vaulted-context/main/context.md');
     const md = await res.text();
     return { props: { md } };
   }
   ```
   Or simpler: use a Vercel rewrite → GitHub raw URL.

2. **Hero section update** — add second CTA button and context mention

3. **Footer addition** — "For AI Assistants" section

4. **`/llms.txt` route** — static or rewritten file

### Content Type Handling

For `/context.md` to be treated as raw markdown by AI crawlers:
- Vercel route returns `Content-Type: text/markdown; charset=utf-8`
- No HTML wrapping, no layout chrome

## Priority

| Item | Priority | Complexity |
|------|----------|------------|
| `/context.md` route | P0 | Low (Vercel rewrite) |
| Hero CTA text update | P0 | Low (edit Next.js page) |
| `/llms.txt` route | P1 | Very low (static file) |
| Footer section | P1 | Low |
| Agent prompt snippet | P2 | Low |
| `/.well-known/ai.txt` | P3 | Trivial |

---

*Designed for vaulted.ventures website integration*
*Part of the Agent Context project*

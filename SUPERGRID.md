# SuperGrid

> Working doc for exploring and building SuperGrid. Nothing here is final. Each area is a starting point for brainstorming, and the open questions matter as much as the answers.

---

## 0. Hackathon context

**Event:** [Meta Global AI Developer Hackathon](https://cerebralvalley.ai/e/meta-ai-developer-hackathon) (remote)
**Dates:** Oct 26 – Nov 4, 2026. **Submissions due Nov 4, 9:00 PM ET.** Judging Nov 4–8, finals Nov 9.
**Models and tools:** Muse Spark series, Muse Wonder, Muse Image, SAM 3.1 (through the Meta Model API), Meta open-weights models (Baseten / Modal / Fireworks credits), Muse Code coding agent.
**Tracks:**
- Indie: judged on creativity, technical ambition, and whether it works.
- Startup: judged on technical execution and whether it holds up in production.

**Theme fit:** "Build for Your World": a problem known firsthand (20 years of SaaS, ContentBlocks, OutcomeApp, and the clone problem happening to those products right now).

**Decisions**
- **Track: Startup.** The bar is technical execution and whether it holds up in production, so the core path has to be real, not mocked.
- **Muse models (Meta Model API) power the in-app AI:** the web app's agents and AI features, running on Cloudflare (Workers, AI Gateway).
- **Muse Code is the local story:** the demo shows Muse Code changing the site locally, then pushing to deploy.

**What this means for scope**
- Meta models are visible in the product: Muse is the AI inside SuperGrid, and Muse Code is the AI on the user's machine.
- About 8 days. The demo needs one complete end-to-end path more than broad coverage (see §18).

---

## 1. Why: history and the problem

**Background:** I built ContentBlocks (courses, AI agents for education/coaching) and OutcomeApp (the "Outcome Funnel"). Both are traditional SaaS: log in, use the UI, build a thing, launch it, get a URL, add a domain. Stop paying and you lose access.

**What changed:**
- Users don't want to pay for smaller apps.
- They have their own AI (Claude, Codex, Muse…) "clone" the app as a blueprint for their own course or funnel.

**Why cloning is bad for everyone:**
- **For builders:** it's hard to run a business when the product can be copied from what's visible. Even a demo video becomes a prompt. Games are getting cloned from trailers before launch. So builders stop building in public.
- **For users:** they copy the *visible* parts and lose the *nuance*: the taste, reasoning, and decisions behind it. The AI infers a lot, but not that.
- **Where it ends up:** another vibe-coded app on its own URL, disconnected from everything else they run. They get ownership, but it's fragmented and doesn't work well.

**The opportunity:** agent coding opens the door for non-technical experts (coaches, specialists) who have a real solution and real taste. Many already build their own "SaaS" in Lovable, Overskill, or Claude, rebuilding auth, databases, and workflows for one niche problem, often bloating it with features. Then they hit the same problem real SaaS companies have: they get copied.

**The question:** how can SuperGrid go *with* this new behaviour instead of against it?

---

## 2. Who: the avatar

A **ClickFunnels / GoHighLevel user**:
- Usually a single creator.
- Markets their own services (coaching, community, offers), markets for clients, or both.
- Non-technical to semi-technical, increasingly using AI to build.
- Secondary: **experts and creators** who want to package their solution/taste and get paid for it (the supply side of the marketplace).

---

## 3. What the audience wants

Some of these come from my own view, some I've observed, and some people have told me directly.

| Want | What it means | Why |
|---|---|---|
| **Ownership** | Their own data (database, CRM) and their own server, which is their own Cloudflare account (most AI builders like Lovable already run on Cloudflare) | Control, and it's nearly free at low/moderate scale |
| **Their own AI** | Use the subscriptions they already pay for (Claude, Codex, Muse) where possible, their own API keys where not | They dislike "token resellers" (Lovable, Overskill) that charge premiums on credits. API pricing for top models is also higher than personal subscriptions. They want to be as close to cost as possible |
| **Quality** | Things work, and work well | Cloning and vibe coding lose the nuance and lower quality. **This is the core of SuperGrid.** |

---

## 4. The shift: SaaS then vs. now

| | **Then** | **Now** |
|---|---|---|
| **Code** | Expensive, specialised, devs were costly | Nearly free and fast. Everyone can "code" "anything" |
| **Infra** | Technical, hard, expensive DevOps. Had to be built for *everyone scale*: multi-tenant, every user paying for everyone else, noisy neighbours | Nearly free at *you scale* (Cloudflare, Vercel). Most sites get modest traffic. Users see hosting as an unnecessary cost, not a benefit |
| **Solution / UX** | *The* product: the UI, the opinions, the feel. Founders were the DNA; quality = their taste | Agents are now the main users, so human editing UIs (funnel builders, block editors) get used less. Design is getting cheap too. People want **focused point solutions**, not bloated do-everything products (GHL had to build every feature because it was all shared) |

**What stays valuable:** the insight, taste, and quality of the solution. Care and thought go into the *unseen* parts. Voice-to-text transcription vs. Granola is the example: the difference is in parts that aren't obvious, and AI cloning can't see into them.

---

## 5. Thesis: what SuperGrid is

SuperGrid is **the connector and UX layer** between:
1. **The user's own infrastructure** (their Cloudflare, their data),
2. **The user's own AI** (their subscriptions/keys), and
3. **Quality solutions from experts** (plugins that carry taste, and reward the people who made them).

| Pillar | SuperGrid answer |
|---|---|
| Code | Use your own AI subscriptions; own your AI API costs |
| Infra | Hosted on your own Cloudflare: your workers, your D1, your repos |
| Quality | A safe way to integrate thoughtful solutions you own, while rewarding their creators (plugins + marketplace) |

**Ownership is built in:** plugins become part of *your* code on *your* worker. If you leave SuperGrid, it's still yours.

**Start small, add only what fits:** a few ecom funnels, or coaching + social growth, or a niche SaaS. No paying for features you don't use.

---

## Areas to build

The rest of this doc is the top-level areas. Each has **what it is** (from my notes so far) and **open questions** to work through.

---

## 6. Web app (SuperGrid control plane)

**What it is**
- The main web app, owned and hosted by me.
- Sign up → connect Cloudflare account.
- Create and manage workspaces.
- In-browser editing of site/backend, using AI through the user's own Cloudflare AI Gateway (their tokens).
- Plugin marketplace, expert profiles, installs, purchases.
- Accessible for non-technical users; the "it just works" path.

**Open questions**
- How does Cloudflare connect work? OAuth vs. scoped API token. What permissions are needed, and how is trust communicated?
- What does SuperGrid store itself vs. what lives only in the user's account?
- What's the editing experience in the web app: chat-first, visual preview, file tree, all three?
- Who pays SuperGrid, and for what? (Marketplace cut? Subscription for the control plane? Both?)

---

## 7. Workspaces / organizations

**What it is**
- Users create workspaces to organise their work (their own business, each client, each product).
- Each workspace has its own resources on the user's Cloudflare: D1, worker(s), repos, etc.

**Open questions**
- Is a workspace = one site? Or can one workspace hold several sites/funnels?
- Agency case: a marketer running workspaces for clients. Whose Cloudflare account? Can a workspace be handed over to the client?
- Team members / collaborators per workspace?
- Naming/provisioning conventions so resources are easy to find in the user's Cloudflare dashboard.

---

## 8. Site / frontend

**What it is**
- The core site worker: **Astro** + a design system (**base-ui**), components, sections, layouts, pages.
- By default, just the basics that make it run.
- Its own **Cloudflare Artifact (git)** repo; deploys on push.
- Two ways to edit:
  - **Locally:** their own Claude Code / Codex / Muse Code, test locally, build pages and components, push.
  - **In the web app:** AI via their own Cloudflare AI Gateway.

**Open questions**
- What's in the base starter exactly? (Pages, layouts, sections, theme tokens, SEO, forms?)
- How are themes/design systems swapped (as plugins?)
- How does the frontend talk to the backend: service bindings, RPC, HTTP?
- Preview environments per branch?

---

## 9. Backend / system

**What it is**
- A backend repo, probably another worker connected to the frontend.
- Handles events, workflows, queues, data access, and other backend concerns.
- Same idea as the site: Artifact repo, git, deploy on push.

**Open questions**
- One backend worker or several? Where do plugin backends live, inside it or as their own workers?
- Core event model: what events exist by default (page view, form submit, purchase, signup…)?
- Workflow primitives: Cloudflare Workflows, Queues, Durable Objects. Which is used for what?
- How do triggers/hooks work so plugins can react to core events?

---

## 10. Data

**What it is**
- **D1** core database: CRM (contacts, tags, notes), analytics, logging.
- **Durable Objects** for stateful/realtime needs.
- Plugins can add their own tables or Durable Objects (e.g. form submissions, course progress).

**Open questions**
- Core schema v0: contacts, events, pageviews, forms, orders? What's the minimum?
- How do plugin schemas get added safely (migrations, namespacing, conflicts)?
- Analytics: raw events in D1, or Cloudflare Analytics Engine?
- Data export / portability story (part of ownership).

---

## 11. Agent guidance layer (structure, skills, docs, guardrails)

This came up in both the site and backend sections, but it may be **the most important area**, because it's how quality survives when the user's AI edits the code.

**What it is**
- Both repos ship with a file structure, skills, docs, and rules that tell any agent (Claude Code, Codex, Muse Code, the web app's agent):
  - what's **safe to customise** vs. **core/don't touch**,
  - how to work within the framework (add a page, add a section, add a workflow step, add a plugin),
  - the *why* behind the decisions.

**Open questions**
- Format: `AGENTS.md` / `CLAUDE.md`, skills folders, per-directory rules, MCP tools, or all of these?
- How do we enforce boundaries? (Lint rules, CI checks, protected paths, tests?)
- Can the guidance be agent-agnostic so it works the same with Claude, Codex, and Muse?

---

## 12. AI layer (bring your own AI)

**What it is**
- **Local:** user's own subscriptions (Claude Code, Codex, Muse Code), working directly on the repos.
- **Web app:** user's own API keys, routed through **their Cloudflare AI Gateway** (their account, their costs, their logs).
- No token reselling and no markup.

**Hackathon build: Muse + Cloudflare**
- **In-app agents and AI features run on Muse models** through the Meta Model API, called from Cloudflare Workers and routed through the user's AI Gateway.
  - Muse Spark: web app agent (site edits, plugin setup), plugin agents (e.g. copywriting).
  - Muse Image: images for pages and sections.
  - Muse Wonder: candidate for heavier reasoning/planning tasks (TBD).
- **Muse Code is the local agent** for the demo (see §16).

**Open questions**
- Agent loop: our own loop on Workers (Durable Object per session?) vs. something else?
- How do plugins that *are* agents (copywriting agent, sales demo agent) get their model access? Always the user's key?
- Model routing: which Muse model for which task (code edits vs. copy vs. images)?
- After the hackathon: keep Muse as the default, with other providers as options?

---

## 13. Plugin system

**What it is**
Plugins extend a workspace. Start small, add exactly what you or your client needs.

**Plugin types** (one plugin can be several):
- **Frontend:** design theme, custom components, experiences (Outcome Funnel, course/membership area)
- **Backend:** routes, systems, workflow steps, triggers/hooks
- **Data schema:** its own tables / Durable Objects
- **Integration:** a 3rd-party connection
- **Admin / management:** admin UI, data views, agent chat components, MCP calls (a plugin can be admin-only, e.g. a simple data dashboard)
- **Agent:** skills, system prompt, RAG/knowledge

**Distribution models**
- **Copy-in** (shadcn style): code lands in your repo and you own it fully.
- **Managed** (npm package or similar): the creator keeps it updated (official plugins, "managed plugins").

**Examples from my notes**
- Official **auth + access/entitlements + billing** plugin for a niche SaaS.
- Official **Course / Membership** plugin (free): adds screens, components, data, and backend. It becomes part of your code.
- An expert's (e.g. Dan Martell) **internal plugins**: sales demo system, copywriting agent, etc.

**Open questions**
- **Plugin manifest:** what does a plugin declare (types, routes, tables, events it listens to, admin screens, required integrations, env/secrets)?
- **Install mechanics:** who applies it, the user's agent, the web app, or a CLI? How are conflicts with user edits handled?
- **Updates:** copy-in plugins that the user's AI has since modified. How do updates merge? (The classic shadcn problem.)
- **Taste transfer:** a plugin should ship *code + the reasoning* (decision docs, skills, agent guidance) so the user's AI can extend it without breaking what makes it good. That's SuperGrid's answer to "cloning loses the nuance."
- **Plugin isolation:** what can a plugin touch? Permissions model?

---

## 14. Marketplace and expert profiles

**What it is**
- Discover plugins from experts. Free or paid, sold singly or in bundles.
- Expert profiles: who they are, all their tools and plugins.
- Trust funnel: add a free plugin → like it / like them → buy more (paid plugins, bundles).
- Rewards creators for their taste, the opposite of being cloned.

**Open questions**
- **The core tension:** if a paid plugin is copied into the buyer's repo and they own it, what stops it from being redistributed or re-cloned? Possible angles:
  - value lives in *updates* and *managed* parts (ongoing improvement),
  - value lives in parts that run on the creator's side (agent knowledge/RAG, data, services),
  - licensing + reputation + convenience (it's easier to buy than to steal),
  - accepting some leakage the way music and software already do.
- Payments: who processes them, what cut SuperGrid takes, and payouts to creators.
- Quality bar / review process for marketplace listings?
- How do experts *build* plugins? (A plugin SDK/template, built with their own AI too.)

---

## 15. Integrations and connectors

**What it is**
- Connections to the user's 3rd-party systems (email, payments, calendars, social, CRMs, etc.).

**Open questions**
- Built into core, or always plugins?
- Where do secrets live? (Cloudflare secrets on the user's worker, ideally.)
- Which integrations matter most for the avatar? (Stripe, email provider, calendar, Meta/Instagram?)

---

## 16. Local dev bridge (desktop app / MCP / CLI)

**What it is**
- A way for local Claude Code (Codex, Muse Code, etc.) to manage a workspace.
- Maybe a slim desktop app, an MCP server, and/or a CLI for:
  - dev previews,
  - pushing / deploying,
  - pulling skills/rules,
  - installing plugins.
- **Hackathon demo: Muse Code.** Clone the workspace's site repo, have Muse Code make a change guided by the repo's agent guidance (§11), preview locally, push → deploys to the user's Cloudflare.

**Open questions**
- MCP-first vs. CLI-first vs. desktop app. Which is smallest for the hackathon?
- Local preview of both workers + D1 together (wrangler dev?)
- How does the local setup stay in sync with web-app edits (it's all git, so pull/push)?

---

## 17. Ownership and exit story

**What it is**
- Everything (code, data, workers) lives on the user's Cloudflare.
- Leaving SuperGrid means losing the connector/UX/marketplace, **not** the site, data, or installed plugins.

**Open questions**
- What exactly breaks if they leave? Managed plugin updates? Web app editing? Anything at runtime? (Ideally nothing at runtime.)
- Make this a visible selling point ("eject" / "you already own it").

---

## 18. Hackathon slice (Startup track)

One end-to-end story beats many half-built areas. Startup track means this path has to really work on a real Cloudflare account.

1. Sign up → **connect Cloudflare**
2. Create a **workspace** → provision site worker + backend worker + D1 + repos
3. Edit the site in the web app with the **Muse-powered agent** (Meta Model API via the user's AI Gateway)
4. Browse the marketplace → **install a plugin** (e.g. Outcome Funnel or Course/Membership), which adds pages, tables, backend, and agent guidance. Its AI features also run on Muse.
5. Open the same repo locally in **Muse Code**, make a change, push → deploys
6. Live URL on their own Cloudflare. "This is yours. Leave anytime and it keeps running."

**Open questions**
- Which plugin is the hero for the demo? (Outcome Funnel is my own work and taste, so it makes a strong story.)
- What must be production-solid vs. what can be thin (e.g. marketplace payments)?

---

## Parking lot

Ideas and questions that don't have a home yet:

- Business model, license and pricing: see [BUSINESS-MODEL.md](BUSINESS-MODEL.md).
- Accounts, workspaces, sites, hosts and the open-core split: see [TENANCY.md](TENANCY.md).

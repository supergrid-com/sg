# SuperGrid tenancy and distribution

> How the system is shaped so one codebase serves a single creator, an agency, a reseller and our own managed offering. Companion to [BUSINESS-MODEL.md](BUSINESS-MODEL.md) (money and license) and [SUPERGRID.md](SUPERGRID.md) (product). Decisions are the current direction; open questions are at the end.

---

## 1. The one idea

**Everything is an operator running the same control plane; the only variables are who hosts the control plane, where a workspace's resources live, and whether strangers can sign up.**

- A **single creator** uses our hosted control plane (free) and their own Cloudflare.
- An **agency** uses the same control plane with many workspaces, one per client.
- A **reseller** runs their own copy of the control plane with their brand, signup turned on, and customers' resources provisioned into the reseller's Cloudflare.
- **SuperGrid managed** is a reseller instance operated by us at supergrid.com.

So "managed" and "reseller" are not two products. They are one mode of the control plane (**operator-hosted**) that we happen to run first. Everything built for managed (signup, plans, metering, provisioning into a shared account, eject) is what a reseller gets.

---

## 2. Hierarchy

Three levels, not two. The middle one answers the "namespace" question.

```
Operator                 the party running a control-plane instance (us, or a reseller)
└── Account              a person or team that signs in; identity + billing owner
    └── Workspace        the tenant: ONE set of infrastructure, shared by its sites
        └── Site         a hostname with its own pages, routes and theme
```

### Account
Who logs in and who pays. Has members (team) with roles. An account can hold many workspaces. For a creator the account is them; for an agency it is the agency; in a reseller's instance each customer is an account.

### Workspace (the tenant / namespace)
**Decision: the workspace is the resource and data boundary, and it holds many sites.**

A workspace owns exactly one of each: D1 database, R2 bucket, KV namespace, AI Gateway, Artifacts repo, Worker. Inside it, every site shares:

- contacts / CRM, events, analytics, logs
- media
- themes, components, sections, plugins
- integrations and secrets
- hooks and workflows
- team members and roles

**Why this boundary:** a coaching business wants one CRM across its home site, its summit funnel and its course site. A dog-breeding business must not see those contacts. That is two workspaces, not two sites. It maps cleanly to a Cloudflare dashboard too: one `supergrid-<slug>` D1, one bucket, one Worker per business, easy to find and easy to hand over.

A workspace also carries the **host**: where its resources are provisioned (§4).

### Site
A site is a hostname (plus workers.dev / preview hostnames) with its own pages, routes, redirects, theme and settings, living inside the workspace's single repo and its public `site` Worker. Sites share the workspace's data; events and pages carry a `site_id`.

**Decision: two Workers per workspace (`site` and `admin`), with sites routed by hostname inside the `site` Worker, rather than one Worker per site.** One public Worker renders every site in the workspace; one admin Worker owns the data plane (§8). Per-site Workers would mean a deploy and a Builds trigger per site, and would burn the per-account Worker cap for agencies on the `own` host. A site can later be split out to its own app in the same repo if it needs separate scaling or deploys; that is an escape hatch, not the default.

---

## 3. Personas mapped onto the hierarchy

| Persona | Control plane | Accounts | Workspaces | Sites | Host (where resources live) |
|---|---|---|---|---|---|
| Single creator | app.supergrid.com, free (or self-hosted) | 1 | 1 per business (coaching, dog breeding) | many per workspace | `own`: their Cloudflare |
| Agency | same | 1 (agency team) | 1 per client, plus the agency's own | many per client | `own`: agency's Cloudflare, or the client's Cloudflare (transferable) |
| Reseller, self-hosted | their own instance, their brand and domain | 1 per customer (self-serve signup) | per plan, usually 1 | per plan | `operator`: the reseller's Cloudflare |
| SuperGrid managed | app.supergrid.com | 1 per customer | per plan | per plan | `operator`: SuperGrid's Cloudflare |
| Managed reseller | an instance we run at their domain | 1 per customer | per plan | per plan | `operator`: SuperGrid's Cloudflare, pooled per reseller |

**Reseller's "namespace" answer:** the reseller does not get "many workspaces". The reseller gets an *operator instance*; their customers get accounts, and each account gets workspaces. The reseller sees all of it from a partner console (§6).

**Agency "client = workspace":** yes. Whether that client has one site or ten is inside the workspace and costs nothing extra in structure. Handover is a workspace transfer (§5).

---

## 4. Host: where a workspace's resources live

Each workspace has a `host`:

| `host` | Provisioned into | Who pays Cloudflare | Used by |
|---|---|---|---|
| `own` | A Cloudflare account the account owner connected via OAuth | the user | free self-hosted, agencies |
| `operator` | The operator's Cloudflare account, in a Workers for Platforms dispatch namespace | the operator, metered back to the account | managed, resellers |

From the control plane's point of view `operator` is the *same* code path whether the operator is SuperGrid or MedSpaPro. The operator instance has one Cloudflare connection of its own (its "pool"), and workspaces land there.

### `own`
What exists today: `provisionWorkspace()` in `@supergrid/cloudflare` creates named resources in the user's account and sets a Workers Builds trigger on the Artifacts repo. Nothing changes except the naming now covers sites inside the workspace, not one site per workspace.

### `operator`
Same resources, same names, but:

- the Worker is uploaded into a **dispatch namespace** (`supergrid-hosted`, or `supergrid-hosted-<reseller>` for managed resellers) instead of the account's top level, with per-script `limits` (CPU ms, subrequests) and outbound rules;
- a **dispatch Worker** owned by the operator routes `<hostname>` → workspace script, resolving custom domains through **Cloudflare for SaaS** custom hostnames;
- D1, R2 and the Artifacts repo are created per workspace in the operator account, same as `own`. Bucket per workspace, not prefix per workspace, so eject is a bucket copy and limits are per bucket;
- Builds deploys into the namespace;
- usage is read per script, per database and per bucket and written to the control plane's `usage` table for metering.

**Decision: resources per workspace are identical across hosts.** That is what makes eject a move, not a rewrite.

### Never one big app

**Decision: on every host, every workspace is its own repo, its own D1, its own bucket and its own two Workers. Managed and reseller customers are not tenants in a shared application.** This is what Workers for Platforms is for: one script per customer, at any count. A shared app would put every customer's plugins and agent-written code in one process, make eject a data extraction instead of a move, and recreate the multi-tenant SaaS the thesis rejects.

What "managed" adds is only that the operator owns the Cloudflare account the resources sit in. Provisioning forks the template repo in Artifacts (`forkFrom`) instead of pushing it, so a new workspace is a fork, two Builds triggers and three resource creates. Shared platform code is not shared in git; it is `@supergrid/core` from npm, so the operator upgrades a fleet by bumping a version in each repo with a bot, and a customer who has edited their repo gets an ordinary dependency update, not a merge.

Per-account limits to verify before scale: Artifacts repos per account, AI Gateways per account (operator-hosted may need one shared gateway with per-workspace keys), Workers for Platforms per-script pricing. D1 (tens of thousands per account) and R2 buckets are not a constraint.

### Mixing hosts
An account may hold workspaces on different hosts. An agency can keep its own site on `own`, run a client on the client's connected Cloudflare, and run another on `operator` because that client wants nothing to do with Cloudflare. Billing is per workspace on `operator` host, free on `own`.

---

## 5. Lifecycle operations every host must support

These are the operations that make the business model honest. They are the same API on every host.

| Operation | What it does | Who uses it |
|---|---|---|
| **Provision** | Create the workspace's resources and push the starter repo | everyone |
| **Transfer** | Move a workspace to another account (same host) | agency → client handover; reseller customer leaves |
| **Migrate host** | Move a workspace's repo, D1, R2, secrets, domains from one Cloudflare account to another | eject from managed; agency → client's Cloudflare; client → managed |
| **Export** | Download repo, D1 dump, R2 files, config as a bundle | ownership promise, backups |
| **Deprovision** | Tear down, with keep flags for media / repo / database | exists today |

"Eject" is **Transfer + Migrate host(operator → own)**. It has to be one button and tested on every release; it is the proof behind "you own it".

---

## 6. What the operator instance is

The control plane is **operator software**. One deployment = one operator. Its config:

```
operator
  brand: name, logo, colours, domain, support email
  auth: provider (WorkOS by default), allowed signup (on/off, invite only, open)
  hosts: own (on/off), operator (on/off + pool Cloudflare connection + dispatch namespace)
  plans: [{ id, price, included: { workspaces, sites, requests, storage, ai }, overage rates }]
  billing: Stripe keys (operator's own)
  marketplace: on/off, source (supergrid.com, or none)
  partner console: on/off
```

**Decision: single operator per deployment.** Not a multi-operator database. Reasons: no operator scoping on every query, strong isolation between resellers, and a reseller's customer data never sits next to ours. A managed reseller is simply an instance we run for them, which is cheap once the control plane itself is a Worker: one Worker and one D1 per reseller on our account, their domain via Cloudflare for SaaS.

**Which configs are which product**

| Product | signup | hosts | plans/billing | brand |
|---|---|---|---|---|
| Free self-hosted (our app.supergrid.com) | open, free accounts | own | none for `own` workspaces | SuperGrid |
| Free self-hosted (someone's own deploy) | off or invite | own | none | theirs, allowed by FSL |
| SuperGrid managed | open | own + operator | ours | SuperGrid |
| Reseller | open | operator (optionally own) | theirs | theirs, needs reseller license |

The **license test** from BUSINESS-MODEL §2 becomes a config check: `signup: open` + `hosts.operator: on` + your brand = you are a reseller.

---

## 7. Open core split

**Decision: the control plane repo is private. The workspace template and the packages are open.** Rule: **the open part must be a complete product for a creator or agency. The private part is the thing that runs many of them.**

| Repo or package | License | Who has it |
|---|---|---|
| Workspace template (`site/`) | FSL → MIT after two years | everyone; forked per workspace |
| `@supergrid/core`, `@supergrid/ui`, `@supergrid/cloudflare`, CLI | FSL → MIT | everyone, from npm |
| `@supergrid/sdk`, plugin API, theme contract | MIT | everyone; plugin and theme authors pick their own license |
| Control plane (`app/`), including signup, plans, billing, metering, `operator` host, partner console | Commercial | us; operator licensees get source access |

No `@supergrid/operator` package to carve out. The whole control plane is the operator product, and the license gate is repo access rather than a runtime key. This is what the ownership promise allows: everything the customer runs (sites, admin, data, plugins) is in their open workspace repo; the control plane is the convenience layer for running many workspaces, and nothing in a workspace depends on it at runtime.

**Who needs the control plane source**

| Who | Gets | Pays |
|---|---|---|
| Creator | hosted app.supergrid.com with `own` host, or no control plane at all | nothing |
| Agency | hosted app.supergrid.com with unlimited workspaces on `own` host, plus the open CLI and `@supergrid/cloudflare` for scripting provisioning, fleet upgrades and transfers | nothing |
| Agency that wants its own brand on the admin, its own deployment, or to extend the control plane | operator license: source access, self-host, signup off | the one-time operator license |
| Reseller | the same operator license with signup on, plus the resources tier | license + resources |
| Managed reseller | an instance we run; no source needed | monthly |

So an agency only pays when it wants what a reseller wants: its own deployment. Automation does not require the source, because provisioning, transfer and upgrades are in the open packages the control plane itself uses. Extension mostly happens in the workspace (plugins with admin screens), which is open.

**What this changes in BUSINESS-MODEL.md:** "reseller license" becomes "operator license"; a self-hosted agency control plane moves from free to licensed; the free self-hosted path is the hosted control plane plus your own Cloudflare, or the template alone.

### The repos

Three different things ship from this codebase, and only one of them is ever copied per customer.

| Repo | Today | Lifecycle | Who runs it |
|---|---|---|---|
| **Workspace template** | `site/` | Forked once per workspace; after that it is the customer's and diverges | every workspace, on any host |
| **Control plane** (private) | `app/` | One deployment per operator; upgraded by the operator, never forked per customer | us at supergrid.com, each licensed operator (reseller or agency) |
| **Published packages** (`@supergrid/core`, `@supergrid/ui`, `@supergrid/cloudflare`, official plugins) | `app/packages/*` and parts of `site/src/server` | Versioned on npm; workspace repos and the control plane both depend on them | nobody runs them; they are dependencies |

The workspace repo and the control plane never share git history. They share npm packages. A workspace does not contain operator code, and the control plane does not contain site code. That is what keeps a customer's repo small and lets a self-hosted agency ignore the control plane entirely.

For development they can stay side by side in this one folder, as now. At release, the template is published as its own git repo (the thing Artifacts forks), and the packages go to npm.

Our managed deploy and a reseller's deploy are the same build with different config. This is the "shared growth" the business model wants: every managed feature is a reseller feature by construction.

---

## 8. The workspace repo

**A workspace is a repo.** One repo = one workspace = one data boundary, with as many sites inside as the owner wants. An account in the control plane can hold many of them, but the repo never knows about accounts.

**The control plane is optional.** A creator or agency can clone the template, add their sites and their clients' sites, and deploy with wrangler. No signup, no accounts, no SuperGrid servers; the repo is the product. The control plane exists to do this many times (provisioning, workspace switching, transfer, marketplace, the hosted agent) and to run the operator-hosted mode. An agency self-hosting chooses its own granularity: one repo holding every client as a site when the agency owns all the leads, or one repo per client when the client must be able to take it with them. Both are the same template.

**Decision: the workspace repo holds both the public sites and the admin that manages them, as a Turborepo with two deployable apps.** The admin is not a dashboard bolted onto the site. It is the management plane: logs, analytics, pages, content, CRM, data, media, plugins, workflows, settings, all exposed as an API and an MCP server that agents use, with screens as the human view of the same API. What the user deploys to Cloudflare is their whole system, not just the frontend.

```
<workspace repo>
  apps/
    site/            Astro SSR. Public sites, one folder per site, routed by hostname.
                     Emits events, calls the admin API through a service binding.
    admin/           React app on Worker assets + Hono. API, MCP server, queue consumer,
                     workflows, cron, admin screens. Owns the data plane.
  packages/
    plugins/
      course/        a copy-in plugin (shape below)
      contact-form/
  migrations/        merged core + plugin migrations (one ordered folder, D1 needs it)
  supergrid.config.ts   sites, theme, plugins (one list)
  AGENTS.md, .claude/skills, docs/   agent guidance
```

From npm, not copied: `@supergrid/core` (schema, events, hooks, workflows, analytics compiler, SDK, admin shell, MCP server), `@supergrid/ui` (shared with the control plane so plugin screens look native). Updating the platform is a version bump the agent can make. The user's code is sites, theme, plugins and config.

**Deploys as two Workers** sharing resources by id:

| Worker | Bindings | Does |
|---|---|---|
| `supergrid-<slug>-site` | D1 `DB` (read), R2 `MEDIA`, KV `CACHE`, Queue producer `EVENTS_QUEUE`, service `ADMIN` | Render sites, emit events, proxy form/API calls to admin |
| `supergrid-<slug>-admin` | D1 `DB`, R2 `MEDIA`, KV `CACHE`, Queue consumer, `WORKFLOW`, cron, `IMAGES`, `AI` + `AI_GATEWAY_URL`, secrets | API `/api/*`, MCP `/mcp`, admin UI, ingest, hooks, workflows, migrations on deploy |

Why two rather than one: an admin deploy does not redeploy the public sites, the public Worker stays small, admin auth is a separate boundary, and the two apps can use different frameworks. `provisionWorkspace()` already creates two Workers and two repos; the second one is renamed from `-backend` to `-admin`, and the two repos collapse into one (two Builds triggers on one repo, each with its own root directory). For the hackathon the admin can stay mounted under `/dashboard` inside the site Worker; splitting it out is moving `src/server`, the queue and workflow exports, and the dashboard pages into `apps/admin`.

### A plugin is one package with three entries

```
packages/plugins/course/
  package.json       exports: ./manifest ./site ./server ./admin
  manifest.ts        id, version, nav entries, settings schema, permissions, per-site enable
  site/              sections, components, Astro pages, islands      → imported by apps/site
  server/            Hono routes, events, hooks, workflow steps, MCP tools, integrations → apps/admin
  admin/             React screens built on @supergrid/ui, registered into the admin shell's router and nav → apps/admin
  db/                schema.ts (Drizzle) + migrations/              → synced into /migrations
  GUIDE.md           the reasoning and taste, read by agents before they edit
```

A Course plugin adds lesson sections and a members area to `site/`, a `course_*` data model, an API and MCP tools for progress and enrolment to `server/`, and a course manager screen to `admin/`. All four ship in one package and deploy in one push. Copy-in plugins live in `packages/plugins`; managed plugins are the same shape installed from npm. Both appear once in `supergrid.config.ts`, which replaces today's three registration lists.

### Changes this implies

**Site app (`apps/site`, today `site/`)**
- Add a `sites` table: `id, slug, name, hostnames (JSON), theme, settings (JSON), created_at`. One row seeded on provision.
- Add `site_id` to `pages`, `redirects`, `events`, `settings` (where per-site), and to the event envelope. Analytics presets and funnels filter by site; contacts stay workspace-wide.
- Middleware resolves `request.host` → site (cached in KV, fallback to the default site), sets `locals.site`, and rewrites to `src/sites/<slug>/…` for file routes. Shared components, sections and themes stay at the app root; each site picks a theme in its row.
- Theme becomes per site. Plugins stay workspace-wide, with a per-site enable flag in `plugin_installs`.

**Admin app (`apps/admin`)**
- Extracted from the site Worker: Hono API, queue consumer, Workflow class, cron, logs, analytics, dashboard pages. Adds the MCP server (every API route gets a tool) and a site switcher.
- Signed SSO handoff from the control plane replaces `ADMIN_TOKEN`.

**Cloudflare package (`app/packages/cloudflare`)**
- `provisionWorkspace()`: one repo, two Workers (`-site`, `-admin`), two Builds triggers with root directories `apps/site` and `apps/admin`.
- Add a `host` provider interface with two implementations: `OwnAccountHost` (today's code) and `OperatorHost` (dispatch namespace upload, Cloudflare for SaaS hostname, usage reads). The second lives in `@supergrid/operator`.
- Add `transferWorkspace`, `migrateWorkspaceHost`, `exportWorkspace`. Migrate = provision on the target, push the repo, D1 export/import, R2 copy, re-create secrets by name (values re-entered by the user), move domains, deprovision source.

**Control plane (`app/apps/web`)**
Thin. It owns accounts, Cloudflare connections, provisioning, the workspace switcher, marketplace, billing and the agent. It does not re-implement contacts, data or logs; those screens move to the workspace's admin and the control plane links or embeds them with the SSO handoff. The agent talks to each workspace through that admin's MCP server, so data never passes through SuperGrid servers on the `own` host.

Its own database, keyed by the WorkOS user:
```
accounts, users, account_members(role)
workspaces(account_id, slug, host, cf_account_id | pool_id, infra manifest JSON, plan_id)
workspace_members(role)              // lets a client be a guest in an agency's workspace
sites(workspace_id, …)               // mirror of the Worker's sites table for navigation
cloudflare_connections(user_id, …)   // the OAuth grants (replaces the sealed cookie)
usage(workspace_id, period, metric, value)
plans, subscriptions                 // operator package
operator_settings                    // §6, one row
```
The `Workspace.plan` and `kind: "own" | "client"` in today's mock become `host` and `account` relations.

---

## 9. Phasing

1. **Hackathon (now):** `own` host, one default site per workspace, admin still mounted inside the site Worker. Add the `sites` table and `site_id` now so nothing has to be migrated later, but only one row exists. Move the plugin folder to the three-entry package shape and collapse the three registration lists into `supergrid.config.ts`. Provisioning: one repo, two Workers named `-site` and `-admin`.
2. **Admin app and multi-site:** extract `apps/admin` with the MCP server; hostname routing, per-site theme, site switcher. Control plane drops its data screens in favour of the SSO handoff. Unlocks agencies properly.
3. **Transfer + export:** ownership story becomes real; agency handover.
4. **Operator host (managed):** dispatch namespace, Cloudflare for SaaS, metering, migrate-host (eject). This is where the `@supergrid/operator` package starts.
5. **Signup, plans, billing, partner console:** finishes managed for us and is the reseller product.
6. **Managed reseller:** automate "deploy one operator instance per reseller" on our account.

---

## 10. Open questions

- **Control plane as a Worker.** Everything in §6 assumes the control plane deploys as one Worker with one D1 so that per-reseller instances are cheap. The current Next.js app would need to run on Workers (OpenNext) or be rewritten; decide before step 6.
- **Contacts per site or per workspace.** Decided workspace-wide, but an agency might want one client workspace with two unrelated brands. The answer should be "that is two workspaces"; confirm it holds for real agency cases.
- **Shared media across workspaces** (an agency's stock library). Not supported by the boundary; the workaround is the marketplace or a "copy from workspace" action. Decide if that is enough.
- **Per-site plugins.** A plugin's tables and routes are workspace-wide; only enablement is per site. Is that enough for a funnel plugin that one site uses and another must not expose?
- **Admin framework.** A React app on Worker static assets plus Hono is the smallest thing that shares `@supergrid/ui` with the control plane. Astro with islands (today's dashboard) also works. Decide when `apps/admin` is extracted.
- **Embed or link.** Whether the control plane iframes the workspace admin inside its shell or just deep-links with SSO. Linking is simpler; embedding keeps the agent side panel next to the data.
- **Site Worker's D1 access.** Should the public site read D1 directly (fast, DB pages) or only through the admin service binding (one data plane)? Direct read, writes through admin, is the likely answer.
- **Agency branding on app.supergrid.com.** Operator-level only in this design, so an agency that wants its logo on the admin either self-hosts the control plane (free) or buys a managed reseller instance. Is a lightweight per-account logo on our hosted plane worth the exception?
- **Metering unit for resellers.** Usage is per workspace and rolls up to account, then to the operator instance. Our bill to a managed reseller is the instance total; what they charge their customers is theirs. Needs rates.
- **Secrets on migrate.** Values cannot be read back from Cloudflare, so migrate-host re-prompts for each secret by name. Acceptable, or store encrypted copies in the control plane for `operator` host only?
- **Preview environments per site** vs per workspace, and whether previews share D1.

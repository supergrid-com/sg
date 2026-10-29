# SuperGrid business model

> Working doc for license, pricing and monetization. The decisions below are the current direction. Open questions are at the end. Companion to [SUPERGRID.md](SUPERGRID.md). How the system is shaped to serve each tier is in [TENANCY.md](TENANCY.md).

---

## 1. Positioning

**Pitch:** "Self-hosted ClickFunnels + GoHighLevel, built by a co-founder of ClickFunnels."

**Piggyback on behaviours these platforms already created:**
- **Funnel hacking and hired funnel builders (ClickFunnels):** freelancers who build funnels and sites for clients.
- **White-labeling and agencies (GoHighLevel):** resellers who package the platform for a niche market. Example: an expert in South Florida med spas sells "MedSpaPro" to med spas, a market GoHighLevel doesn't target directly.

**The gap:** GoHighLevel's reseller tier is $497/month, and the product is bloated for a niche. SuperGrid lets that expert ship a focused product at a lower price and keep the margin.

---

## 2. License

**Decision: Functional Source License (FSL-1.1).** Each release converts to MIT or Apache 2.0 two years after it ships.

**What it allows**
- Read, run, modify and self-host for your own business.
- Professional services: building, deploying and managing sites for clients.

**What it blocks**
- "Competing use": offering a hosted, self-serve product that substitutes for SuperGrid, unless you hold a reseller license (see §3).

**Alongside the license**
- **Trademark policy.** Forks are allowed, but they can't use the SuperGrid name, logo or marketplace. This, not the license, is what stops a rebranded fork, the same way it works for WordPress.
- **MIT-licensed SDK and plugin API.** Plugin and theme authors choose their own license, so paid plugins can exist in the marketplace.
- **Contributor license agreement.** Lets outside contributions also ship under the commercial reseller license.

**Where the free/paid line sits**

| Use | License |
|---|---|
| Self-host your own sites and funnels | Free |
| Freelancer builds and deploys a funnel for a client | Free |
| Agency manages client sites through the hosted admin app, on its own or its clients' Cloudflare | Free |
| Agency runs its own branded deployment of the control plane | Operator license |
| Agency runs a self-serve SaaS where customers sign up, pay and get a workspace provisioned (MedSpaPro) | Operator license + resources tier |

The test is **your own deployment of the control plane**. Managing clients through our hosted app is a service and stays free. Running your own copy, with or without signups, needs the license. The control plane repo is private; the license is access to it (see [TENANCY.md](TENANCY.md) §7).

---

## 3. Reseller / white-label license

**What it grants:** the right to run SuperGrid as a rebranded, self-serve SaaS. That includes their own domain, brand, plans, signup flow, billing and provisioning, with the end customer never seeing SuperGrid.

**Decision: they pay for the license and resources, not for code updates.** Core code ships to everyone under the FSL and updates all the time, so updates aren't what the reseller buys. The one-time license covers future versions.

**Pricing (draft)**
- **License, one-time: around $297.** The right to resell, for all versions. Compared with GoHighLevel's $497/month reseller tier.
- **Resources, monthly, price TBD.** What makes it easier to build and sell a micro SaaS:
  - reseller plugins and features, such as signup, plans, billing and the partner console,
  - a private community and support,
  - courses and education on building the micro SaaS, picking a niche, and marketing it.

**Where it runs**
- **Self-hosted:** the reseller runs their SaaS on their own Cloudflare account.
- **Managed reseller, around $99/month:** we host their SaaS on the SuperGrid Cloudflare account, the same model as managed (§5), with overages metered.

**The reseller keeps 100% of what they charge their customers.**

---

## 4. Monetization

**Order of importance**
1. **Free, self-hosted.** The core product and the core angle. The code is open and they own it. Hosting is on their own Cloudflare account at Cloudflare's prices, not ours.
2. **Managed.** The main paid entry point (§5).
3. **Reseller license, resources and managed reseller.** For agencies and niche experts (§3).
4. **Later:** certification, marketplace, courses and other adjacent products (§6).

**Decision: no cut of their sales.** Whatever users sell is theirs. There is no transaction fee and no required payment processor. How and where they sell, through Stripe, PayPal or anything else, is up to them.

---

## 5. Managed

**Decision: managed runs on the SuperGrid Cloudflare account, not the user's.**
- We provision the resources: a Worker per site in a Workers for Platforms dispatch namespace, a D1 database per workspace, R2 storage under a per-workspace prefix, and custom domains through Cloudflare for SaaS.
- This is the most self-serve version. No Cloudflare account, no wrangler, no git required to get started.
- **They still get the git repo.** They can clone it, change it and push. We own the deploy pipeline.
- **Eject at any time.** Move the site, data and repo to their own Cloudflare account and continue on the free self-hosted path.

**Why not provision into their Cloudflare account**
- Many users don't have Cloudflare and never want to think about it.
- One account is simpler to provision, meter, monitor and support.

**Pricing (draft)**
- **Around $20/month base**, covering most users.
- **Metered overages** above the included usage: requests, storage, AI use and similar.

**What this changes elsewhere**
- **Ownership (SUPERGRID.md §17).** Managed users don't hold the infrastructure until they eject. The promise becomes "you own the code and data and can take them anywhere, any time". Eject has to be a real, tested, one-click path, not a support ticket.
- **AI (SUPERGRID.md §12).** Metering AI use sits against "no token reselling, no markup". Options: pass AI through at cost, or let managed users bring their own key and skip the AI charge.
- **We carry infra cost and abuse risk.** Spam funnels and phishing pages on our hostnames become our problem. Needs terms of service, abuse reporting and per-workspace limits from day one.
- **Workers for Platforms is required from the start** for managed, rather than a later scaling step.

---

## 6. Later

- **Certification and a hire-a-builder directory.** Certified builders get leads. Fits the hired-funnel-builder culture.
- **Marketplace.** Plugins, themes, features and courses, with a percentage of each sale (SUPERGRID.md §14).
- **Courses and education.** This audience already buys training.
- **Desktop app / IDE.** Manages everything locally. Possibly $20/month or bundled into a paid tier. Test as an MVP after launch, not part of launch.

---

## 7. Open questions

- **FSL wording is broad.** "Substantially similar functionality" could be read to cover an agency hosting client sites. Publish a plain-language FAQ or an additional grant that makes agency client management explicitly free.
- **MIT or Apache 2.0** as the conversion license.
- **Price of the monthly resources tier**, and exactly what's in it at launch.
- **Managed reseller overages.** A reseller has many workspaces, so metering probably needs to be per workspace. Model this together with agency pricing.

**Later**
- **Managed limits.** What's included in the base price, and the overage rates.
- **Eject.** Everything a site needs to keep running on the user's own account: repo, D1 database, R2 files, Worker and wrangler config, secret names, domains and DNS, and any queues, workflows and cron triggers.
- **Agency pricing on managed.** Flat per account or per workspace. Needs a cost and revenue model before deciding.

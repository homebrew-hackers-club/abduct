---
name: setup-company-profile
description: Research the user's company and set up its profile on hacklab using the hacklab CLI (hacklab org claim/create/edit). Use when an employer wants to set up, claim, or polish their company profile on hacklab.
---

# setup-company-profile

Set up a company profile on hacklab so hackers can find the company and see that it is hiring. You research the company and prepare every field; the user drives the `hacklab org` editor (it is interactive — there is no non-interactive apply for orgs), pasting values from the sheet you prepared.

## Org fields

The `hacklab org` editor covers these fields (org page lives at `hacklab.so/o/<slug>`):

| field | meaning | shape |
| --- | --- | --- |
| Name | company name | text |
| Slug | URL handle | lowercase letters, numbers, hyphens |
| Logo URL | logo image | URL |
| Website | company site | URL |
| Short description | one-liner | text |
| Long description | the pitch | text, a few sentences |
| YC batch / YC slug / YC URL | YC identity, if a YC company | e.g. `W24`, `ycombinator.com/companies/<slug>` |
| WaaS URL | Work at a Startup jobs page | URL |
| Team size | headcount | whole number |
| Status / Stage | e.g. Active / Seed, Series A | text |
| Hiring? | the flag hackers filter by | yes / no |
| Industries / Tags / Locations | discovery metadata | comma-separated lists |

## Step 1 — Preflight

1. Check the CLI: `hacklab --version` (if missing: `npm i -g hacklab`).
2. Check auth: `hacklab whoami`. If not logged in, have the user run `hacklab login` (interactive browser flow). Note: claim eligibility partly comes from the email domain of the account — a work email helps.

## Step 2 — Identify the company and the path in

Ask which company this is for, then determine the route:

- **Claim** — hacklab pre-seeds YC companies. If the company might already exist (any YC company, or one a teammate added), the user runs `hacklab org claim`: it lists companies claimable via membership or matching email domain. This is the preferred route — it keeps the existing slug and any seeded data.
- **Create** — otherwise `hacklab org create` (name, slug, website, short description). If create hits an existing slug, the CLI itself offers to claim when possible; if the slug is taken by someone else, pick a different slug.

## Step 3 — Research the company

Fill every field in the table from public sources:

1. **Company website**: description copy, logo (prefer a direct image URL — check `og:image`, `/favicon`, press/brand pages), locations, careers page.
2. **YC directory**, if applicable: `ycombinator.com/companies/<slug>` gives batch, team size, status, stage, industries, and the Work at a Startup URL.
3. **GitHub org**: engineering presence, extra description material.
4. **Web search** for anything still missing (headcount, locations). Mark anything not from a first-party source as unverified.

Rules: never invent a value; verify every URL resolves; prefer the company's own copy over your paraphrase for descriptions (light editing is fine).

## Step 4 — Ask the employer (two required questions)

**1. "Are you hiring right now?"** — always ask; this sets `Hiring?`, the flag hackers filter by. Never infer it from a careers page that may be stale.

**2. "Here's everything I found — confirm each field."** Present the full researched field sheet with sources, let them correct or drop values:

```
Name          Acme Robotics          (site)
Slug          acme-robotics
Website       acme.com               (verified)
Logo URL      acme.com/logo.png      (og:image)
Short desc    Robots for warehouses. (site hero)
YC batch      W24                    (YC directory)
Team size     12                     (YC directory — confirm?)
Locations     Warsaw, Remote         (careers page)
...
```

Descriptions especially should be approved verbatim — this is their public pitch to hackers.

## Step 5 — Apply via the interactive editor

The org editor is interactive and autosaves per field. Walk the user through it:

1. Run `hacklab org claim` or `hacklab org create` per Step 2 (after create/claim the CLI offers "edit its details now?" — say yes).
2. For the remaining fields, run `hacklab org`: pick a field, paste the value from the confirmed sheet, repeat. Every save is immediate; quitting loses nothing.
3. Keep the confirmed sheet visible in your final message so the user can copy values line by line.

Warn before slug changes: old `/o/<old-slug>` links stop resolving.

## Step 6 — Verify

Open `https://hacklab.so/o/<slug>` and check it renders: logo loads, links work, hiring badge matches the answer from Step 4. Report the URL.

## Failure modes

- `not logged in` → `hacklab login`.
- "no companies to claim" → the user is not a member of an unclaimed company and their email domain does not match any; go the `create` route.
- Slug already claimed by someone else → different slug, or the user asks the current owner to add them as a member.
- Sparse research (stealth company) → name, slug, website, one-line description, and the hiring flag is a perfectly good profile. Do not pad.

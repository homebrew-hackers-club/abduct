---
name: setup-company-profile
description: Research the user's company and set up its profile on hacklab using the hacklab CLI (hacklab org claim/create/edit). Use when an employer wants to set up, claim, or polish their company profile on hacklab.
---

# setup-company-profile

Set up a company profile on hacklab so hackers can find the company and see that it is hiring. You research the company, confirm the findings with the user, then apply everything yourself with the CLI's non-interactive org verbs (`hacklab org list/claim/create/apply --json`, CLI >= 0.7.6). The user only answers questions and approves — no prompts to drive.

## Org fields

Field names as `hacklab org set` / `org apply` take them (org page lives at `hacklab.so/o/<slug>`):

| field | meaning | shape |
| --- | --- | --- |
| `name` | company name | text |
| `slug` | URL handle | lowercase letters, numbers, hyphens |
| `logo` | logo image | URL |
| `website` | company site | URL |
| `description` | one-liner | text |
| `long-description` | the pitch | text, a few sentences |
| `yc-batch` / `yc-slug` / `yc-url` | YC identity, if a YC company | e.g. `W24`, `ycombinator.com/companies/<slug>` |
| `waas` | Work at a Startup jobs page | URL |
| `team-size` | headcount | whole number |
| `status` / `stage` | e.g. Active / Seed, Series A | text |
| `hiring` | the flag hackers filter by | yes / no |
| `industries` / `tags` / `locations` | discovery metadata | lists (arrays in yaml, or comma-separated) |

## Step 1 — Preflight

1. Check the CLI: `hacklab --version` — must be >= 0.7.6 (if missing or older: `npm i -g hacklab`).
2. Check auth: `hacklab whoami`. If not logged in, have the user run `hacklab login` (interactive browser flow). Note: claim eligibility partly comes from the email domain of the account — a work email helps.
3. Get the lay of the land: `hacklab org list --json` → `organizations` (already managed) and `claimable` (with the reason: member / email domain).

## Step 2 — Identify the company and the path in

Ask which company this is for, then pick the route from the `org list` output:

- **Already managed** — it's in `organizations`: skip straight to research + apply.
- **Claim** — it's in `claimable` (hacklab pre-seeds YC companies): `hacklab org claim <slug> --json`. Preferred — keeps the existing slug and seeded data. Idempotent: re-claiming your own org is a no-op success.
- **Create** — otherwise: `hacklab org create --name "<name>" [--slug <slug>] [--website <url>] --json` (slug is derived from the name when omitted). On a slug collision the CLI exits with `error.code: "slug_exists"` plus `existing` and `claimable` — if `claimable` is true, reroute to `org claim <slug>`; otherwise pick a different slug.

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

## Step 5 — Apply

Write the approved fields to a throwaway `org.yaml` in a scratch directory (never the user's repo), then apply in one shot:

```yaml
name: Acme Robotics
description: Robots for warehouses.
long-description: >
  Acme builds picking robots that slot into existing warehouse racking.
  Founded 2024, YC W24.
website: acme.com
logo: acme.com/logo.png
yc-batch: W24
team-size: 12
hiring: true
industries: [Robotics, Logistics]
locations: [Warsaw, Remote]
```

```sh
hacklab org apply org.yaml --json          # or pipe: ... | hacklab org apply - --json
```

Only fields present in the file are touched. `--org <slug>` is only needed when the account manages more than one company (a missing flag errors with `ambiguous_org` and the slugs). Delete the yaml afterwards. Avoid changing `slug` on an existing org — the response's `warnings` array will tell you old links broke; do it only if the user explicitly wants it.

If `apply` rejects a field, fix or drop that field and re-apply — the server owns validation and re-applying is safe.

## Step 6 — Verify

Check `hacklab org view --json` reflects every approved field, then open `https://hacklab.so/o/<slug>`: logo loads, links work, hiring badge matches the answer from Step 4. Report the URL.

## Failure modes

- `not logged in` → `hacklab login`.
- `claimable` empty and the company isn't yours → the user is not a member of an unclaimed company and their email domain does not match any; go the `create` route.
- `slug_exists` with `claimable: false` → someone else owns it: different slug, or the user asks the current owner to add them as a member.
- `ambiguous_org` → pass `--org <slug>`.
- Sparse research (stealth company) → name, slug, website, one-line description, and the hiring flag is a perfectly good profile. Do not pad.

---
name: setup-hacklab-profile
description: Research the user (GitHub, personal site, blog, socials) and set up their hacklab profile using the hacklab CLI. Use when the user asks to set up, fill in, or polish their hacklab profile.
---

# setup-hacklab-profile

Set up the user's hacklab profile by researching who they are and applying the findings with the `hacklab` CLI. The goal is a profile that looks hand-written by the user: accurate links, a bio in their voice, nothing invented.

## Profile fields

The CLI edits these fields (via `hacklab profile`):

| field | meaning | shape |
| --- | --- | --- |
| `name` | display name | text |
| `bio` | short bio | text, keep it to 1-2 punchy sentences |
| `website` | personal site | URL |
| `blog` | blog | URL |
| `x` | X / Twitter | handle or URL (CLI normalizes) |
| `youtube` | YouTube channel | URL |
| `instagram` | Instagram | handle or URL (CLI normalizes) |
| `rss` | RSS feed | URL |
| `open-to-work` | job-seeking flag | yes / no |

GitHub is linked automatically from the hacklab login — do not try to set it.

## Step 1 — Preflight

1. Check the CLI exists: `hacklab --version` (if missing: `npm i -g hacklab`).
2. Check auth and identity: `hacklab whoami`. If not logged in, ask the user to run `hacklab login` themselves (it is an interactive browser flow — do not run it for them in the background).
3. Fetch current state: `hacklab profile view --json`. Note which fields are already set — never overwrite a non-empty field without telling the user.

## Step 2 — Research the person

Build a picture from sources in this order (stop early when the picture is complete):

1. **Local machine**: `git config user.name`, `git config user.email`, `gh api user` (GitHub name, bio, blog, location, socials via `gh api user/social_accounts`).
2. **GitHub profile deep-dive**: `gh api users/<login>` and the profile README (`gh api repos/<login>/<login>/readme` — decode base64). Pinned repos and READMEs often carry the site, blog, and socials.
3. **Their website**, if found: fetch it and look for bio copy, an about page, social links, and an RSS/Atom feed (`<link rel="alternate" type="application/rss+xml">`, or try `/rss.xml`, `/feed.xml`, `/atom.xml`, `/feed`).
4. **Web search** to sweep for whatever is still missing: search their name + GitHub handle for an X account, YouTube channel, blog. Collect everything plausibly theirs — but mark each find as verified (cross-linked from a confirmed source, same handle) or unverified. Doubtful finds are not dropped and not included silently; they go to the user flagged as unverified in Step 4.

Rules:

- Never invent a link. Everything shown to the user must actually exist — verify every URL resolves (a quick fetch or `curl -sI`).
- Prefer links the person publishes about themselves (GitHub profile, their own site) over search results.
- Nothing goes into the profile without passing through the Step 4 checklist — the user picks, you don't.

## Step 3 — Draft the bio

Write a 1-2 sentence bio from the research: what they build, what they are known for, in their own tone (mirror the voice of their GitHub bio / site copy — do not corporate-ify a hacker). If they already have a bio somewhere, adapt it rather than writing fresh.

## Step 4 — Ask the user (two required questions)

These two conversations are mandatory — never skip them, never assume the answers:

**1. "Are you currently looking for a job?"**

Ask this as a direct question. The answer sets `open-to-work`. Job-seeking status is personal and time-sensitive — it must come from the person, never from research (a "hire me" banner on an old site proves nothing about today).

**2. "Here's everything I found — which links should go on your profile?"**

Present ALL personal links the research turned up as a checklist — website, blog, X, YouTube, Instagram, RSS, and anything else you found — each with where you found it, e.g.:

```
[x] website    adalovelace.dev            (from GitHub profile)
[x] blog       adalovelace.dev/blog       (linked from site nav)
[x] x          @adalovelace               (cross-linked from site)
[ ] youtube    youtube.com/@ada-old-vlogs (web search — unverified, last upload 2019)
[x] rss        adalovelace.dev/rss.xml    (site <link> tag)
```

Default to including everything first-party; let the user check/uncheck. Include even doubtful finds in the list (clearly flagged) — the user decides, not you. Also show the drafted bio here for approval or edits.

## Step 5 — Build profile.yaml and confirm

Write the approved fields to a `profile.yaml` (in a scratch directory, not the repo), using CLI field names:

```yaml
name: Ada Lovelace
bio: First programmer. Writing engines before engines were cool.
website: adalovelace.dev
blog: adalovelace.dev/blog
x: adalovelace
rss: adalovelace.dev/rss.xml
open-to-work: yes
```

Only include fields the user approved. Flag anything that would overwrite an existing non-empty value before applying.

## Step 6 — Apply and verify

```sh
hacklab profile apply profile.yaml --json
```

The response lists `updated` fields. Then run `hacklab profile view` and show the result plus the profile URL (printed at the bottom of the view output) so the user can check it live.

If `apply` rejects a field, fix that one field (or drop it) and re-apply — the server owns validation.

## Failure modes

- `not logged in` → ask the user to run `hacklab login`.
- Unknown field error → you used a wrong key; valid keys are the field names in the table above (or their API aliases).
- Research turns up nothing beyond GitHub → that is fine; a name, bio, and GitHub link is a respectable minimal profile. Do not pad it.

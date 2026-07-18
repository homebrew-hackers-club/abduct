# abduct

A gstack-style skill pack for employers on [hacklab](https://hacklab.so): find hackers, set up your company presence, hire.

Skills in this repo are agent workflows: each one is a directory under `skills/` with a `SKILL.md` the agent loads and follows.

## Skills

| Skill | What it does |
| --- | --- |
| `/setup-company-profile` | Researches your company (site, YC directory, GitHub) and sets up its hacklab org profile via the `hacklab` CLI — claim or create, every field prepped and confirmed with you. |

More coming: hacker discovery (search open-to-work hackers by stack and activity), outreach, and hiring-pipeline skills.

## Install

Clone and symlink the skills into your project (or user) skills directory:

```sh
git clone git@github.com:homebrew-hackers-club/abduct.git
ln -s "$(pwd)/abduct/skills/setup-company-profile" ~/.claude/skills/setup-company-profile
```

Or add the repo as a Claude Code plugin (it ships a `.claude-plugin/plugin.json`; skills are auto-discovered from `skills/`).

## Requirements

- The `hacklab` CLI: `npm i -g hacklab` (or `npx hacklab`)
- A hacklab account: `hacklab login` — use your work email; org claiming matches on email domain

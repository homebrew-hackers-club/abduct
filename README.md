# abduct

A gstack-style skill pack for interacting with [hacklab](https://hacklab.so) and looking for a job.

Skills in this repo are agent workflows: each one is a directory under `skills/` with a `SKILL.md` the agent loads and follows.

## Skills

| Skill | What it does |
| --- | --- |
| `/setup-hacklab-profile` | Researches who you are (GitHub, personal site, socials) and sets up your hacklab profile via the `hacklab` CLI. |

More coming: job-search skills (find openings, tailor applications, track pipeline) built on top of your hacklab profile.

## Install

Clone and symlink the skills into your project (or user) skills directory:

```sh
git clone git@github.com:homebrew-hackers-club/abduct.git
ln -s "$(pwd)/abduct/skills/setup-hacklab-profile" ~/.claude/skills/setup-hacklab-profile
```

Or add the repo as a Claude Code plugin (it ships a `.claude-plugin/plugin.json`; skills are auto-discovered from `skills/`).

## Requirements

- The `hacklab` CLI: `npm i -g hacklab` (or `npx hacklab`)
- A hacklab account: `hacklab login`

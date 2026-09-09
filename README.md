# CAT skills

Skills for coding agents (Claude Code, Codex, Cursor and the like) that teach them how to use
CAT, the data-testing tool: one folder per situation, each with a `SKILL.md` your agent loads
when it matches what you asked for.

This folder is generated from the `cat-tools` repository by the CAT CLI release pipeline and
is not edited here — a pull request against this repository will not be reviewed.

## Install

- `npx skills add justcatit/cat-skills` — installs into your agent's skills folder.
- `git clone https://github.com/justcatit/cat-skills` — the folders under `skills/` in the
  clone are what your agent needs; copy or symlink them into its skills folder.
- The zip on the [Releases page](https://docs.justcat.it/releases/) — for a firewalled
  install; unpack it into your agent's skills folder. It also carries the packaged
  documentation pages the skills link to, for when your agent cannot fetch a URL.

## Versions

The tip of `main` always carries the newest CAT CLI's skills. An older CLI's skills are on
the tag `v<version>` matching it.

---

The skill files in this repository are MIT licensed (see LICENSE). CAT itself is a commercial
product licensed under its own terms of use: https://docs.justcat.it/license/

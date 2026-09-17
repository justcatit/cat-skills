---
name: cat-install-cat
description: >
  Getting CAT onto a machine: which of the four tools to install (CAT Studio, CAT CLI, the
  PowerShell module, the Python module), how on Windows, Linux and macOS, what to check
  afterwards, and how the plan unlocks it (a sign-in token for Team, a license key for
  Enterprise; nothing on the command line for Starter and Professional). Obligatory reading
  before installing or updating any CAT tool, before setting a license key or a portal
  token, and whenever a CAT command refuses to run with an exit code from 2 to 8.
---

# Install CAT

CAT is four tools that open the same project file and run the same engine; the choice is
how you talk to it. Nothing works until the plan unlocks the tool, so installing and
unlocking are one job. Every fact is on the documentation page it links; the same page as
markdown is at the URL with `index.md` appended, which is the form to read when your fetch
tool summarizes HTML.

## Which tool

- A person with no command line: **CAT Studio**, Windows only, from the Microsoft Store.
- A machine, a script, a pipeline, an agent: **CAT CLI**, a single executable with no
  runtime to install, on Windows, Linux and macOS. The default for an agent.
- A team already on PowerShell 7: the **PowerShell module**, results as objects.
- A Python environment: the **Python module** (`justcatit` on PyPI), which needs a .NET
  runtime next to it.

The comparison, with what each cannot do: https://docs.justcat.it/reference/basics/cat-tools/

"I have the CLI, I have everything" holds for running tests and writing outputs; it does
not hold for the Windows-only providers (DAX, Power BI, the OLE DB file providers), which
need a Windows machine whichever tool runs them.

## Install, then verify

Read the platform section of the page before running anything, and pin the version;
`<version>` in the commands is the one on the Releases page:

- CAT CLI, Windows (WinGet or the signed installer), Linux (archive into `/opt` or the home
  directory, needs `curl` and ICU), macOS (Homebrew tap): https://docs.justcat.it/reference/cat-cli/installation/
- CAT Studio: https://docs.justcat.it/reference/cat-studio/installation/
- PowerShell module: https://docs.justcat.it/reference/powershell-module/installation/
- Python module: https://docs.justcat.it/reference/python-module/installation/
- What is not there off Windows, and the providers that need a driver of their own:
  the CLI page's last sections, and https://docs.justcat.it/how-to-guides/licensing-and-admin/cat-on-linux/

The check is `catcli --version` (or the tool's equivalent) in a new terminal; a terminal
opened before the install does not see the path. The verbs `instance`, `docs`, `help` and
`version` work without any plan, so the installation can always be inspected.

## Unlock it

The plan decides what runs; the machine holds the unlock:

- **Enterprise**: a license key. Ask the person to put it into the environment, or into the
  sandbox's secrets settings, as `CAT_LICENSE_KEY` — never ask for the key in the
  conversation, and never read it: a key you handle ends up in the transcript and in the
  command log. On a machine the person owns it can instead be stored once per machine and
  account with `catcli instance --setLicenseKey`. Either way every tool then runs without
  sign-in, offline, unattended. The variable wins over the stored key, and a value it cannot
  use refuses the run (exit code `8`) rather than falling back. Where it
  lives and what it is not: https://docs.justcat.it/how-to-guides/licensing-and-admin/apply-license-key/
- **Team**: a personal access token from the portal in the environment variable
  `CAT_PORTAL_TOKEN`; interactive use only, a pipeline or a scheduler is refused.
- **Starter and Professional**: CAT Studio only; the command-line tools refuse.

Which plan does what, trials, and what happens when a plan ends:
https://docs.justcat.it/how-to-guides/licensing-and-admin/get-license/
The sign-in section of the CLI introduction, and the exit codes that tell the cases apart:
https://docs.justcat.it/reference/cat-cli/introduction/

`catcli instance -s` shows the plan, the key and how CAT thinks it was launched; it is the
first command to run when a verb refuses.

## The traps

- A refusal is a plan or sign-in problem, never a broken install: `2` no usable token, `3`
  the plan has no command-line tools, `4` portal unreachable and no cache, `5` expired, `6`
  interactive-only plan started by automation, `7` a per-project limit, `8` `CAT_LICENSE_KEY`
  holds no key CAT can use. Report the message;
  do not work around it, and never put a key or a token into a project file.
- The key and the token are per machine **and per account**: a key set under one user is
  not there for the service account a scheduler runs as.
- A valid Enterprise key makes `CAT_PORTAL_TOKEN` irrelevant; with both present the key
  wins.
- Linux without ICU stops at startup with a message about a missing ICU package.
- `~/.local/bin` is on the path from the next login, not in the current shell.
- Pin the version in anything automated; upgrade on purpose, after a test.

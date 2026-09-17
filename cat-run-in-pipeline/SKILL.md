---
name: cat-run-in-pipeline
description: >
  CAT running unattended, in a CI/CD pipeline (Azure DevOps, GitHub Actions, GitLab,
  Jenkins) or a scheduler (cron, Windows Task Scheduler, SQL Server Agent, a notebook job):
  what the step needs around the one run command, which plan allows it, where the key and
  the secrets live, how the run becomes a red or green verdict — CAT CLI's own exit code, or
  a publish step with the PowerShell module. Obligatory reading before writing or changing
  any pipeline, workflow, job or schedule that runs CAT, and when such a run is refused.
---

# Run CAT in a pipeline or a scheduler

The run itself is one command; what surrounds it is where pipelines go wrong: the plan, the
key, the version, the secrets, and the verdict. Every fact is on the documentation page it
links; the same page as markdown is at the URL with `index.md` appended, which is the form to
read when your fetch tool summarizes HTML.

## Read this page first, then the platform's

The one page with everything a step needs, plan, key, tool, allow-list, secrets, results,
exit codes, and the three questions to settle before the first step (where it runs, how it
is managed, what it must reach): https://docs.justcat.it/reference/integrations/

Then the platform guide, a starting point, not a finished pipeline:

- Azure DevOps Pipelines https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/cat-in-azure-devops-pipelines/
  and Classic Releases https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/ados-classic-releases/
- GitHub Actions https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/cat-in-github-actions/
- GitLab https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/cat-in-gitlab/
- Jenkins https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/cat-in-jenkins/
- cron on Linux https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/cat-in-cron/
- Windows Task Scheduler https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/cat-in-windows-task-scheduler/
- SQL Server Agent https://docs.justcat.it/how-to-guides/pipelines-and-schedulers/sql-agent-cat-cli/
- Databricks notebooks https://docs.justcat.it/how-to-guides/data-platforms/cat-in-databricks-notebooks/

## The rules a step must obey

- **Enterprise only.** A run started by a CI/CD platform or a scheduler is refused on every
  other plan, and from CAT 3.0 a pipeline without a valid license key stops before any test
  runs. Map the secret that holds the key to a `CAT_LICENSE_KEY` environment variable on the
  step: that variable is the key, and a hosted agent needs no set-key step at all. A
  self-hosted agent can instead keep the key stored under the account the agent runs as, and
  then must not map the variable. Load the install skill for the tool and the key on that machine.
- **Pin the version** and keep it, with the key, in pipeline variables; upgrading is one
  variable change.
- **Secrets are environment variables of the step**, never text in the project file: the
  file says `%NAME%` and the platform injects `NAME`. Load the data-sources skill for the
  project side. https://docs.justcat.it/how-to-guides/organize-and-run-tests/work-with-secrets/
- **With CAT CLI the step fails by itself on a failed test** (exit code `10`). A step that
  also publishes a report needs the platform's always-run guard on the publish step
  (`if: always()` on GitHub Actions, `condition: always()` on Azure Pipelines, `when: always`
  on GitLab, a `post` block on Jenkins), because a red step skips what follows it —
  `--exitCode run` on `catcli run` restores the old `0`-whatever-the-results behaviour if a
  pipeline needs it instead. With the PowerShell module, the output is still the verdict: ask
  the project for `junit` (every platform) or `trx` (Azure DevOps, Visual Studio), publish
  it, and let the publish step fail on failed tests; or read the JSON output's counts in a
  script and exit non-zero yourself. Load the outputs skill for the file.
- **A non-zero exit code does not always mean the run did not happen.** With CAT CLI, `10`
  means it did, with a test that did not pass; `1` the project did not open, `2` to `8` the
  sign-in and plan check refused it, `6` being exactly "interactive-only plan started by
  automation". https://docs.justcat.it/reference/cat-cli/introduction/#exit-codes
- **Network position.** The agent must reach every data source the tests compare; with the
  key in place it needs no other address at run time, and the package source only at install
  time.

## The traps

- The key and the secrets belong to an account: a scheduler running as a service account
  does not see the key set by the person who installed CAT, and cron starts with almost no
  environment. The scheduler pages show where each one goes.
- A hosted agent is fresh every run: install and key in the pipeline, every time.
- A relative `File` in the output setting resolves against the project file, not the
  working directory of the step; the publish step must look where CAT wrote.
- `Output: junit` writes a timestamped name by default; a publish step wants a fixed name.
- `-n` on the run skips the outputs, and with them the report the publish step expects.
- A pipeline that worked on 2.x and breaks on 3.0 with exit code `6`, `8` or a key message is
  missing the license key, not the tool.
- On Azure DevOps, a step that maps a `CAT_LICENSE_KEY` nobody defined passes the literal
  text `$(CAT_LICENSE_KEY)`, and CAT refuses the run with exit code `8` instead of using a
  stored key. Define the variable, or remove the mapping.
- A CAT CLI pipeline written for 2.x that started going red on 3.0 is seeing failed tests it
  used to hide, not a new bug; `--exitCode run` restores the old behaviour if that is wanted.

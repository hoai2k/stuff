# Claude working agreement

## Deploy state: **ENABLED**

Deploy branch: `deploy`

This repository deploys from the `deploy` branch, not from `main`. Pushing to
`main` runs no workflow and costs nothing; the site is published only when
`deploy` is moved forward to `main`. This switch overrides every other
instruction in this file about deploying.

- **ENABLED** means deploy after every change: commit, push `main` as the rest
  of this file says, then publish it with `git push origin main:deploy`. That
  push deploys. A deploy that is already running always finishes, and pushes
  waiting behind it collapse into the newest one.
- **DISABLED** means deploys are off. Commit and push `main` as usual, so
  finished work still lands on `main`, but never push, merge or fast-forward
  `deploy`. End every change with this warning to the owner:
  > ⚠️ Deploys are disabled for this repo: the change is on `main` but NOT
  > deployed. The live site is still whatever `deploy` holds.
- **Turning deploys on** happens only when the owner explicitly asks to deploy,
  to enable deploys, or for a push to `main` that should go live. Change the
  heading above to `**ENABLED**`, commit and push `main`, then
  `git push origin main:deploy`. If the owner disabled the deploy workflow in
  the Actions tab, remind them to re-enable it.
- **Turning deploys off**, when the owner asks: change the heading to
  `**DISABLED**`, commit and push `main`. It costs nothing.
- `git push origin main:deploy` is always a fast-forward. Never force-push
  `deploy`. If the push is refused, `deploy` has commits `main` lacks: merge
  `origin/deploy` into `main` first, then push both.
- `.claude/hooks/deploy-guard.sh` enforces this for Claude sessions: while the
  heading says DISABLED it refuses any push to `deploy`. It reads the heading
  at the moment of the push, so flipping it to ENABLED is all it takes to let
  the push through.

Repository-wide instructions for Claude Code sessions in this repository.

## Where it deploys

The repository root is `www.hoai.net/stuff/` on the owner's DreamHost server
(`/home/craft/hoai/stuff`). `.github/workflows/deploy.yml` uploads it with
rsync over SSH, using a key that `rrsync` on the server confines to that folder
and forbids from deleting. The secrets it needs are `SSH_PRIVATE_KEY`,
`SSH_KNOWN_HOSTS`, `SSH_HOST` and `SSH_USER` (optional `SSH_PORT`,
`SSH_PATH`).

**Never overwrite or delete server data.** The live site holds files that exist
only on the server, above all the calendar log written by `cal/api.php`:

- The deploy never passes `--delete`, so files missing from the repo are left
  alone. Keep it that way.
- `.deployignore` lists paths that are never uploaded (repo-only files and the
  data files `cal/api.php` writes). Never commit a file under one of those
  paths expecting it to go live, and add a project's server-written data files
  to `.deployignore` before its first deploy.
- `cal/api.php` is not in the repository (the server runs it, so it can't be
  downloaded). If it is ever added, it deploys like any other file.

Actions → Deploy → Run workflow does a dry run by default and lists what would
change; use it before a deploy that touches something new.

## Repository layout: one folder per project

Each project lives in its own top-level folder, published at
`www.hoai.net/stuff/<folder>/`, with a `README.md` saying what it is, what
lives only on the server, and how to check it. The root `README.md` links
every project.

| Folder | What | Live |
| --- | --- | --- |
| `cal/` | Calendar checkmark tracker | https://www.hoai.net/stuff/cal/ |

## Always commit finished work to `main`

Every task ends on `main`. When a task is complete, do not leave the work
sitting on a feature branch or uncommitted: commit it, merge it into `main`,
push `main`, and say so in the final message with the resulting commit.

1. Commit the work on the session's development branch as each piece is
   finished, and push the branch (pushes to a working branch run nothing).
2. Verify the change by running the checks in the README of each project it
   touched. A deploy uploads without running any tests.
3. Merge that branch into `main` and push `main`, once, at the end of the task,
   then (while deploys are ENABLED, see "Deploy state" above) publish it with
   `git push origin main:deploy`.
4. Report the merge and the resulting `main` commit.

Open a pull request only when the user explicitly asks for one; otherwise merge
directly. Never merge work that is unfinished, untested, or known to be broken —
finish it or say plainly what is blocking, rather than merging a broken `main`.

## Agents

Use subagents where they save effort, and pick the model by the work (see
"Models" below): `sonnet` for mechanical edits, lookups, file-by-file chores,
well-specified implementation, tests and reviews with a clear contract; `opus`,
or the model left unset so it inherits, for design, architecture and anything
open-ended. Give an agent the exact files, interfaces and checks it must
satisfy so a cheaper model can finish the job without guessing.

## No Git LFS

Don't use Git LFS anywhere in this repository. GitHub never frees LFS storage:
a deleted or replaced LFS file keeps counting against the account's allowance
until the whole repository is deleted and recreated, and the allowance is
shared by every project. Files over GitHub's 100 MB limit go somewhere else, or
ask.

## Tests and deploys

Deploys (pushes to `deploy`) upload only; they run no tests. Tests are your job
before you commit: run the checks of each project your change touches and say
in the commit message what you ran and that it passed.

Batch a task's work into one deploy at the end rather than deploying after
every small step. Pushes to `main` and to working branches run nothing. A
deploy that is already running always finishes — never cancel one unless the
owner asks. Pushes that only change Markdown or `.claude/` don't deploy at all
(`paths-ignore` in the deploy workflow), and the job carries a
`timeout-minutes`, so a hung run cannot bill for hours. Keep all three when
editing the workflow.

## Models

- Only a session already running on Fable may start Fable subagents, workers or helper agents. Opus,
  Sonnet and every other model never pass model "fable"; they leave the model unset so it inherits, or
  pick Opus or Sonnet.
- Don't switch a session to Fable on your own. If a task seems to need Fable, ask me first and say why.
- This is enforced in `.claude/settings.json`: sessions in this repository default to Opus, and a
  PreToolUse hook (`.claude/hooks/block-fable.sh`) reads the calling model from the transcript and
  refuses a subagent or workflow call that asks for Fable unless the caller is itself Fable. When I ask
  for Fable, switch the main session with `/model`; don't edit the hook to get round it.

## Deployment reminder

`deploy` is the deployed branch. Pushing it (`git push origin main:deploy`)
uploads the site to `www.hoai.net/stuff/`; treat every deploy as a release and
check afterwards that the Deploy run is green and the changed pages still work.

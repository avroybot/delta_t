---
name: merge
description: >-
  Land this project's current changes on GitHub by pushing local `main` to
  `origin` (https://github.com/avroybot/delta_t.git). Invoke only when the
  user has explicitly requested a merge or landing (such as the Merge Change
  action, /merge, or wording like "merge this" or "land it"); never for
  review, preparation, passing checks, or skill installation.
disable-model-invocation: true
metadata:
  delta-action: land
---

# Merge to origin/main

Merge requests here are fulfilled by pushing local `main` to `origin`
(`https://github.com/avroybot/delta_t.git`). This repository has no test
suite, CI, or review requirements: the definitive verification is that the
push succeeds and `origin/main` matches local `main`.

The user's request is the authorization to merge. Once invoked, work through
the steps below without re-asking whether to merge. Stop only for the genuine
blockers listed at the end.

## 1. Inspect state

- `git remote get-url origin` — must be `https://github.com/avroybot/delta_t.git`.
  If a different remote is configured, ask before pushing anywhere else.
- `git --no-pager branch --show-current` — the current branch must be `main`.
  If not, ask which branch to land; never discard work to switch.
- `git --no-pager status --porcelain` and `git --no-pager log --oneline -n 5` —
  identify the commits (and any uncommitted changes) that will land.

## 2. Commit uncommitted work

- If `git status --porcelain` is non-empty, list the files that will be
  committed before staging anything, so the report is transparent.
- Stage with `git add -A` and commit with `GIT_EDITOR=true git commit -m
  "<message>"`, deriving the message from `git --no-pager diff` /
  `git --no-pager diff --cached`.
- Never commit credential-like files (`.env`, `*.pem`, token files) or obvious
  local junk: skip them and say so in the report.
- Agent tooling directories such as `.agents/` are not auto-committed; include
  them only when the user explicitly asks.
- Continue even if the tree was already clean; do not stop here when there is
  nothing to commit.

## 3. Sync with the destination

- `git fetch origin`
- If local `main` contains everything on `origin/main` (no commits in
  `main..origin/main`), continue to the push step.
- If `origin/main` has commits local `main` lacks, the branches have diverged.
  Per the established preference, do not resolve automatically: stop, show the
  divergence with `git --no-pager log --oneline main..origin/main` and
  `git --no-pager log --oneline origin/main..main`, and wait for the user's
  decision. Proceed only after they choose how to handle it.
- Never use `--force` or `--force-with-lease` unless the user explicitly
  requests it.

## 4. Push

- `git push origin main`
- On an auth failure or rejected push, stop and report the exact error. Do not
  retry with force, and do not modify the user's Git configuration.

## 5. Verify and report

- `git fetch origin`
- Compare `git rev-parse main` with `git rev-parse origin/main`. When the two
  hashes match, the merge succeeded; report it with the remote tip from
  `git --no-pager log --oneline -n 3 origin/main`.
- If the hashes differ or the fetch fails, report that the merge did not
  complete and why.

## Stop and report (blockers)

In each case, state plainly that the changes have not merged and what is
needed to proceed:

- The push is rejected or authentication fails.
- `origin/main` has diverged from local `main` (established preference: pause
  for the user's decision).
- `origin` points at an unexpected repository.
- Nothing to land: the tree is clean and `main` already equals `origin/main` —
  say the change is already merged; do not invent work.

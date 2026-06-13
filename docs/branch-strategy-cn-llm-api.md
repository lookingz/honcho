# Branch Strategy For `main-cn-llm-api`

## Purpose

This repository is maintained as a fork of upstream Honcho with long-term
customizations for domestic LLM API compatibility. The goal of this branching
strategy is to make upstream sync safer while keeping local adaptations easy to
maintain.

This document is for the local fork workflow, not for upstream contribution.

## Branch Roles

### `upstream/main`

- Represents the latest official Honcho code.
- Never commit directly to it.
- Only use it as the source for future syncs.

### `main`

- `main` is now intentionally kept as a clean mirror of upstream `main`.
- Do not put domestic LLM API adaptations or local-only fixes on `main`.
- Use it as a reference baseline and as the fork branch that stays aligned with upstream.
- If GitHub shows "Sync Fork" for `main`, that action should now be safe in principle because `main` no longer carries fork-specific product logic.

### `main-cn-llm-api`

- This is the long-term primary development branch.
- All domestic LLM API adaptations live here.
- Continue normal feature work, fixes, and configuration adjustments here.
- Current policy decisions already carried by this branch include:
  - Keep `MAX_BATCH_SIZE`
  - Accept upstream `send_dimensions` and `dimensions_mode`
  - Accept deprecation of `VECTOR_STORE_DIMENSIONS`
  - Follow upstream `TOOL_CHOICE` behavior

### `sync/upstream-YYYYMMDD`

- Temporary branch used only for syncing new upstream commits.
- Create it from `main-cn-llm-api`.
- Merge `upstream/main` into it.
- Resolve conflicts and run verification there first.
- Merge it back into `main-cn-llm-api` after validation.

## Current Repository Status

- As of `2026-06-13`, `main` has been reset to a clean upstream mirror.
- The previous fork-specific `main` history was preserved in backup branch
  `backup/main-before-upstream-mirror-20260613`.
- `main-cn-llm-api` remains the only long-term branch for local product behavior,
  domestic LLM API compatibility, and conflict resolution decisions.

## Why `main` Must Stay Clean

- A dirty fork `main` makes GitHub fork sync confusing because the web UI treats
  fork-specific commits as conflicts against upstream.
- A clean `main` gives a stable answer to "what is upstream doing right now?"
- Keeping all local behavior on `main-cn-llm-api` makes future merges easier to
  reason about because there is one customization line instead of two.
- If a local change matters for production, it belongs on `main-cn-llm-api`, not
  on `main`.

## Naming Convention

### Long-term branch

Use:

```bash
main-cn-llm-api
```

This name emphasizes that the branch is the main customized line for domestic
LLM API compatibility.

### Temporary upstream sync branches

Use:

```bash
sync/upstream-20260522
sync/upstream-20260601
```

Use the actual sync date in `YYYYMMDD` format.

## Daily Development Workflow

Do normal development on `main-cn-llm-api`.

```bash
git switch main-cn-llm-api
git pull --ff-only
```

Make changes, then commit and push normally:

```bash
git add .
git commit -m "Describe the change"
git push
```

Do not start daily work from `main` unless you intentionally need to inspect the
latest upstream baseline.

## Upstream Sync Workflow

Before syncing upstream:

- Commit or stash local uncommitted work.
- Do not start an upstream merge with a dirty working tree.

Recommended sync steps:

```bash
git fetch upstream
git switch main
git pull --ff-only
git switch main-cn-llm-api
git pull --ff-only
git switch -c sync/upstream-YYYYMMDD
git merge upstream/main
```

After the merge:

- Resolve conflicts.
- Run targeted tests for changed areas.
- Run broader tests if the merge touches core config, embeddings, vector store,
  routing, or SDK behavior.

If everything looks good:

```bash
git switch main-cn-llm-api
git merge sync/upstream-YYYYMMDD
git push
```

Then remove the temporary branch:

```bash
git branch -d sync/upstream-YYYYMMDD
```

## Conflict Resolution Rules

When the same areas change both locally and upstream, use these standing
decisions unless a future requirement changes them:

- Preserve domestic LLM API compatibility work on `main-cn-llm-api`
- Keep `MAX_BATCH_SIZE`
- Accept upstream `send_dimensions` and `dimensions_mode`
- Treat `EMBEDDING_VECTOR_DIMENSIONS` as authoritative
- Treat `VECTOR_STORE_DIMENSIONS` as deprecated
- Follow upstream `TOOL_CHOICE` default behavior unless a new product decision
  overrides it

If a future merge touches these same areas again, start from these decisions
instead of re-deciding from scratch.

## Safe Working Rules

- Do not develop on `temp/*` branches long term.
- Do not merge upstream directly into a dirty branch.
- Do not put fork-only commits on `main`.
- Do not force `main` to mirror `main-cn-llm-api`.
- If `main` ever drifts again, back it up first, then restore it to the upstream
  mirror instead of carrying local fixes there.
- Prefer one sync branch per upstream merge event.
- Verify conflict-heavy merges before pushing.

## Quick Command Reference

### Continue normal work

```bash
git switch main-cn-llm-api
git pull --ff-only
```

### Refresh clean `main`

```bash
git switch main
git pull --ff-only
```

### Start an upstream sync

```bash
git fetch upstream
git switch main-cn-llm-api
git pull --ff-only
git switch -c sync/upstream-YYYYMMDD
git merge upstream/main
```

### Finish an upstream sync

```bash
git switch main-cn-llm-api
git merge sync/upstream-YYYYMMDD
git push
git branch -d sync/upstream-YYYYMMDD
```

## Optional Future Adjustment

If `main` becomes unnecessary later, it can remain as a passive baseline branch.
There is no need to force daily usage on `main` as long as `main-cn-llm-api`
remains the primary maintained branch.

---
name: leftovers
description: End-of-branch cleanup. Finds what this branch added that the finished work doesn't use (abandoned first attempts, helpers that duplicate existing ones, unused new files, exports, packages and options, debug logging, commented-out code, agent summary files) and removes it once you confirm. Use before committing or opening a PR, or when the user asks to "clean up leftovers", "tidy up after the agent", or "remove what we didn't use".
argument-hint: "[base branch]"
disable-model-invocation: true
allowed-tools: "Bash(git diff:*), Bash(git log:*), Bash(git merge-base:*), Bash(git rev-parse:*), Bash(git status:*), Bash(git ls-files:*), Bash(git grep:*), Bash(git show:*), Bash(npx -y knip:*), Bash(npx knip:*), Read, Grep, Glob"
---

# Leftovers

## Arguments

Raw arguments: $ARGUMENTS

Parse the arguments as the **base branch**. If empty, use the repo's default branch (`git rev-parse --abbrev-ref origin/HEAD`, falling back to `main`).

## Goal

Before this work is committed or merged, find everything **this branch added** that the finished solution doesn't use, and remove it once the user agrees. Agents try things, change approach and move on without clearing up. This is the clear-up.

**Scope is this branch only.** Cruft that existed before the branch is out of scope; mention `/cruft-hunt` for that if you notice any, in one line at most.

## Instructions

### 1. Collect what the branch changed

- `git diff --name-status <base>...HEAD` for committed changes.
- `git status --porcelain` for uncommitted and untracked files.
- `git log --oneline <base>..HEAD` to see how the work went: a commit that "switches to" or "replaces" something is a sign a first attempt may still be there.

If nothing changed, output `✅ No leftovers. This branch has no changes.` and stop.

### 2. Look for leftovers in what changed

- **First attempts**: two ways of doing the same job, both added on this branch, where only one is wired up.
- **Duplicate helpers**: a new helper that does what an existing one (from before the branch) already does. Search for the job, not just the name.
- **Unused additions**: new files, exports, functions, components, routes or config nothing in the final code uses. For JS/TS run `npx -y knip --no-progress` and keep only findings in files this branch touched.
- **Packages**: dependencies the branch added that aren't imported, or are used once for something the platform or an existing dependency already does.
- **Newly orphaned**: things that existed before the branch but that this branch removed the last use of: a package whose only import was replaced, a helper or type nothing calls any more. The branch caused it, so it's a leftover. Say "this branch removed its last use" and cite the commit.
- **Just-in-case**: new options, parameters or config keys nothing passes or reads.
- **Litter**: debug logging, commented-out code, TODOs for work that's now done, agent-written summary files (`IMPLEMENTATION_SUMMARY.md`, `*_NOTES.md`) the user didn't ask for, tests for code that was later removed.

**A test is not a caller.** A new function only its own new test uses is a leftover.

Before listing an item, check it isn't reached another way (a string, a folder scan, config, a script, a route something outside could call). If it might be, leave it out.

### 3. Output the list

Use this format exactly:

```
## Leftovers on `feature/subscriber-search` (since main)

1. `client/src/hooks/useSubscriberSearch.ts`. Added in a8a496f, replaced by server-side search in c68ee09. Nothing imports it.
2. `lodash`. Added for `debounce` in Subscribers.tsx; that's the only use.
3. `console.log('DEBUG search', …)` at server/routes/subscribers.ts:12.
4. `IMPLEMENTATION_SUMMARY.md`. Agent summary, nothing links to it.

4 leftovers. Remove all, some (give the numbers), or none?
```

If there are none, output exactly:

```
✅ No leftovers. Everything this branch added is used.
```

### 4. Remove (only after the user answers)

- Remove exactly the items the user picked: nothing else. For packages, use the package manager so the lockfile updates.
- Replace a removed package's single use with the platform equivalent only if the user picked that item and the replacement is a few lines.
- Run the project's checks (from `AGENTS.md`, `CLAUDE.md`, `README*` or the manifest's scripts).
- If a check fails, put that item back and report it. **Never edit a test to make a check pass.**
- Don't commit unless the user asks. Report what was removed and the check results in a few lines.

## Rules

- **This branch only.** Pre-existing cruft is out of scope, unless this branch removed its last use.
- **Evidence on every item**: a commit hash, a `file:line`, or tool output.
- **When in doubt, leave it out.** Anything that might be reached another way isn't a leftover.
- **Ask before removing anything.**
- **No preamble.** Start with the `## Leftovers` heading or the ✅ line.

---
name: zombies
description: Suggest tests worth writing for a feature using the ZOMBIES heuristic (Zero, One, Many, Boundaries, Interface, Exceptions, Simple scenarios). Pass a free-text feature description, or omit args to use the current branch's diff. Outputs only the categories that apply — not a full ZOMBIES checklist.
argument-hint: "[feature description]"
disable-model-invocation: true
allowed-tools: "Bash(git diff:*), Bash(git log:*), Bash(git merge-base:*), Bash(git rev-parse:*), Bash(git status:*), Bash(find:*), Bash(ls:*), Bash(grep:*), Read, Grep, Glob"
---

# ZOMBIES

## Arguments

Raw arguments: $ARGUMENTS

If arguments are provided, treat them as a **free-text feature description** (e.g. "sign-in code login flow", "image upload validation"). Locate the relevant code and tests yourself using `Grep`/`Glob`.

If arguments are empty, run `git diff main...HEAD` and use the diff as the feature scope.

## Goal

Identify the **most valuable tests to write** for the feature, using ZOMBIES as a thinking tool. The output is a list of test ideas the user can stub out themselves — **do not write or stub the tests**.

ZOMBIES stands for:

- **Z**ero — no inputs / empty state
- **O**ne — a single input / the happy path
- **M**any — multiple inputs, ordering, pagination, concurrency
- **B**oundaries — limits, off-by-one, min/max lengths, timing edges, type edges
- **I**nterface — the contract/shape of the public API (return types, status codes, redirects)
- **E**xceptions — invalid input, failures, expired/used/missing state, auth failures
- **S**imple scenarios — the common everyday usage paths a real user takes

**Skip categories that don't apply.** A read-only endpoint may have nothing under "Many". A pure validator may have nothing under "Interface". Only list tests that are genuinely worth writing — quality over coverage.

## Instructions

### 1. Locate the feature

- **With args**: search the codebase with `Grep`/`Glob` for files matching the description. Read the implementation files (controllers, models, actions, validators) and the existing test file if one exists.
- **Without args**: run `git diff --name-status main...HEAD` and `git diff main...HEAD`, then read the changed implementation files. Skip auto-generated files (lockfiles, compiled assets, generated route/type definitions).

### 2. Generate ZOMBIES suggestions

For each ZOMBIES letter, ask: *is there a test here that would catch a real bug or document real behaviour?* If yes, list it. If no, skip the letter.

**Cross-reference against the existing test file.** This skill outputs tests *to write*, not a coverage map — so:

- **Skip behaviours already fully covered** by an existing test. A covered behaviour is not a test to write; listing it buries the gaps that matter.
- **Keep a behaviour that's only partially covered** — where a test exercises it but misses an important assertion (e.g. asserts a successful login redirect but never checks the code is consumed). Prefix the bullet with `[partial]` and name the missing assertion.
- When unsure whether a test covers a behaviour, keep the bullet rather than dropping it — a false "already covered" silently hides a real gap.

Suggestion quality bar:

- **Specific, not generic.** "Test maximum email length (255 chars)" beats "Test boundaries".
- **Reference real values from the code** when possible — column lengths from migrations, expiry windows from config, validation rules from FormRequests.
- **One test per bullet.** Don't combine "test A and B" into one line.
- **Phrase as a behaviour to verify**, not as a method name. "Expired sign-in code returns 422" beats "test_expired_code".

### 3. Output the report

Group by feature area first (if the diff covers multiple features), then by ZOMBIES letter within each. **Letters always appear in ZOMBIES order — Zero, One, Many, Boundaries, Interface, Exceptions, Simple.** Skipping a letter never reorders the rest: the letters you do show must still run top-to-bottom in that fixed sequence (e.g. show Boundaries before Interface before Exceptions, even if Zero, One, and Many were all skipped). Use this format exactly:

```
## 🧟 [Feature Area]

**Boundaries**
- Email field rejects values longer than 255 chars (matches migration column)
- Sign-in code expires exactly at the 15-minute mark

**Exceptions**
- Expired code returns a validation error
- Already-used code cannot be redeemed twice
- Submitting code for an unknown email fails silently (no user enumeration)

**Simple**
- Requesting a code emails the user and creates a `SignInCode` row
- [partial] valid code logs the user in — existing test asserts the redirect but never asserts the code is consumed
```

The `🧟` prefix and `##` level are what make a feature heading stand out from the bold `**Letter**` sub-headings — keep both. If multiple features are in scope, repeat the block per feature, separating each with a `---` horizontal rule so the boundaries between features are obvious.

End with a one-line summary: `X test ideas.`

If there's nothing worth testing (e.g. trivial rename, pure config change), output exactly:

```
✅ Nothing worth writing tests for.
```

## Rules

- **Don't stub the tests.** This skill outputs ideas only — the user writes the tests.
- **Skip ZOMBIES letters that don't apply.** Do not write "(none)" placeholders. Quality over coverage.
- **Gaps only.** Skip behaviours an existing test already fully covers. Keep partially-covered behaviours, prefixed with `[partial]` and naming the missing assertion. When unsure, keep the bullet — never silently drop a real gap.
- **Always preserve ZOMBIES order.** The displayed sections must follow Zero → One → Many → Boundaries → Interface → Exceptions → Simple. Skipping letters is fine; reordering the remaining ones is not.
- **One heading per letter, per feature area.** Each ZOMBIES letter appears at most once within a feature area — collect all of that letter's bullets under its single heading. Never repeat a letter's heading.
- **Be specific.** Reference actual lengths, timings, statuses, route names from the code. Generic suggestions are worthless.
- **One behaviour per bullet.** No "and" joining two tests.
- **No implementation hints.** Don't suggest assertions, factories, or test setup — just what to verify.
- **Group by feature first, then by letter.** Don't dump everything under one giant ZOMBIES list when the diff spans multiple features.
- **No preamble.** No "Here are the tests I'd suggest…". Start with the first `## [Feature Area]` heading.
- **No closing advice** beyond the `X test ideas.` summary line.

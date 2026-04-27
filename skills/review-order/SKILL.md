---
name: review-order
description: Prepare a structured, scannable review checklist for a branch or diff, grouped by feature change and applying the four-pass review order (Types, Data Flow, Business Logic, Edge Cases) within each feature. Use when the user asks to "prep a review", "review order", or wants a checklist to manually walk through changes.
argument-hint: "[branch]"
disable-model-invocation: true
allowed-tools: "Bash(git diff:*), Bash(git log:*), Bash(git merge-base:*), Bash(git rev-parse:*), Bash(git status:*), Read, Grep, Glob"
---

# Review Order

## Arguments

Raw arguments: $ARGUMENTS

Parse the arguments as the **base branch** to diff against. If empty, default to `main`.

## Goal

Produce a **scannable map** of the changes between the base branch and `HEAD`, grouped by feature and following the four-pass review order (Types → Data Flow → Business Logic → Edge Cases) within each feature.

The user is going to read this map, then jump into the code themselves. So every bullet needs a `file:line` so they can click straight in. Keep prose minimal — one short clause per bullet. Think index, not essay.

**Do not perform the review. Do not make suggestions. Do not flag things to verify, confirm, double-check, or consider. Do not propose tests, fixes, or improvements. Do not ask "is this intended?" or "is this safe?".**

Your job is purely descriptive: name the types, point at where data flows, state what the logic does, and *point at* where edge-case surfaces live. The user reads the map and decides for themselves what to check.

## The review order (why this structure)

Most people read diffs top to bottom. That misses things. The user reads in this order *within each feature*:

1. **Types and interfaces first.** Describe the data shapes the code expects and produces.
2. **Data flow second.** Trace where data comes in, gets transformed, and goes out.
3. **Business logic third.** State what the code does — methods, guards, branches.
4. **Edge cases last.** Point at the *locations* where edge-case surfaces exist (branches, nullable values, external input, concurrent paths, queue-context assumptions). Just the location and what makes it an edge-case surface. No advice.

Your job is to make this order easy to follow per feature, not to mash everything into four global buckets.

## Instructions

### 1. Get the diff

Run `git diff <base>...HEAD` (three-dot — i.e. changes on this branch since it diverged) to see what's changed. Also run `git diff --name-status <base>...HEAD` for a quick file-level overview. If there are no changes, tell the user and stop.

### 2. Read the changed files

For every changed file, read enough of the file (and surrounding files only when needed to understand a data flow) to categorise it.

Skip auto-generated files unless they contain something unexpected:
- Lockfiles (`composer.lock`, `package-lock.json`, `pnpm-lock.yaml`, etc.)
- Compiled assets (`public/build/`, `dist/`)
- Generated route/type definitions (e.g. Wayfinder action/route files, generated TS types)

### 3. Identify the feature changes

Group the diff into the **distinct user-facing or behavioural changes** the branch introduces. A feature is a coherent unit of behaviour, not a file or a layer. Examples of how to split:

- "Unsubscribe via signed link" — controller, route, blade unsubscribe-CTA, page, tests
- "Auto-subscribe on idea creation" — observer hook, migration relationship usage, test
- "Notify subscribers on status change" — observer, notification class, mail template, tests

Heuristics for identifying features:
- A new model/migration usually anchors at least one feature (the thing it represents).
- A new route + controller + page is almost always one feature.
- A new notification class + its trigger (observer/event) + its mail template is one feature.
- Side-effects added to *existing* flows (e.g. "voting now also subscribes you") are their own small feature.
- Pure refactors with no behavioural change can be grouped as a single "Refactor" feature at the end.

If the branch is genuinely a single feature, emit one feature heading. Don't force a split.

### 4. Output the map

Return **only** the markdown document. No preamble, no summary at the end. Order features roughly by importance / blast radius (auth/security first, refactors last).

**Voice.** Bullets, not paragraphs. Each bullet is `path/to/file.ext:LINE — short clause`. The clause names what's there in plain words; the `file:line` is what the user clicks. One line per bullet. No multi-sentence explanations. Think `git grep` output with a human-readable suffix, not a code review.

Use this structure:

```
## Feature: [short name of the change]

One sentence describing what behaviour this introduces or changes.

### 1. Types & Interfaces
- `path/file.php:12` — what's there (e.g. "new `subscribers` BelongsToMany on `Idea`")
- `path/migration.php:14` — `idea_subscribers` table, unique on `(idea_id, user_id)`

### 2. Data Flow
- `path/controller.php:23` — entry: `POST /x` → invokable
- `path/observer.php:18` — fan-out: queries subscribers, dispatches notification
- `path/notification.php:30` — exit: queued mail via `mail.foo` template

### 3. Business Logic
- `path/observer.php:22` — short-circuit on `is_internal`
- `path/observer.php:28` — actor-exclusion gate
- `path/controller.php:14` — `detach` runs unconditionally on bound pair

### 4. Edge Cases
- `path/observer.php:25` — `Auth::id()` is null in queue/console contexts
- `path/controller.php:14` — public route, no auth check, signed URL has no expiry

---

## Feature: [next change]

...
```

Omit a section if there's nothing meaningful in it. A small feature might be just Logic + Edges.

## Rules

- **Every bullet has a `file:line`.** That's the whole point — the user clicks it to jump. No bullet without a citation. If you genuinely need to say something that has no specific line (a feature-level note), put it in the one-sentence intro under the feature heading, not in the bullets.
- **One short clause per bullet.** Aim for under ~15 words after the em-dash. If you need more, you're writing a review, not a map. Split into two bullets at different lines, or cut.
- **No suggestions, no advice, no questions.** Banned phrasings: "verify that…", "confirm…", "make sure…", "consider…", "check whether…", "is this intended?", "is this safe?", "should we…", "could…", "might want to…". State facts only.
- **No emojis.** If something is security-sensitive, say so in words in the feature intro — don't decorate.
- **Description over prescription.** "`detach` runs unconditionally on the bound pair" is fine. "Confirm it's safe that `detach` runs unconditionally" is not.
- **Edge Cases name the surface, not the fix.** Nullable input, unguarded branch, external state, queue-context assumption, concurrent path. Don't tell the reviewer what to do about it.
- **Group by feature, not by review pass.** The four passes go *inside* each feature, in order, every time.
- **Omit a pass within a feature if nothing fits it.** Don't pad with filler.
- **Skip auto-generated files** (lockfiles, compiled assets, generated route/type definitions) unless they contain something unexpected.
- **Order features by blast radius**: security/auth-sensitive first, user-facing behaviour next, internal/refactor last. Flag the sensitive surface in the feature intro sentence.
- **A file can appear under multiple features** (e.g. a controller that gained two unrelated endpoints). Within one feature, a given line should only appear once — pick the most useful pass for it.

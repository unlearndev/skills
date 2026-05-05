---
name: triage
description: Triage a branch or diff by grouping changed files into feature areas, assigning each a risk tier (High/Medium/Low), and producing a scannable summary that helps decide where to spend review time. Use when the user asks to "triage" a branch, PR, or diff.
argument-hint: "[branch]"
disable-model-invocation: true
allowed-tools: "Bash(git diff:*), Bash(git log:*), Bash(git merge-base:*), Bash(git rev-parse:*), Bash(git status:*), Read, Grep, Glob"
---

# Triage

## Arguments

Raw arguments: $ARGUMENTS

Parse the arguments as the **base branch** to diff against. If empty, default to `main`.

## Goal

Produce a **risk-ordered map** of the changes between the base branch and `HEAD`, grouped by feature area, so the user can decide where to spend their review time. The output is a triage report — not a review.

**Do not perform the review. Do not make suggestions. Do not propose fixes. Do not flag specific lines to verify.** The user reads this map and decides for themselves what to dig into.

## Instructions

### 1. Get the diff

Run `git diff --name-status <base>...HEAD` for a file-level overview, then `git diff <base>...HEAD` to see the actual changes. If there are no changes, tell the user and stop.

### 2. Read the changed files

For every changed file, read enough of the file to understand what feature area it belongs to and how risky it is. You don't need to read every line of every file — read enough to categorise it.

### 3. Group by feature area

Group files by **what the code does**, not by file type. Good group names: "Authentication", "Notification Delivery", "User Data Mutations", "Schema & Config". Bad group names: "Controllers", "PHP Files", "Vue Components".

Heuristics:
- A controller + its route + its page + its tests usually form one feature group.
- A migration + the model changes that go with it form one feature group.
- A notification class + its trigger + its mail template form one feature group.
- A side-effect added to an existing flow (e.g. "voting now also subscribes you") is its own small group.
- Pure refactors with no behavioural change can be a single "Refactor" group.

Each file appears in exactly one group — pick the most relevant one. Keep group names short (2–4 words).

### 4. Assign a risk tier

Tier each group as **High**, **Medium**, or **Low**.

**Always High** if the group:
- Touches authentication or authorization
- Mutates user data, payments, or billing
- Modifies permissions or access control
- Handles sensitive data (passwords, tokens, PII)

**Bump up one tier** if the group:
- Has complex conditionals or branching logic
- Interacts with external services or APIs
- Contains a large amount of generated code (100+ lines that look like a single AI generation)

Otherwise: pick the tier that matches the blast radius of the change.

### 5. Identify auto-generated files

Auto-generated files go in a "Skip" section at the bottom. These include:
- Lockfiles (`composer.lock`, `package-lock.json`, `pnpm-lock.yaml`)
- Compiled assets (`public/build/`, `dist/`)
- Generated route/type definitions (Wayfinder action/route files, generated TS types)

### 6. Output the report

Use this exact format:

```
### ⚠️ High Risk

**[Feature Area Name]**
Reason: [one sentence explaining why this is high risk]
- path/to/file.php (added/modified)
- path/to/other-file.php (modified)

**[Another Feature Area]**
Reason: [one sentence]
- path/to/file.php (added)

### Medium Risk

**[Feature Area Name]**
Reason: [one sentence]
- path/to/file.php (added)

### Low Risk

**[Feature Area Name]**
- path/to/file.php (modified)

### Skip

**Auto-generated**
- path/to/generated/files... (auto-generated — skip)
```

End with a one-line summary: `X groups, Y high-risk files to focus on`.

## Rules

- **Group by feature area, not by file type.** "Auth flow" not "Controllers". "Notification delivery" not "PHP files".
- **One group per file.** Pick the most relevant one.
- **Group names: 2–4 words.** Short and descriptive.
- **One-line reason per group** explaining WHY it's that risk level. Only required for High and Medium tiers — Low can skip the reason.
- **Status suffix on each file**: `(added)`, `(modified)`, or `(deleted)`. Match `git diff --name-status`.
- **Order tiers from highest to lowest risk.** High first, then Medium, then Low, then Skip.
- **Omit a tier section if it's empty.**
- **If everything fits in one tier**, say so and suggest reviewing everything: "This diff is small — review all files".
- **No suggestions, no advice.** This is a triage map, not a review. Don't say "verify…", "consider…", "make sure…".
- **No emojis** other than the ⚠️ on the High Risk heading.
- **End with a one-line summary**: `X groups, Y high-risk files to focus on`.

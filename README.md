# Skills

A collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for product development workflows by [Unlearn](https://unlearn.dev).

## Install

```sh
npx skills add unlearndev/skills
```

This uses [skills.sh](https://skills.sh) to install all skills into your project's `.claude/skills/` directory. Once installed, Claude Code will automatically detect and use them based on your prompts.

To install a single skill:

```sh
npx skills add unlearndev/skills --skill spec-generator
```

## Skills

| Skill | Description |
|-------|-------------|
| [spec-generator](#spec-generator) | Turn a vague idea into a detailed product spec |
| [feature-generator](#feature-generator) | Expand a spec into an implementation-ready feature list |
| [code-review](#code-review) | Review staged changes or a specific area of the codebase |
| [checklist](#checklist) | Convert plans or reviews into persistent markdown checklists |
| [review-order](#review-order) | Prepare a structured, scannable review map of a branch's changes |
| [first-five](#first-five) | Triage a branch against the First Five checklist and flag only real concerns |
| [triage](#triage) | Group a branch's changes into feature areas and assign risk tiers |
| [zombies](#zombies) | Suggest the most relevant tests to write for a feature using the ZOMBIES heuristic |
| [warm](#warm) | Evaluate newly added or upgraded dependencies against the WARM check |

### spec-generator

Generate a detailed product specification from a rough idea, description, or supporting materials (screenshots, notes, wireframes).

```
> Write up a spec for a Slack bot that summarizes daily standups
> Turn this idea into a requirements doc
> /spec-generator
```

Outputs a structured markdown spec covering overview, user stories, data model, API, UI, and edge cases.

### feature-generator

Expand a `spec.md` into a full `features.md` ordered by implementation dependency, or keep the two files in sync when either changes.

```
> Generate features from the spec
> Sync spec and features
> /feature-generator
```

Features are ordered for optimal AI agent implementation: foundation first, then simple flows, complex features, admin features, and cross-cutting concerns.

### code-review

Review staged git changes or a specific area of the codebase. Optionally delegate to an alternative agent (codex, aider, goose).

```
> Review my staged changes
> /code-review codex auth-module
> /code-review
```

Findings are classified by severity: Critical, Error, Warning, Suggestion, and Nitpick.

### checklist

Convert the current plan, code review, or any structured content into a persistent markdown checklist saved to `.claude/plans/`.

```
> Turn this plan into a checklist
> /checklist
```

Creates a trackable file with checkboxes that the agent checks off as it works through the implementation.

### review-order

Prepare a scannable map of a branch's changes, grouped by feature and ordered for review (Types → Data Flow → Business Logic → Edge Cases) with `file:line` citations.

```
> Prep a review of this branch
> /review-order
> /review-order develop
```

Outputs a descriptive map (no suggestions, no questions) so you can jump straight into the code yourself.

### first-five

Scan a branch or diff against the **First Five** checklist (Error Handling, Input Boundaries, External Calls, State Mutations, Assumed Dependencies) and report only genuine concerns — verified against the codebase, not flagged on suspicion.

```
> Run the first five on this branch
> /first-five
> /first-five develop
```

Outputs a short, scannable triage list with `file:line` citations and ⚠️ markers for high-severity findings. No fixes proposed, no clean-section placeholders.

### triage

Map a branch's changes into feature-area groups, each tiered as High / Medium / Low risk, so you can decide where to spend your review time. This is a triage map — not a review.

```
> Triage this branch
> /triage
> /triage develop
```

Outputs a risk-ordered report grouped by feature area (e.g. "Authentication", "Notification Delivery"), with auto-generated files pushed to a Skip section. No suggestions, no fixes — just a map.

### zombies

Identify the most valuable tests to write for a feature using the ZOMBIES heuristic (Zero, One, Many, Boundaries, Interface, Exceptions, Simple scenarios). Pass a free-text feature description, or omit the argument to use the current branch's diff.

```
> /zombies
> /zombies sign-in code login flow
> /zombies image upload validation
```

Outputs a grouped list of test ideas — only the ZOMBIES categories that genuinely apply, with specific values pulled from the code (column lengths, expiry windows, throttle limits) so you can see at a glance what's worth covering and write the tests yourself.

### warm

Evaluate newly added or upgraded dependencies against the **WARM** check (Worth it, Alive, Right-sized, Maintained securely). Covers client- and server-side dependencies in any language by diffing the changed manifests against a base.

```
> WARM check this branch
> /warm
> /warm develop
```

Outputs a per-dependency report scoring each WARM letter with a one-word verdict (Keep, Reconsider, Patch/Pin, Replace), ordered by concern. Facts come from real registry, repo, and advisory lookups — nothing fabricated. Evaluates and reports only; it never edits manifests or runs installs.

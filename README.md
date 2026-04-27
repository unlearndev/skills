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

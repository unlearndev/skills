---
name: spec-generator
description: Generate a detailed product specification document from a vague idea, rough description, or supporting materials (UI sketches, screenshots, notes, existing docs). Use this skill whenever the user wants to turn a rough concept into a structured spec — even if they say things like "write up a spec for...", "turn this idea into a requirements doc", "I have this rough idea for a product", "help me spec this out", "create a PRD for...", or uploads images/files and wants a spec written from them. Covers all product types including internal tools. Trigger this skill even if the request seems simple — a vague one-liner idea deserves a proper spec just as much as a detailed brief.
---

# Spec Generator Skill

Turn a vague product idea — possibly supported by sketches, screenshots, notes, or other documents — into a clear, well-structured product specification document that a development team can act on.

This always produces a **product spec**. That covers customer-facing products, internal tools, and staff-facing systems — they're all products. Never produce a feature spec, API spec, or integration spec.

## What "good" looks like

A good output spec should:
- Be concise but complete — no padding, no gaps
- Use plain language, not corporate jargon
- Be opinionated where needed (e.g. suggest a sensible default stack or behaviour) while flagging where the team still needs to decide
- Surface implicit assumptions and make them explicit
- Feel like it was written by a thoughtful senior PM or tech lead who has thought through the edges

---

## Process

### Step 1: Gather and understand inputs

Take stock of everything the user has provided:
- **Verbal description** — even one sentence is enough to start
- **Uploaded files** — read all of them: notes, existing docs, data models, wireframes, screenshots, mockups
- **Images** — examine carefully; extract implied features, flows, and constraints

Before drafting, review everything the user has provided and identify any gaps that would meaningfully affect the spec. Always ask about the intended technical stack if it hasn't been provided — this can influence the direction of the project and its features. Ask any other questions that cannot be reasonably inferred from the available inputs — don't ask about things that are already clear or can be decided with a sensible default. Keep the questions concise and grouped in one go, not spread across multiple turns.

### Step 2: Draft the spec

Use the template below. Adapt sections as needed — not every section is required for every spec. Use your judgment. Omit sections that don't apply rather than filling them with placeholder text.

---

## Spec Template

```markdown
# [Product Name] — Specification

> **Context:** [1–2 sentences on why this is being built and what informed this spec — e.g. team discussions, user research, a UI sketch.]

## Overview

[2–4 sentences. What is this? Who uses it? What problem does it solve? What is it NOT?]

---

## Goals

- [Concrete, measurable goal]
- [Another goal]
- [Keep this list short — 3–6 items]

---

## User Roles

### [Role Name]
- [What they can do]
- [What they cannot do / what they see or don't see]
- [How they authenticate or access the system]

[Repeat for each role]

---

## Core Features

### 1. [Feature Name]

[Description. Be specific — what does the user do, what happens, what are the constraints? Mention edge cases where relevant.]

### 2. [Feature Name]

[...]

[Continue for all major features, numbered.]

---

## Technical Stack

- **[Layer]** — [Technology]
- **[Layer]** — [Technology]
```

---

## Writing Guidelines

**Be specific, not vague.** Instead of "users can manage their account", write "users can update their email address and reset their password via a link sent to their registered email."

**Resolve ambiguity with a sensible default.** If the user hasn't specified something (e.g. pagination limit), pick a reasonable value and note it. Don't leave blanks.

**Separate customer-facing from internal.** If there are two audiences (e.g. end users + staff), make this structurally clear — separate sections, separate role descriptions.

**Flag implicit constraints.** If something in the inputs implies a constraint (e.g. a sketch shows no search bar → search is probably out of scope), surface it in the relevant feature description.

**Technical Stack is optional.** Only include it if the stack is known or strongly implied. If unknown, omit the section entirely. When suggesting technologies, web search first to ensure recommendations are current and still the standard choice — avoid packages or libraries that have been superseded or fallen out of common use.

**Never include an "Out of Scope" section.** The spec template is the only structure — do not add any sections beyond what is in the template.

---

## Output Format

Produce the spec as a `.md` file saved to `/mnt/user-data/outputs/spec.md`. Present it to the user using `present_files`.

After presenting the file, give a brief summary (1–2 sentences max) noting any significant assumptions you made.
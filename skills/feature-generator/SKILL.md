---
name: feature-generator
description: Generate a detailed features.md document from a spec.md, or sync changes between spec.md and features.md when either file is updated. Use this skill whenever the user wants to expand a product spec into a full feature list, asks to "generate features", "create features.md", "expand the spec", "update features from spec", or "sync spec and features". Also trigger when the user has modified either spec.md or features.md and wants to keep them in sync. Always use this skill when both files are in play together.
---

# Feature Generator Skill

Expand a `spec.md` into a detailed `features.md`, ordered by the sequence an AI agent would most successfully implement them (dependency order). Keep both files in sync when either is modified.

---

## Step 1: Check for spec.md

Before doing anything, check whether `spec.md` exists in the current context or has been provided by the user.

**If spec.md does not exist:**
Tell the user: "There's no spec.md yet — you'll need to create one first. If you have the spec-generator skill enabled, I can kick that off for you now. Would you like to do that?"

Do not proceed until spec.md is available.

---

## Step 2: Generating features.md

Read spec.md fully. Extract every discrete piece of functionality implied by the spec — from user roles, core features, and technical stack. Then produce `features.md` following the format below.

### Ordering rules

Order features by implementation dependency — what must exist before something else can be built. Think like an AI agent working top to bottom through the file:

1. Foundation first — database, auth, core models
2. Then the simplest user-facing flows that depend on nothing else
3. Then features that build on those
4. Admin/internal features after customer-facing equivalents
5. Cross-cutting concerns (notifications, rate limiting, etc.) last

### features.md format

Each feature follows this exact structure:

```markdown
# [Product Name] — Features

---

## [N]. [Feature Name]

[One sentence describing what this feature does and who it's for.]

**User flow**
1. [Step]
2. [Step]
3. [Step]

**UI overview**
[2–4 sentences describing the key UI elements involved.]

---
```

Rules:
- Every feature gets a number, a user flow, and a UI overview — no exceptions
- User flows are written from the perspective of the actor (customer, team member, visitor)
- UI overview describes what the user sees and interacts with, not implementation detail
- No sub-bullets inside user flows — numbered steps only
- Horizontal rules (`---`) between every feature and after the title

---

## Step 3: Syncing changes

When the user modifies either `spec.md` or `features.md` and asks to sync:

**If spec.md was modified:**
- Identify what changed (new features added, features removed, features altered)
- Update only the relevant features in `features.md` to reflect those changes
- Add new features in the correct dependency order position
- Remove features that no longer exist in the spec
- Do not touch features that are unaffected

**If features.md was modified:**
- Identify what changed (features added, removed, or substantially altered)
- Update only the relevant sections of `spec.md` to reflect those changes
- Preserve all other spec content exactly as-is
- Do not rewrite or reformat untouched sections

Always confirm with the user what changed before syncing, so they can verify the right things are being updated.

---

## Output

Save `features.md` to `/mnt/user-data/outputs/features.md` and present it using `present_files`.

If syncing changes to `spec.md`, save the updated version to `/mnt/user-data/outputs/spec.md` and present both files.
---
name: checklist
description: Convert the current plan, code review, or structured content in context into a persistent markdown checklist file under .claude/plans/
disable-model-invocation: true
allowed-tools: Read, Write, Glob, Bash(mkdir:*), Bash(ls:*)
---

# Checklist

## Your task

Convert the most recent structured content in this conversation (plan, code review, task list, implementation steps, etc.) into a markdown checklist file saved to the project.

## Instructions

### 1. Identify the content

Look at the current conversation context. Find the most recent plan, code review, implementation steps, or other structured content. This is what you will convert.

### 2. Generate a name

Derive a short, descriptive kebab-case name from the content's topic (e.g., `auth-refactor`, `api-migration`, `payment-integration`). The file will be saved as `.claude/plans/<name>-checklist.md`.

### 3. Check for existing file

Before writing, check if `.claude/plans/<name>-checklist.md` already exists. If it does, ask the user whether to overwrite, choose a different name, or cancel. Do NOT proceed without their answer.

### 4. Convert to checklist format

Transform the content into a checklist file following these rules:

- **Preserve the original structure exactly** — keep all headings, sections, groupings, nesting, and ordering intact
- **Only add `- [ ]` checkbox markers** to actionable items — do not rewrite, summarize, or rephrase any content
- Keep non-actionable text (descriptions, context, rationale) as-is without checkboxes
- Maintain heading hierarchy (##, ###, etc.) exactly as in the original
- If items already have bullet points or numbered lists, convert them to `- [ ]` format
- Nested items should use indented checkboxes (e.g., `  - [ ]` for sub-items)

### 5. Add the tracking header

Add this exact block at the very top of the file, before any other content:

```
<!-- CHECKLIST INSTRUCTIONS
IMPORTANT: When working on items in this checklist, you MUST check off each item
(change `- [ ]` to `- [x]`) AS SOON AS it is implemented/completed.
Do not wait until the end — check off each item immediately after finishing it.
This file is the source of truth for tracking progress.
-->
```

### 6. Ensure the directory exists

Create `.claude/plans/` if it doesn't already exist.

### 7. Write the file

Save the checklist to `.claude/plans/<name>-checklist.md`.

### 8. Confirm

Tell the user the file path and a brief summary of what was captured. Remind them that items will be checked off as they are completed.

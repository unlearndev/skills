---
name: judge
description: Judge the current branch's changes against the repo's DECISIONS.md design rules. Use when asked to judge, check, or review a change or branch against the decisions.
---

Judge the current branch's changes against the design rules in DECISIONS.md.

## Look

1. Read `DECISIONS.md` in the repo root. If it doesn't exist, stop and
   report that instead of judging.
2. Find the base: the repo's default branch, unless the user names one.
3. Read the full diff (`git diff <base>...HEAD`) and open every changed
   file, so each change is judged in the context of the file it lands in.

## Judge

Check the change against each rule in DECISIONS.md, in order.

- Only the rules in the file. No other findings, no style feedback, no
  improvement suggestions.
- A violation needs evidence you can point at: a file and line in this
  change. If it doesn't clearly break a written rule, it isn't a violation.
- The same wrong pattern in two places is two violations.

## Report

Output exactly this, and nothing else:

```
JUDGMENT · <branch> vs <base> · <n> violations

1. Rule <k> · <file>:<line>
   <what the code does, one sentence>
   If it ships: <what breaks, one sentence>
```

One numbered entry per violation, ordered by rule then file. If there are
none:

```
JUDGMENT · <branch> vs <base> · clean

Every rule in DECISIONS.md holds for this change.
```

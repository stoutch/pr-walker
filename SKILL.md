---
name: pr-walker
description: Use when the user wants to understand or get up to speed on a pull request or branch — they give a PR link or branch name and want the intention explained, not a line-by-line code review. Triggers: "help me understand this PR", "walk me through this branch", "what is this PR doing", "/pr-walker".
---

# PR Walker

## Overview

Get the user up to speed on a PR **fast**. The goal is understanding INTENTION — what the PR is trying to do and why the code is shaped the way it is — not a code review. This is a learning aid, not an audit.

**Core principle: be concise. The user can always ask to dig deeper. Default to the short version.**

## Workflow

1. **Checkout the code.** Input is a PR link or a branch name.
   - PR link → `gh pr checkout <number-or-url>`
   - Branch name → `git fetch && git checkout <branch>`
   - This mutates the working directory to the PR branch. If there are uncommitted local changes, warn the user before switching.

2. **Find the target (base) branch from GitHub — do NOT assume `main`.** PRs are often stacked, so the base may be a parent branch.
   - `gh pr view <branch-or-number> --json baseRefName,number,title -q '.baseRefName'` gives the real base branch.
   - If there's no PR yet for the branch, `gh pr view` returns no result — fall back to `main` and tell the user you assumed `main` because no PR exists.
   - Use the detected base for the diff. Example: `BASE=$(gh pr view --json baseRefName -q .baseRefName)` then diff against `origin/$BASE`.

3. **Determine the changed files.** Diff against the merge base with the detected target branch: `git diff --stat $(git merge-base HEAD origin/$BASE)...HEAD`. Using the merge base (not a plain `origin/$BASE...HEAD`) keeps a stacked PR's diff scoped to *only this PR's* changes, not the parent PR's. Also check `git status` for uncommitted working-tree changes and include them (note they're uncommitted). State which base branch you diffed against in the overview.

4. **Give a high-level overview** (see format below), then list the files in the order you'll walk them.

5. **Walk files ONE AT A TIME.** Present exactly one file, then STOP and wait for the user to say continue. Do not present the next file until they give the go-ahead.

6. **Stop after the last file.** No recap, no offer of next steps. Done.

## High-level overview format

Keep it tight — a few sentences on what the PR accomplishes, then 2–4 bullets grouping the work by concern. Then a numbered file map of **only the files you'll actually walk** (see below), in the order you'll walk them.

**Order files logically by dependency/story** (e.g. schema → backend → routing → UI), not alphabetically or diff order.

**Do NOT walk trivial / boilerplate files.** Walking a file costs the user a "next" — don't spend that on files with nothing to explain. Bucket every changed file as you scan:
- **Trivial/boilerplate** — pure registration, one-line additions, imports, type plumbing, generated files, lockfiles, mechanical renames. List these together in ONE short batch note in the overview (e.g. "Skipping as boilerplate: `useSubpage.ts` (registers the subpage), `FreightSearchContext.tsx` (one-line case)…"). Do not put them in the numbered walk list.
- **Non-trivial** — anything with real logic, intention, or a non-obvious "why". These are the only files in the numbered walk list.

If a file is *mostly* boilerplate with one interesting piece, walk it but only cover the interesting piece.

## Per-file format (the important part)

**This must be a clean, scannable summary — NOT a wall of text.** The baseline failure this skill fixes is over-explaining. A one-line change gets one line.

For each file use this shape:

```
## File N of M: `path/to/file`

**What changed:** one or two sentences.

**Why it's written this way:** the intention and why it has to be this way. This is the reason the file made the walk list — keep it focused on the non-obvious parts.
```

Files that are entirely boilerplate never reach this format — they're batched in the overview instead (see above).

Hard limits:
- A meaty file: a short paragraph or a few bullets max for the "why" — not multiple paragraphs.
- Do NOT paste diff hunks unless the user asks.
- Do NOT critique, flag bugs, suggest improvements, or note risks. Intention only. If you spot something, hold it unless the user asks.

End each file with a single line like: `Ready for file N+1 (\`name\`) when you are.`

## Red flags — you're doing it wrong if:

- A boilerplate/one-line file is in the numbered walk list → pull it out, batch it in the overview instead.
- You made the user type "next" to get past a file with nothing to explain → that file shouldn't have been walked.
- You wrote "worth flagging…" or critiqued a choice → delete it, this is intention-only.
- You presented two files before the user said continue → stop, wait for go-ahead.
- The user has to scroll to read one file's summary → too long.

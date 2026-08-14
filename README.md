# PR Walker

Understand a pull request's intent—one meaningful file at a time.

PR Walker is an agent skill for getting up to speed on a pull request or branch. It explains what the change is trying to accomplish and why the code is structured that way, without turning the walkthrough into a code review.

## What it does

- Detects the PR's actual base branch, including stacked PRs
- Scopes the diff using the merge base
- Groups trivial changes such as imports, generated files, and registration code
- Orders meaningful files by dependency and narrative
- Explains one file at a time
- Focuses on intention—not bugs, risks, or suggested improvements
- Includes uncommitted working-tree changes in the walkthrough

## Example

Ask your agent:

```text
Help me understand https://github.com/acme/widgets/pull/123
```

Or provide a branch:

```text
Walk me through the feature/new-checkout branch
```

PR Walker begins with a concise overview and file map:

```text
This PR adds an expedited checkout path while reusing the existing
payment flow. I compared it with the PR's actual base branch,
feature/checkout-foundation.

- Adds the expedited-checkout domain model
- Connects it to the checkout API
- Exposes the new path in the UI

Skipping as boilerplate: route registration and generated types.

1. src/checkout/expedited.ts
2. src/api/createCheckout.ts
3. src/components/CheckoutButton.tsx
```

It then explains the first meaningful file and waits for you before continuing.

## Prerequisites

PR Walker expects:

- Git
- A local checkout of the repository you want to explore
- [GitHub CLI](https://cli.github.com/) for pull-request URLs
- An authenticated GitHub CLI session:

```bash
gh auth login
```

> [!IMPORTANT]
> PR Walker checks out the requested PR or branch, which changes the active branch in your working directory. If you have uncommitted changes, preserve them before starting.

## Installation

Clone this repository:

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/pr-walker.git
```

### Claude Code

Copy the skill into your Claude skills directory:

```bash
mkdir -p ~/.claude/skills/pr-walker
cp pr-walker/SKILL.md ~/.claude/skills/pr-walker/SKILL.md
```

### Codex

Copy the skill into your agent skills directory:

```bash
mkdir -p ~/.agents/skills/pr-walker
cp pr-walker/SKILL.md ~/.agents/skills/pr-walker/SKILL.md
```

Restart your agent session if the skill is not detected immediately.

## Usage

Give your agent either a pull-request URL:

```text
Use pr-walker to explain https://github.com/acme/widgets/pull/123
```

Or a branch name:

```text
Use pr-walker to walk me through feature/new-checkout
```

You can also use natural prompts such as:

```text
Help me understand this PR.
What is this branch doing?
Walk me through these changes.
```

After each file, reply with `continue` or `next` to proceed.

## How it works

1. Checks out the requested PR or branch.
2. Reads the PR's target branch from GitHub instead of assuming `main`.
3. Finds the merge base and scopes the diff to the current PR.
4. Includes relevant uncommitted changes.
5. Separates boilerplate from meaningful implementation.
6. Presents a short overview and logically ordered file map.
7. Explains each meaningful file individually, waiting between files.
8. Stops after the final file.

If a branch does not have a corresponding PR, PR Walker falls back to `main` and tells you that it made that assumption.

## What PR Walker is not

PR Walker is a learning aid, not a code-review tool. It intentionally does not:

- Hunt for bugs
- Flag risks
- Critique implementation choices
- Suggest refactors
- Dump full diff hunks
- Explain every mechanical change

Ask for a separate code review if you want an audit.

## Repository structure

```text
pr-walker/
├── README.md
└── SKILL.md
```

`SKILL.md` contains the complete agent instructions.

## Contributing

Contributions that make walkthroughs clearer, shorter, or more accurate are welcome.

When proposing a change, preserve the core behavior:

- Intention over critique
- Meaningful files over boilerplate
- One file per interaction
- Concise explanations by default
- Correct handling of stacked PRs

## License

No license has been specified yet. Add a `LICENSE` file before publishing if you want others to use, modify, or redistribute this skill.

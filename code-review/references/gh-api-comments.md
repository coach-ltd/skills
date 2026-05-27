# GitHub API — PR Comments (CI Reference)

How to post comments from inside `claude-code-action`. All commands must be simple `gh api` calls — no heredocs, no variable prefixes, no file writing.

## Context Variables

Get these ONCE at the start of your review:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number} --jq '.head.sha'
```

Replace `{owner}`, `{repo}`, `{pr_number}` with actual values from the PR context.

## Global Comment (summary, intention, specs)

```bash
gh api repos/{owner}/{repo}/issues/{pr_number}/comments -f body='Your markdown here'
```

Note: PRs are issues in GitHub API — use `/issues/` for global comments.

## Inline Comment (single line)

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body='**Important** — *Agent: Performance*

Your finding here.' \
  -f commit_id='abc123' \
  -f path='src/file.swift' \
  -F line=42 \
  -f side='RIGHT'
```

- `line` — line number in the NEW file (right side of diff)
- `side` — always `RIGHT`
- `commit_id` — HEAD SHA of the PR branch
- `path` — relative from repo root
- Use `-F` (not `-f`) for integer fields like `line`

## Inline Comment (multi-line range)

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body='Your finding here.' \
  -f commit_id='abc123' \
  -f path='src/file.swift' \
  -F start_line=40 \
  -F line=45 \
  -f start_side='RIGHT' \
  -f side='RIGHT'
```

## With Code Suggestion

GitHub renders ` ```suggestion ` blocks as one-click apply buttons:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body='**Best Practice** — *Agent: Architecture*

Consider splitting this.

```suggestion
validateInput(data);
persist(data);
```' \
  -f commit_id='abc123' \
  -f path='src/file.swift' \
  -F line=42 \
  -f side='RIGHT'
```

## RULES — Read carefully

1. **ONE `gh api` call per comment.** Post inline comments one by one. Do NOT try to batch them into a JSON file.
2. **No heredocs.** Never use `<< 'EOF'` or `<< EOF`. Put the body directly in the `-f body='...'` argument.
3. **No variable prefixes.** Never write `VAR=value gh api ...`. If you need a variable, use a separate `gh api` call or hardcode the value.
4. **No file writing.** Never use `cat >`, `python3`, or `node` to write JSON files for `--input`. Just use `-f` flags directly.
5. **No chaining.** Never use `&&` or `||` to chain multiple commands. One command per Bash call.
6. **Escape quotes** in the body with `'\''` (end single quote, escaped quote, start single quote) if needed.
7. **Use `-F` for integers** (`line`, `start_line`) and `-f` for strings (`body`, `path`, `side`, `commit_id`).

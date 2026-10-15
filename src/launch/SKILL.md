---
name: launch
description: "Launch one herdr agent tab per Linear issue to implement it and open a PR, no merge. Claude (Opus 5.5 high, local worktree) by default, or Amp on an orb (GPT 6.1 Sol). Also closes tabs whose PRs merged. Use for /launch, 'do same for <issue urls>', 'launch agents for these issues', 'close tabs with merged prs'."
disable-model-invocation: true
---

# Launch

Load the herdr skill first (`~/.agents/skills/herdr/SKILL.md`) for CLI details.

Input: Linear issue URLs, plus optional agent choice (`claude` default, `amp sol` or `amp claude`) and extra per-issue instructions. Needs herdr: check `test "${HERDR_ENV:-}" = 1`. Workspace is the current one (`herdr pane list` shows it). Launch issues in parallel where possible.

## Names

Ticket `OPT-12345` gives:
- tab label `opt12345 : <2 word description>`, e.g. `opt12345 : date remove`
- agent name `opt12345`
- branch `lawsonoates/opt-12345-<kebab-of-title>`
- worktree `/Users/lawsonoates/support-opt-12345` (Claude only)

## Claude (default)

1. `git fetch -q origin main && git worktree add -q -b <branch> <worktree> origin/main`. Never `scripts/worktree/new.sh`.
2. `herdr tab create --workspace <ws> --cwd <worktree> --label "<label>" --no-focus`. Take `.result.root_pane.pane_id`.
3. `herdr agent start <agent> --kind claude --pane <pane> --timeout 60000 -- --model claude-opus-5-5 --effort high`

## Amp on an orb

Orbs run in a remote clone at `/home/user/workspace/repo` and cannot see local folders. Make no worktree.

1. `herdr tab create --workspace <ws> --cwd /Users/lawsonoates/support --label "<label>" --no-focus`
2. `herdr agent start <agent> --kind amp --pane <pane> --timeout 60000 -- --executor orb -m <mode>`, both on high effort:
   - `amp sol`: mode `sol-high` (user plugin, GPT-6.1 Sol at high effort). Header shows "sol-high".
   - `amp claude`: mode `external-agent:claude-code`. Header shows "claude code · claude opus 5.5 · high effort".
   An unknown `-m` silently falls back to `medium`, so check the header shows the mode and "In the Orb". List modes with `amp plugins list | grep "agent mode"` and `amp config external-agents list`.
3. The prompt says: work in this orb checkout, fetch origin, create `<branch>` off `origin/main` here. Orb git defaults to `Amp <amp@ampcode.com>`, so the prompt must also say to run `git config user.name "Lawson Oates"` and `git config user.email "lawsonoates@gmail.com"` before the first commit.

## Prompt

`herdr agent prompt <agent> "<text>"`, then `herdr agent wait <agent> --until working --timeout 15000`. The text contains:
- Implement <ticket> "<title>": <url>. Read the issue first. Branch and location as above.
- The user's extra instructions, verbatim.
- Commit, push and open a PR that links the issue and fills `.github/pull_request_template.md` with human confirmation unchecked. Conventional Commits with `[TICKET]` prefix. Do NOT merge, do NOT enable auto-merge.
- Author: only `Lawson Oates <lawsonoates@gmail.com>` for author and committer, gh as lawsonoates, no co-author trailers. Stop and ask if any other identity would be used.
- pnpm may rewrite `pnpm-lock.yaml`. Restore it and never commit unrelated lockfile churn.
- Name sibling issues launched in the same batch and say to stay in scope.
- Exploration issues: write an uncommitted `<TICKET>-report.md` at the repo root, with no implementation, commit or PR.

After launch, read each pane (`herdr pane read <pane> --source recent-unwrapped --lines 40`) and confirm the agent is working, not waiting on a question.

## Close merged tabs

For each tab in the workspace, find the PR(s) from its branch (`gh pr list --head <branch> --state all --json number,state`, or from the pane). Close with `herdr tab close <tab>` only when every PR is MERGED. Leave open, closed-unmerged and unknown tabs, and never close this session's own tab. Keep worktrees.

---
name: github-cli
description: Use the GitHub CLI (gh) to create repos, connect/push local git repos to GitHub, and create, list, view, and update issues and projects. Use when plain git can't do the job (anything that talks to GitHub itself, not just the repo).
---

# GitHub CLI (gh)

## When to use gh vs git
- **git** handles the repo: commit, branch, merge, pull, push to an existing remote.
- **gh** handles GitHub: create a repo on GitHub, auth, issues, PRs, projects, repo settings.
- Rule of thumb: if the command would need a browser on github.com, use `gh`.

## Ground rules
1. Check auth first if any gh command fails: `gh auth status`. Fix with `gh auth login`.
2. Unsure of a flag? Run `<command> --help` before guessing. Never invent flags.
3. Default new repos to `--private` unless the user says public.
4. Ask before anything destructive or public: `gh repo delete`, changing visibility, closing many issues.
5. Before the first push, check for secrets: `.env`, keys, tokens. Make sure `.gitignore` covers them.
6. Prefer `--json` output when you need to read results, so they parse cleanly.

## 1. New repo from an existing local folder (most common)
```sh
cd path/to/project
git init                      # skip if already a git repo
git add -A && git commit -m "Initial commit"
gh repo create my-project --private --source=. --remote=origin --push
```
- `--source=.` uses the current folder, `--remote=origin` adds the remote, `--push` pushes it.
- Omit the name to use the folder name. Add `--description "..."` if wanted.

## 2. New empty repo on GitHub, then clone it
```sh
gh repo create my-project --private --clone
gh repo create my-project --private --gitignore Node --license MIT --clone
```

## 3. Push / sync
```sh
git push -u origin main       # first push of a branch (git is enough)
git pull                      # normal sync (git is enough)
gh repo sync                  # sync a fork's default branch with its upstream
gh repo view --web            # open the repo in the browser
gh repo clone owner/repo      # clone without typing the full URL
```
Local repo has no remote yet? `gh repo create --source=. --remote=origin --push` (section 1).

## 4. Issues (track work)
```sh
gh issue create --title "Add login page" --body "Details here"
gh issue create --title "Fix bug" --label bug --assignee @me
gh issue comment 12 --body "Progress note"
gh issue edit 12 --add-label "in progress"
gh issue close 12 --comment "Done in abc123"
gh issue reopen 12
```
Labels must already exist: `gh label list`, create with `gh label create "in progress"`.

## 5. Fetch issues (todo list)
```sh
gh issue list                                   # open issues, current repo
gh issue list --assignee @me
gh issue list --label bug --state open
gh issue list --json number,title,labels,state  # machine-readable
gh issue view 12                                # full issue + body
gh issue view 12 --comments
gh issue status                                 # what's assigned to / mentions me
```
Different repo: add `-R owner/repo` to any issue command.

## 6. Projects (boards)
Projects need an extra auth scope once: `gh auth refresh -s project`
```sh
gh project list --owner @me
gh project view 1 --owner @me
gh project item-list 1 --owner @me --format json
gh project item-add 1 --owner @me --url https://github.com/owner/repo/issues/12
gh project create --owner @me --title "Roadmap"
```

## 7. Discover anything else with help
gh is self-documenting. When a task isn't covered above:
```sh
gh help                       # top-level command list
gh <command> --help           # e.g. gh issue --help
gh <command> <sub> --help     # e.g. gh issue list --help  (flags + examples)
gh help environment           # env vars like GH_REPO, GH_TOKEN
gh help formatting            # --json, --jq, --template
```
Workflow: `gh help` → pick the command → `--help` on the subcommand → read the EXAMPLES section → run it. Prefer this over guessing.

Last resort for anything without a command: `gh api` (see `gh api --help`).

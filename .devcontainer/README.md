<!--
SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Devcontainer & Claude Code

This repository holds only Markdown documentation, checked by two CIs:
**markdownlint** and **REUSE** (SPDX) compliance. The dev container ships exactly
the tools needed to work on those docs from inside a reproducible environment - no
build toolchain.

It builds from [`Dockerfile`](./Dockerfile). Every tool is installed there
directly - **no devcontainer features** (they have proven more trouble than help,
so they are deliberately avoided):

- a recent **git** (built from source, `>= 2.48`) for relative-path worktrees;
- **reuse** (via `pipx`) - the tool behind the *REUSE Compliance* CI;
- **gh** (the GitHub CLI, from the official GitHub CLI `.deb`) - for pull-request
  / issue workflows;
- **Node.js** and **markdownlint-cli2** (`npm -g`, pinned) - the tool behind the
  *Markdown Lint* CI, so you can lint locally before pushing.

The one exception is the [Claude Code](https://claude.com/claude-code) CLI, added
in `postCreateCommand` (see [Claude Code install & auth](#claude-code-install--auth)).

It is **Docker Compose based** ([`docker-compose.yml`](./docker-compose.yml),
service `dev`). The clone is mounted at `/workspaces/act-contributing`, which the
static `workspaceFolder` in `devcontainer.json` points at. Unlike a code dev
container there is no privileged mode, host networking or X11 forwarding - none of
it is needed here.

## Local checks

Both CIs can be reproduced inside the container:

```bash
# Markdown lint (same tool as the CI)
markdownlint-cli2 "**/*.md"

# REUSE / SPDX compliance (same tool as the CI)
reuse lint
```

## Git worktrees for parallel agents

Worktrees work out of the box, with no extra configuration: they live **inside the
clone**, under the gitignored [`../worktrees/`](../worktrees/) directory, so they
come along with the single workspace mount and need no host layout change.

From **inside the container**, at the repository root:

```bash
git worktree add worktrees/my-feature -b my-feature
```

Two constraints (see [`../worktrees/README.md`](../worktrees/README.md) for the
full explanation):

- **inside the container, not on the host** - the container's git sets
  `worktree.useRelativePaths` (set in `postCreateCommand`) and is recent enough
  (`>= 2.48`) to honour it; a host with an older git writes absolute paths that
  do not resolve inside the container;
- **under `worktrees/`, nowhere else** - only the clone is bind-mounted, so a
  worktree created elsewhere lives only in the container's writable layer and is
  lost on rebuild.

Since this is a docs-only repo, a fresh worktree needs no bring-up: it is usable
straight after `git worktree add`.

## Claude Code install & auth

Claude Code is installed by `postCreateCommand` (see `devcontainer.json`) via the
official installer (`https://claude.ai/install.sh`), which fetches a
self-contained native binary. It is installed there rather than in the Dockerfile
so its ~250MB of binaries - and every version it self-updates to - live in the
container's writable layer, where the CLI's own cleanup can reclaim them, instead
of being pinned in an image layer forever. The auto-updater is left **on**.

**Authentication is interactive - run `claude` once and sign in through the
wizard.** There is no token to mint or paste.

The login persists, so **you only do this once**. `CLAUDE_CONFIG_DIR` (set in
`devcontainer.json`) points Claude's entire config - both the `~/.claude` data dir
and `~/.claude.json`, which holds the credentials, onboarding and trusted-folder
state - at an **isolated named Docker volume** (`act-contributing-claude-config`,
declared in [`docker-compose.yml`](./docker-compose.yml)). A named volume, not a
host bind mount: Claude's plugin index stores **absolute in-container paths**, so
sharing the host `~/.claude` would leak them across containers. The volume persists
across rebuilds and **starts empty** - the image pre-creates `~/.claude` so the
fresh volume is owned by the container user, and on the first build you sign in
once.

## GitHub CLI (gh) & git auth

`gh` is installed from the Dockerfile (official GitHub CLI `.deb`). There is **no
token in a file**: authenticate the same way as every other tool, from inside the
container, with the tool's own mechanism.

- **git** operations authenticate through the host `~/.ssh` mount (the remote is
  `git@github.com:All-Circuits-Technologies/ACT-Contributing.git`), so pushing and
  pulling need nothing extra.
- **gh** API calls (PRs, issues) authenticate interactively - run it once:

  ```bash
  gh auth login
  ```

  Like Claude, the login persists: gh's config (`~/.config/gh`) is an **isolated
  named Docker volume** (`act-contributing-gh-config`, declared in
  [`docker-compose.yml`](./docker-compose.yml)), pre-created in the image so the
  fresh volume is owned by the container user, and it survives rebuilds. You sign
  in once.

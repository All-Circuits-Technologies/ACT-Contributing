<!--
SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Devcontainer & Claude Code

This repository holds only Markdown documentation, checked by two CIs:
**markdownlint** and **REUSE** (SPDX) compliance. The dev container ships exactly
the tools needed to work on those docs from inside a reproducible environment - no
build toolchain.

It builds from [`Dockerfile`](./Dockerfile), which provides:

- a recent **git** (built from source, `>= 2.48`) for relative-path worktrees;
- **reuse** (via `pipx`) - the tool behind the *REUSE Compliance* CI.

On top of that it adds, as devcontainer features and a post-create step:

- **Node.js**, the **GitHub CLI** (`gh`) and the [Claude Code](https://claude.com/claude-code)
  CLI (features);
- **markdownlint-cli2** (`npm -g`, in `postCreateCommand`) - the tool behind the
  *Markdown Lint* CI, so you can lint locally before pushing.

It is **Docker Compose based** ([`docker-compose.yml`](./docker-compose.yml),
service `dev`). The clone is always mounted at `/workspaces/act-contributing`, so
the static `workspaceFolder` works whichever mount mode (below) you pick. Unlike a
code dev container there is no privileged mode, host networking or X11 forwarding -
none of it is needed here.

## Local checks

Both CIs can be reproduced inside the container:

```bash
# Markdown lint (same tool as the CI)
markdownlint-cli2 "**/*.md"

# REUSE / SPDX compliance (same tool as the CI)
reuse lint
```

## Git worktrees for parallel agents (optional)

By default the container runs in **simple mode**: just this clone is mounted at
`/workspaces/act-contributing` and nothing else from the host is visible. You
don't have to do anything.

To run several agents on different branches in parallel, switch to **worktree
mode**, which mounts the *project root* one level above the clone so the clone and
its sibling worktrees are all visible inside the container.

1. **Lay out the project as `root/act-contributing` on the host.** Keep an outer
   folder and move the clone into an `act-contributing/` subfolder beside which
   worktrees will live:

   ```text
   ~/git/ACT-Contributing/            <- project root, mounted at /workspaces
     act-contributing/                <- this clone; OPEN THIS in VS Code -> /workspaces/act-contributing
     <some-branch>/                   <- worktrees added later -> /workspaces/<some-branch>
   ```

   The in-container path `/workspaces/act-contributing` is owned by
   `workspaceFolder` in devcontainer.json; the simple-mode default mount target in
   docker-compose.yml must match it (they cross-reference each other in comments).

   For an existing clone (close VS Code first, then from the parent dir):

   ```bash
   cd ~/git
   mv ACT-Contributing ACT-Contributing.tmp
   mkdir ACT-Contributing
   mv ACT-Contributing.tmp ACT-Contributing/act-contributing
   ```

   Re-open the **`act-contributing/`** folder in VS Code (not the root).

2. **Enable the mount.** Copy the template and uncomment both lines:

   ```bash
   cp .devcontainer/.env.example .devcontainer/.env
   # WORKSPACE_MOUNT_SOURCE=../..
   # WORKSPACE_MOUNT_TARGET=/workspaces
   ```

   `.devcontainer/.env` is gitignored (per-user). Rebuild the container.

3. **Create worktrees from inside the container**, as siblings of
   `act-contributing`:

   ```bash
   git worktree add ../my-feature -b my-feature
   ```

   `worktree.useRelativePaths` is set globally in the container (postCreate), so
   worktree links use relative paths and resolve no matter where `/workspaces` is
   mounted. This needs git `>= 2.48`, which the container's git (built from source)
   provides - **do not** run `git worktree add` from the host if its git is older
   (it writes absolute paths that won't resolve in the container).

## Claude Code install & auth

Claude Code is installed via the official devcontainer feature
(`ghcr.io/anthropics/devcontainer-features/claude-code`); the `node` feature is
added alongside it because the base image has no Node.js. The GitHub CLI (`gh`) is
installed the same way (`ghcr.io/devcontainers/features/github-cli`) for PR/issue
workflows.

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

`DISABLE_AUTOUPDATER=1` is set so the feature-managed binary doesn't try to
self-update.

### Other tool tokens (gh, ...)

There's no Claude token to manage. For tools that authenticate from an env var -
currently just `gh` (`GH_TOKEN`) - put the value in the gitignored
`.devcontainer/.env` (see [`.env.example`](./.env.example)). Docker Compose
auto-loads that file and `docker-compose.yml` forwards `GH_TOKEN` into the
container via its `environment:` block. The file is optional: without it, `gh`
just falls back to `gh auth login`.

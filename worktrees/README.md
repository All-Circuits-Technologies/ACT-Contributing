<!--
SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Git worktrees

This directory is where `git worktree` checkouts of this repository live, so that
several branches can be worked on in parallel without juggling a single checkout
(useful, for instance, to run several agents at once). Everything here except this
README is gitignored, so the checkouts never show up as untracked content.

This is a documentation-only repository: a fresh worktree has no submodules and no
generated files, so unlike a code project it is usable straight after
`git worktree add` - there is no bring-up step. Only the two constraints below
matter.

## Create

Always from **inside the dev container**, from the repository root:

```bash
git worktree add worktrees/my-feature -b my-feature
```

Two constraints, both of which silently produce a broken worktree if ignored:

- **Inside the container, not on the host.** The container's system git config
  sets `worktree.useRelativePaths` (see `postCreateCommand` in
  [`../.devcontainer/devcontainer.json`](../.devcontainer/devcontainer.json)), so
  the links between the worktree and the main clone are relative and resolve both
  at the container path (`/workspaces/act-contributing`) and at the
  host's own clone path, which is different. This needs git >= 2.48; a host
  shipping an older git writes absolute paths that do not resolve in the
  container.
- **Under `worktrees/`, nowhere else.** Only the clone is bind-mounted into the
  container. A worktree created outside it (`../my-feature`, `/tmp/...`) exists
  solely in the container's writable layer and is lost when the container is
  rebuilt.

## Remove

```bash
git worktree remove worktrees/my-feature
```

If the worktree still holds changes or untracked files git refuses to remove it;
add `--force` once you are sure nothing there is worth keeping.

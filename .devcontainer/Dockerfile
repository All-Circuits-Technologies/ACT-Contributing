# SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>
#
# SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1

# =============================================================================
# ACT-Contributing documentation dev container
# =============================================================================
# This repository holds only Markdown documentation (contributing guides, coding
# standards) checked by two CIs: markdownlint and REUSE (SPDX) compliance. The
# image therefore ships NO build toolchain - just what is needed to lint the
# docs, keep them REUSE-compliant, and drive git/gh/Claude Code from inside.
#
# Tooling installed here:
#   * a recent git (built from source, >= 2.48) for relative-path worktrees
#   * reuse (pipx) - matches the REUSE Compliance CI
# Node.js, the GitHub CLI and Claude Code are added as devcontainer features
# (see devcontainer.json); markdownlint-cli2 is installed via npm in
# postCreateCommand once the node feature is present.
# =============================================================================

FROM debian:trixie AS base

# Avoid interactive prompts
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Base utilities plus the toolchain strictly required to build git from source
# below (compiler + the libraries git links against). Nothing here is used to
# build the repository itself - it has no code.
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    xz-utils \
    less \
    sudo \
    locales \
    pipx \
    # openssh-client provides `ssh` (git's SSH transport for git@github.com
    # remotes) plus `ssh-add`/`ssh-agent`. The base image ships none of them, so
    # without this both SSH pushes and VS Code's forwarded ssh-agent are unusable.
    openssh-client \
    # apt `git` is installed only so dpkg-based packages that declare a `git`
    # dependency (notably the `gh` .deb pulled by the github-cli feature) can be
    # configured. The newer git built from source below lands in /usr/local/bin,
    # which precedes /usr/bin on PATH, so it is the one actually used at runtime.
    git \
    # `file` is required by reuse to detect file encodings (without it every
    # `reuse` invocation aborts with NoEncodingModuleError)
    file \
    # git build dependencies (libcurl provides curl/curl.h + curl-config, needed
    # for git's HTTP(S) transport - without it the build fails compiling http.o)
    gcc \
    make \
    zlib1g-dev \
    libssl-dev \
    libcurl4-openssl-dev \
    libexpat1-dev \
    libpcre2-dev \
    gettext \
    && rm -rf /var/lib/apt/lists/*

# =============================================================================
# Recent git from source (>= 2.48 for relative worktree support)
# =============================================================================
# The apt git on Debian trixie is 2.47.x, which predates the `relativeWorktrees`
# repository extension (git 2.48) that the worktree workflow relies on for
# `git worktree` with relative paths (worktree.useRelativePaths, see
# devcontainer.json). Build a newer git from the official release tarball into
# /usr/local, which precedes /usr/bin on PATH.
ARG GIT_VERSION=2.51.0
RUN curl -fsSL "https://mirrors.edge.kernel.org/pub/software/scm/git/git-${GIT_VERSION}.tar.xz" \
       | tar -xJ -C /tmp \
    && make -C "/tmp/git-${GIT_VERSION}" prefix=/usr/local NO_TCLTK=1 USE_LIBPCRE=YesPlease -j"$(nproc)" all \
    && make -C "/tmp/git-${GIT_VERSION}" prefix=/usr/local NO_TCLTK=1 USE_LIBPCRE=YesPlease install \
    && rm -rf "/tmp/git-${GIT_VERSION}" \
    && git --version

# Set locale
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen \
    && locale-gen

ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# =============================================================================
# User setup
# =============================================================================
ARG USERNAME=debian

# Use bash as the login shell: the Node.js devcontainer feature runs its nvm
# setup as this user via `su`, which uses the login shell. The default
# (/bin/sh -> dash) lacks `source` and fails the build.
RUN useradd -m -s /bin/bash $USERNAME \
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# =============================================================================
# REUSE (SPDX compliance tool)
# =============================================================================
# Installed as the user via pipx so its shim lands on the user PATH. Matches the
# REUSE Compliance CI so `reuse lint` can be run locally before pushing.
USER ${USERNAME}
RUN pipx install reuse \
    && pipx ensurepath
USER root

# Default to the user's home; the real workspace is provided by the
# docker-compose bind mount at runtime (VS Code cd's into workspaceFolder).
WORKDIR /home/${USERNAME}

# Pre-create the workspace parent so it is owned by the user, not root. In
# simple mode docker-compose bind-mounts only /workspaces/act-contributing;
# Docker would otherwise auto-create the /workspaces parent as root, leaving the
# user unable to `git worktree add` sibling worktrees. In worktree mode
# /workspaces is itself the bind mount, which harmlessly shadows this dir. Keep
# this path in sync with the mount target in docker-compose.yml / devcontainer.json.
RUN mkdir -p /workspaces && chown ${USERNAME}:${USERNAME} /workspaces

USER ${USERNAME}

# Pre-create ~/.claude (owned by the user) so the isolated named volume mounted
# here (act-contributing-claude-config, see docker-compose.yml) inherits user
# ownership when Docker first initialises it. Without this the fresh volume is
# root-owned and Claude can't write to it. CLAUDE_CONFIG_DIR (devcontainer.json)
# points Claude's whole config in here, so a single login persists across rebuilds.
RUN mkdir -p /home/${USERNAME}/.claude

SHELL ["/bin/bash", "-c"]

CMD ["/bin/bash"]

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
# Everything is installed here in the Dockerfile rather than through devcontainer
# features: features are deliberately avoided (they have caused more trouble than
# they saved), so the toolchain is explicit and reproducible in one place.
#
# Tooling installed here:
#   * a recent git (built from source, >= 2.48) for relative-path worktrees
#   * reuse (pipx) - matches the REUSE Compliance CI
#   * gh (official GitHub CLI .deb) - for PR/issue workflows
#   * Node.js and markdownlint-cli2 (npm) - the tool behind the Markdown Lint CI
# Claude Code is the ONE exception: it is installed by postCreateCommand (see
# devcontainer.json), which keeps its ~250MB of self-updating binaries out of the
# image layers and lets it self-update in the container's writable layer.
# =============================================================================

# Pinned by digest (RG30): bump the tag and the digest together.
FROM debian:trixie@sha256:9cc080028c43b27d2074d63a5f9caf7166d731494965616c1a6d2827a004585c AS base

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
    # dependency (notably the `gh` .deb installed below) can be configured. The
    # newer git built from source below lands in /usr/local/bin, which precedes
    # /usr/bin on PATH, so it is the one actually used at runtime.
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
# /usr/local, which precedes /usr/bin on PATH. Not strictly a MUST - once the
# Debian git catches up to >= 2.48 this whole stage can be dropped and the apt
# git used instead.
ARG GIT_VERSION=2.51.0
RUN curl -fsSL "https://mirrors.edge.kernel.org/pub/software/scm/git/git-${GIT_VERSION}.tar.xz" \
       | tar -xJ -C /tmp \
    && make -C "/tmp/git-${GIT_VERSION}" prefix=/usr/local NO_TCLTK=1 USE_LIBPCRE=YesPlease -j"$(nproc)" all \
    && make -C "/tmp/git-${GIT_VERSION}" prefix=/usr/local NO_TCLTK=1 USE_LIBPCRE=YesPlease install \
    && rm -rf "/tmp/git-${GIT_VERSION}" \
    && git --version

# =============================================================================
# GitHub CLI (gh)
# =============================================================================
# For pull-request / issue workflows from inside the container. Installed from
# the official GitHub CLI release .deb (first-party, pinned) rather than a
# devcontainer feature. Authentication is interactive: run `gh auth login` once
# (see README.md); the login lands in the gh config volume (see
# docker-compose.yml) and survives rebuilds.
ARG GH_VERSION=2.101.0
RUN curl -fsSL -o /tmp/gh.deb \
       "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_amd64.deb" \
    && apt-get update \
    && apt-get install -y --no-install-recommends /tmp/gh.deb \
    && rm -f /tmp/gh.deb \
    && rm -rf /var/lib/apt/lists/* \
    && gh --version

# =============================================================================
# Node.js and markdownlint-cli2
# =============================================================================
# markdownlint-cli2 is the CLI behind the Markdown Lint CI, so docs can be linted
# locally with the same tool before pushing. Node.js (with npm) is its runtime.
# Both come from Debian rather than a devcontainer feature; the pinned CLI version
# keeps local and CI behaviour aligned.
ARG MARKDOWNLINT_CLI2_VERSION=0.23.2
RUN apt-get update && apt-get install -y --no-install-recommends \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g markdownlint-cli2@${MARKDOWNLINT_CLI2_VERSION} \
    && markdownlint-cli2 --version

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

# bash (not the default /bin/sh -> dash) as the login shell: pipx ensurepath and
# the Claude Code installer write shell snippets that expect bash, and it is what
# an interactive terminal expects anyway.
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

# Pre-create the workspace parent so it is owned by the user, not root:
# docker-compose bind-mounts the clone at /workspaces/act-contributing, and
# Docker would otherwise auto-create the /workspaces parent as root. Baking the
# dir in with user ownership avoids a runtime chown. Keep this path in sync with
# the mount target in docker-compose.yml / devcontainer.json.
RUN mkdir -p /workspaces && chown ${USERNAME}:${USERNAME} /workspaces

USER ${USERNAME}

# Pre-create the two config directories that docker-compose.yml mounts named
# volumes on, so they are owned by the user: Docker initialises a fresh named
# volume from the image content at its mount point, and a mount point that does
# not exist in the image yields a root-owned volume the tool cannot write to.
# Baking the dirs in avoids any runtime chown. Keep both paths in sync with the
# volume targets in docker-compose.yml.
# - ~/.claude: CLAUDE_CONFIG_DIR (devcontainer.json) points Claude's whole config
#   - including ~/.claude.json - in here, so a single login persists across rebuilds.
# - ~/.config/gh: gh writes its hosts/token here, so a single `gh auth login`
#   persists across rebuilds.
RUN mkdir -p /home/${USERNAME}/.claude /home/${USERNAME}/.config/gh

SHELL ["/bin/bash", "-c"]

CMD ["/bin/bash"]

<!--
SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# ACT-Contributing - Agent Guidelines

> This is the unified entry point for AI coding agents working in this repository, following the
> [agents.md](https://agents.md/) convention. It is read directly by tools such as GitHub Copilot
> and by Claude Code (via the `CLAUDE.md` symlink). It links out to the human-facing documentation
> rather than duplicating it.

This repository holds **only Markdown documentation**: it is the canonical source of the ACT
contributing rules and coding standards that every other ACT software repository is expected to
follow. There is no application code here, and no build step - the "product" is the prose.

## Repository layout

| Path                                                           | Content                                      |
| -------------------------------------------------------------- | -------------------------------------------- |
| [`README.md`](README.md)                                       | Entry point and index                        |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                           | REUSE duties, PR/MR rules, Code of Conduct   |
| [`README-reuse.md`](README-reuse.md)                           | How licensing/REUSE is organised here        |
| [`software/CODING-STANDARDS.md`](software/CODING-STANDARDS.md) | Coding standards index and glossary          |
| `software/CODING-STANDARDS_global.md`                          | Language-agnostic rules (the `RGxx` rules)   |
| `software/CODING-STANDARDS_c.md`, `_cpp.md`, `_qt.md`          | Per-language rules                           |
| `LICENSES/`                                                    | License texts referenced by SPDX headers     |
| `.markdownlint.yaml`                                           | Markdown lint configuration (enforced by CI) |
| `.github/workflows/`                                           | CI: `markdown_lint` and `reuse_compliance`   |

## Golden rule

This repository *defines* the rules. Any change here must itself be an exemplary application of
those rules: it has to pass the same lint and licensing checks, and its wording must follow the
authoring rules below. Do not document a rule you are breaking on the same line.

## Authoring rules (always apply, override any local style)

These are the durable writing rules for every change in this repo. They are inlined here because
they apply to all edits:

- ASCII-only typography in prose: no em/en dashes, typographic arrows, curly quotes, ellipsis, or
  checkmark characters in the Markdown. Write `-`, `->`, `<->`, `'`, `...` instead. Unicode stays
  only where it carries meaning: physical units (`microL`, `degC`), non-English proper names, and
  box-drawing characters inside diagrams and directory trees.
- Issue numbers (GitHub, GitLab, Redmine, Jira) go in the commit message and the PR/MR title, but
  never in the documentation body. Issues become invalid over time; the docs stay. Explain *what*
  and *why* in plain words; `git blame` recovers the ticket. (The stable `RGxx` standard
  identifiers are not tickets - referencing those is expected.)
- A general document must not name a specific consumer of it. Describe what a rule guarantees, not
  who currently relies on it.

## Checks before committing

Both CIs can - and should - be reproduced locally before pushing. The dev container
([`.devcontainer/`](.devcontainer/README.md)) ships both tools:

```bash
# Markdown lint - same tool and config (.markdownlint.yaml) as the Markdown Lint CI
markdownlint-cli2 "**/*.md"

# REUSE / SPDX compliance - same tool as the REUSE Compliance CI
reuse lint
```

Key points enforced by `.markdownlint.yaml`: ATX headings (`#`), dash bullets (`-`), lines up to
100 characters (tables and long single-word links may exceed it), and only the three-dash form for
horizontal rules.

## Licensing (REUSE)

Every file must carry SPDX information, or the REUSE CI fails. See [`CONTRIBUTING.md`](CONTRIBUTING.md)
and [`README-reuse.md`](README-reuse.md) for the full duties. In short:

- Add a header to each new file, using a comment style the file's format supports:

  ```text
  SPDX-FileCopyrightText: <year> <Full Name> <email>

  SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
  ```

- For files that cannot hold a comment (pure JSON, binaries), add a sibling `<file>.license` text
  file carrying the same two tags.
- Use your full name and a valid email address. Add yourself to the header of files you update.
- Only reference a license that has a text file in [`LICENSES/`](LICENSES/). The licenses in use
  here are `LicenseRef-ALLCircuits-ACT-1.1` (default for this project's content), `MIT`, and
  `CC0-1.0`.

## Contribution workflow

- **Language:** everything is written in **English** - documentation, commits, branch names, and
  everything on GitHub/GitLab (PRs, MRs, issues, comments).
- **Work in a development branch**, never directly on `master` / the stable branch
  ([RG8](software/CODING-STANDARDS_global.md#rg8---work-in-development-branches)).
- **Every pull/merge request is linked to an issue**
  ([RG10](software/CODING-STANDARDS_global.md#rg10---feature-related-code)) and merged to the stable
  branch only after review by an authorized reviewer
  ([RG26](software/CODING-STANDARDS_global.md#rg26---the-code-in-stable-branch-has-to-be-peer-reviewed)).
- Repository members branch inside the repo; external contributors work from a fork. See
  [`CONTRIBUTING.md`](CONTRIBUTING.md#pull-and-merge-request).

## The standards themselves

When a task touches the coding standards, read the relevant file in full before editing - the rules
cross-reference each other by `RGxx` identifier, and those identifiers are a stable public API that
other repositories link to. Do not renumber or remove an `RGxx` rule without understanding who links
to it.

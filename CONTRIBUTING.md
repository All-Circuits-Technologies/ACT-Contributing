<!--
SPDX-FileCopyrightText: 2024 - 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Contributing

How to contribute to an ACT repository. Read this before opening a pull or merge request.

Everything is written in English: documentation, commits, branch names, and everything on
GitHub/GitLab (pull/merge requests, issues, comments).

By contributing, you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Table of contents

- [Coding standards](#coding-standards)
- [Licensing (REUSE)](#licensing-reuse)
- [Pull and merge requests](#pull-and-merge-requests)

## Coding standards

Follow the [coding standards](software/coding-standards.md). They apply to every ACT software
repository and are enforced in review.

## Licensing (REUSE)

We use [REUSE](https://reuse.software) to manage licensing: every file must carry SPDX information,
or the REUSE check fails. Follow
[RG22](software/coding-standards-global.md#rg22---carry-spdx-licensing-on-every-file) for SPDX headers
(and `.license` siblings) and
[RG23](software/coding-standards-global.md#rg23---keep-a-licenses-directory) for the `LICENSES/`
folder. Use your full name and a valid email, and add yourself to the header of any file you create
or update.

Before pushing, you can check compliance locally, so you avoid extra commits until CI is happy:

```shell
reuse lint
```

To install `reuse` locally: `python3 -m pipx install reuse` (see the
[pipx](https://pipx.pypa.io/) and [reuse](https://reuse.software) docs).

## Pull and merge requests

Every pull or merge request is linked to a task or issue
([RG10](software/coding-standards-global.md#rg10---tie-code-to-a-requested-feature)), and is merged
into the stable branch only after review by an authorized reviewer
([RG26](software/coding-standards-global.md#rg26---peer-review-code-before-it-enters-a-stable-branch)).

How you create the branch depends on your access:

- **Repository members** (GitHub or GitLab): create a development branch inside the repository,
  following [RG8](software/coding-standards-global.md#rg8---work-in-development-branches).
- **External contributors** (GitHub): fork the repository, then open a pull request against its
  `master` branch.

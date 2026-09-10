<!--
SPDX-FileCopyrightText: 2024 - 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Coding standards

The coding standards to follow in every ACT software repository. Start here: this page explains how
the standards are organised, how they combine, and where to find each one.

Every rule is blocking: a merge request that violates one is not merged until it is fixed.

## Table of contents

- [What belongs here](#what-belongs-here)
- [How the standards combine](#how-the-standards-combine)
- [Rule ids](#rule-ids)
- [The standards](#the-standards)

## What belongs here

These standards cover only what a linter cannot enforce: architectural decisions, conventions with
more than one reasonable option, and process rules. Anything that can be checked automatically and
expressed in a linter configuration file - formatting, indentation, import order, naming casing, and
the like - does not belong here: configure the linter and let CI enforce it
([RG27](coding-standards-global.md#rg27---build-and-lint-the-project-with-ci)). A rule that merely
restates a linter setting adds maintenance cost and drifts out of sync with the tool that actually
enforces it.

A rule may still record the *value* a linter enforces - the line-length limit, the indentation
width - when that value is a shared convention worth stating once and referencing across projects.

## How the standards combine

The standards come in three layers. A more specific layer may tighten or override a value set by a
broader one:

```text
project > language > global
```

For example, if the global standard sets "lines cannot exceed 100 characters" and the C standard
sets 80, then C projects use 80. If a specific project then sets 120, that project uses 120 - the
project layer wins over both.

## Rule ids

Each rule has a unique id, unique across every standard (global, per-language, per-project). The id
prefix tells you which standard a rule belongs to:

| Prefix                | Standard         |
| --------------------- | ---------------- |
| `RG`                  | Global           |
| `RC`                  | C                |
| `RCPP`                | C++              |
| `RQT`, `RQTCPP11`     | Qt               |

An id is a stable public API: other repositories cite it in code review and link to it. A rule's
wording may change, but its id never moves to a different rule and is never reused.

## The standards

- [Global coding standards](coding-standards-global.md)
- [C coding standards](coding-standards-c.md)
- [C++ coding standards](coding-standards-cpp.md)
- [Qt coding standards](coding-standards-qt.md)

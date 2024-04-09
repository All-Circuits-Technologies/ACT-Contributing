<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Global coding standards

## Table of contents

- [Global coding standards](#global-coding-standards)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Standards list](#standards-list)
    - [RG1 - No more than 1000 lines](#rg1---no-more-than-1000-lines)
    - [RG2 - No more than 100 characters per line](#rg2---no-more-than-100-characters-per-line)
    - [RG3 - Indent](#rg3---indent)
    - [RG4 - Code written in English](#rg4---code-written-in-english)
    - [RG5 - Documentation written in English](#rg5---documentation-written-in-english)
    - [RG6 - Code is stored in a version control system (Git, SVM, etc.)](#rg6---code-is-stored-in-a-version-control-system-git-svm-etc)
    - [RG7 - At least one "stable" branch](#rg7---at-least-one-stable-branch)
    - [RG8 - Work in development branches](#rg8---work-in-development-branches)
    - [RG9 - The code builds](#rg9---the-code-builds)
    - [RG10 - Feature-related code](#rg10---feature-related-code)
    - [RG11 - Update task/issue in your tasks/issues tracker](#rg11---update-taskissue-in-your-tasksissues-tracker)
    - [RG12 - Working code](#rg12---working-code)
    - [RG13 - Unit testing and features](#rg13---unit-testing-and-features)
    - [RG14 - Unit tests and dependencies](#rg14---unit-tests-and-dependencies)
    - [RG15 - Unit testing and automation](#rg15---unit-testing-and-automation)
    - [RG16 - Unit testing and success](#rg16---unit-testing-and-success)
    - [RG17 - Third-party software licenses](#rg17---third-party-software-licenses)
    - [RG18 - Software license](#rg18---software-license)
    - [RG19 - Design and maintainability](#rg19---design-and-maintainability)
    - [RG20 - Design and evolution](#rg20---design-and-evolution)
    - [RG21 - Use UTF-8 to encode files](#rg21---use-utf-8-to-encode-files)
    - [RG22 - SPDX license in headers](#rg22---spdx-license-in-headers)
    - [RG23 - LICENSES directory](#rg23---licenses-directory)
    - [RG24 - No dead code, without a right justification](#rg24---no-dead-code-without-a-right-justification)
    - [RG25 - TODO and FIXME comment style](#rg25---todo-and-fixme-comment-style)

## Introduction

This file contains all the global standards to apply when working in our repositories.

First read: [coding standards](CODING_STANDARDS.md) to understand how the standards apply on
projects and the overloading process.

## Standards list

### RG1 - No more than 1000 lines

- Severity: Non-blocking

Files shouldn't exceed **1000 lines**.

### RG2 - No more than 100 characters per line

- Severity: Non-blocking

Lines shouldn't exceed **100 characters**.

### RG3 - Indent

- Severity: **Blocking**

The code must be **indented with spaces** and each level of indentation corresponds to **4 spaces**.

### RG4 - Code written in English

- Severity: **Blocking**

The code must be written in **English**.

### RG5 - Documentation written in English

- Severity: **Blocking**

The documentation in code must be written in **English**.

### RG6 - Code is stored in a version control system (Git, SVM, etc.)

- Severity: **Blocking**

**Any code** developed must be stored in a version control system (Git, SVN, etc.).

Either by using your version control system (GitHub, GitLab, Bitbucket, etc.) or the one of your
client.

### RG7 - At least one "stable" branch

- Severity: **Blocking**

**All code repositories** must have a so-called **"stable" branch**.

**A branch is considered as "stable", if at any time it is possible to recover a fully functional
application, library, firmware, etc. from this branch.** All bugs found on this branch must be
logged in an tasks/issues tracker (Redmine, Jira, Issues in GitHub or GitLab, etc.) and analyzed for
categorization: to be corrected urgently or to be planned for in a future development cycle.

The "stable" branch may be named "master", "main", "stable"... as you wish.

**Adding code to this branch can only be done through peer code review.**

### RG8 - Work in development branches

- Severity: **Blocking**

**The code must be developed in working branches**, the name of the branches follows the following
pattern:

```
tmp/xxxx-branch-name
```

With:

- `xxxx`: the id of the associated (main) task in your tasks/issues tracker (Redmine, Jira, GitHub,
  GitLab, etc.)
- `branch-name`: a short abstract of the task purpose

For example:

```
tmp/845-add-main-app-logo
```

### RG9 - The code builds

- Severity: **Blocking**
- Applicability: if the source code goes through a compiler

The building of the code must be successful.

> [!NOTE]
> Here, **it is necessary to clearly differentiate the code under development**, in a development
> branch, **and the code under review**, to be merged into the "stable" branche.
> _On your development branch, you may have code that does not work or does not build_ — in this
> case, it is always good to specify this in the commit you make — precisely because you are
> developing.
> On the other hand, **the code in review must build and work as expected**, that's the basis.

### RG10 - Feature-related code

- Severity: **Blocking**

The code must be related to feature requested by customer, ordering party or discussed together.

_It is as important to not forget features as to not add extra features which are not needed._

> [!NOTE]
> When you add not needed features, you might spare your time — when you are working for you, it's
> your own choice — but you also add features as you conceive them FOR NOW: you think, that in
> future, you may need this feature as you think it will be needed.
> However, when this feature is really asked and needed, it won't be the exact thing you have
> developed and so you will need to do some refactoring.
> If the code to refactor is a leaf of the "big project tree", it's ok, it won't take too much time;
> but if you have created extra features based on extra features, etc. and the code to refactor is
> the basis of a lot of others classes (a branch or a trunk of the "big project tree"), it will
> take a lot of time (and potential bugs added) to refactor it.

> [!NOTE]
> In a context of a shared library, open source project, etc. it's important to discuss the features
> to add (their relevance, priority) before developping them.

### RG11 - Update task/issue in your tasks/issues tracker

- Severity: **Blocking**

Since code is developed against features and these features are represented by task or issue in
your tasks/issues tracker.
**You must update the task/issue with the progress of your development** (you begin to work on it,
it's in review by a peer, it's merged in stable branch, etc.).

### RG12 - Working code

- Severity: **Blocking**

The code must be working as we expect it should be _(subjective and subject to discussion with the
reviewers)_.

During reviews, it is important for reviewers to verify that the code under review is working. But
also if what it has been developed matched what has been requested.

> [!NOTE]
> Here, **it is necessary to clearly differentiate the code under development**, in a development
> branch, **and the code under review**, to be merged into the "stable" branche.
> _On your development branch, you may have code that does not work or does not build_ — in this
> case, it is always good to specify this in the commit you make — precisely because you are
> developing.
> On the other hand, **the code in review must build and work as expected**, that's the basis.

> [!NOTE]
> Code review is, normally, just a review (reading) of the code, so it's complicated to know if
> the code is really working, or not.
> Therefore, _if no unit tests have been developed_, and if the feature is a tricky one (or the code
> is complex), it's better to test _yourself_ (as a reviewer) if the code works, or not.

### RG13 - Unit testing and features

- Severity: **Blocking**
- Applicability: if unit tests must be developped

The features requested by customer, ordering party or discussed together must be covered by unit
tests.

### RG14 - Unit tests and dependencies

- Severity: **Blocking**
- Applicability: if unit tests must be developped

If unit tests are **dependent on third-party elements** provided by customer or ordering party then
software stubs must be developed.

> [!NOTE]
> When working with third-parties and when a bug is detected, it's always complicated to know whose
> fault it is. Therefore, to work with software stubs, allow

### RG15 - Unit testing and automation

- Severity: **Blocking**
- Applicability: if unit tests must be developped

Unit tests must be automatable.

### RG16 - Unit testing and success

- Severity: **Blocking**
- Applicability: if unit tests must be developped

All **unit tests must be successful** before merging the code in the stable branch.

### RG17 - Third-party software licenses

- Severity: **Blocking**

The **licenses of third-party libraries** used in the software must be in accordance with:

- your own library/application license(s),
- the context in which you develop the code (in a big company, for open source project, etc.), and
- if you develop a software for a customer, the way it wants to distribute it.

### RG18 - Software license

- Severity: **Blocking**

You must not use a library, software, source codes, etc. for which you haven't the right to use it,
in the context you use it.

### RG19 - Design and maintainability

- Severity: **Blocking**

**The code must be** clear enough to be **maintainable by another developer** _(subjective and
subject to discussion)_.

This discussion can take place during design and/or during peer code reviews _(but doing it in code
reviews is almost too late)_.

### RG20 - Design and evolution

- Severity: **Blocking**

The **code must be designed** in such way it **can evolve according to the future and already stated
needs** of the customer or ordering party _(subjective and subject to discussion)_.

It is important to always find out about the purpose of the program to get a glimpse of what the
software could be in a few years. This is very dependent on the client/ordering party's vision and
their ability to project themselves. This is why it remains very subjective and subject to
discussion.

However, most of the time we have this information for medium-term visions, which is why it is
important to think about it when doing the software design _(while remaining as close as possible
to the client/ordering party's primary needs)_.

To illustrate this: if the client wants us to design shoes for him, we could either go for sandals,
sneakers, dress shoes, etc. But if in future, he wants that people run with it for jogging.
It is probably preferable to create a design that would be suitable for this, here, sneakers.

### RG21 - Use UTF-8 to encode files

- Severity: **Blocking**

The files containing codes must be encoded in UTF-8.

### RG22 - SPDX license in headers

- Severity: **Blocking**

The project's textual files must contain a header, consisting of at least:

```
SPDX-FileCopyrightText: YYYY First name Name <email>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-XZ
```

This header has to be commented according to the language, i.e. prefixed with `//` for C/C++
languages ​​and `#` for Python and shell languages ​​for example, and with the updated license version
if necessary.

Binary or non-commentable files must be accompanied by a file `xxxx.license` that contains this
header.
Example: `icon.png.license`.

In the case of a complex or third-party tree, it is possible to associate a license with a folder
by creating a .reuse/dep5 file (do not overuse it):

```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Upstream-Name: name of your repository without extension, for example: shoes-shop-sw
Upstream-Contact: Firstname Lastname <email>
Source: link to your version control system, for instance: http://hostname/shoes-shop-sw

Files: folder1/*
Copyright: YYYY Firstname Lastname <email>
License: LicenseRef-ALLCircuits-ACT-X.Z

Files: folder2/*
Copyright: YYYY-YYYY Firstname Lastname <email>
License: MIT
```

It is the responsibility of the reviewer to ensure that old license headers are updated when
necessary.

Optionally, developers can install `reuse` on their workstation if they wish to run locally
`reuse lint`.
Here is the installation procedure:

```shell
%PYTHON3% —m pip install pipx
%PYTHON3% —m pipx install reuse
%PYTHON3% —m pipx ensurepath
```

### RG23 - LICENSES directory

- Severity: **Blocking**

A folder `LICENSES/` must be at the root of your repository, with at least the file
`LicenseRef-ALLCircuits-ACT-X.Z.txt` inside.

There must be one file per license used.
For example there must be an MIT.txt file if the MIT license is used.

This is simplified by using the reuse tool (`reuse download MIT` for example).

### RG24 - No dead code, without a right justification

- Severity: **Blocking**

What we mean by dead code is:

- commented code, and
- code never called

**You can't have dead code in your projects**, it's the main rule. Because, it adds complexity to
the reading and may lead developpers to wrongly understand how the features work.

_However, sometimes, we want to keep dead code for right reasons_:

- we have developped classes and methods not used for now but will be used in future,
- we keep a commented code for examples and explain some things,
- etc.

In that case, you HAVE TO explain why you want to keep the dead code by adding comments (added
before the dead code).

The reviewer of your merge request will judge if your choice is relevant or not (or is it misses
information)

### RG25 - TODO and FIXME comment style

- Severity: Non-blocking

Sometimes, we want to add TODO and FIXME comments in our code; because we want to express the fact
that some elements have to be done in future or has to be fixed.

For adding the TODO and FIXME comments, you follow the Flutter linter rule:
[flutter_style_todos](https://dart.dev/tools/linter-rules/flutter_style_todos)

> Use Flutter TODO format: // TODO(username): message, https://URL-to-issue.
>
> [...]
>
> From the
> [Flutter docs](https://github.com/flutter/flutter/wiki/Style-guide-for-Flutter-repo#comments):
>
> > TODOs should include the string TODO in all caps, followed by the GitHub username of the person
> > with the best context about the problem referenced by the TODO in parenthesis. A TODO is not a
> > commitment that the person referenced will fix the problem, it is intended to be the person with
> > enough context to explain the problem. Thus, when you create a TODO, it is almost always your
> > username that is given.
>
> **GOOD:**
>
> ```
> // TODO(username): message.
> // TODO(username): message, https://URL-to-issue.
> ```

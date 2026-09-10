<!--
SPDX-FileCopyrightText: 2024 - 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Global coding standards

Language-agnostic rules that apply to every ACT software repository. Each rule has a stable id
(`RGxx`) so it can be cited in code review and linked from other repositories. Treat these ids as a
public API: their wording may change, but a code (`RG8`) never moves to a different rule and is
never reused.

These global rules are the base layer. A language standard (C, C++, Qt, Flutter) or a project
standard may tighten a value; the most specific one wins: `project > language > global`. See
[coding-standards.md](coding-standards.md) for the override mechanism.

**Every rule below is blocking: a merge request that violates one is not merged until it is fixed.**

## Table of contents

- [Language and formatting](#language-and-formatting)
- [Version control and workflow](#version-control-and-workflow)
- [Building and delivery](#building-and-delivery)
- [Testing](#testing)
- [Design and code quality](#design-and-code-quality)
- [Licensing](#licensing)

## Language and formatting

### RG1 - No more than 1000 lines

Keep a source file under 1000 lines. Past that, split it by responsibility - a file that long is
almost always doing several jobs.

### RG2 - No more than 100 characters per line

Keep lines under 100 characters. A language or project standard may set a different limit (80 in C,
for example); the most specific one wins.

### RG3 - Indent with 4 spaces

Indent with spaces, never tabs. One indentation level is 4 spaces.

### RG4 - Write code in English

Write code in English: identifiers, symbols, everything the compiler sees.

### RG5 - Write code documentation in English

Write in-code documentation (comments, doc blocks) in English.

### RG21 - Encode files in UTF-8

Encode every source file in UTF-8.

## Version control and workflow

### RG6 - Store code in a version control system

Store any code you develop in a version control system (Git, SVN, etc.), whether on your own
platform (GitHub, GitLab, Bitbucket) or the client's.

### RG7 - Keep at least one stable branch

Every repository has a stable branch. A branch is stable when, at any time, you can recover a fully
functional application, library, or firmware from it. Bugs found on it are logged in the tracker
and triaged: fix now, or plan for a later cycle. Name it `master`, `main`, or `stable` as you wish.

### RG8 - Work in development branches

Develop in a working branch, never directly on the stable branch. Branch names follow the pattern:

```text
tmp/xxxx-branch-name
```

- `xxxx`: the id of the associated task in your tracker (Redmine, Jira, GitHub, GitLab, etc.)
- `branch-name`: a short summary of the task's purpose

For example:

```text
tmp/845-add-main-app-logo
```

### RG10 - Tie code to a requested feature

Write only code that serves a feature requested by the customer, the ordering party, or agreed
together. Forgetting a feature and adding an unrequested one are equally wrong.

Speculative features cost more than they save: when the need becomes real it rarely matches what you
guessed, so you refactor anyway. If that speculative code sits at the base of the dependency tree,
the refactor is expensive and bug-prone. On a shared library or open-source project, discuss a
feature's relevance and priority before building it.

### RG11 - Keep the task tracker up to date

Because code is written against features tracked as tasks or issues, keep each task's status current
as you progress: started, in review, merged into the stable branch, and so on.

### RG26 - Peer review code before it enters a stable branch

Code reaches a stable branch (`master`, `main`, `stable`) only through peer code review.

## Building and delivery

### RG9 - Keep the code building

Code that compiles must build successfully before it is merged into the stable branch.

Distinguish work in progress from merge candidates. A development branch may hold code that does not
build or run - say so in the commit message. Code ready to merge into the stable branch must build.

### RG12 - Merge only working code

Code merged into the stable branch works as expected. This is partly subjective and open to
discussion with reviewers.

A review reads the code, so a reviewer cannot always tell whether it runs. When there are no unit
tests and the feature or code is tricky, the reviewer should run it to confirm it works, and check
that what was built matches what was requested.

### RG27 - Build and lint the project with CI

Build and lint the project with continuous integration tools (Jenkins, GitLab CI, GitHub Actions,
etc.). CI proves the code builds on a machine other than the author's - developers install many
tools that can make software work by accident - and enforces the rules that can be checked
automatically.

### RG28 - Produce deliverables with CI/CD

When you release the result of a build, produce the deliverable with CI/CD tools, not on your own
machine. A developer's machine has an uncontrolled toolchain, so its output is not reproducible; and
if you are away or your machine breaks, a colleague still needs to be able to deliver. Applies when
the source goes through a compiler.

### RG29 - Provide a README with a presentation and quick start

Every repository has a `README.md` at its root, with a short presentation of the project and a quick
start covering how to install the dev environment, build the library or application, and deploy it.

## Testing

Rules in this section apply when unit tests are to be developed for the project.

### RG13 - Cover requested features with unit tests

Cover the features requested by the customer, the ordering party, or agreed together with unit
tests.

### RG14 - Stub third-party dependencies in unit tests

When unit tests depend on third-party elements provided by the customer or ordering party, develop
software stubs for them. Stubbing isolates your code, so a detected bug points clearly at your side
rather than the third party's.

### RG15 - Make unit tests automatable

Write unit tests so they can be automated.

### RG16 - Pass all unit tests before merging

All unit tests pass before the code is merged into the stable branch.

## Design and code quality

### RG19 - Keep the code maintainable

Write code clear enough for another developer to maintain. This is subjective and best settled
during design, or during peer review - though by review it is almost too late.

### RG20 - Design for foreseeable evolution

Design code so it can evolve toward the customer's future and already-stated needs. This is
subjective and depends on how well the client can project themselves, but medium-term direction is
usually known, so account for it during design while staying close to the primary need.

For example, if the client asks for shoes and expects to run in them later, sneakers are a better
base design than sandals or dress shoes.

### RG24 - No dead code without a justification

No dead code: neither commented-out code nor code that is never called. Dead code raises the reading
cost and misleads readers about how the feature actually works.

Keep dead code only for a stated reason - a class not wired up yet, an example kept on purpose - and
write that reason in a comment right before it. The reviewer judges whether the justification holds.

### RG25 - Use the TODO and FIXME comment style

Write TODO and FIXME comments in the Flutter linter style,
[flutter_style_todos](https://dart.dev/tools/linter-rules/flutter_style_todos): `TODO` in all caps,
then the username of the person with the most context on the problem, then the message, optionally
followed by a link to the issue.

```c
// TODO(username): message.
// TODO(username): message, https://URL-to-issue.
```

## Licensing

### RG17 - Check third-party licenses for compatibility

The licenses of third-party libraries you use must be compatible with your own license(s), the
context you develop in (company, open source, etc.), and, for a customer project, how the customer
intends to distribute it.

### RG18 - Use only software you have the right to use

Do not use a library, program, or source you do not have the right to use in the context you use it
in.

### RG22 - Carry SPDX licensing on every file

Every file's copyright and license are declared in SPDX form:

```text
SPDX-FileCopyrightText: YYYY First name Name <email>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
```

Declare it in one of three ways, in order of preference:

- an inline header, commented in the file's own syntax (`//` for C/C++, `#` for Python and shell,
  etc.);
- a sibling `<file>.license` holding the same header, for a binary or non-commentable file (for
  example `icon.png.license`);
- a `REUSE.toml` entry covering a folder or a set of files, for a complex or third-party tree (use
  it sparingly).

Update the license version when needed. Reviewers make sure outdated headers are updated.

### RG23 - Keep a LICENSES directory

Keep a `LICENSES/` folder at the repository root with one text file per license in use, including at
least `LicenseRef-ALLCircuits-ACT-X.Z.txt`. For example, add `MIT.txt` when the MIT license is used.

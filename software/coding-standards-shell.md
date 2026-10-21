<!--
SPDX-FileCopyrightText: 2023 Anthony Loiseau <anthony.loiseau@allcircuits.com>
SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Shell coding standards

Unix shell script conventions, reusable across ACT projects. Each rule has a stable id (`RSHELLx`)
so it can be cited in code review.

This standard overloads the global standards:
[global standards](coding-standards-global.md). First read
[coding standards](coding-standards.md) to understand how the standards apply and the override
mechanism.

This guide does **not** restate what the linter enforces: every script passes
[ShellCheck](https://www.shellcheck.net)
([RSHELL0](#rshell0---pass-shellcheck)), configured once and enforced by CI. The rules below cover
only what ShellCheck cannot see.

## Linting

### RSHELL0 - Pass ShellCheck

Check every shell script with ShellCheck, in CI
([RG27](coding-standards-global.md#rg27---build-and-lint-the-project-with-ci)). Enable these
optional checks in a `.shellcheckrc` file at the repository root:

```text
enable=add-default-case
enable=avoid-nullary-conditions
enable=check-unassigned-uppercase
enable=quote-safe-variables
enable=require-variable-braces
```

Silence a justified finding on the line before it, with `# shellcheck disable=SCxxxx`.

## Syntax and error handling

### RSHELL1 - Match the shebang to the syntax

Start every script with a shebang matching the syntax it uses: `#!/bin/sh` for POSIX shell,
`#!/bin/bash` for Bash. Write POSIX shell when the script must be portable or run in a minimal
environment.

A `#!/bin/sh` script may use these widespread non-POSIX features, silencing ShellCheck:
`set -o pipefail` ([SC3040](https://www.shellcheck.net/wiki/SC3040)) and `local`
([SC3043](https://www.shellcheck.net/wiki/SC3043)).

### RSHELL2 - Stop on any error

Make the script exit on any failing command, failing pipe and unset variable:

```sh
set -e
set -u
# shellcheck disable=SC3040
set -o pipefail
```

The `shellcheck disable` line is only needed under `#!/bin/sh`.

### RSHELL3 - Use braces to read variables (removed)

Removed: enforced by ShellCheck (`require-variable-braces`), see RSHELL0.

### RSHELL4 - Quote variables (removed)

Removed: enforced by ShellCheck ([SC2086](https://www.shellcheck.net/wiki/SC2086) and
`quote-safe-variables`), see RSHELL0.

## Naming

### RSHELL5 - Name global constants in UPPER_SNAKE_CASE and make them read-only

```sh
THIS_DIR="$(cd "$(dirname "$0")" && pwd)"
readonly THIS_DIR
```

### RSHELL6 - Name functions in snake_case

```sh
print_help() {
    grep "^##" "$0" | sed 's/^## \?//'
}
```

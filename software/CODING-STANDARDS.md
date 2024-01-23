<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Coding standards

## Table of contents

- [Coding standards](#coding-standards)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Glossary](#glossary)
  - [Standards overload](#standards-overload)
  - [Standards ids](#standards-ids)
  - [List of coding standards](#list-of-coding-standards)

## Introduction

This readme contains the list of coding standards to use with our Git repositories.

## Glossary

Severity of facts when doing a code review

| ID           | Description                                                                                                      |
| ------------ | ---------------------------------------------------------------------------------------------------------------- |
| Blocking     | Rule which blocks the review validation                                                                          |
| Non-blocking | Better to have it, and may be corrected in the long term but not blocking the validation of the review by itself |

## Standards overload

**Global standards can be overloaded by language-specific standards, which themselves can be
overloaded by project standards:**

`Project standards > Language standards > Global standards`

First example:

> If a global standard specifies that "Lines cannot exceed 100 characters" but a C standard
> specifies that "Lines cannot exceed 80 characters" then the C standard wins.
>
> So in C projects, lines cannot exceed 80 characters.

Second example:

> If a global standard specifies that "Lines cannot exceed 100 characters" and a C standard
> specifies that "Lines cannot exceed 80 characters", but a project standard specifies that "Lines
> cannot exceed 120 characters" then it is the standard of the project which prevails.
>
> So in this specific project, the lines should not exceed 120 characters.

## Standards ids

Each standard rule has an unique id and it's unique for all the standards (global, by languages,
etc.)

## List of coding standards

- [Global coding standards](CODING-STANDARDS_global.md)
- [C++ coding standards](CODING-STANDARDS_cpp.md)
- [Qt coding standards](CODING-STANDARDS_qt.md)

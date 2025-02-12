<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Coding standards <!-- omit from toc -->

## Table of contents

- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
- [Glossary](#glossary)
- [Standards override](#standards-override)
- [Standards ids](#standards-ids)
- [List of coding standards](#list-of-coding-standards)

## Introduction

This readme contains the list of coding standards to use with our Git repositories.

## Glossary

We distinguish different types of project. Depending on the the project type, the severity on rules
may be different. The project types are:

| ID      | Description                                                                                             |
| ------- | ------------------------------------------------------------------------------------------------------- |
| Default | This is the default projects, which can be with client or internal to the company.                      |
| PoC     | This is Proof Of Concept projects, the goal of those projects are only to test or proove some concepts. |
| *All*   | All the projects.                                                                                       |

Severity when doing a code review:

| ID           | Description                                                                                                      |
| ------------ | ---------------------------------------------------------------------------------------------------------------- |
| Blocking     | Rule which blocks the review validation                                                                          |
| Non-blocking | Better to have it, and may be corrected in the long term but not blocking the validation of the review by itself |

## Standards override

**Global standards can be overriden by language-specific standards, which themselves can be
overriden by project standards:**

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

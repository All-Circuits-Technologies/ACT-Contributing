<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Coding standards <!-- omit from toc -->

## Table of contents

- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
- [C++ code standards](#c-code-standards)

## Introduction

This contains our coding standards for the C projects.

This overload the global standards: [global standards](CODING-STANDARDS_global.md)

First read: [coding standards](CODING_STANDARDS.md) to understand how the standards apply on
projects and the overloading process.

## C code standards

### RC1 - Documentation

- Severity: **Blocking**

Blocking Code documentation is written in English with doxygen.
See: [Génération de la documentation du code écrit](https://extranet.allcircuits-technologies.com/projects/engineering/wiki/G%C3%A9n%C3%A9ration_de_la_documentation_du_code_%C3%A9crit)

Example:

### RC2 - Function documentation

- Severity: **Blocking**

Functions **MUST** be documented and at the beginning of their implementation
except for API functions which are documented in the header file.

This documentation contains a brief description of what the function does,
what it returns, and its arguments.

The tags used are as follows (and in the following order):

Example:

```cpp
/**
 * @brief Brief description of what the function does
 * @note A note on the function which precises an important element
 * @note A second note about an other important element
 * @warning A warning about the limit of the function, this is useful when it can't 
 *          be tested in the function itself
 * @see struct_a_t // This is a direct reference to a method or struct
 * @param[in] pwd What is the meaning of the param, how it is used 
 * @param[in] sloop What is the meaning of the param, how it is used 
 * @return What the function returns and what is the meaning of the value returned 
 */
int prefix_my_function(struct_a_t pwd, int sloop);
```

### RC3 - Structures documentation

- Severity: **Blocking**

Structures **MUST** be documented and at the beginning of their declarations.
This documentation contains a brief description of what the structure represents
and what it is used for.

The tags used are as follows (and in the following order):

```c
/**
 * @brief Brief description of what the structure does
 * @note A note on the structure which precises an important element
 * @note A second note about an other important element
 * @warning A warning about the limit of the class
 */
typedef struct 
{
} prefix_my_struct_t;
```

The tags @note and @warning should only address one subject at a time.
If needed, multiply these tags.

All elements of the structure optionally documented if structure's documentation
is not enough to understand the meaning of the element. The documentation is
done on line above the element declaration.

```c
/** 
 * @brief Brief description of what the structure does
 * @note A note on the structure which precises an important element
 * @note A second note about an other important element
 * @warning A warning about the limit of the class
 */
typedef struct 
{
    /** @brief Id of the structure */
    int id;           
    /** @brief  Active or not */
    bool is_active;
} prefix_my_struct_t;
```

### RC4 - Documentation of enums

- Severity: **Blocking**

Enumerations **MUST** be documented and at the beginning of their declarations.
This documentation contains a brief description of what the enumeration represents
and what it is used for.

The tags used are as follows (and in the following order):

```c
/**
 * @brief Brief description of what the structure does
 * @note A note on the structure which precises an important element
 * @note A second note about an other important element
 * @warning A warning about the limit of the class
 */
typedef enum 
{
} prefix_my_enum_t;
```

The tags @note and @warning should only address one subject at a time.
If needed, multiply these tags.

All elements of the enumeration optionally documented if enumeration's documentation
is not enough to understand the meaning of the element. The documentation is
done on line above the element declaration.

```c
/**
 * @brief Brief description of what the structure does
 * @note A note on the structure which precises an important element
 * @note A second note about an other important element
 * @warning A warning about the limit of the class
 */
typedef enum 
{
    /** @brief Value zero */
    my_enum_eZERO = 0,
    /** @brief Value one */
    my_enum_eONE,
    my_enum_eNBR
} prefix_my_enum_t;
```

### RC5 - Organisation of header files

- Severity: Not blocking

For clarity and consistency, the header files must be organized in a specific
way. And respect skeleton file found at root of the project. For new projects,
see [actprojectskeletons](http://gitlab.act.lan:8011/internal-libraries/actprojectskeletons.git)

### RC6 - TODO

- Severity: **Blocking**

Format of the TODO comment:

```c
/* TODO (XXX): mon message */
```
or
```c
/* TODO XXX mon message */
```

Where XXX is the trigram of the person who will do the TODO.

### RC7 - One file per class or fonctionnality

- Severity: **Blocking**

One file must contain only one class or one functionality and name with the name
of the class or functionality. Name of file must be in lowercase.
Example: RGB led driver is in file `rgb_led.c`, `rgb_led.h` and rgb_led_types.h.

[!Note]
xxx_types.h files are used to define types that may used by c file that not
have directly access to rgb_led.h.

### RC8 - Naming conventions

- Severity: **Blocking**

All names of functions, variables, structures, enumerations, and macros **MUST**
be prefixed with a unique prefix that identifies the project or module.

To limit the risk of name collisions, the prefix should be unique to the project.
The prefix should be in lowercase and separated by two underscore `__`.
But macros prefix in lowercase joined by name of the macro in uppercase.

Prefixes should shorter as possible, 3 to 5 characters is a good length.
Like accronym of the module.

Example in `pwm.c`:

```c
/* Define enum */
typedef enum
{
    /** @brief Value for VAR0 */
    pwm__enum_eVAR0 = 0,
    /** @brief Value for VAR1 */
    pwm__enum_eVAR1,
    pwm__enum_eNBR
} pwm__enum_t;

/* Define constant */
#define pwmCONST (0)

/* Define variable */
PRIVATE uint8_t pwm__var;

/* Define functions */
PUBLIC void pwm__init(void)
{
    ...
}

PUBLIC void pwm__set_var(uint8_t test_var)
{
    ...
}
```

### RC9 - Braces

- Severity: Not blocking

Open braces `{` must be on their own line but closing braces `}` should be on
the same line as declaration of the structure, enumeration, or function.

```c
PUBLIC void prefix__function(void)
{
    ...
}

typedef enum
{
    ...
} prefix__my_enum_t;
```

### RC10 - Maximum number of characters per line

- Severity: **Blocking**

The maximum number of characters per line is **80**. there can be an exception
for comments.

### RC11 - Indentation

- Severity: **Blocking**

For C code indentation, tabs are not allowed. The indentation is done with
**4 spaces**.

### RC12 - Initialisation of structures

- Severity: **Blocking**

Structures **MUST** be initialized with fields in the same order as they are
declared. Using the field order is forbidden.

```c
prefix_my_struct_t my_struct = {
    .id = 0,
    .is_active = true
};
```

### RC13 - Table pointer usage

- Severity: **Blocking**

Using "table" or "&table[ 0 ]" then becomes a matter of style, generally:

- Using "table" is preferred when we talk about the whole table.
- Using "&table[ X ]" is preferred when we talk about the element X of the
table.
- Using "table + X" is preferred when we talk about sub-table from element X.

### RC14 - Enumeration values

- Severity: **Blocking**

All enumeration values **MUST** contain last value with `eNBR` suffix.
This value is used to know the number of elements in the enumeration.

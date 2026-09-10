<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>
SPDX-FileCopyrightText: 2025 Pierre-Noel Bouteville <pierre-noel.bouteville@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# C coding standards

Coding standards for C projects. This standard overloads the
[global standards](coding-standards-global.md); first read the
[coding standards index](coding-standards.md) for how the standards combine and the override
mechanism. Like every ACT standard, it covers only what a linter cannot enforce (see
[what belongs here](coding-standards.md#what-belongs-here)).

## Table of contents

- [C code standards](#c-code-standards)
  - [RC1 - Documentation](#rc1---documentation)
  - [RC2 - Function documentation](#rc2---function-documentation)
  - [RC3 - Structures documentation](#rc3---structures-documentation)
  - [RC4 - Documentation of enums](#rc4---documentation-of-enums)
  - [RC5 - Organisation of header files](#rc5---organisation-of-header-files)
  - [RC6 - TODO](#rc6---todo)
  - [RC7 - One file per class or fonctionnality](#rc7---one-file-per-class-or-fonctionnality)
  - [RC8 - Naming conventions](#rc8---naming-conventions)
  - [RC9 - Braces](#rc9---braces)
  - [RC10 - Maximum number of characters per line](#rc10---maximum-number-of-characters-per-line)
  - [RC11 - Indentation](#rc11---indentation)
  - [RC12 - Initialisation of structures](#rc12---initialisation-of-structures)
  - [RC13 - Table pointer usage](#rc13---table-pointer-usage)
  - [RC14 - Enumeration values](#rc14---enumeration-values)
  - [RC15 - Macro global parentheses](#rc15---macro-global-parentheses)
  - [RC16 - Macro arguments parentheses](#rc16---macro-arguments-parentheses)

## C code standards

### RC1 - Documentation

Blocking Code documentation is written in English with doxygen.
See: [Génération de la documentation du code écrit](https://extranet.allcircuits-technologies.com/projects/engineering/wiki/G%C3%A9n%C3%A9ration_de_la_documentation_du_code_%C3%A9crit)

Example:

### RC2 - Function documentation

Functions **MUST** be documented with doxygen above their definition in the
`.c` file. The header file only carries the bare prototype.

This documentation contains a brief description of what the function does,
what it returns, and its arguments.

The tags used are as follows (and in the following order):

Example in `pwm.c`:

```c
/**
 * @brief Brief description of what the function does
 * @note A note on the function which precises an important element
 * @note A second note about an other important element
 * @warning A warning about the limit of the function, this is useful when it
 *          can't be tested in the function itself
 * @see struct_a_t // This is a direct reference to a method or struct
 * @param[in] pwd What is the meaning of the param, how it is used
 * @param[in] sloop What is the meaning of the param, how it is used
 * @return What the function returns and what is the meaning of the value
 *         returned
 */
PUBLIC int pwm_my_function(struct_a_t pwd, int sloop)
{
    ...
}
```

The only exception is a header file implemented by **several** `.c` files (one
per port or backend, chosen at build time). The documentation is then the
contract shared by every implementation, so it **MUST** be written in the `.h`,
above the prototype. Each `.c` only carries, above the function definition, a
plain comment which points back to the header holding the doxygen.

Example in `include/sdcard.h`:

```c
/**
 * @brief Mount the SD card
 * @return 0 if the SD card has been mounted, a negative error code otherwise
 */
PUBLIC int sdcard_mount(void);
```

Example in `src/sdcard_spi.c`:

```c
/*
 * see ../include/sdcard.h
 */
PUBLIC int sdcard_mount(void)
{
    ...
}
```

The goal of this rule is to keep exactly **one** copy of the documentation:
with one implementation, the `.c` is that place; with several, duplicating the
same `@brief` in each backend guarantees they drift apart.

A comment which is specific to one implementation still belongs to its own
`.c`, as a plain comment: it is not part of the contract.

### RC3 - Structures documentation

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

For clarity and consistency, the header files must be organized in a specific
way. And respect skeleton file found at root of the project. For new projects,
see [actprojectskeletons](http://gitlab.act.lan:8011/internal-libraries/actprojectskeletons.git)

### RC6 - TODO

Format of the TODO comment:

```c
/* TODO (XXX): mon message */
```

or

```c
/* TODO XXX mon message */
```

Where XXX is the initial with last name of the person who will do the TODO.
Example: tmagne

### RC7 - One file per class or fonctionnality

One file must contain only one class or one functionality and name with the name
of the class or functionality. Name of file must be in lowercase.
Example: RGB led driver is in file `rgb_led.c`, `rgb_led.h` and rgb_led_types.h.

[!Note]
xxx_types.h files can be used to define types that may be used by c file which
do not have a direct access to rgb_led.h.

### RC8 - Naming conventions

All names of functions, variables, structures, enumerations, and macros **MUST**
be prefixed with a unique prefix of 3-6 characters **without** _ that
identifies the project or module.

To limit the risk of name collisions, the prefix should be unique to the project.
The prefix should be in lowercase and separated by one underscore `_`.
But macros prefix in lowercase joined by name of the macro in uppercase.

Example in `pwm.c`:

```c
/* Define enum */
typedef enum
{
    /** @brief Value for VAR0 */
    pwm_enum_eVAR0 = 0,
    /** @brief Value for VAR1 */
    pwm_enum_eVAR1,
    pwm_enum_eNBR
} pwm_enum_t;

/* Define constant */
#define pwmCONST (0)

/* Define variable */
PRIVATE uint8_t pwm_var;

/* Define functions */
PUBLIC void pwm_init(void)
{
    ...
}

PUBLIC void pwm_set_var(uint8_t test_var)
{
    ...
}
```

### RC9 - Braces

Open braces `{` must be on their own line but closing braces `}` should be on
the same line as declaration of the structure, enumeration, or function.

```c
PUBLIC void prefix_function(void)
{
    ...
}

typedef enum
{
    ...
} prefix_my_enum_t;
```

### RC10 - Maximum number of characters per line

The maximum number of characters per line is **80**. There can be an exception
for comments.

### RC11 - Indentation

For C code indentation, tabs are not allowed. The indentation is done with
**4 spaces**.

### RC12 - Initialisation of structures

Structures **MUST** be initialized with fields' name.
Using the fields order during initialisation is forbidden.

```c
/* GOOD */
prefix_my_struct_t my_struct = {
    .id = 0,
    .is_active = true
};

/* GOOD */
prefix_my_struct_t my_struct = {
    .is_active = true,
    .id = 0
};

/* BAD */
prefix_my_struct_t my_struct = {
    0,
    true
};
```

### RC13 - Table pointer usage

Using "table" or "&table[ 0 ]" then becomes a matter of style, generally:

- Using "table" is preferred when we talk about the whole table.
- Using "&table[ X ]" is preferred when we talk about the element X of the
table.
- Using "table + X" is preferred when we talk about sub-table from element X.

### RC14 - Enumeration values

All enumeration values **MUST** contain last value with `eNBR` suffix.
This value is used to know the number of elements in the enumeration.

### RC15 - Macro global parentheses

All macros **MUST** be defined with parentheses around the whole macro
definition.

```c
#define prefixMY_MACRO (8 + 3)
```

### RC16 - Macro arguments parentheses

All macros **MUST** be defined with parentheses around the arguments.

```c
#define prefixMY_MACRO(x) ((x) * (x))
```

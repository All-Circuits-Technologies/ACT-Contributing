<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Coding standards

## Table of contents

- [Coding standards](#coding-standards)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Standards overloading](#standards-overloading)
  - [C++ code standards](#c-code-standards)
    - [RCPP1 - Instance attributes of a private class](#rcpp1---instance-attributes-of-a-private-class)
    - [RCPP2 - Attribute accessors](#rcpp2---attribute-accessors)
    - [RCPP3 - Simple accessors in headers](#rcpp3---simple-accessors-in-headers)
    - [RCPP4 - Use reference](#rcpp4---use-reference)
    - [RCPP5 - Objects destroying](#rcpp5---objects-destroying)
    - [RCPP6 - The destructor is "virtual"](#rcpp6---the-destructor-is-virtual)
    - [RCPP7 - Methods to override are "virtual"](#rcpp7---methods-to-override-are-virtual)
    - [RCPP8 - The constructor is "explicit"](#rcpp8---the-constructor-is-explicit)
    - [RCPP9 - Constant method](#rcpp9---constant-method)
    - [RCPP10 - Include in headers](#rcpp10---include-in-headers)
    - [RCPP11 - The `#pragma once`](#rcpp11---the-pragma-once)
    - [RCPP12 - Syntactic organization of header files](#rcpp12---syntactic-organization-of-header-files)
    - [RCPP13 - Syntactic organization of source files](#rcpp13---syntactic-organization-of-source-files)
    - [RCPP14 - Code documentation](#rcpp14---code-documentation)
    - [RCPP15 - Code documentation of functions/methods](#rcpp15---code-documentation-of-functionsmethods)
    - [RCPP16 - Code documentation of classes](#rcpp16---code-documentation-of-classes)
    - [RCPP17 - Code documentation of structures](#rcpp17---code-documentation-of-structures)
    - [RCPP18 - Code documentation of enumeration](#rcpp18---code-documentation-of-enumeration)
    - [RCCP19 - Classes and namespaces naming](#rccp19---classes-and-namespaces-naming)
    - [RCCP20 - Local variables naming](#rccp20---local-variables-naming)
    - [RCCP21 - Class methods naming](#rccp21---class-methods-naming)
    - [RCCP22 - Class member naming](#rccp22---class-member-naming)
    - [RCPP23 - Method parameters naming](#rcpp23---method-parameters-naming)
    - [RCPP24 - Enumeration naming](#rcpp24---enumeration-naming)
    - [RCPP25 - Structure naming](#rcpp25---structure-naming)
    - [RCPP26 - Static methods naming](#rcpp26---static-methods-naming)
    - [RCPP27 - Constant static class members naming](#rcpp27---constant-static-class-members-naming)
    - [RCPP28 - File naming](#rcpp28---file-naming)
    - [RCPP29 - Macro naming](#rcpp29---macro-naming)
    - [RCPP30 - Add units in names](#rcpp30---add-units-in-names)
    - [RCPP31 - One brace equals one line](#rcpp31---one-brace-equals-one-line)
    - [RCPP32 - One file per class, namespace, enum or structure](#rcpp32---one-file-per-class-namespace-enum-or-structure)
    - [RCPP33 - Use the latest revision of C++](#rcpp33---use-the-latest-revision-of-c)
    - [RCPP34 - Default pointer initialization](#rcpp34---default-pointer-initialization)
    - [RCPP35 - Namespace and utility functions](#rcpp35---namespace-and-utility-functions)
    - [RCPP36 - Utility class and functions](#rcpp36---utility-class-and-functions)
  - [C++11 code standards](#c11-code-standards)
    - [RCPP11-1 - Use iterators](#rcpp11-1---use-iterators)
    - [RCPP11-2 - Use constant iterators](#rcpp11-2---use-constant-iterators)
    - [RCPP11-3 - Virtual method override](#rcpp11-3---virtual-method-override)
    - [RCPP11-4 - Attributes default initialization](#rcpp11-4---attributes-default-initialization)
    - [RCPP11-5 - Use nullptr](#rcpp11-5---use-nullptr)
    - [RCPP11-6 - Default pointer initialization](#rcpp11-6---default-pointer-initialization)
    - [RCPP11-7 - Constant values](#rcpp11-7---constant-values)
    - [RCPP11-8 - constexpr, constant values and literal type](#rcpp11-8---constexpr-constant-values-and-literal-type)
    - [RCPP11-9 - Global constant values](#rcpp11-9---global-constant-values)
  - [Qt specific code standards - with standard C++](#qt-specific-code-standards---with-standard-c)
    - [RQT1 - Inherits from QObject](#rqt1---inherits-from-qobject)
    - [RQT2 - QObject and inheritance](#rqt2---qobject-and-inheritance)
    - [RQT3 - Macro Q\_OBJECT](#rqt3---macro-q_object)
  - [Qt specific code standards - with C++11](#qt-specific-code-standards---with-c11)

## Introduction

This contains our coding standards for the Qt projects.

This overload the C++ standards: [C++ standards](CPP-CODING-STANDARDS.md)

First read: [coding standards](CODING_STANDARDS.md) to understand how the standards apply on
projects and the overloading process.

## Qt specific code standards - with standard C++

### RQT1 - Inherits from QObject

- Severity: **Blocking**

By default, and unless there is a logical reason for this, **all classes must be inherited**
_(directly or indirectly)_ from **QObject**.

### RQT2 - QObject and inheritance

- Severity: **Blocking**

By default, and unless there is a logical reason for this, **all class constructors deriving from
QObject must have an optional "parent" parameter**.

Qt has developed a sort of garbage collector based on the QObject inheritance tree. To use it, it
is necessary that the new instances created define a “parent”. Otherwise, it would be necessary to
manually manage their destruction via a delete.

Examples:

```cpp
class MyClass : public QObject
{
    Q_OBJECT

    public:
        MyClass(QObject * parent = NULL);
};

MyClass::MyClass(QObject *parent) :
    QObject(parent)
{
}
```

```cpp
// parent will be "automatically" deleted with this
MyClass *parent = new MyClass(this); // "this" is a QObject

// myClass will be "automatically" deleted with parent
MyClass *myClass = new MyClass(parent);

// No parent has been given, the object must be deleted manually with `delete`
MyClass *otherClass = new MyClass();
delete otherClass;
```

### RQT3 - Macro Q_OBJECT

- Severity: **Blocking**

By default, and unless there is a logical reason for this, all classes derived from QObject must
add the Q_OBJECT macro.

```cpp
class MyClass : public QObject
{
    Q_OBJECT

    public:
        MyClass(QObject *parent = NULL);
};
```

## Qt specific code standards - with C++11

### RQTCPP11-1 - Signals and slots

- Severity: **Blocking**

In the Qt application and across Qt threads, connections between signals and slots must use,
wherever possible, pointer syntax instead of the SIGNAL and SLOTS macros.

Example:

```cpp
class MyClass : public QObject
{
    Q_OBJECT

    public:
        MyClass(QObject *parent = nullptr);

    public slots:
        void onElementCall(int value);

    signals:
        void elementCalled(int value);
};

MyClass::MyClass(QObject *parent) :
    QObject(parent)
{
    connect(this, &MyClass::elementCalled, this, &MyClass::onElementCall);
}
```

### RQTCPP11-2 - Signals and slots, excluding library

- Severity: **Blocking**

If you use a connection between a non-Qt thread (e.g. Posix Thread) and a Qt thread, the
connections between signals and slots must use, as much as possible, the SIGNAL and SLOT macros,
instead of the pointer syntax.

```cpp
class MyClass : public QObject
{
    Q_OBJECT

    public:
        MyClass(QObject *parent = nullptr);

    signals:
        void elementCalled(int value);
};

class MyOtherClass : public QObject
{
    Q_OBJECT

    public:
        MyOtherClass (QObject *parent = nullptr);

    public slots:
        void onElementCall(int value);
};
```

```cpp
// It exists two instances of MyClass and MyOtherClass:
// MyClass *myClass => which is in the main Qt thread
// MyOtherClass *myOtherClass => which is in a Posix thread

connect(myClass, SIGNAL(elementCalled(int)), myOtherClass, SLOT(onElementCall(int)));
```

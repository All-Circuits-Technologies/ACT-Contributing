<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Qt coding standards

Coding standards for Qt projects. This standard overloads the
[C++ standards](coding-standards-cpp.md); first read the
[coding standards index](coding-standards.md) for how the standards combine and the override
mechanism. Like every ACT standard, it covers only what a linter cannot enforce (see
[what belongs here](coding-standards.md#what-belongs-here)).

## Table of contents

- [Qt specific code standards - with standard C++](#qt-specific-code-standards---with-standard-c)
  - [RQT1 - Inherits from QObject](#rqt1---inherits-from-qobject)
  - [RQT2 - QObject and inheritance](#rqt2---qobject-and-inheritance)
  - [RQT3 - Macro Q\_OBJECT](#rqt3---macro-q_object)
- [Qt specific code standards - with C++11](#qt-specific-code-standards---with-c11)
  - [RQTCPP11-1 - Signals and slots](#rqtcpp11-1---signals-and-slots)
  - [RQTCPP11-2 - Signals and slots, excluding library](#rqtcpp11-2---signals-and-slots-excluding-library)

## Qt specific code standards - with standard C++

### RQT1 - Inherits from QObject

By default, and unless there is a logical reason for this, **all classes must be inherited**
_(directly or indirectly)_ from **QObject**.

### RQT2 - QObject and inheritance

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

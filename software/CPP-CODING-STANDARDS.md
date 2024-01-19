<!--
SPDX-FileCopyrightText: 2024 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Coding standards

## Table of contents

- [Coding standards](#coding-standards)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
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

## Introduction

This contains our coding standards for the C++ and Qt projects.

This overload the global standards: [global standards](GLOBAL-CODING-STANDARDS.md)

First read: [coding standards](CODING_STANDARDS.md) to understand how the standards apply on
projects and the overloading process.

## C++ code standards

### RCPP1 - Instance attributes of a private class

- Severity: **Blocking**

The class's instance attributes must be **always** private. (_They cannot be public or protected_)

Example:

```cpp
class MyClass
{
    private:
        int _id;
};
```

> [!NOTE]
> To be able to access instance attributes from outside or from derived classes, it is necessary to
> use accessors.

### RCPP2 - Attribute accessors

- Severity: Non-blocking

The attributes of a class may have:

- a getter (returns the const value), ex: `const X &getY() const;`
- an accessor (returns the pointed and modifiable value), ex: `X &accessY();`
- a setter (affects the value), ex: `void setY(const X &var);`

Example:

```cpp
class ClassA
{
};

class MyClass
{
    public:
       const ClassA *getClassALink() const { return _classALink; }

       ClassA *accessClassALink() { return _classALink; }

       void setClassALink(const ClassA *classALink)
       { _classALink = classALink; }

    private:
        ClassA *_classALink{nullptr};
};
```

### RCPP3 - Simple accessors in headers

- Severity: Non-blocking

If the setters, getters and accessors only contain a return or a simple assignment, their
implementations should be written in the header (to be seen as `inline`).

For the sake of readability, this code must be written on one line (_and will not respect
the rule of: one brace per line; but it's ok_).

> [!TIP]
> By putting the implementation of the methods in the header, they become inline, which tells the
> compiler that it should copy the contents of these methods instead of making a call. If the
> content of the method is simple, it saves time without wasting space.
> But interest fades as the code becomes more complex.
> For further reading on the subject:
> https://docs.microsoft.com/fr-fr/cpp/cpp/inline-functions-cpp?view=msvc-160

Example:

```cpp
class MyClass
{
    public:
        int getBeansNumber() const { return _beansNumber; }

        void setBeansNumber(int beansNumber) { _beansNumber = beansNumber; }

    private:
        int _beansNumber;
};
```

### RCPP4 - Use reference

- Severity: **Blocking**

**Objects must be passed**, wherever possible, **by reference**.

References are used to avoid unnecessary object copying.

_All primitive types are not affected by this rule: bool, int, etc._

Example:

```cpp
class ClassA
{
};

class MyClass
{
    public:
        const ClassA &getCoolStuff() const { return _coolStuff; }

        bool doSomethingCool(const ClassA &element);

        int getSuperCool() const { return _superCool; }

        void mySuperCoolMethod(int superCoolParam);

    private:
        ClassA _coolStuff;
        int _superCool;
};
```

This is also true for class attributes: if we are sure that the object is initialized in the class
constructor, getters can return objects by reference rather than by pointer:

```cpp
class ClassA
{
};

class MyClass
{
    public:
        // This can crash at runtime if _coolStuff is nullptr, so it's important
        // to be sure _coolStuff can't never be null when we call the getter.
        const ClassA &getCoolStuff() const { return *_coolStuff; }

    private:
        ClassA *_coolStuff{nullptr};
};
```

> [!NOTE]
> In this last example, if the _coolStuff attribute is null then the call to `getCoolStuff` will
> crash the app; hence the need to ensure that the attribute is indeed created in the constructors
> before returning its content by reference.

### RCPP5 - Objects destroying

- Severity: **Blocking**

The **destruction of objects instantiated** with the "new" operator **must be handled in the code**.

C++ does not have a garbage collector, so it is very important to avoid memory leaks to ensure
that the destruction of all created objects is managed.

### RCPP6 - The destructor is "virtual"

- Severity: **Blocking**

The **destructor** must be prefixed with the keyword **"virtual"**.

> [!TIP]
> Without the "virtual" keyword applied to the destructor, when deleting from the base class, the
> destructors of derived classes are not called.
> For more reading on the subject:
> https://cpp.developpez.com/faq/cpp/?page=Les-destructeurs#Why-et-quand-faut-il-creer-un-destructeur-virtuel

### RCPP7 - Methods to override are "virtual"

- Severity: **Blocking**

All **methods** of a base class that can be **overridden** in derived classes **must be "virtual"**.

Example:

```cpp
class BaseClass
{
    protected:
        virtual void myMethod();

        virtual void specificMethod() = 0;
};

class DerivedClass : public BaseClass
{
    protected:
        virtual void myMethod();

        virtual void specificMethod();
};
```

### RCPP8 - The constructor is "explicit"

- Severity: Non-blocking

By default, **constructors** should be prefixed with the keyword **"explicit"**.

_This is the nominal case, if you do not put the keyword, you must be able to justify why._

> [!TIP]
> When the word "explicit" is not used, the compiler will try to do an implicit conversion.
> In other words, if the constructor of class A has class B as a parameter, and the constructor of
> class B has an integer as parameter. By writing the integer instead of an instance of class B in
> the constructor of class A, the compiler will automatically instantiate a class B.
> In itself, this is not "bad", but it can be a source of error, because it is too permissive.
> For further reading on the subject:
> https://www.it-swarm-fr.com/fr/c%2B%2B/que-signifie-le-mot-cle-explicite/958392226/ ; see the
> concept of “Converting constructor”:
> https://en.cppreference.com/w/cpp/language/converting_constructor

### RCPP9 - Constant method

- Severity: Non-blocking

Methods which don't modify members of a class should be constant.

Example:

```cpp
class MyClass
{
    public:
        int getMyVar() const { return _myVar; }

    private:
        int _myVar;
};
```

### RCPP10 - Include in headers

- Severity: Non-blocking

To avoid to include unnecessarily code in cascade, **it is important to only add in the headers the
`#include` actually used in the headers themselves**.

As far as possible, the following two points should be encouraged:

- Only add `#include` in source files
- Create Forward declarations in header files

Example:

✅ **good:**

```cpp
class QTimer;

class MyClass
{
    private:
        QTimer *_timer;
};
```

❌ **bad:**

```cpp
#include <QTimer>

class MyClass
{
    private:
        QTimer *_timer;
};
```

### RCPP11 - The `#pragma once`

- Severity: Non-blocking

If the compiler allows it, **prefer to use `#pragma once` instead of `#define`** in headers.

### RCPP12 - Syntactic organization of header files

- Severity: Non-blocking

For greater intelligibility but also to better organize the code, the header files must be
constructed as follows and in this order:

1. REUSE header (see
   [RG22 - SPDX license in headers](GLOBAL-CODING-STANDARDS.md#rg22---spdx-license-in-headers))
2. `#pragma once` or `#define`
3. The `#include` of the class we inherit (if we inherit from a class)
4. All includes to system classes (eg: stdio.h; sorted in alphabetical order)
5. All includes to third-party framework classes (e.g. Qt classes; sorted in alphabetical order)
6. All includes to classes developed within the project and the BE (sorted in alphabetical order)
7. All Forward declarations (sorted in alphabetical order)
8. Two empty lines (to clearly separate what the class needs/imports from what the class actually
   does)
9. The class, structure, namespace, enum, etc.
   1. Under a section `public:` add any structures or enums used to interact with the class
    (_always preferred adding enums in another file_)
   2. Under a section `private:` add any structures or enums only used in the class
   3. Under a section `public:`:
      1. Add all constructors (if necessary)
      2. Add the copy constructors (if necessary)
      3. Add the destructor (if necessary)
   4. Under a section `protected:`:
      1. Add all constructors (if necessary)
      2. Add the copy constructors (if necessary)
   5. Under a section `private:`:
      1. Add all constructors (if necessary)
      2. Add the copy constructors (if necessary)
   6. Under a section `public:` add all the public methods of the class
   7. Under a section `public:` add all static public functions of the class
   8. Under a section `protected:` add all the protected methods of the class
   9. Under a section `protected:` add all static protected functions of the class
   10. Under a section `private:` add all the private methods of the class
   11. Under a section `private:` add all the static private functions of the class
   12. Under a section `public:` add all public static constants dedicated to the class
   13. Under a section `protected:` add all protected static constants dedicated to the class
   14. Under a section `private:` add all private static constants dedicated to the class
   15. Under a section `private:` add all the attributes of the class

For information, and without counter-argument, all groups of elements are separated by a single
empty line.

```cpp
// SPDX-FileCopyrightText: 2019 - 2024 Magnus Carlsen <magnus.carlsen@mail.com>
//
// SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1

#pragma once

#include <stdio.h>

#include <QHash>
#include <QString>

#include "exemple/exemple.hpp"
#include "hmidisplayhelper.hpp"
#include "otherexemple/otherexemple.hpp"

class ForwardedClass;
                                                              // Two empty lines

/** @brief Display a simple view with description, an image and buttons */
class HmiDisplaySimpleTestView : public HmiDisplayHelper
{
    public:
        /** @brief List of buttons which can be displayed in the view */
        enum ButtonEnum : quint8
        {
            OkButton        = 0x01, //!< @brief Display a green success button
            NotOkButton     = 0x02, //!< @brief Display a red fail button
            CancelButton    = 0x04  //!< @brief Display a red fail button
        };

    private:
        /** @brief Contains infomations about a button */
        struct ButtonInfo
        {
            QString label;
            char shortcut;
        };

    public:
        /** @brief Class constructor
            @param commonItf The common interface to use in order to ask the display of a GUI
            @param parent The parent class */
        explicit HmiDisplaySimpleTestView(ICommonInterface& commonItf, QObject *parent = nullptr);

        /** @brief Copy constructor
            @param other The other to copy */
        HmiDisplaySimpleTestView(HmiDisplaySimpleTestView &other);

        /** @brief Class destructor */
        virtual ~HmiDisplaySimpleTestView() override;

    protected:
        /** @brief Class constructor
            @param parent The parent class */
        explicit HmiDisplaySimpleTestView(QObject *parent = nullptr);

    public:
        /** @brief Format and set the element to display
            @note To call before @ref HmiDisplayHelper::displayHmi
            @attention The @ref setButtonName method has to be called before this
                       one
            @param sequenceToken The sequence token linked to the current module sequence
            @param name The name of the view
            @param description The description to display in the view
            @param buttons Flags which has to contain ButtonEnum and allow to choose the buttons to
                           display
            @param imagePath The path of the image to display, if empty no image will be searched
                             and so displayed
            @return True if no problem occurs */
        bool formatAndsetElemToDisplay(const QString &sequenceToken,
                                       const QString &name,
                                       const QString &description,
                                       quint8 buttons = OkButton | NotOkButton,
                                       const QString &imagePath = QString(),
                                       ButtonEnum defaultButton = CancelButton);

        /** @brief Change the button label name displayed
            @attention To work, this method has to be called before the
                       @ref formatAndsetElemToDisplay method
            @param button The button to change its name
            @param labelName The new button label name to display
            @param keyShortCut The new button shorcut */
        void setButtonName(ButtonEnum button, const QString &labelName, char keyShortCut);

    public:
        /** @brief This function does something cool
            @return If the method has really done something cool */
        static bool doSomethingCool();

    protected:
        /** @brief Set the buttons managed by the class
            @note This method won't change the element to display. This info is only used to parse
                  results */
        void setButtons(quint8 buttons) { _buttons = buttons; }

        /** @brief Set the HMI name
            @note This method won't change the element to display. This info is only used to parse
                  results */
        void setName(const QString &name) { _name = name; }

        /** @brief Parse the display result and say if the HMI ended in succes or not
            @param valuesSet The values got from HMI
            @return True if the HMI ended in success */
        virtual bool parseDisplayResult(const JsonArray &valuesSet) override;

    protected:
        /** @brief This function does something so cool
            @return If the method has really done something so cool */
        static bool doSomethingSoCool();

    private:
        /** @brief Doing some tests */
        void doingSomeTests() const;

    private:
        /** @brief This function does something very cool
            @return If the method has really done something very cool */
        static bool doSomethingVeryCool();

    public:
        static const constexpr char* OkButtonKey = "okKey";
        static const constexpr char* OkButtonLabel = QT_TR_NOOP("Ok");
        static const constexpr char  OkKeyShorcut = 'o';
        static const constexpr char* OkFontColor = "#FFFFFF";
        static const constexpr char* OkBackgroundColor = "#43A047";

    protected:
        static const constexpr char* NotOkButtonKey = "notOkKey";
        static const constexpr char* NotOkButtonLabel = QT_TR_NOOP("Not ok");
        static const constexpr char  NotOkKeyShorcut = 'n';
        static const constexpr char* NotOkFontColor = "#FFFFFF";
        static const constexpr char* NotOkBackgroundColor = "#E53935";

    private:
        static const constexpr char* CancelButtonKey = "cancelKey";
        static const constexpr char* CancelButtonLabel = QT_TR_NOOP("Cancel");
        static const constexpr char  CancelKeyShorcut = 'c';
        static const constexpr char* CancelFontColor = "#FFFFFF";
        static const constexpr char* CancelBackgroundColor = "#E53935";

    private:
        ButtonEnum _buttonClicked{0};
        QString _name;
        quint8 _buttons;

        QHash<ButtonEnum, ButtonInfo> _buttonsInfoOverriden;
};
```

### RCPP13 - Syntactic organization of source files

- Severity: Non-blocking

For greater intelligibility but also to better organize the code, the source files must be
constructed as follows and in this order:

1. REUSE header (see
   [RG22 - SPDX license in headers](GLOBAL-CODING-STANDARDS.md#rg22---spdx-license-in-headers))
2. The `#include` of the header file (with the shortest path, ex: `#include "myclass.hpp"`)
4. All includes to system classes (eg: stdio.h; sorted in alphabetical order)
5. All includes to third-party framework classes (e.g. Qt classes; sorted in alphabetical order)
6. All includes to classes developed within the project and the BE (sorted in alphabetical order)
7. Class static members creation
8. Two empty lines (to clearly separate what the class needs/imports from what the class actually
   does)
9. The code implementation

For information, and without counter-argument, all groups of elements are separated by a single
empty line.

```cpp
// SPDX-FileCopyrightText: 2019 - 2024 Magnus Carlsen <magnus.carlsen@mail.com>
//
// SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1

#include "hmidisplaysimpletestview.hpp"

#include <QDir>

#include "bankcoreshared/usebyplugin/icommoninterface.hpp"
#include "testbedcore/parsers/bankjsonformatter.hpp"
#include "testbedcore/parsers/bankjsonparser.hpp"
#include "testbedcore/testbedglobal.hpp"

bool HmiDisplaySimpleTestView::_myStaticElement = true;
                                                                              // Two empty lines

HmiDisplaySimpleTestView::HmiDisplaySimpleTestView(ICommonInterface &commonItf, QObject *parent) :
    HmiDisplayHelper(commonItf, parent)
{
}

bool HmiDisplaySimpleTestView::formatAndsetElemToDisplay(const QString &sequenceToken,
                                                         const QString &name,
                                                         const QString &description,
                                                         quint8 buttons,
                                                         const QString &imagePath,
                                                         ButtonEnum defaultButton)
{
    // Do something
    return true;
}

void HmiDisplaySimpleTestView::setButtonName(HmiDisplaySimpleTestView::ButtonEnum button,
                                             const QString &labelName,
                                             char keyShortCut)
{
    // Do something
}

bool HmiDisplaySimpleTestView::parseDisplayResult(const JsonArray &valuesSet)
{
    // Do something
    return false;
}

```

### RCPP14 - Code documentation

- Severity: **Blocking**
- Overload: [RG5](GLOBAL-CODING-STANDARDS.md#rg5---documentation-written-in-english)

The documentation in code must be written in **English** with doxygen.

Example:

```cpp
/** @brief Class A documentation */
class ClassA
{
}
```

### RCPP15 - Code documentation of functions/methods

- Severity: **Blocking**

The functions/methods must be documented and included in the header files, above their declarations.
The documentation must contain a brief description of what the function/method does, what it
returns and what it takes as arguments.

The tags used are as follows (and in the following order):

```cpp
/** @brief Brief description of what the function does
    @note A note on the function which precises an important element
    @note A second note about an other important element
    @warning A warning about the limit of the function, this is useful when it can't
             be test in the function itself
    @see ClassA
    @param classA What is the meaning of the param, how it is used
    @param sloop What is the meaning of the param, how it is used
    @return What the function returns and what is the meaning of the value returned */
bool myFunction(const ClassA &classA, int &sloop);
```

The tags `@note` and `@warning` should only cover one topic/subject at a time. If necessary, add
multiple tags.

### RCPP16 - Code documentation of classes

- Severity: **Blocking**

The classes must be documented and included in the header files, above their declarations. The
documentation must contain a brief description of what the class represents and what it is used for.
This is where we explain the algorithms linked to the class and what it achieves. But also the links
(if they exist) between the methods and attributes of the class.

The tags used are as follows (and in the following order):

```cpp
/** @brief Brief description of what the class does
    @note A note on the class which precises an important element
    @note A second note about an other important element
    @warning A warning about the limit of the class
    @see ClassA */
class MyClass
{
}
```

The tags `@note` and `@warning` should only cover one topic/subject at a time. If necessary, add
multiple tags.

### RCPP17 - Code documentation of structures

- Severity: **Blocking**

The structures must be documented and included in the header files, above their declarations. The
documentation must contain a brief description of what the structure represents and what it is used
for.

The tags used are as follows (and in the following order):

```cpp
/** @brief Brief description of what the structure does
    @note A note on the structure which precises an important element
    @note A second note about an other important element
    @warning A warning about the limit of the class
    @see ClassA */
struct MyStruct
{
}
```

The tags `@note` and `@warning` should only cover one topic/subject at a time. If necessary, add
multiple tags.

All the elements of the structure _may_ be documented _(not compulsory)_. It can be (compulsory) if
the documentation of the structure is not sufficient to understand what the element does or if the
name of the element is not understandable by itself.

_For uniformity, try to align the doxygen elements together._

The tags used are as follows (and in the following order):

```cpp
/** @brief Brief description of what the structure does
    @note A note on the structure which precises an important element
    @note A second note about an other important element
    @warning A warning about the limit of the structure
    @see ClassA */
struct MyStruct
{
    int id;           //!< @brief Id of the structure
    MyClassA myClass; /*!< @brief Class value
                           @note Class note */
}
```

### RCPP18 - Code documentation of enumeration

- Severity: **Blocking**

The enumerations must be documented and included in the header files, above their declarations. The
documentation must contain a brief description of what the enum represents and what it is used
for.

The tags used are as follows (and in the following order):

```cpp
/** @brief Brief description of what the enum does
    @note A note on the enum which precises an important element
    @note A second note about an other important element
    @warning A warning about the limit of the enum
    @see ClassA */
enum MyEnum
{
}
```

The tags `@note` and `@warning` should only cover one topic/subject at a time. If necessary, add
multiple tags.

All the elements of the enum _may_ be documented _(not compulsory)_. It can be (compulsory) if
the documentation of the enum is not sufficient to understand what the element does or if the
name of the element is not understandable by itself.

_For uniformity, try to align the doxygen elements together._

The tags used are as follows (and in the following order):

```cpp
/** @brief Brief description of what the enum does
    @note A note on the enum which precises an important element
    @note A second note about an other important element
    @warning A warning about the limit of the enum
    @see ClassA */
enum MyEnum
{
    Id,     //!< @brief Id enum
    MyClass /*!< @brief MyClass enum
                 @note MyClass enum note */
}
```

### RCCP19 - Classes and namespaces naming

- Severity: **Blocking**

**Class and namespace names** must respect **UpperCamelCase**.

Example:

```cpp
namespace MyNamespace
{
};

class MyClass
{
};
```

### RCCP20 - Local variables naming

- Severity: **Blocking**

The **name of local variables** must respect the **lowerCamelCase**.

Example:

```cpp
int myVar;
```

### RCCP21 - Class methods naming

- Severity: **Blocking**

The **method name** must respect the **lowerCamelCase**.

Example:

```cpp
class MyClass
{
    public:
        void myMethod();
};
```

### RCCP22 - Class member naming

- Severity: **Blocking**

A **class member name** must respect the **lowerCamelCase** and begins with an underscore.

Example:

```cpp
class MyClass
{
    private:
        int _myElement = 0;
};
```

### RCPP23 - Method parameters naming

- Severity: **Blocking**

The **name of method parameters** must respect the **lowerCamelCase**.

Example:

```cpp
void MyClass::myMethod(bool myParam1, int myParam2)
{
    // Do something clever
}
```

### RCPP24 - Enumeration naming

- Severity: **Blocking**

The **name of the enumeration and its members** must respect the **UpperCamelCase**.

Example:

```cpp
enum AnimalType
{
    Dog,
    Cat
};
```

### RCPP25 - Structure naming

- Severity: **Blocking**

The **name of the structure** must respect the **UpperCamelCase**.

The **name of the structure members** must respect the **lowerCamelCase**.

Example:

```cpp
struct MyStruct
{
    int myValue;
};
```

### RCPP26 - Static methods naming

- Severity: **Blocking**

The **name of the static methods** must respect the **lowerCamelCase**.

Example:

```cpp
class MyClass
{
    public:
        static void myStaticMethod(int param);
};
```

### RCPP27 - Constant static class members naming

- Severity: **Blocking**

The **name of constant static class members** must respect the **UpperCamelCase**.

Example:

```cpp
class MyClass
{
    private:
        static const int MaxNumber;
};
```

### RCPP28 - File naming

- Severity: **Blocking**

Each **file** must be **named according to the class, namespace, enum or struct it contains**.

The **file name is written in lowercase**, **does not contain spaces, hyphens or underscores**.

Example:

> If I have a class named: `MyBeautifulClass`, the name of the header and source files which
> contain the class must be: `mybeautifulclass.hpp` and `mybeautifulclass.cpp`.

### RCPP29 - Macro naming

- Severity: **Blocking**

The **macro name** must respect the **SCREAMING_SNAKE_CASE**.

Example:

```cpp
#define MY_CONSTANT_VALUE (0)

class MyClass
{
};
```

### RCPP30 - Add units in names

- Severity: **Blocking**

When a **constant**, a **variable**, a **method refers** in their names to something quantifiable,
the **unit must be specified in the name**.

Examples:

```cpp
#define TIMEOUT_IN_MS (1000)

class MyTimer
{
    private:
        int getTimeoutInMs() const;
};
```

```cpp
int distanceInKm = 2;
```

### RCPP31 - One brace equals one line

- Severity: **Blocking**

A brace must be alone on its own line.

_There is one exception to this rule with the inline methods written in the header next to their
definitions (for readability)._

Examples:

```cpp
class MyClass
{
    public:
        int getValue() const { return _value; }

    private:
        int _value;
};
```

```cpp
if(test)
{
}
```

### RCPP32 - One file per class, namespace, enum or structure

- Severity: **Blocking**

Each **class or namespace** must have its **own file**.

If **an enum or a structure** is **not contained in a class**, it must **have its own file**.

### RCPP33 - Use the latest revision of C++

- Severity: Non-blocking

_On new projects and if the compiler allows it_, you should use the latest compatible C++ revision.

The C++ revisions add new features but also make certain mechanics simpler. But depending on the
version of your compiler, it may not support all revisions of C++, and sometimes only in part.

### RCPP34 - Default pointer initialization

- Severity: **Blocking**

**All pointers** that **do not point to a created instance must be initialized with 0** (or MACRO,
like NULL).

Example:

```cpp
OtherClass *otherClass = NULL;
OtherClass *anotherClass = new OtherClass();
```

### RCPP35 - Namespace and utility functions

- Severity: **Blocking**

Functions that can be grouped by theme must be grouped in namespaces.

If those functions must call "private functions" common to the others functions, see:
[RCPP36](#rcpp36---utility-class-and-functions).

Example:

```cpp
namespace Utilities
{
    int doSomethingCool(const MyClass &myClass);
};
```

### RCPP36 - Utility class and functions

- Severity: **Blocking**

_If grouping **utility functions** cannot be done in a namespace_, because some functions has to be
private, then they must be grouped in a class as a static methods.

This could also be because we want to use the class as a "feature class", only used by static
methods.

Examples:

```cpp
class Utilities
{
    public:
        static int doSomethingCool(const MyClass &myClass);

    private:
        static int sharedCoolStuff(const MyClass &myClass);
};
```

```cpp
class CoolStuff
{
    private:
        explicit CoolStuff();

    public:
        static bool doVeryCoolStuff();

    private:
        void doCoolStuff();

    private:
        int _coolVariable{0};
};
```

## C++11 code standards

### RCPP11-1 - Use iterators

- Severity: Non-blocking

To iterate over a list, map, etc. your should use iterators instead of indexes.

_Iteration forward:_

✅ **good:**

```cpp
vector<int> myVector = { 0, 1, 2, 3, 4 };

for(vector<int>::iterator iter = myVector.begin(); iter != myVector.end(); ++iter)
{
    // Do something clever
}
```

🟡 **to avoid:**

```cpp
vector<int> myVector = { 0, 1, 2, 3, 4 };

for(int idx = 0; idx < myVector.length(); ++idx)
{
    // Do something clever
}
```

_Backward iterations:_

✅ **good:**

```cpp
vector<int>::iterator iter = myVector.end();

while(iter != myVector.begin())
{
    --iter;
    // Do something clever
}
```

🟡 **to avoid:**

```cpp
vector<int> myVector = { 0, 1, 2, 3, 4 };

for(int idx = (myVector.length() - 1); idx >= 0 ; --idx)
{
    // Do something clever
}
```

### RCPP11-2 - Use constant iterators

- Severity: **Blocking**

Iterators which are not intended to modify containers must be constant.

Example:

```cpp
vector<int> myVector = { 0, 1, 2, 3, 4 };

for(vector<int>::const_iterator citer = myVector.begin(); citer != myVector.end(); ++citer)
{
    // Do something clever
}
```

> [!NOTE]
> With the C++ default containers, the `begin()` and `end()` methods return either a constant or
> non-constant iterator. What they return depends on the iterator type, if the iterator type is
> `const_iterator`, they will return a constant value.
> This is also true in Qt, but we prefer to use the methods `constBegin()`, `cbegin()` and
> `constEnd()`, `cend()`, to clearly mark the difference.

### RCPP11-3 - Virtual method override

- Severity: **Blocking**

All derived class methods that override virtual methods must have an `override` suffix.

Example:

```cpp
class BaseClass
{
    public:
        virtual ~BaseClass();

    protected:
        virtual void myMethod();

        virtual void specificMethod() = 0;
};

class DerivedClass : public BaseClass
{
    public:
        virtual ~DerivedClass() override;

    protected:
        virtual void myMethod() override;

        virtual void specificMethod() override;
};
```

> [!NOTE]
> The keyword "override" is just a safeguard. It simply validates that the method which is
> overloaded is indeed virtual (otherwise, an error is produced by the compiler).
>
> For further reading on the subject: https://en.cppreference.com/w/cpp/language/override

> [!NOTE]
> The destructor must also be overriden for consistency.
>
> For further reading on the subject:
> - https://stackoverflow.com/a/54694226
> - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-override

### RCPP11-4 - Attributes default initialization

- Severity: Non-blocking

When **attributes** have **default values** _​​(true for all constructors)_, these values **​​must be
put in the header rather than in the constructors**.

This allows you to avoid making mistakes when adding new constructors, or when you want to change
the value.

Example:

```cpp
class MyClass
{
    private:
        int _myValue{0};
        bool _myTest{false};
};
```

### RCPP11-5 - Use nullptr

- Severity: **Blocking**

The keyword: `nullptr`, must always be used to nullify a pointer or test its nullity.

From C++11, it's better to user `nullptr` instead of `0`, or macro returning `0` like `NULL`.

Example:

```cpp
MyClass *myClassPtr = nullptr;

if(myClassPtr == nullptr)
{
    // Do something clever
}
```

### RCPP11-6 - Default pointer initialization

- Severity: **Blocking**
- Overload: [RCPP34](#rcpp34---default-pointer-initialization)

**All pointers** that **do not point to a created instance must be initialized with `nullptr`**.

Examples:

```cpp
class OtherClass;

class MyClass
{
    private:
        OtherClass *_otherClass = nullptr;
};
```

```cpp
OtherClass *otherClass = nullptr;
OtherClass *anotherClass = new OtherClass();
```

### RCPP11-7 - Constant values

- Severity: **Blocking**

Constant values must be defined by `static const` rather than macros (`#define`).

Example:

✅ **good:**

```cpp
class MyClass
{
    private:
        static const constexpr int TimerValueInMs{0};
};
```

❌ **bad:**

```cpp
#define TIMER_VALUE_IN_MS (0)

class MyClass
{
};
```

### RCPP11-8 - constexpr, constant values and literal type

- Severity: **Blocking**

All constant values ​​that are "Literal types" must be "`constexpr`".

Example:

```cpp
class MyClass
{
    private:
        static const constexpr bool DefaultBoolValue{false};
};
```

> [!NOTE]
> The `constexpr` keyword lets the compiler know that the constant value can be assigned at
> compilation (rather than at code execution).
>
> For further reading on the subject:
> https://docs.microsoft.com/en-us/cpp/cpp/constexpr-cpp?view=msvc-160

### RCPP11-9 - Global constant values

- Severity: Non-blocking

All global constant values ​​of a project should be stored in dedicated header files and specific
namespaces.

Example:

```cpp
namespace ProjectConstants
{
    namespace Database
    {
        static const constexpr int DdAddressPort{5520};
    };
};
```

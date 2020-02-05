Coding Style Guidelines
=======================

Coding style guides are a set of guidelines and best practices to help
programmers write consistent and manageable code. Not only does it reduce the
variance in the kind of code produced by a team, it also makes it easier for the
programmer to make the right design choices for the project. 
While we list a couple of simple points here, the general idea is to
mimic the existing style in the project.

However, significantly more important than the style, is the following thought
when writing code:

**Write code for a reader - who has no prior idea of what you are doing.**


Style
=====

## Python

The industry standard for Python styling is PEP8.
Details of the style guide can be found at 
https://www.python.org/dev/peps/pep-0008/.
It's a good read, and adhering to it would go a long way in making our product
consistent.

That said, of course it's hard to stick to PEP8 format perfectly (and to
remember all the guidelines unless you're a seasoned PEP8 python programmer).
To help out in those cases, the following tools are extremely useful and highly
recommended to be installed -

1. [pycodestyle](https://pypi.org/project/pycodestyle/)
A tool to check for PEP8 compliance. It flags items that don't adhere to the
PEP8 style.
2. [autopep8](https://pypi.org/project/autopep8/0.8/) - A tool to automatically
format your python code to be PEP8 compliant. Works based on the output of the
style checker above.

Once we have our own repository set up, we can even consider making the style
checker a standard part of our tooling to ensure that code that gets checked in
is of good quality.

For docstrings, we should use [google style docstrings](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html)


## C++

### Formatting

The following are some formatting standards being following in Clara Genomics

* **Class names** in `CamelCase`
* **Function names** and public member variables in `snake_case`
  Private member variables in `snake_case_ending_with_underscore_`
* **Enums** Prefer enum classes `enum class MyEnum` over C-style enums
  enum members in `snake_case`
* **Macros** in `ALLCAPS`
  Macros to be prefixed with `CGA_<MODULE>_VAR_NAME`

In addition, we use [clang-format](https://clang.llvm.org/docs/ClangFormat.html)
to automate common code formatting.
If clang-format is installed, running `make format` in the build directory
will automatically adapt all source files to a common format. 


Interface Design
================

## C++

### Modules

Each module/package must have a public header file `<module>.hpp`.
This header file:

* Must include the declaration of an init function `StatusType init();`
  (in the package's namespace).
  The function is required, even if it is empty, and the empty definition should
  not be inline - it should be in `src`.
* Should include any shared definitions for the package, especially any status
  enumerants.
* Should include the proper package doxygen tags (see below)

### Public Scoping

* ALL public Clara Genomics packages and their functions, typedefs, constants,
  etc MUST be encapsulated in the namespace `claragenomics`.  
* Within that namespace, it is up to the developer as to whether they feel the
  need to further scope their package in a sub-package.
* However, developers should avoid polluting the claragenomics namespace with
  generic names like "Error"; if these are needed, consider wrapping them or
  having claragenomics adopt a standard.

### Public Class Interfaces

Wherever possible, public interfaces that are C++ classes should be
pure interfaces.

The common way of doing this is a pure virtual base class with
a hidden implementation subclass (or several).
The class should consist of nothing but public, pure virtual functions and
(usually) a public, virtual destructor `~myclass() = default`. 
Where possible, the public API class class should contain
* no non-public members
* no non-virtual, non-static member functions
* no data members.  
* no inline functions
All implementation should be done in a subclass whose name and declaration
should NOT be in the public headers Construction of the concrete subclass(es)
should be done via some form of static/global factory function in the public
API.

The goals here are simple:
* The API headers should be for the API and should not be polluted with
implementation details
* Changing the implementation of the API should not require the client binary to
recompile.  It should only require a relink in the case of a static lib, and no
rebuild in the case of a shared library.

This assumes that none of the public API functions are lightweight enough that
virtual function calling overhead would present a significant performance
problem.  This is the case that should, in general be sought.

This does mean that your API classes cannot be layered directly into another
object - they must be referred to via pointers (or possibly references).
This is actually preferable, as it makes explicit the creation and deletion of
the objects.

### Public Headers

Public headers should only declare and define entities that are required to
interact with the library, not library internal-only declarations and
definitions.
The public headers should be placed in the projects `include` directory.

Wherever possible, public headers should follow these guidelines:

* Self-sufficient: Each header should compile without error if included in an
otherwise-empty cpp file.
* Minimal: The minimal set of headers needed to attain self-sufficiency should
be included; no more
* Least-coupled: Prefer `class <foo>` forward declarations over including a
class's header wherever possible
(e.g. when a class is only referenced via pointer, not layering)

Headers should use `#pragma once` as include guards.

Headers that are only needed by the implementation (not the public APIs) should
be in the `src` directory, not `include`.

## Doxygen Tags

In the `<package>.hpp` header, a block definition a group name for the package
must be included, e.g. the following. Note the open and close blocks:

```
#pragma once

/// \defgroup mypackage My new package
/// Base docs for the mypackage package (tbd)
/// \ingroup mypackage
/// \{
namespace claragenomics {
  namespace mypackage {
    enum class StatusType {
      success = 0,
      generic_error
    };

    StatusType init();
  };
};

/// \}
```

In addition, public headers for all items in the package should scope themselves
into this group, e.g.:

```
namespace claragenomics {
  namespace mypackage {
    /// \addtogroup mypackage
    /// \{

    // <stuff>

    /// \}
  };
};
```

.. Copyright (c) 2016, Ruslan Baratov
.. All rights reserved.

Project declaration
-------------------

The next must-have command is
`project <https://cmake.org/cmake/help/latest/command/project.html>`__.
The command ``project(foo)`` will set languages to C and C++ (default),
declare some ``foo_*`` variables and run basic build tool checks.

.. admonition:: CMake documentation

  * `project <https://cmake.org/cmake/help/latest/command/project.html>`__

Tools discovering
=================

.. _project tools discovering:

By default on calling the ``project`` command, CMake will try to detect compilers
for default languages: C and C++. Let's add some variables and check where
they are defined:

.. literalinclude:: /examples/project-examples/set-compiler/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 3-5,9-11

.. admonition:: Examples on GitHub

  * `Repository <https://github.com/cgold-examples/project-examples>`__
  * `Latest ZIP <https://github.com/cgold-examples/project-examples/archive/master.zip>`__

Run test on ``Linux``:

.. code-block:: none
  :emphasize-lines: 2, 4-5, 9, 15, 21-22

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hset-compiler -B_builds
  Before 'project':
    C: ''
    C++: ''
  -- The C compiler identification is GNU 4.8.4
  -- The CXX compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Check for working CXX compiler: /usr/bin/c++
  -- Check for working CXX compiler: /usr/bin/c++ -- works
  -- Detecting CXX compiler ABI info
  -- Detecting CXX compiler ABI info - done
  -- Detecting CXX compile features
  -- Detecting CXX compile features - done
  After 'project':
    C: '/usr/bin/cc'
    C++: '/usr/bin/c++'
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

CMake will run tests for other tools as well, so we should try to avoid
checking for anything before ``project``. Place all checks
**after project declared**.

Also ``project`` is a place where the toolchain file will be read.

.. literalinclude:: /examples/project-examples/toolchain/CMakeLists.txt
  :language: cmake

.. literalinclude:: /examples/project-examples/toolchain/toolchain.cmake
  :language: cmake

.. code-block:: none
  :emphasize-lines: 2-5, 9, 12, 15-17, 20, 23, 26-28, 30

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Htoolchain -B_builds -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake
  Before 'project'
  Processing toolchain
  Processing toolchain
  -- The C compiler identification is GNU 4.8.4
  -- The CXX compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  Processing toolchain
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  Processing toolchain
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  Processing toolchain
  Processing toolchain
  Processing toolchain
  -- Detecting C compile features - done
  -- Check for working CXX compiler: /usr/bin/c++
  Processing toolchain
  -- Check for working CXX compiler: /usr/bin/c++ -- works
  -- Detecting CXX compiler ABI info
  Processing toolchain
  -- Detecting CXX compiler ABI info - done
  -- Detecting CXX compile features
  Processing toolchain
  Processing toolchain
  Processing toolchain
  -- Detecting CXX compile features - done
  After 'project'
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

.. note::

  You may notice that the toolchain is read several times.

Languages
=========

If you don't have or don't need support for one of the default languages you can
set language explicitly after the name of the project. This is how we setup a 
C-only project:

.. literalinclude:: /examples/project-examples/c-compiler/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 7

There are no checks for the C++ compiler and the variable with the path to the C++ compiler
is empty for now:

.. code-block:: none
  :emphasize-lines: 2, 8, 15

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hc-compiler -B_builds
  Before 'project':
    C: ''
    C++: ''
  -- The C compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  After 'project':
    C: '/usr/bin/cc'
    C++: ''
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

Of course you will not be able to build C++ targets anymore. Since CMake
thinks that ``*.cpp`` extension is for C++ sources (by default), there will
be an error reported if C++ is not listed (discovering of C++
tools will not be triggered):

.. literalinclude:: /examples/project-examples/cpp-not-found/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 2, 4

.. code-block:: none
  :emphasize-lines: 2, 11-12

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hcpp-not-found -B_builds
  -- The C compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Configuring done
  CMake Error: Cannot determine link language for target "foo".
  CMake Error: CMake can not determine linker language for target: foo
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

We can save some time by using special language ``NONE`` when we don't need any
tools at all:

.. literalinclude:: /examples/project-examples/no-language/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 2

No checks for C or C++ compiler as you can see:

.. code-block:: none

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hno-language -B_builds
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

.. note::

  This form will be used widely in examples in cases when we don't need to
  build targets.

.. note::

  For CMake 3.0+, the sub-option ``LANGUAGES`` is added, since it will be:

  .. code-block:: cmake

    cmake_minimum_required(VERSION 3.0)
    project(foo LANGUAGES NONE)

.. _project variables:

Variables
=========

Command ``project`` declares the ``*_{SOURCE,BINARY}_DIR`` variables. From version
``3.0`` onwards you can add ``VERSION`` which additionally declares the 
``*_VERSION_{MAJOR,MINOR,PATCH,TWEAK}`` variables:

.. literalinclude:: /examples/project-examples/variables/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 1, 9

.. code-block:: none
  :emphasize-lines: 2, 4-7, 23-26

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hvariables -B_builds
  Before project:
    Source:
    Binary:
    Version:
    Version (alt): ..
  -- The C compiler identification is GNU 4.8.4
  -- The CXX compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Check for working CXX compiler: /usr/bin/c++
  -- Check for working CXX compiler: /usr/bin/c++ -- works
  -- Detecting CXX compiler ABI info
  -- Detecting CXX compiler ABI info - done
  -- Detecting CXX compile features
  -- Detecting CXX compile features - done
  After project:
    Source: /.../project-examples/variables
    Binary: /.../project-examples/_builds
    Version: 1.2.7
    Version (alt): 1.2.7
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

You can use alternative ``foo_{SOURCE,BINARY}_DIRS``/
``foo_VERSION_{MINOR,MAJOR,PATCH}`` synonyms. This is useful
when you have a hierarchy of projects:

.. literalinclude:: /examples/project-examples/hierarchy/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6-8

.. literalinclude:: /examples/project-examples/hierarchy/boo/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6-9

.. code-block:: none
  :emphasize-lines: 22

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hhierarchy -B_builds
  -- The C compiler identification is GNU 4.8.4
  -- The CXX compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Check for working CXX compiler: /usr/bin/c++
  -- Check for working CXX compiler: /usr/bin/c++ -- works
  -- Detecting CXX compiler ABI info
  -- Detecting CXX compiler ABI info - done
  -- Detecting CXX compile features
  -- Detecting CXX compile features - done
  From top level:
    Source (general): /.../project-examples/hierarchy
    Source (foo): /.../project-examples/hierarchy
  From subdirectory 'boo':
    Source (general): /.../project-examples/hierarchy/boo
    Source (foo): /.../project-examples/hierarchy
    Source (boo): /.../project-examples/hierarchy/boo
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

As you can see, we are still able to use ``foo_*`` variables even if the new
command ``project(boo)`` is called.

When not declared
=================

CMake will implicitly declare ``project`` in case there is no such command
in the top-level CMakeLists.txt. This will be equal to calling ``project``
before any other commands. It means that ``project`` will be called **before**
``cmake_minimum_required``, so it could lead to problems as described in the 
:ref:`previous section <cmake_minimum_required should be first>`:

.. literalinclude:: /examples/project-examples/not-declared/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 3

.. literalinclude:: /examples/project-examples/not-declared/boo/CMakeLists.txt
  :language: cmake

.. code-block:: none
  :emphasize-lines: 17

  [project-examples]> rm -rf _builds
  [project-examples]> cmake -Hnot-declared -B_builds
  -- The C compiler identification is GNU 4.8.4
  -- The CXX compiler identification is GNU 4.8.4
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Check for working CXX compiler: /usr/bin/c++
  -- Check for working CXX compiler: /usr/bin/c++ -- works
  -- Detecting CXX compiler ABI info
  -- Detecting CXX compiler ABI info - done
  -- Detecting CXX compile features
  -- Detecting CXX compile features - done
  Before 'cmake_minimum_required'
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../project-examples/_builds

Summary
=======

* You must have ``project`` command in your top-level ``CMakeLists.txt``
* Use ``project`` to declare non divisible monolithic hierarchy of targets
* Try to minimize the number of instructions before ``project`` and verify
  that variables are declared in such block of CMake code

.. Copyright (c) 2016, Ruslan Baratov
.. All rights reserved.

Cache variables
---------------

Cache variables are saved in :ref:`CMakeCache.txt` file:

.. literalinclude:: /examples/usage-of-variables/cache-cmakecachetxt/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4

.. code-block:: none
  :emphasize-lines: 2, 7

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hcache-cmakecachetxt -B_builds
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> grep abc _builds/CMakeCache.txt
  abc:STRING=687

No scope
========

Unlike regular variables CMake cache variables have global scope:

.. literalinclude:: /examples/usage-of-variables/cache-no-scope/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6, 8

.. literalinclude:: /examples/usage-of-variables/cache-no-scope/boo/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 3

.. code-block:: none
  :emphasize-lines: 2-3

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hcache-no-scope -B_builds
  A: 123
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

Double set
==========

If a variable is already in cache then the command ``set(... CACHE ...)`` will have no
effect - old variable values will be used:

.. literalinclude:: /examples/usage-of-variables/double-set/1/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4

.. code-block:: none
  :emphasize-lines: 1, 3-4, 9

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cp double-set/1/CMakeLists.txt double-set/
  [usage-of-variables]> cmake -Hdouble-set -B_builds
  Variable from cache: 123
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> grep abc _builds/CMakeCache.txt
  abc:STRING=123

Update :ref:`CMakeLists.txt <cmakelists.txt>` (don't remove cache!):

.. literalinclude:: /examples/usage-of-variables/double-set/2/CMakeLists.txt
  :diff: /examples/usage-of-variables/double-set/1/CMakeLists.txt

.. code-block:: none
  :emphasize-lines: 2-3, 8

  [usage-of-variables]> cp double-set/2/CMakeLists.txt double-set/
  [usage-of-variables]> cmake -Hdouble-set -B_builds
  Variable from cache: 123
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> grep abc _builds/CMakeCache.txt
  abc:STRING=123

-D
==

Cache variables can be set using the ``-D`` command line option.  Variable that are set via 
``-D`` take priority over the ``set(... CACHE ...)`` command.

.. code-block:: none
  :emphasize-lines: 1-2, 7

  [usage-of-variables]> cmake -Dabc=444 -Hdouble-set -B_builds
  Variable from cache: 444
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> grep abc _builds/CMakeCache.txt
  abc:STRING=444

Initial cache
=============

If there are a lot of variables to set it's not so convenient to use ``-D``.
In this case the user can define all variables in a separate file and load
it via ``-C``:

.. literalinclude:: /examples/usage-of-variables/initial-cache/cache.cmake
  :language: cmake
  :emphasize-lines: 3-5

.. literalinclude:: /examples/usage-of-variables/initial-cache/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6-8

.. code-block:: none
  :emphasize-lines: 2, 4-6

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -C initial-cache/cache.cmake -Hinitial-cache -B_builds
  loading initial cache file initial-cache/cache.cmake
  A: 123
  B: 456
  C: 789
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

Force
=====

If you want to set a cache variable even if it's already present in the cache file,
you can use the keyword ``FORCE``:

.. literalinclude:: /examples/usage-of-variables/force/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4

.. code-block:: none
  :emphasize-lines: 2-3

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -DA=456 -Hforce -B_builds
  A: 123
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

This is quite a surprising behavior for users and conflicts with the nature of
cache variables that were designed to store variable values once and globally.

.. warning::

  ``FORCE`` usually it is an indicator of badly designed CMake code.

Force as a workaround
=====================

``FORCE`` can be used to fix the problem that was described
:ref:`eariler <cache confusing>`:

.. literalinclude:: /examples/usage-of-variables/no-force-confuse/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-5

.. code-block:: none
  :emphasize-lines: 2-3, 7-8

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hno-force-confuse -B_builds
  A: 456
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> cmake -Hno-force-confuse -B_builds
  A: 123
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

With ``FORCE``, the variable will be set even if it's already present in cache, so
a regular variable with the same name will be unset at each run:

.. literalinclude:: /examples/usage-of-variables/force-workaround/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-5

.. code-block:: none
  :emphasize-lines: 2-3, 7-8

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hforce-workaround -B_builds
  A: 456
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> cmake -Hforce-workaround -B_builds
  A: 456
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

Cache type
==========

Although the type of any variable is **always** string, you can add some hints which
will be used by CMake-GUI:

.. literalinclude:: /examples/usage-of-variables/cache-type/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-7

Run configure:

.. image:: cache-gui/01-generate.png
  :align: center

Variable ``FOO_A`` will be treated as a boolean. Uncheck box and run configure:

.. image:: cache-gui/02-bool.png
  :align: center

Variable ``FOO_B`` will be treated as a path to the file. Click on ``...``:

.. image:: cache-gui/03-filepath.png
  :align: center

Select file:

.. image:: cache-gui/04-change-filepath.png
  :align: center

Run configure:

.. image:: cache-gui/05-ok-filepath.png
  :align: center

Variable ``FOO_C`` will be treated as a path to the directory. Click on ``...``:

.. image:: cache-gui/06-path.png
  :align: center

Select directory:

.. image:: cache-gui/07-change-path.png
  :align: center

Run configure:

.. image:: cache-gui/08-ok-path.png
  :align: center

Variable ``FOO_D`` will be treated as string. Click near variable name and
edit:

.. image:: cache-gui/09-string.png
  :align: center

Run configure:

.. image:: cache-gui/10-ok-string.png
  :align: center

Description of variable:

.. literalinclude:: /examples/usage-of-variables/cache-type/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 7

Will pop-up as a hint for users:

.. image:: cache-gui/11-popup-description.png
  :align: center

.. admonition:: CMake documentation

  * `Cache entry <https://cmake.org/cmake/help/latest/command/set.html#set-cache-entry>`__

Enumerate
=========

The selection widget can be created for variables of string type:

.. literalinclude:: /examples/usage-of-variables/cache-enum/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4, 6

.. image:: cache-gui/12-enum.png
  :align: center

.. admonition:: CMake documentation

  * `STRINGS property <https://cmake.org/cmake/help/latest/prop_cache/STRINGS.html>`__

Internal
========

Variable with type ``INTERNAL`` will not be shown in CMake-GUI (again, real type
is a string still):

.. literalinclude:: /examples/usage-of-variables/internal-gui/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-6

.. image:: cache-gui/13-gui-internal.png
  :align: center

Also such type of a variable implies ``FORCE``:

.. literalinclude:: /examples/usage-of-variables/internal-force/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-6, 8-10

Variable ``FOO_A`` will be set to ``123`` then **reset** to ``456`` because the 
behavior is similar to variable **with FORCE**, then one more time to ``789``,
so the final result is ``789``. Variable ``FOO_B`` is a cache variable with **no
FORCE** so first ``123`` will be saved in cache, then since ``FOO_B`` is already
in cache, ``456`` and ``789`` **will be ignored**, so the final result is ``123``:

.. code-block:: none
  :emphasize-lines: 2-4

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hinternal-force -B_builds
  FOO_A (internal): 789
  FOO_B (string): 123
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

Advanced
========

If a variable is marked as advanced:

.. literalinclude:: /examples/usage-of-variables/advanced-gui/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 5, 8

it will not be shown in CMake-GUI if ``Advanced`` checkbox is not set:

.. image:: cache-gui/14-gui-no-advanced.png
  :align: center

.. image:: cache-gui/15-gui-advanced.png
  :align: center

.. admonition:: CMake documentation

  * `mark_as_advanced <https://cmake.org/cmake/help/latest/command/mark_as_advanced.html>`__

Use case
========

.. _cache use case:

The ability of cache variables respect user's settings fits perfectly for
creating project's customization option:

.. literalinclude:: /examples/usage-of-variables/project-customization/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-5

Default value:

.. code-block:: none
  :emphasize-lines: 2-4

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hproject-customization -B_builds
  FOO_A: Default value for A
  FOO_B: Default value for B
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

User's value:

.. code-block:: none
  :emphasize-lines: 1-2

  [usage-of-variables]> cmake -DFOO_A=User -Hproject-customization -B_builds
  FOO_A: User
  FOO_B: Default value for B
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

Note that such approach doesn't work for regular CMake variable ``FOO_B``:

.. code-block:: none
  :emphasize-lines: 1, 3

  [usage-of-variables]> cmake -DFOO_B=User -Hproject-customization -B_builds
  FOO_A: User
  FOO_B: Default value for B
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds

Option
======

Command ``option`` can be used for creating boolean cache entry:

.. literalinclude:: /examples/usage-of-variables/option/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-5

.. code-block:: none
  :emphasize-lines: 2-4, 9-10

  [usage-of-variables]> rm -rf _builds
  [usage-of-variables]> cmake -Hoption -B_builds
  FOO_A: OFF
  FOO_B: ON
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /.../usage-of-variables/_builds
  [usage-of-variables]> grep FOO_ _builds/CMakeCache.txt
  FOO_A:BOOL=OFF
  FOO_B:BOOL=ON

.. admonition:: CMake documentation

  * `option <https://cmake.org/cmake/help/latest/command/option.html>`__

Unset
=====

If you want to remove variable ``X`` from cache you need to use
``unset(X CACHE)``. Note that ``unset(X)`` will remove regular variable from
current scope and have no effect on cache:

.. literalinclude:: /examples/usage-of-variables/unset-cache/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 4-5, 8, 11

When we have both cache and regular ``X``, the regular ``x`` has higher 
priority and will be printed:

.. literalinclude:: /examples/usage-of-variables/unset-cache/configure.log
  :emphasize-lines: 3

Command ``unset(X)`` will remove regular variable so cache variable will be
printed:

.. literalinclude:: /examples/usage-of-variables/unset-cache/configure.log
  :emphasize-lines: 4

Command ``unset(X CACHE)`` will remove cache variable too. Now no variables
left:

.. literalinclude:: /examples/usage-of-variables/unset-cache/configure.log
  :emphasize-lines: 5

Since ``option`` modifies cache, the same logic applies here:

.. literalinclude:: /examples/usage-of-variables/unset-cache/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 18, 21, 14-15

.. literalinclude:: /examples/usage-of-variables/unset-cache/configure.log
  :emphasize-lines: 6-8

.. _cache name recommendation:

Recommendation
==============

Because of the global nature of cache variables and options
(well it's cache too) you should prefix it with the name of the project to
avoid clashing in case several projects are mixed together by
``add_subdirectory``:

.. literalinclude:: /examples/usage-of-variables/grouped-options/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6-7

.. literalinclude:: /examples/usage-of-variables/grouped-options/foo/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6-7

.. literalinclude:: /examples/usage-of-variables/grouped-options/boo/CMakeLists.txt
  :language: cmake
  :emphasize-lines: 6-7

.. seealso::

  * :ref:`Module names <module name recommendation>`
  * :ref:`Function names <function name recommendation>`

Besides the fact that both features can be set independently, now CMake-GUI also
groups them nicely:

.. image:: cache-gui/grouped.png
  :align: center

Summary
=======

* Use cache to set **global** variables
* Cache variables fits perfectly for expressing customized options: default
  value and respect user's value
* Type of cache variable helps CMake-GUI users
* Prefixes should be used to avoid clashing because of the global nature of
  cache variables

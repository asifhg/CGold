.. Copyright (c) 2016, Ruslan Baratov
.. All rights reserved.

CMake listfiles
---------------

There are several places where CMake code can live:

* In ``CMakeLists.txt`` listfiles loaded by the ``add_subdirectory`` command will help
  you create a source/binary tree. This is a skeleton of your project.

* ``*.cmake`` modules help you organize/reuse CMake code.

* CMake scripts can be executed via``cmake -P`` and help you solve problems
  in a cross-platform fashion without relying on system specific tools like bash,
  or without introducing external tool dependency like Python.

.. admonition:: Examples on GitHub

  * `Repository <https://github.com/cgold-examples/cmake-sources>`__
  * `Latest ZIP <https://github.com/cgold-examples/cmake-sources/archive/master.zip>`__

.. toctree::
  :glob:
  :maxdepth: 2

  cmake-sources/subdirectories
  cmake-sources/includes
  cmake-sources/common
  cmake-sources/scripts

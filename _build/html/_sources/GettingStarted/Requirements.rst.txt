.. _Requirements:

Requirements
=============

*13 May 2021*

This page describes the hardware and software required to build and run
the PLEXIL Executive and its supporting programs.

.. contents::


.. _development_environment:

Development environment
-----------------------

Any modern Unix-like system with the required development software
should be able to build and run the PLEXIL Executive and its associated
programs.

The source distribution requires about 200 MB of file system space to
build fully.

.. _operating_systems:

Operating systems
~~~~~~~~~~~~~~~~~

The development team regularly builds and maintains PLEXIL on the
following operating systems:

-  Linux (Red Hat Enterprise Linux, versions 5 and newer; Ubuntu
   releases since 12.04 LTS) on Intel x86-64 and AMD64 hardware
-  macOS versions 10.8 and newer, on x86-64 Macs

The PLEXIL Executive has been successfully built on these additional
platforms.

-  Windows Subsystem for Linux 2 (WSL 2); see the `detailed
   instructions <Media:Using_PLEXIL_with_WSL_2_Install_Instructions.pdf>`__
   (PDF).
-  `FreeBSD <https://www.freebsd.org/>`__
-  `Cygwin <http://cygwin.com/>`__ Unix-like development environment on
   Microsoft Windows.

The PLEXIL development team welcomes feedback and patches from users of
platforms other than Linux and macOS. While we make every effort to
assure portability, we may not be able to offer support for alternative
platforms.

.. _development_software:

Development software
~~~~~~~~~~~~~~~~~~~~

The PLEXIL Executive and the StandAlone Simulator are written in C++.
PLEXIL C and C++ code from the PLEXIL-4 releases should be buildable on
any C++03 and C99 compliant compiler, and should run unmodified in most
POSIX-compliant environments. A C++11 compliant compiler is required for
the forthcoming PLEXIL 6 release. The 'develop' branch requires a C++14
compiler.

The following compiler suites are actively used in development, and
known to work across multiple releases and platforms:

-  `gcc (GNU Compiler Collection) <https://gcc.gnu.org/>`__
-  `clang (LLVM project C/C++ compiler) <https://clang.llvm.org/>`__

There are no known issues on desktop systems using either GNU
libc/libcpp/libstdc++ or LLVM libc++, the standard libraries associated
with the compiler suites above.

Other development tools required by PLEXIL can be found in a typical
Linux or macOS development environment. The ones that aren't
preinstalled are often available through native package managers on
Linux distributions, `MacPorts <https://www.macports.org/>`__ or
`Homebrew <https://brew.sh/>`__ on macOS, or the ports systems on BSDs.

The following are required to build PLEXIL from the tarball releases:

-  POSIX-compliant 'tar' utility with support for gzip compression
-  POSIX-compliant shell (e.g. `GNU
   bash <https://www.gnu.org/software/bash/>`__,
   `Dash <https://wiki.archlinux.org/title/Dash>`__,
   `zsh <http://zsh.sourceforge.net/>`__,
   `Busybox <https://www.busybox.net/>`__, etc.)
-  Either `GNU Make <https://www.gnu.org/software/make/>`__ or
   `CMake <https://cmake.org/>`__

If you are building the Plexil Executive from a Git clone, you will also
need:

-  `GNU gperf <https://www.gnu.org/software/gperf/>`__ perfect hash
   utility
-  CMake, or
-  The GNU autotools:
   `autoconf <https://www.gnu.org/software/autoconf/>`__,
   `automake <https://www.gnu.org/software/automake/>`__, and
   `libtool <https://www.gnu.org/software/libtool/>`__

**Caution: The ``libtool`` utility provided with
Apple's**\ `Xcode <https://developer.apple.com/xcode/>`__\ **development
tools is not GNU libtool.** You must install GNU libtool if you are
planning to build PLEXIL from a Git clone.

The PLEXIL compilers, PLEXIL Viewer, and other development tools are
written in portable Java. A Java 8 or newer runtime is required. PLEXIL
is regularly tested with the following Java JDK environments:

-  `OpenJDK <https://jdk.java.net/>`__ - Oracle licensed
-  `AdoptOpenJDK <https://adoptopenjdk.net/>`__ - open source

The following software packages are required for building PLEXIL Java
software from source:

-  `Apache ant <https://ant.apache.org/>`__ version 1.7 or newer -
   requires the antlr add-ons to build the Plexilscript compiler
-  Java compiler for the chosen Java runtime

To build the RoboSim graphical robot simulator example, additional `X
Window System <https://www.x.org/wiki/>`__ and
`OpenGL <https://www.opengl.org/>`__ packages are required:

-  libXi (X11 Input Extension) and its development header files
-  libXmu and its development header files
-  `mesa <https://www.mesa3d.org/>`__ - open source implementation of
   OpenGL
-  libGLU (OpenGL Utilities library) and its development header files
-  `freeglut <https://freeglut.sourceforge.net/>`__ (OpenGL Utilities
   Toolkit) and its development header files

.. _runtime_environment:

Runtime environment
-------------------

Any environment capable of building the PLEXIL Executive should be able
to run it. In addition, the PLEXIL Executive has been successfully
ported to several embedded runtime environments.

Hardware
~~~~~~~~

The Executive code base is 64-bit clean. It is actively maintained on
Intel/AMD x86 architectures in both 32- and 64-bit modes. It has been
successfully demonstrated on 32-bit PowerPC and ARM platforms by the
development team. It should be portable to other 32- and 64-bit
architectures with minimal effort.

Hardware floating point is desirable, but not required, if a software
floating point library is available.

Runtime memory requirements vary by platform and application. The
Executive should run adequately on most systems with at least 16 MB of
RAM, including embedded platforms. Large PLEXIL plans may require more
RAM.

.. _operating_systems_1:

Operating systems
~~~~~~~~~~~~~~~~~

The PLEXIL Executive has been demonstrated on these embedded platforms:

-  `VxWorks <https://www.windriver.com/products/vxworks>`__ real time
   operating system on 32-bit PowerPC
-  Buildroot embedded Linux on 32-bit ARM

.. _required_libraries:

Required libraries
~~~~~~~~~~~~~~~~~~

The default C and C++ runtimes are GNU libc and libstdc++, or LLVM
libc++ when the Clang compiler is used.

The PLEXIL Executive has run on embedded Linux platforms using
`uClibc-ng <https://www.uclibc-ng.org/>`__, a lightweight open-source
libc implementation, and on the standard VxWorks C runtime, both with
minor modifications to the source.

The development team has no experience with reduced C++ Standard Library
implementations; we welcome user feedback about them.

Use of the following APIs is optional, and can be disabled at build
time:

-  POSIX threads (pthreads). The universalExec application requires
   threads, semaphores, and mutexes. Other thread APIs may be usable
   with some effort; e.g. the macOS build uses the native Mach
   semaphores. But the Executive core is entirely single-threaded, and
   an Executive application can be built without threads. The NASA AOS
   PLEXIL-cFS app (link to be supplied) uses the Executive core in this
   fashion.

-  The POSIX dynamic loading API. While useful for (e.g.) loading
   interface libraries during development, dynamic loading is not
   required.

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

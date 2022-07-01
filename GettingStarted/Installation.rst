.. _Installation:

Installation
=============

*13 May 2021*

This page describes how to get and install PLEXIL. Familiarity with a
Unix-like operating system (e.g. Linux, macOS) is assumed.

.. contents::

.. _getting_plexil:

Getting PLEXIL
--------------

There are two ways to get PLEXIL:

-  Download a compressed archive (*tarball*) of the sources and prebuilt
   Java jars.
-  Clone the source code from the ``git`` repository hosted here on
   Sourceforge.

In either case, the PLEXIL Executive must be compiled from source. The
tarball format includes prebuilt Java jars for the Standard PLEXIL and
Plexilscript compilers, the PLEXIL Viewer, and the PLEXIL static
checker; therefore it is the recommended distribution format for
exploring PLEXIL for the first time.

Tarball
~~~~~~~

The most recent tarball (compressed source archive file) is available
for download
`here <https://sourceforge.net/projects/plexil/files/latest/download>`__.
It is built with the GNU ``tar`` utility.

The archive may be expanded by the archive utility on your platform, or
at the command line:

``tar xzf plexil-4.5.0RC3.tar.gz``

This will expand to a directory named ``plexil-4.5.0RC3`` with all the
source and example files. This is the directory to use as
``PLEXIL_HOME`` in the procedures
`below <Installation#Building_Plexil>`__.

Note that as the released version number changes, the directory name in
the archive will change accordingly.

git
~~~

PLEXIL source code may be checked out from the git repository on
Sourceforge.

The following command will get the latest stable version of the source
code (currently the ``releases/plexil-4`` branch):

`` git clone -b releases/plexil-4 ``\ ```https://git.code.sf.net/p/plexil/git`` <https://git.code.sf.net/p/plexil/git>`__\ `` plexil``

This will create a directory named ``plexil`` with all the source files.

.. _subversion_svn:

Subversion (svn)
~~~~~~~~~~~~~~~~

The original Subversion repository still exists, but has not been
updated since the git repository was established. If you have been using
SVN, we recommend you switch to git.

.. _installing_plexil:

Installing PLEXIL
-----------------

The PLEXIL Executive and its examples are buildable with either `GNU
Make <https://www.gnu.org/software/make/>`__ or
`CMake <https://cmake.org/>`__. The C and C++ code complies with the
applicable language standards. See `Requirements <Requirements>`__ for
details.

The top level ``README`` has additional information on configuring,
building, and running PLEXIL.

.. _basic_installation:

Basic installation
~~~~~~~~~~~~~~~~~~

These instructions presume a POSIX-compliant shell and GNU make.

#. Unpack or clone the sources. See `#Tarball <#Tarball>`__ or
   `#git <#git>`__ above.

*NOTE: Plexil may not build correctly if installed in a directory that
is a symbolic link, such as ``/tmp`` in macOS.*

#. Change into the source directory.

#. Set up the environment. (This is not strictly necessary for the
   build, but will be for running programs.)

| `` PLEXIL_HOME="$PWD"``
| `` export PLEXIL_HOME``
| `` source "$PLEXIL_HOME/scripts/plexil-setup.sh"``

#. Build the desired components:

   #. For the PLEXIL Executive (universalExec), TestExec, compilers,
      checker, and PLEXIL Viewer: ``make tools``
   #. For all the above plus examples: ``make everything``

#. The PLEXIL Executive and related programs will be installed into
   $PLEXIL_HOME/bin ; libraries are installed into $PLEXIL_HOME/lib, and
   include files to $PLEXIL_HOME/include. There are additional scripts
   (e.g. plexilexec, plexilc) in $PLEXIL_HOME/scripts .

.. _advanced_installation:

Advanced installation
~~~~~~~~~~~~~~~~~~~~~

The PLEXIL Executive can be configured for a wide variety of
applications and environments using either the GNU autotools or CMake.
See the top level README file for an outline.

--------------

**Copyright (c) 2006-2021, Universities Space Research Association
(USRA). All rights reserved.**

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

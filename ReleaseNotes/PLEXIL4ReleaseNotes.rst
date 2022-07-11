.. _PLEXIL4ReleaseNotes:

PLEXIL-4 Release Notes
=======================

*11 June 2015*

|PLEXIL| 4 is a major release, with significant changes from previous
releases. Here are some of the highlights.

.. contents::

.. _building_plexil_4:

Building PLEXIL 4
-----------------

The |PLEXIL| C++ source code should be compatible with most modern and
recent C++ compilers. It is regularly tested against `gcc and
g++ <http://gcc.gnu.org/>`_ and `clang and
clang++ <http://clang.llvm.org/>`_ on both Linux and Mac OS X.

|PLEXIL| is now available for download as a gzip compressed ``tar``
archive file (or *tarball*). Prebuilt packages for Linux and Mac OS X
are no longer available due to lack of resources.

Building |PLEXIL| from an SVN checkout now requires the GNU "autotools"
suite: `libtool <https://www.gnu.org/software/libtool/>`_,
`automake <https://www.gnu.org/software/automake/>`_, and
`autoconf <https://www.gnu.org/software/autoconf/>`_. These programs
should not be required in the tarball release, as they have already been
run prior to archiving. Please see :ref:`Software Requirements <Installation>` for more
information.

It is again possible to build the |PLEXIL| suite from the top level
Makefile, whether downloaded as an archive or checked out from the
repository:

::

    cd plexil-4
    make

This builds most of the tools that users are likely to want, plus
several example applications.

.. _plexil_language:

PLEXIL Language
---------------

The language has had the following functions added.

.. _real_to_integer_conversion_functions:

Real-to-Integer Conversion Functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The |PLEXIL| language has added the following functions for converting
Real numbers to Integer values:

-  ``real_to_int(x)`` - Converts a Real value which is exactly an
   Integer to the corresponding Integer.
-  ``trunc(x)`` - Rounds the Real value towards 0.
-  ``round(x)`` - Rounds the Real value as defined in the C language
   standard.
-  ``ceil(x)`` - Returns the least positive Integer that is equal to or
   greater than x.
-  ``floor(x)`` - Returns the most positive Integer that is equal to or
   less than x.

When the resulting number would not fit into an Integer value, or for
``real_to_int`` a Real value is not exactly representable as an Integer,
``UNKNOWN`` is returned.

.. _strlen_function:

strlen Function
~~~~~~~~~~~~~~~

The ``strlen()`` function returns the length of a String value as an
Integer.

.. _array_size_functions:

Array Size Functions
~~~~~~~~~~~~~~~~~~~~

Array sizes can be determined with the following new functions:

-  ``arraySize(a)`` returns the current size of the array a as an
   Integer.
-  ``arrayMaxSize(a)`` returns the maximum (declared) size of the array
   a as an Integer.

.. _plexil_executive:

PLEXIL Executive
----------------

The |PLEXIL| Executive's performance has been significantly improved.
Major areas of improvement:

-  The plan reader's intermediate representation has been removed. XML
   is now translated directly into internal representations.
-  Expression representations were reengineered from the ground up. See
   :ref:`Expressions <expressions_release4>` below for more
   information.
-  The innermost routines of the Executive have been streamlined.
-  The ``Id<T>`` "smart pointer" template has been eliminated.
-  The executive core no longer uses any hash tables.

The result is much faster overall execution time - typically 2-5x faster
than |PLEXIL| 3 for medium sized plans - and a smaller but still
significant improvement in memory usage. Many memory leaks have also
been eliminated.

.. _expressions_release4:

Expressions
~~~~~~~~~~~

Previous versions of the Executive used double-precision floating point
representation for all values in the language. |PLEXIL| 4 replaces this
system with strongly typed ``Expression`` and ``Value`` objects.

As a result the |PLEXIL| Executive is now much stricter about expression
types:

-  Boolean is now a distinct type, incompatible with numeric types.
-  The Executive will report an error if a Real value is assigned to an
   Integer variable.

Since the |PLEXIL| language was originally designed to be a strongly typed
language, this should have no effect on conformant plans. However,
non-conforming plans which used to run in earlier releases due to lax
type checking by the Executive may no longer work as before.

.. _string_representation:

String Representation
~~~~~~~~~~~~~~~~~~~~~

String values were interned (stored in a symbol table) in previous
versions of the |PLEXIL| Executive. String values are now treated just as
any other value. Intermediate values in string expressions are now
discarded, rather than stored permanently. Plans which do string
manipulation will use less memory as a result.

.. _external_interfacing:

External Interfacing
~~~~~~~~~~~~~~~~~~~~

The APIs for interfacing to external systems have changed significantly.

This |PLEXIL| wiki now has much larger, and we hope much improved,
documentation on interfacing to the |PLEXIL| executive. See the sidebar
menu at left. The |PLEXIL| team eagerly seeks feedback on this
documentation.

The new ``Value`` class can represent any |PLEXIL| value for input or
output. See the :ref:`API Reference <APIReference>` for details.

.. _extended_plexil:

Extended PLEXIL
---------------

The Extended |PLEXIL| translator generates smaller and faster Core |PLEXIL|
XML for several common language features.

.. _standard_plexil_compiler:

Standard PLEXIL Compiler
------------------------

The Standard |PLEXIL| compiler has added an optimization phase. Some
moderately high-value optimizations have already been implemented. More
will be forthcoming.

See :ref:`PLEXIL Language <plexil_language>` above
for other changes.

.. _plexil_checker:

PLEXIL Checker
--------------

The |PLEXIL| Checker had not been updated in quite a while. It is now up
to date with the current |PLEXIL| schema, and many bugs have been fixed.
The Checker now performs a schema conformance check before any semantic
checks.

.. _plexil_viewer:

PLEXIL Viewer
-------------

The |PLEXIL| Viewer had also suffered from a lack of maintenance in
previous releases. Many bugs have been fixed in the Viewer, and
performance should be quite a bit better.

**The PLEXIL Viewer now requires Java 8.** Please let us know if you are
unable to update from earlier versions, so we can develop a workaround.

The Viewer now defaults to TCP port 49100. Previously it used a port
number in the ephemeral range. As before, users can specify a different
port number.

.. _improved_shell_scripts:

Improved Shell Scripts
----------------------

The ``plexil``, ``plexilexec``, ``plexiltest``, and ``plexilsim`` shell
scripts have been revised. Checking of TCP ports in use has been
significantly improved.

.. _regression_tests_and_examples:

Regression Tests and Examples
-----------------------------

Most, if not all, of the |PLEXIL| plans in the regression test suites and
the examples directory have been checked for conformance with the
current schema and fixed where necessary. Many of these plans had never
been updated since early in development.


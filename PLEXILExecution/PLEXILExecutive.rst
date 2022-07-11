.. _PLEXILExecutive:

PLEXIL Executive
====================

*11 May 2015*

.. contents::


Introduction
------------

The *PLEXIL Executive* (formerly called the Universal Executive, or UE),
is an implementation of the |PLEXIL| language. It is also a runtime
environment that provides or facilitates the following capabilities:

-  loading and execution of |PLEXIL| plans and libraries
-  commanding and querying state of external systems
-  notifications about execution status:

   -  to the :ref:`Plexil Viewer <PLEXILViewer>`
   -  to :ref:`external systems <InterfacingOverview>`

.. _plexil_application:

PLEXIL Application
~~~~~~~~~~~~~~~~~~

The |PLEXIL| Executive, often referred to simply as the "executive", is,
strictly speaking, a software library written in C++. To use the
executive, it must run as part of a *PLEXIL application*. |PLEXIL|
applications consist of an executive, one or more *interface adapters*,
and one or more external systems or simulators. External interfacing is
discussed at length elsewhere. This chapter describes how to operate the
executive itself within the context of a |PLEXIL| application; namely, how
to start the executive and specify its application configuration,
operating parameters, and textual debugging output.

If you are new to |PLEXIL|, we suggest you start with the simulators
described in :ref:`Simulating Plan Execution <PLEXILSimulator>`.

.. _running_the_executive:

Running the Executive
---------------------

There are two ways to start the |PLEXIL| Executive: the ``plexilexec``
script, and the ``universalExec`` executable.

.. _the_plexilexec_script:

The plexilexec script
~~~~~~~~~~~~~~~~~~~~~

The most flexible way to start the |PLEXIL| Executive is by running the
``plexilexec`` shell script. ``plexilexec`` can launch a graphical user
interface, run various checks on a plan file, and actually run the plan.

Let's assume you have a Plexil plan filed as ``foo.plx``. The most basic
invocation of the script at Unix command line is as follows. (The second
line is an abbreviated form of the first).

::

     plexilexec -plan foo.plx
     plexilexec -p foo.plx

An :ref:`interface configuration file <InterfaceConfigurationFile>` is
required to run the Plexil executive, and defaults to a "dummy"
configuration filed as ``plexil/examples/dummy-config.xml``. You can
specify an interface configuration file (call it ``my-config.xml``) as
follows.

::

     plexilexec -plan foo.plx -config my-config.xml
     plexilexec -p foo.plx -c my-config.xml

If your plan uses libraries (discussed in :ref:`Plexil Reference <PlexilReferences>`), they may be specified as well. In the
following example, assume two library files named ``lib1.plx`` and
``lib2.plx``.

::

     plexilexec -plan foo.plx -config my-config.xml -library lib1.plx -library lib2.plx
     plexilexec -p foo.plx -c my-config.xml -l lib1.plx -l lib2.plx

There are many more command line options available; type
``plexilexec -help`` to see a listing. We provide a few pointers here:

-  The ``-check`` (or ``-ch``) option runs Plexil's :ref:`static type checker <PlexilChecker>` on the plan prior to having it loaded.
-  The ``-debug`` (or ``-d``) option specifies a Debug Configuration
   file (see next section).
-  The ``-quiet`` (or ``-q``) option suppresses a leading printed
   summary and default debug messages during execution.

The :ref:`PLEXIL Viewer <PLEXILViewer>` chapter describes
additional options that launch the plan viewer, which allows monitoring
of plan execution down to the microstep level.

.. _the_universalexec_executable:

The universalExec executable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``universalExec`` executable is the actual |PLEXIL| Executive program.
Unlike the ``plexilexec`` script, all it does is run plans.

``universalExec`` accepts the following command line options:

-  ``-c``\ *``config_file``* - interface configuration file; if not
   supplied, a basic configuration is auto-generated
-  ``-d``\ *``debug_file``* - debug configuration file; default is
   Debug.cfg in the current directory
-  ``-l``\ *``library_name_or_file``* - library to preload; this option
   can appear multiple times
-  ``-L``\ *``library_directory``* - directory to search for libraries;
   this option can appear multiple times
-  ``-p``\ *``plan_file``* - the plan file to run
-  ``-v`` - connect to a |PLEXIL| Viewer; suboptions:

   -  ``-b`` - allow Viewer to block execution (e.g. for breakpoints);
      default is non-blocking
   -  ``-h``\ *``hostname_or_ip_addr``* - the host running the Viewer;
      default is localhost
   -  ``-n``\ *``port_number``* - the TCP port number of the Viewer;
      default is 49100

Using the examples above, typical invocations might look like:

::

     universalExec -p foo.plx -c my-config.xml
     universalExec -p foo.plx -c my-config.xml -l lib1.plx -l lib2.plx

.. _output_and_the_debug_configuration_file:

Output and the Debug Configuration File
---------------------------------------

The |PLEXIL| Executive can generate optional text output. This output can
be used to debug a plan or simply study its behavior. To have such
output printed as the executive runs, a *debug configuration* file (or
*debug file* for short) is needed. This file must exist in the directory
from which the executive is started (which is not necessarily the same
directory as that of the plan). By default, a file named ``Debug.cfg``,
if it exists, is used. A debug file with another name can be specified
using the ``-debug`` or ``-d`` command line option to ``plexilexec``
(described in previous section), or the similar commands ``plexiltest``
and ``plexilsim`` described in the :ref:`simulator chapter <PLEXILSimulator>`.

The debug file is a text file. Lines should start with either a comment
character ('#') or a colon, after which should be the *tag*. A tag is an
ad-hoc string, consisting of words separated by colons, with no spaces.
Debugging messages encoded in the executive that match any of the tags
in the file will be printed to the standard output (i.e. the terminal in
which the executive runs) as the plan is executed.

The following are some useful entries to place in this file.

::

    :Node:outcome

This prints the final outcome of every node.

::

    :Node:clock

This prints the start and end times (along with outcome) of every node.

::

    :Node:transition

This prints the state transitions of every node. It can be quite
verbose.

Unfortunately, the debug file possibilities are not well documented. An
incomplete, though fairly current, listing of possible debug tags is
found in the file ``plexil/doc/CompleteDebugFlags.cfg``. A somewhat less
complete listing of debug tag explanations is found in
``plexil/doc/DebugFlagDefinitions.txt``.

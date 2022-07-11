.. _InterfacingOverview:

Interfacing Overview
=======================

*27 April 2015*

This chapter provides a high-level overview of how to use and extend the
external interfaces of the :ref:`PLEXIL Executive <PLEXILExecutive>`.

.. contents::

Overview
--------

The PLEXIL Application Framework provides a flexible mechanism for
adding external interfaces to the :ref:`PLEXIL Executive <PLEXILExecutive>`,
and to applications built on the PLEXIL
Application Framework.

.. _plexil_executive_facilities:

PLEXIL Executive facilities
---------------------------

These artifacts are required to enable interaction between the :ref:`PLEXIL Executive <PLEXILExecutive>`
and an external system or environment:

#. One or more shared library files which implement interface adapters
   and/or exec listeners (C++ code).
#. An :ref:`interface configuration file <InterfaceConfigurationFile>` describing
   the adapters and listeners to be used.

.. _standard_interfaces:

Standard Interfaces
~~~~~~~~~~~~~~~~~~~

The PLEXIL distribution comes with several standard interface adapters
and exec listeners; see :ref:`Standard Libraries <StandardLibraries>` for a summary.

.. _configuration_file:

Configuration File
~~~~~~~~~~~~~~~~~~

See :ref:`Interface Configuration File <InterfaceConfigurationFile>` for
details.

.. _custom_interfaces:

Custom Interfaces
-----------------

For an overview of developing your own custom adapters and listeners,
please see :ref:`Implementing Custom Interfaces <ImplementingCustomInterfaces>`.

.. _writing_your_own_application:

Writing Your Own Application
----------------------------

In some applications, the user may prefer an executable which has all
interfaces pre-loaded. In other situations, the operating system and/or
build toolchain may not support dynamic loading of interface libraries.
Or maybe the application requires that the executive run under the
control of a real-time OS, in a particular threading enviroment.

In situations like these, it is straightforward to build a dedicated C++
executable which does exactly what the application requires. The process
is outlined in :ref:`Implementing Custom PLEXIL Applications <ImplementingCustomApplications>`.



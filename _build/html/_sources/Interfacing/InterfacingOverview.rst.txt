.. _InterfacingOverview:

Interfacing Overview
=======================

*27 April 2015*

This chapter provides a high-level overview of how to use and extend the
external interfaces of the `PLEXIL Executive <PLEXIL_Executive>`__.

.. contents::

Overview
--------

The PLEXIL Application Framework provides a flexible mechanism for
adding external interfaces to the `PLEXIL
Executive <PLEXIL_Executive>`__, and to applications built on the PLEXIL
Application Framework.

.. _plexil_executive_facilities:

PLEXIL Executive facilities
---------------------------

These artifacts are required to enable interaction between the `PLEXIL
Executive <PLEXIL_Executive>`__ and an external system or environment:

#. One or more shared library files which implement interface adapters
   and/or exec listeners (C++ code).
#. An `interface configuration file <#Configuration_File>`__ describing
   the adapters and listeners to be used.

.. _standard_interfaces:

Standard Interfaces
~~~~~~~~~~~~~~~~~~~

The PLEXIL distribution comes with several standard interface adapters
and exec listeners; see `Standard
Libraries <Standard_Interface_Libraries>`__ for a summary.

.. _configuration_file:

Configuration File
~~~~~~~~~~~~~~~~~~

See `Interface Configuration File <Interface_Configuration_File>`__ for
details.

.. _custom_interfaces:

Custom Interfaces
-----------------

For an overview of developing your own custom adapters and listeners,
please see `Implementing Custom
Interfaces <Implementing_PLEXIL_Interfaces>`__.

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
is outlined in `Implementing Custom PLEXIL
Applications <Implementing_Custom_PLEXIL_Applications>`__.

--------------

Copyright (c) 2006-2015, Universities Space Research Association (USRA).
All rights reserved.

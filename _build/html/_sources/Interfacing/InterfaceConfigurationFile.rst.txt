.. _InterfaceConfigurationFile:

Interface Configuration File
===============================

*15 May 2015*

The *interface configuration file* tells the `PLEXIL
Executive <PLEXIL_Executive>`__ how to route commands, lookups, and
planner updates to the appropriate interface adapters and exec
listeners.

.. contents::

.. _interface_configuration_format:

Interface Configuration Format
------------------------------

The *interface configuration file* is an XML file supplied to the
`PLEXIL Executive <PLEXIL_Executive>`__ that describes all adapters and
listeners available to the Executive, and (optionally) the mapping
between command and lookup names, and the adapters which implement those
commands and lookups.

This file always has the ``Interfaces`` element as its root. The
``Interfaces`` element body may contain any of these optional elements:

-  ``Adapter`` - describes an interface adapter, an object that
   implements commands, lookups, and/or planner updates;
-  ``Listener`` - describes an exec listener, an object that reports on
   node state transitions in the plan, and other execution events.

``Adapter`` and ``Listener`` elements may appear in any order, any
number of times, inside the ``Interfaces`` element. In general, at least
one ``Adapter`` should be supplied for any plan that interacts with its
environment in any way.

The outline of a typical interface configuration file is:

.. code-block:: html 

    <Interfaces>
     <Adapter AdapterType="adapter_type" ... more attributes as required ... >
      ... configuration data for adapter_type required ...
     </Adapter
      ... more Adapter elements ...
     <Listener ListenerType="listener_type" ... more attributes as required ... >
      <Filter FilterType="filter_type" ... more attributes as required ... >
       ... configuration data for filter_type as required ...
      </Filter>
       ... configuration data for listener_type as required ...
     </Listener>
      ... more Listener elements ...
    </Interfaces>
    
    
.. _adapter_format:

Adapter Format
~~~~~~~~~~~~~~

An ``Adapter`` element specifies what kind of adapter will be
constructed. There is one required attribute, ``AdapterType``.
``Adapter`` elements may also take the following optional attribute:

-  ``LibPath`` - names the shared library file containing the adapter
   implementation. The library file name defaults to
   lib\ *AdapterType*.so on Linux and lib\ *AdapterType*.dylib on Mac OS
   X.

All ``Adapter`` elements may contain one or more of the following
optional elements, which specify how the Executive may use them:

-  ``DefaultAdapter`` - signifies that this adapter should be used for
   all interfaces not registered to another adapter.
-  ``DefaultCommandAdapter`` - signifies that this adapter should be
   used for all commands not registered to another adapter.
-  ``DefaultLookupAdapter`` - signifies that this adapter should be used
   for all lookups not registered to another adapter.
-  ``PlannerUpdate`` - signifies that this adapter should be used for
   planner updates.
-  ``CommandNames`` - contains a comma-separated list of specific
   command names for which this adapter will be used.
-  ``LookupNames`` - contains a comma-separated list of specific state
   names for which this adapter will be used. Optionally takes an
   attribute:

   -  ``TelemetryOnly`` - boolean-valued attribute (default ``false``).
      If ``true``, LookupNow will take its value for these states from
      the most recently received value, instead of attempting a live
      query.

Note that some adapter types automatically register themselves for the
appropriate functions.

``Adapter`` elements may also contain additional attributes and elements
as required by the particular type of adapter.

Note: If a command or lookup is executed for which no adapter is
assigned, the Executive will halt with a run-time exception.

.. _listener_format:

Listener Format
~~~~~~~~~~~~~~~

The ``Listener`` element has one required attribute, ``ListenerType``.
This specifies what kind of exec listener will be constructed.
``Listener`` elements may contain the following optional elements:

-  ``Filter`` - specifies an event filter for the listener. This element
   has one required attribute, ``FilterType``.
-  ``LibPath`` - names the shared library file containing the listener
   implementation. The library file name defaults to
   lib\ *ListenerType*.so on Linux and lib\ *ListenerType*.dylib on Mac
   OS X.

``Listener`` and ``Filter`` elements may contain additional attributes
and elements, as required by the particular listener or filter type.

Example
~~~~~~~

Here is an example of a simple interface configuration file. Several
others can be found in the examples directory of the PLEXIL
distribution.

.. code-block:: html 

    <Interfaces>
     <Adapter AdapterType="Dummy">
      <DefaultAdapter />
     </Adapter>
     <Adapter AdapterType="OSNativeTime" />
     <Listener ListenerType="LuvListener" />
     <Listener ListenerType="PlanDebugListener" />
    </Interfaces>

--------------

Copyright (c) 2006-2015, Universities Space Research Association (USRA).
All rights reserved.

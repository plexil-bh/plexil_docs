.. _StandardLibraries:

Standard Libraries
=====================

*13 May 2015*

These are the interface libraries which are part of the |PLEXIL|
distribution.

.. contents::

.. _predefined_adapters:

Predefined Adapters
-------------------

There are five adapters currently included with |PLEXIL|. These are the
Dummy, OSNativeTime, Utility, IpcAdapter, and UdpAdapter adapters.

.. _dummy_adapter:

Dummy Adapter
~~~~~~~~~~~~~

The ``Dummy`` adapter is a stub for testing purposes. Lookups sent to
this adapter always return ``UNKNOWN``. Commands immediately receive an
acknowledgement value of ``COMMAND_SENT_TO_SYSTEM``; no command return
value is generated.

.. _time_adapter:

Time Adapter
~~~~~~~~~~~~

The ``OSNativeTime`` adapter uses the native operating system's
real-time clock facility for lookups of the ``time`` state. Both
``LookupNow`` and ``LookupOnChange`` are supported.

.. important::

    LookupOnChange of ``time`` requires a non-zero
    tolerance. Omitting a tolerance, or explicitly providing a zero
    tolerance value, will result in a run-time exception.

.. _utility_adapter:

Utility Adapter
~~~~~~~~~~~~~~~

The ``Utility`` adapter provides the ``print`` and ``pprint`` commands,
as described in the :ref:`PLEXIL Reference <utility_commands>`. See the directory
``plexil/src/apps/sample`` for example usage of this adapter.

IpcAdapter
~~~~~~~~~~

The IpcAdapter supports collaboration between multiple |PLEXIL| Executives
and other systems, using the `IPC <http://www.cs.cmu.edu/~IPC/>`_
(Inter-Process Communication) package from Carnegie Mellon University to
implement publish-subscribe communications.

For detailed documentation, please see :ref:`Inter-Executive Communication <Inter-ExecutiveCommunication>`.

UdpAdapter
~~~~~~~~~~

The UdpAdapter supports collaboration between multiple |PLEXIL| Executives
and other systems, using the standard UDP Datagram protocol over
Internet. The UdpAdapter provides point-to-point communications.

For detailed documentation, please see :ref:`UDP Adapter <UDPAdapter>`.

.. _predefined_listeners:

Predefined Listeners
--------------------

There are two predefined Listeners available to all |PLEXIL| applications.

LuvListener
~~~~~~~~~~~

The ``LuvListener`` implements a TCP socket-based connection to the
|PLEXIL| Viewer. For detailed information, please see :ref:`PLEXIL Viewer <PLEXILViewer>`.

PlanDebugListener
~~~~~~~~~~~~~~~~~

The ``PlanDebugListener`` causes a debug message to be generated for the
``Node:clock`` :ref:`debug flag <output_and_the_debug_configuration_file>` every
time a node enters the ``EXECUTING`` or ``FINISHED`` state.

.. _predefined_listener_filters:

Predefined Listener Filters
---------------------------

One listener filter is provided to all |PLEXIL| applications.

.. _nodestate_filter:

NodeState Filter
~~~~~~~~~~~~~~~~

The ``NodeState`` filter determines in which node states the associated
listener will be active.

.. _interface_configuration_reference:

Interface Configuration Reference
---------------------------------

This documents the interface configuration information required to use
the above interface libraries. Please see :ref:`Interface Configuration File <InterfaceConfigurationFile>` 
for an overview of how to use
configuration data.

.. _configuring_adapters:

Configuring Adapters
~~~~~~~~~~~~~~~~~~~~

Adapter descriptions go in the ``Adapters`` element of the interface
configuration. E.g.

::

 <Interfaces>
  <Adapters>
   <Adapter AdapterType="type" ... other attributes ...>
    ... additional configuration info for this adapter ...
   </Adapter>
  </Adapters>
  ... listeners go here ... 
 </Interfaces>

.. _dummy_adapter_configuration:

Dummy Adapter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Dummy adapter has no parameters. However, it must be explicitly
registered for the commands and lookups it should handle.

A common case is for the Dummy adapter to handle all commands and
lookups not yet implemented by other adapters. The declaration for this
case would be:

::

 <Adapter AdapterType="Dummy">
  <DefaultAdapter />
 <Adapter />

To handle only specific commands and lookups, specify them in a
``CommandNames`` or ``LookupNames`` element inside the ``Adapter``
element.

.. _ipcadapter_configuration:

IpcAdapter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^

Please see :ref:`IPC Configuration Reference <configuration_reference>`.

.. _time_adapter_configuration:

Time Adapter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

The Time adapter requires no additional configuration information. It
automatically registers itself for lookups of ``time``.

::

    <Adapter AdapterType="OSNativeTime" />

.. _udpadapter_configuration:

UdpAdapter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^

**WORK IN PROGRESS**

The UdpAdapter takes the following parameters as attributes to the
``Adapter`` element:

-  ``default_local_port`` and ``default_peer_port`` are optional. If
   supplied, they should be valid UDP port numbers (range 1-65535).
-  ``default_peer`` is optional. It should be the name of a network host
   or an IP address. Default is ``localhost``.
-  ``debug`` is optional. If supplied it should be a Boolean value. If
   ``true``, enables debugging output; default is ``false``.

Messages are defined by describing them in the body of the ``Adapter``
element, with a ``Message`` element for each different message type.
``Message`` takes the following parameters as attributes:

-  ``name`` (required) is the name of the message type, and the
   corresponding command name.
-  ``peer`` is the optional name or IP address of a network host. If not
   supplied, it defaults to the value of the ``default_peer`` attribute
   above, or ``localhost`` if neither is supplied.
-  ``local_port`` and ``peer_port`` are optional. If supplied, they
   should be valid UDP port numbers (range 1-65535). If not supplied,
   they default to the value of ``default_local_port`` and
   ``default_peer_port`` respectively. 
   
   .. note:: 
   
    If neither a default port
    nor message port are supplied, the interface will crash with a
    runtime exception. One or the other must be supplied.

The format of a message is defined by ``Parameter`` elements inside the
``Message`` element. ``Parameter`` takes the following attributes:

-  ``desc`` is an optional descriptor (e.g. parameter name);
-  ``bytes`` (required) is the length in bytes of the parameter in the
   message;
-  ``type`` (required) is the type of the parameter, which must be one
   of the following:

   -  ``int`` for the |PLEXIL| ``Integer`` type;
   -  ``float`` for the ``Real`` type;
   -  ``bool`` for the ``Boolean`` type;
   -  ``string`` for the ``String`` type;
   -  ``int-array`` for arrays of ``Integer``;
   -  ``float-array`` for arrays of ``Real``;
   -  ``bool-array`` for arrays of ``Boolean``;
   -  ``string-array`` for arrays of ``String``;

-  ``elements`` is required for array types. It is the (fixed) size of
   the array;

The UdpAdapter automatically registers itself for the following
commands:

-  ``SendMessage``
-  ``ReceiveCommand``
-  ``GetParameter``
-  ``SendReturnValue``

.. _utility_adapter_configuration:

Utility Adapter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Utility adapter requires no additional configuration information. It
automatically registers itself for the commands ``print`` and
``pprint``.

::

    <Adapter AdapterType="Utility" />

.. _configuring_listeners_and_listener_filters:

Configuring Listeners and Listener Filters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Listener descriptions go in the ``Listeners`` element of the interface
configuration. E.g.

::

 <Interfaces>
  ... adapters go here ... 
  <Listeners>
   <Listener ListenerType="type" ... other attributes ...>
    <Filter FilterType="filter_type" ... other filter attributes ...>
     ... additional configuration info for this filter ...
    </Filter>
    ... additional configuration info for this listener ...
   </Listener>
  </Listeners>
 </Interfaces>

.. _luvlistener_1:

LuvListener
^^^^^^^^^^^

The ``LuvListener`` has several optional parameters:

-  ``Blocking`` is a Boolean value. If ``true``, the Executive waits for
   acknowledgement from the Plexil Viewer after every node transition,
   to allow single-stepping and breakpointing of plans. The default
   value is ``false``;
-  ``HostName`` specifies the viewer host. It can be either an IP
   address or a host name. The default is ``localhost``;
-  ``Port`` is a TCP port number. It defaults to 49100.

An example of a custom ``LuvListener`` configuration:

::

    <Listener ListenerType="LuvListener" Blocking="true" HostName="test_console" Port="13579" />

Note that as of |PLEXIL| 4, parameters supplied at the :ref:`PLEXIL Executive <PLEXILExecutive>` command line will override the
``LuvListener`` parameters in an interface configuration file.

.. _plandebuglistener_1:

PlanDebugListener
^^^^^^^^^^^^^^^^^

The PlanDebugListener requires no additional configuration information.

::

    <Listener ListenerType="PlanDebugListener" />

.. _nodestate_filter_1:

NodeState Filter
^^^^^^^^^^^^^^^^

The ``NodeState`` filter allows a listener to selectively report when a
node transitions to or from specific node states.

Selecting the node states is done via one of these optional elements.
One or the other, **but not both**, must be supplied.

-  ``States`` should be a list of node state names, separated by commas.
   The filter will permit the listener to report when a node transitions
   from or to the named states.
-  ``IgnoredStates`` should be a list of node state names, separated by
   commas. The filter will permit the listener to report when a node
   transitions from or to any state *other than* the named states.

Example:

::

 <ExecListener ListenerType="JoesListener">
  <Filter FilterType="NodeState">
   <States>EXECUTING, FAILING, FINISHING, ITERATION_ENDED</States>
  </Filter>
 </ExecListener>

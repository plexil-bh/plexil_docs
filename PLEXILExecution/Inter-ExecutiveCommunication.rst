.. _Inter-ExecutiveCommunication:

Inter-Executive Communication
===================================

*19 May 2015*

This chapter describes how multiple interacting |PLEXIL| executives can be
realized.

.. contents::

Overview
--------

In many cases, it is useful to have multiple |PLEXIL| executives work
together to execute a plan. For instance, in a distributed environment
with many sub-systems, it is more efficient to have |PLEXIL| plans and
executives on each sub-system than to have a single system containing
all plans, which may rely on the network to send out low-level commands.
By distributing plans and executives among sub-systems, the amount of
network traffic can be limited to high-level communication and the
system becomes more easily maintainable.

The |PLEXIL| executive can communicate with another executive via
messages, commands, and lookups. An executive can be set up to receive
messages, commands, and lookups via the interface configuration and the
extended plexil constructs described below.

The communication framework utilizes Carnegie Mellon University's
`Inter-Process Communication (IPC) <http://www.cs.cmu.edu/~IPC/>`_
package (See *plexil/third-party/ipc*, for more information and the
latest distribution of the IPC package).

.. _outgoing_communication:

Outgoing Communication
----------------------

All communication save command return values are performed through a
publish/subscribe model. That is, all communication that is sent to the
central communication server is sent to every other executive connected
to that server. Because of this, it is important that no two executives
expect to handle the same command or lookup, as these are, by
definition, intended for one and only one agent. Having two executives
handling the same command or lookup results in undefined behavior.

Messages are published to all other executives via the SendMessage
command, and commands and lookups are published via the normal methods
of communicating with an external system. This allows executives to
emulate external systems and act as simulators. These simulators can
then be swapped out for a real application or more complex simulator
with no change to the controlling plan. See the example
section for an example plan that communicates with another executive.

.. _incoming_communication:

Incoming Communication
----------------------

When a message or command is received from an external executive, the
action along with its parameters are stored in an internal queue. When
an 'OnCommand', or 'OnMessage' node transitions to ``EXECUTING``, it
will immediately process the oldest matching action in the internal
queue. If there are no matching actions in the internal queue, the
command waits until one arrives. When a single action has been
processed, the node completes.

Lookups are handled differently. The IpcAdapter holds a table of lookup
names and values. When a request for a lookup is received, the adapter
automatically returns the current value of that lookup.

When the parent node of an OnCommand or OnMessage node transitions from
``EXECUTING`` to any other state and the child node is in the
``EXECUTING`` state, the child node transitions to the ``FAILING``
state, canceling the request to handle new incoming actions.

.. _oncommand_and_onmessage:

OnCommand and OnMessage
~~~~~~~~~~~~~~~~~~~~~~~

For detailed semantics of the OnCommand and OnMessage Extended |PLEXIL|
constructs used for receiving commands and messages, see the :ref:`Plexil Reference <PlexilReferences>`.

Lookups
~~~~~~~

Lookups are defined in the :ref:`interface configuration <InterfaceConfigurationFile>` for the IpcAdapter, and
are updated via the command 'UpdateLookup'.

Lookup definitions are specified in the following format within the
``<Adapter>`` tag:

::

    <ExternalLookup>
      <Lookup name="lookup name" type="variable type" value="initial value"/>
    </ExternalLookup>

where:

-  **``lookup``\ ````\ ``name``** is the name of the lookup to implement
-  **``variable``\ ````\ ``type``** is a |PLEXIL| data type (e.g.
   ``Integer``, ``String``, etc.)
-  **``initial``\ ````\ ``value``** is the initial value of the lookup

The ``<Lookup>`` element requires all three attributes.

Multiple lookups can be specified one after another, inside the
``<ExternalLookup>`` element.

.. _example_lookup_configuration:

Example Lookup Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This interface configuration fragment declares an Integer Lookup ``x``
whose initial value is 1, and a String Lookup ``str`` whose initial
value is "Hello".

::

    <ExternalLookups>
      <Lookup name="x" type="Integer" value="1" />
      <Lookup name="str" type="String" value="Hello" />
    </ExternalLookups>

Setup
-----

To enable inter-exec communication, the IpcAdapter must be set up
correctly in the interface config file that is given to the |PLEXIL|
Executive.

As a prerequisite for any communication, the IPC Central process must be
running on any open port on any machine and the IpcAdapter must be
specified in the interface configuration. A sample interface
configuration fragment that ensures communication can be established,
but does not declare specific communication features, is as follows:

::

     <Interfaces>
       <Adapter AdapterType="IpcAdapter" Server="59.60.0.1" AllowDuplicateMessages="true" />
     </Interfaces>

This enables the IpcAdapter and connects it to the IPC Central process
being hosted at IP address 59.60.0.1.

Messaging
~~~~~~~~~

To enable messaging, use a ``<CommandNames>`` block to register the
commands "SendMessage" and "ReceiveMessage" with the IpcAdapter.

Commands
~~~~~~~~

To enable the handling of incoming commands, use a ``<CommandNames>``
element to register the commands "ReceiveMessage", "GetParameters", and
"SendReturnValues" with the IpcAdapter.

To enable the sending of commands, replace the ``<CommandNames>``
element with the element ``<DefaultCommandAdapter/>``. This registers
the IpcAdapter with every command not specified in a ``<CommandNames>``
block in any other adapter.

.. _lookups_1:

Lookups
~~~~~~~

To enable the handling of incoming lookups, specify each lookup to be
handled in an ``<ExternalLookups>`` block. Within this block, each
lookup to be handled is specified in a ``<Lookup>`` element.

Example
-------

Here is an example of two plans that communicate with each other. One
acts as a simulator, and the other acts as a high-level controlling
plan.

.. _controlling_plan:

Controlling Plan
~~~~~~~~~~~~~~~~

*to be supplied*

.. _simulator_plan:

Simulator Plan
~~~~~~~~~~~~~~

This plan implements a simulator that serves:

-  One integer lookup, ``x``
-  The command ``MoveRight(Integer dX)``, which increases X by dX
-  The message ``Quit``, which shuts down the simulator

::

    Command UpdateLookup(String, Integer);
    Command SendReturnValue(Integer);

    Interface:
     {
       //The integer lookup, x
       Integer x = 1;
       Boolean continue = true;
      Loop:
       Concurrence
       {
         // Repeat the loop until the flag is false.
         RepeatCondition continue;
         // One iteration ends when either of the child nodes finishes.
         EndCondition RecMoveRight.state == FINISHED || RecQuit.state == FINISHED;

         //The handler for the command MoveRight(Integer dX)
        RecMoveRight:
         OnCommand "MoveRight" ( Integer modX ) {
           Increment: x = x + modX;
           SetExt: UpdateLookup("x", x);
           RespondMoveRight: SendReturnValue(x);
         }
         //The handler for the message 'Quit'
        RecQuit:
         OnMessage "Quit" {
           Set: continue = false;
         }
       }
     }

Here is the interface configuration for this executive:

::

     <Interfaces>
       <Adapter AdapterType="OSNativeTime" />
       <Adapter AdapterType="IpcAdapter" Server="localhost" AllowDuplicateMessages="true">
         <ExternalLookups>
           <Lookup name="x" type="Integer" value="1" />
         </ExternalLookups>
       </Adapter>
     </Interfaces>

Limitations
-----------

Although not inherent in the system, the related problem of presence
guarantees still remains. While a normal simulator interface directly
connects to the simulator or is the simulator, a |PLEXIL| simulator
operates on a subscription model. Because of this, there is no built-in
way to check to ensure that one and only one simulator is going to
process commands that are being broadcast.

Also, if an exec needs to send commands and lookups to multiple
listening agents, there is no way to differentiate between the two. If
the lookup “wind_speed” exists on two listening agents, any lookup
“wind_speed” from the main plan will receive whichever response comes in
first, leaving the second one to sit in the message queue.

Both of these limitations can be overcome with good planning and
forethought, but the fact that they exist remains.

.. _configuration_reference:

Configuration reference
-----------------------

The IpcAdapter takes the following parameters as attributes in the
``Adapter`` element:

-  ``Server`` is the name or IP address of the IPC central server. It
   defaults to ``localhost``;
-  ``AllowDuplicateMessages`` is a Boolean value; ``true`` means
   multiple messages with the same name can be processed; ``false``
   means they are ignored. The default is ``false``;
-  ``TaskName`` is the identifier by which this executive will be known,
   for the purposes of sending and receiving IPC messages. The default
   is a randomly generated string.

The following elements may appear in the body of the ``Adapter``
element:

-  ``ExternalLookups`` is a list of ``Lookup`` elements describing the
   lookups that this executive will serve for itself and for other IPC
   clients. The ``Lookup`` element takes the following parameters as
   required attributes:

   -  ``name`` is the name of the lookup;
   -  ``type`` is the |PLEXIL| data type of the value returned, one of
      ``Boolean``, ``Integer``, ``Real``, or ``String``;
   -  ``value`` is the initial value of this lookup.

-  ``CommandNames`` is a comma-separated list of command names that will
   be handled by the IpcAdapter. The adapter's built-in command names
   are automatically registered;
-  ``LookupNames`` is a comma-separated list of lookup names that will
   be routed to external IPC servers by the IpcAdapter;
-  ``DefaultCommandInterface`` causes the IpcAdapter to perform all
   commands not explicitly handled by other interfaces;
-  ``DefaultLookupInterface`` causes the IpcAdapter to perform all
   lookups not explicitly handled by other interfaces;
-  ``DefaultAdapter`` causes the IpcAdapter to perform all commands,
   lookups, and planner updates not explicitly handled by other
   interfaces.

An example ``Adapter`` element for a hypothetical robot mobility
controller might look like this:

::

    <Adapter AdapterType="IpcAdapter" Server="RobotCentral" TaskName="MobilityController" AllowDuplicateMessages = "true">
     <ExternalLookups>
      <Lookup name="heading" type="Integer" value="0" />
      <Lookup name="speed" type="Real" value="0.0" />
      <Lookup name="odometer" type="Real" value="0.0" />
     </ExternalLookups>
     <LookupNames>power_status</LookupNames>
     <CommandNames>report_fault</LookupNames>
    </Adapter>


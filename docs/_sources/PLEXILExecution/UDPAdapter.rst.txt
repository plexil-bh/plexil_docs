.. _UDPAdapter:

UDP Adapter
===============

*20 May 2015*

.. contents::

.. _udp_adapter_users_guide:

UDP Adapter User's Guide
------------------------

The UDP Adapter gives access to UDP messages/commands in |PLEXIL| plans.

There is a working example which uses the UdpAdapter in
``plexil/examples/multi-exec/udp``. This example illustrates how to
configure and load the UdpAdapter (``udp.xml``) and how to use it in the
example plans (``test-recv.ple`` and ``test-send.ple``).

.. _working_example_code:

Working Example Code
~~~~~~~~~~~~~~~~~~~~

``plexil/examples/multi-exec/udp`` contains some useful UdpAdapter code.
This code illustrates both how to use the UdpAdapater, plus some useful
idioms for asynchronous communications in Plexil.

.. _udpadapter_demo:

UdpAdapter Demo
^^^^^^^^^^^^^^^

Here follows a brief description of how to run the UdpAdapter
demonstration.

::

    % cd $PLEXIL_HOME
    % make
    % cd examples/multi-exec/udp
    % make demo

This will make sure that the |PLEXIL| Executive and its supporting
programs and libraries are built, that the two test plans
(``test-recv.ple`` and ``test-send.ple``) are compiled, and will call
the ``run-agents`` script to run them.

The output produced by running this demo is intended to help the user
gain an understanding of both what Plexil is doing, and what the
UdpAdapter is doing. Time spent understanding this output will be well
spent.

This directory also demonstrates all of the pieces required to send and
receive UDP messages/commands via the UdpAdapter (for either single or
multi-agent systems):

#. One or more plans which use ``Command`` and ``OnCommand`` nodes to
   send and receive UDP messages/commands, in this case those plans are
   ``test-send.ple`` and ``test-recv.pde`` respectively.
#. One or more interface configuration files which tell the
   ``universalexec`` which adapters to use, in this case the UdpAdapter,
   and what messages/commands can be used in the plans. In this example,
   the configuration file is ``udp.xml``, and in this case, both plans
   use the same configuration file since they exchange only a small
   number of UDP messages/commands (``test_udp_msg``, ``ack_msg``, and
   ``quit_msg``).
#. Finally, there is an optional debugging output configuration file
   which the ``universalexec`` uses to control the its own internal
   debugging output. Note that this is separate from the debugging
   output produced by the plans, and that produced by the UdpAdapter.

Once all of these are in place, the plans can be run and their behavior
inspected. This can be done, as described above, via the ``demo``
makefile target, or by compiling and running the plans by hand. Note
that to run a multi-agent system like this, you will either have to use
more than one shell window or put one of the agents into the background.
This is all handled conveniently by the ``demo`` makefile target and by
the ``run-agents`` script in ``plexil/scripts``.

For example, using two shells, in the first shell:

::

    % cd plexil/examples/multi-exec/udp
    % plexilc test-recv.ple
    % plexilexec -p test-recv.plx -c udp.xml

and in the second shell:

::

    % cd plexil/examples/multi-exec/udp
    % plexilc test-send.ple
    % plexilexec -p test-send.plx -c udp.xml

The ``test-send.plx`` plan/agent will send a UDP message to the
``test-recv.plx`` plan/agent, and both will exit when done.

The same thing can also conveniently be achieved with the ``run-agents``
script (assuming you put ``plexil/scripts`` and ``plexil/bin`` in your
path):

::

    % cd plexil/examples/multi-exec/udp
    % run-agents test-recv -c udp.xml test-send -c udp.xml

.. _debugging_output:

Debugging Output
^^^^^^^^^^^^^^^^

The output produced by these demonstration plans is well worth the
effort it takes to understand what the system is doing. The amount of
debugging output can be affected in the usual way, via the ``Debug.cfg``
file, which has several useful keywords in it which can be commented in
to provide a fair amount of additional output. In addition, for the
purposes of demonstration, some of the very low level UDP machinery
debugging is turned on for the benefit of new users. To turn this low
level debugging off, change the ``debug="true"`` attribute of the ``<Adapter AdapterType="UdpAdapter"/>``
element in the adapter configuration ``udp.xml`` file to
``debug="false"``.

.. _asynchronous_communication_idioms:

Asynchronous Communication Idioms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The demonstration plans illustrate several useful asynchronous
communication idioms, including idioms for

#. open loop communications (``SendUdpCommand1``, ``SendQuitCommand``,
   ``HandleQuitMessage``)
#. closed loop communications (``SendUdpCommand2``, ``SendCommandAck``,
   ``HandleAck``)
#. timeouts (``WaitForTimeoutOrCommand``, ``HandleTimeout``
#. multiple messages in a single handler (``HandleCommand``)
#. successful execution of "planned" failures
   (``WaitForTimeoutOrCommand``)

These idioms are demonstrated by the following makefile targets in the
demo subdirectory:

::

    % cd plexil/examples/multi-exec/udp
    % make demo
    % make timeout1
    % make timeout2

The ``demo`` target demonstrates both open and closed loop idioms, and
multiple messages in a single handler. The ``timeout1`` and ``timeout2``
targets demonstrate timeouts and planned failures. Again, it is worth
reading and understanding the output of these demos and the plans they
arise from.

.. _udp_message_definition:

UDP Message Definition
~~~~~~~~~~~~~~~~~~~~~~

For any given plan, all of the UDP commands/messages that can be used
therein with the UdpAdapter must be defined in the Plexil XML
communication configuration file. For the UdpAdapter, this file
declares:

-  the adapter itself: ``<Adapter AdapterType="UdpAdapter"/>``, including

   -  the required adapter type: ``AdapterType="UdpAdapter"``
   -  one optional default remote peer:
      ``default_peer="remote.host.ip"``
   -  one optional default remote port: ``default_peer_port="8031"``
   -  one optional default local port: ``default_local_port="9876"``
   -  an optional "debug" flag to turn on some internal UdpAdapter
      debugging output: ``debug="true"``

-  each of the messages/commands of interest: ``<Message name="acfs_state"/>``, including

   -  a required name (which must match the name given in the plan):
      ``name="ack_msg"``
   -  an optional remote peer: ``peer="remote.host.ip"``
   -  an optional remote port: ``peer_port="8031"``
   -  an optional local port: ``local_port="9874"``

-  one or more optional message/command specifications, including

   -  a required type: ``type="<type>"`` attribute, where
      \ ``<type> := int | float | bool | string | int-array | float-arry | bool-array | string-array``
   -  a required element encoding length in bytes: ``bytes="<n>"``
      where \ ``:= 1 | 2 | 4 | <n>``\ , depending on the type (see
      :ref:`below <plexil_and_udp_data_types>`),
   -  for arrays, a required array length attribute:
      ``elements="<n>"``, which ``<n>`` is the number of elements in the
      array,
   -  an optional parameter description: ``desc="blah"``

For example:

::

 <Interfaces>
  <Adapter AdapterType="Utility"/>
  <Adapter AdapterType="OSNativeTime"/>
  <Adapter AdapterType="UdpAdapter" debug="true" default_local_port="9876" default_peer_port="9876">
    <Message name="test_udp_msg" local_port="8032" peer_port="8032">
      <Parameter type="string" bytes="3" desc="message id"/>
      <Parameter type="bool" bytes="1" desc="send ack flag"/>
      <Parameter type="int" bytes="4" desc="num_widgets"/>
      <Parameter type="float" bytes="4" desc="arg4"/>
      <Parameter type="int-array" elements="3" bytes="2" desc="16 bit ints"/>
      <Parameter type="float-array" elements="3" bytes="4" desc="32 bit floats"/>
      <Parameter type="bool-array" elements="3" bytes="1"/>
      <Parameter type="string-array" elements="3" bytes="3"/>
    </Message>
    <Message name="ack_msg" local_port="8033" peer_port="8033">
      <Parameter type="string" bytes="4"/>
    </Message>
    <Message name="quit_msg" local_port="8034" peer_port="8034">
      <Parameter type="string" bytes="4"/>
    </Message>
  </Adapter>
 </Interfaces>

For any message/command to be received by OnCommand, either the
``default_local_port`` or the ``local_port`` must be defined, with the
``local_port`` taking precedence. Similarly, for any message/command to
be sent, either the ``default_peer`` or ``peer`` and either the
``default_peer_port`` or ``peer_port`` must be defined, with the more
specific ``<Parameter/>`` setting taking precedence. As a last resort, if neither
``default_peer`` nor ``peer`` are defined, "localhost" will be used.

Unfortunately, there is at this time no way to represent this
information directly in the plan, which means that the plan can not
reason about which port to use or which message to send or receive. This
limitation must be kept in mind with writing plans.

Note that the there is no such thing as an "empty" UDP message. In UDP,
there is only content, which is to say that a if you set up a socket to
listen for zero bytes -- a "content free" message -- the listener will
simply succeed immediately, which is unlikely to be what you want. The
simplest convention to use for "content free" messages is simply to send
the name of the message as its one and only string parameter, e.g.,

::

    Command quit("quit");

and receive it in a similar way, e.g.,

::

    OnCommand quit(String msgName);

.. _plexil_and_udp_data_types:

Plexil and UDP Data Types
~~~~~~~~~~~~~~~~~~~~~~~~~

The mapping between internal Plexil types and external types is this:

+-------------+-----------------+-----------------+-----------------+
| Plexil name | C++ type        | UDP             | Length in Bytes |
+=============+=================+=================+=================+
| Real        | ``double`` (64  | 32 bit floats   | 4               |
|             | bits)           |                 |                 |
+-------------+-----------------+-----------------+-----------------+
| Integer     | ``int32_t`` (32 | 32, 16, or 8    | 4, 2, or 1      |
|             | bits)           | bit integers    |                 |
+-------------+-----------------+-----------------+-----------------+
| Boolean     | ``bool`` (8     | 32, 16, or 8    | 4, 2, or 1      |
|             | bits)           | bit booleans    |                 |
|             |                 | (integers)      |                 |
+-------------+-----------------+-----------------+-----------------+
| String      | ``std::string`` | char[n]         | n > 0           |
+-------------+-----------------+-----------------+-----------------+

Plexil's type system is a little lean. In particular, this means that
bitfields are a bit tricky, therefore we are not trying to support bit
fields of any sort, including single bit booleans. The smallest
"boolean" we can send or receive in Plexil is a 8 bit integer.

These are parsed and checked at configuration file read time by
``UdpAdapter::parseXmlMessageDefinitions``.

Note that strings are fixed size for transport, but that that fixed
length is in effect the maximum size of the string. At encoding time,
all of the characters given will be encoded up to the size in bytes
given in the configuration file. If fewer characters are specified in
the plan than the maximum number in the configuration file, only those
characters given will be put in the outgoing buffer. Since the outgoing
buffer is zeroed out before use, this means that the any remain unused
slots in the buffer will simply have ``NULL``\ s in them, making the
shorter-than-maximum length strings "C" strings. At decoding time, the
decoder will read characters out of the string until either it has read
the maximum number of characters declared in the configuration file, or
until it encounters a ``NULL``. Either way, the resulting string is what
is "received" by Plexil. This "feature" is demonstrated in the example
code in ``examples/multi-exec/udp``.

.. _using_udp_messagescommands_in_a_plexil_plan:

Using UDP Messages/Commands in a Plexil Plan
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The UdpAdapter uses the Plexil ``Command`` interface to send UDP
messages, and the ``OnCommand`` interface to set up a listener for
incoming UDP messages. This interface follows the example of the
IpcAdapter for usage.

Once a UDP message has been defined in the XML configuration file (see
:ref:`above <udp_message_definition>`), it can be used either to send or
to receive that message. In the following examples, we will be using a
simpler definition of ``test_udp_msg``, which is given here:

::

    <Interfaces>
     <Adapter AdapterType="Utility"/>
     <Adapter AdapterType="OSNativeTime"/>
     <Adapter AdapterType="UdpAdapter" debug="true" default_local_port="9876" default_peer_port="9876">
      <Message name="test_udp_msg" local_port="8032" peer_port="8032">
       <Parameter type="string" bytes="3" desc="message id"/>
       <Parameter type="bool" bytes="1" desc="send ack flag"/>
       <Parameter type="int" bytes="4" desc="num_widgets"/>
       <Parameter type="float" bytes="4" desc="arg4"/>
      </Message>
     </Adapter>
    </Interfaces>
    

.. _the_command_interface:

The ``Command`` Interface
^^^^^^^^^^^^^^^^^^^^^^^^^

The ``Command`` interface is used to send a UDP message. For example,
for the definition of ``test_udp_msg`` :ref:`immediately above <using_udp_messagescommands_in_a_plexil_plan>`, the
``test_udp_msg`` command can be called thus:

::

    Command test_udp_msg ("UDP", false, 3, 3.14159);

or, assuming that all of the appropriate variables have been defined and
populated, like this:

::

    Command test_udp_msg (arg1, arg2, arg3, arg4);

All of the usual Plexil language idioms and restrictions apply. For
example, using "-" instead of "_" in the message name will fail during
translation if you are using Standard |PLEXIL|.

.. _the_oncommand_interface:

The ``OnCommand`` Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``OnCommand`` interface works in a similar manner. For example,
given the message definition
:ref:`above <using_udp_messagescommands_in_a_plexil_plan>`, the handler
for a ``test_udp_msg`` command can be invoked thus:

::
    
    OnCommand test_udp_msg (String arg1, Boolean arg2, Integer arg3, Real arg4);

However, note that all of the arguments to the given command must be
declared locally as the argument to the ``OnCommand`` node itself, as in
the example above. Again, this restriction is related to the
PlexilParser version 0.4 and may not apply to later parsers. The
practical consequence of this restriction is that if any of these
received values are to be used outside of the ``OnCommand`` node, they
will have to be passed via ``Assignment`` nodes.

Note that at the present time (8/30/11), neither ``plexilc`` nor
``PlexilCompiler`` support arrays in the ``OnCommand`` interface.

.. _complete_send_and_receive_example_plans:

Complete Send and Receive Example Plans
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For convenience, complete working examples of sending and receiving
plans are given here. The ``print`` command in each is used for
debugging output and to make what the plans are doing easier to
comprehend. Note that the ``print`` command is defined in the Utility
adapter. Also note that the versions of the plan given here may not
match those in ``plexil/examples/multi-exec/udp`` -- those plans may
include additional types and test code.

Using the definition of ``test_udp_msg`` given
:ref:`above <using_udp_messagescommands_in_a_plexil_plan>`:

``test-recv.ple``

::

  test-recv:
  {
    OnCommand test_udp_msg (String arg1, Boolean arg2, Integer arg3, Real arg4)
    {
      Command print ("\ntest-recv: arg1==", arg1, ", arg2==", arg2, ", arg3==", arg3, ", arg4==", arg4, "\n\n");
    }
  }

``test-send.ple``

::

    test-send:
    {
      String arg1 = "UDP";
      Boolean arg2 = false;
      Integer arg3 = 3;
      Real arg4 = 3.14159;
      NodeList:
      SendSetup:
      {
         Command print ("\ntest-send: arg1==", arg1, ", arg2==", arg2, ", arg3==", arg3, ", arg4==", arg4, "\n\n");
      }
      SendCommand:
      {
         Command test_udp_msg (arg1, arg2, arg3, arg4); // Send the UDP message
      }
    }

There are similar, though more interesting demonstration plans in
``plexil/examples/multi-exec/udp``, which illustrate both this basic
functionality plus some other useful idioms for timing out, closed loop
communications, and so on.

.. _outstanding_issues:

Outstanding Issues
------------------

.. _invokeabort_timing_vs_multi_node_translation:

InvokeAbort Timing vs Multi-Node Translation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In testing, we find that if we send a multi-parameter message followed
immediately by another message which breaks the ``InvariantCondition``
of the receiver of the first message, that ``invokeAbort`` is being
called while the multi-node expansion of the receive while it is still
processing parameters (i.e., the ``GetParameter`` nodes). That is, since
the multi-node expansion of ``OnCommand`` is not and atomic action, race
conditions inherently exist therein. This causes a run-time error at the
moment (``invokeAbort`` requires that it only be called on
``ReceiveCommand``, which is not atomic), and it is not at all clear to
me now this should be handled. It may be that this is a fundamental
problem with the multi-node expansions which has simply never been
tested in the past, or it may be simply that the UdpAdapter isn't yet
sophisticated enough to handle this boundary condition. This needs
further thought and discussion, and really should be tested on the
IpcAdapter.

.. _fixed_length_udp_messages:

Fixed Length UDP Messages
~~~~~~~~~~~~~~~~~~~~~~~~~

For a start, all messages are fixed length. Among the features that one
might want to add are:

#. A wrapper UDP buffer which includes both some sort of data definition
   and the data itself, and, perhaps,
#. NULL terminated "C" strings.

In the mean time, arrays can be flattened to sequences of vectors and
strings can be given maximum length.

.. _one_udp_listener_per_incoming_message:

One UDP Listener per Incoming Message
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At present, each ``OnCommand`` node sets up a UDP listener on its own
thread which waits for a UDP message on the given port. This means that
only one listener can be active on any given port at any given time. As
mentioned elsewhere, since none of the host or port information can be
represented in the plan, this may mean some careful planning and
overlapping message definitions in the configuration file if more than
one listener is needed for a given message at a time.

There are at least two possible solutions, both of which should probably
be implemented eventually.

#. Host and ports should be representable in the plans, which would
   allow for a lot of flexibility for both sending and receiving
   messages, and
#. For situations where more than one incoming message is expected on a
   single port, it is possible to set up a single UDP listener loop
   which receives multiple messages on a single port and dispatches them
   appropriately.

At present, neither of these is designed or implemented.

.. _low_level_error_codes_vs_plan_nodes:

Low Level Error Codes vs Plan Nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At present, there is no good way to communicate low level errors up to
the plan level. For example, if a UDP message is declared to use port
8031 and that port is for some reason unavailable, the low level call to
``bind()`` will return -1, and will be unable to receive anything. At
present, this causes an ``assert`` to fail, which is rather heavy
handed, but maybe less obscure than returning bogus data after a warning
message.

A similar situation exists when a plan tries to send a quantity that
can't be represented in 32 bits. There is no way currently for the
UdpAdapter to notify the exec that something un-reasonable is happening.
At present, this limit checking is not yet implemented, but even when it
is, all that can be done for now is to fail an ``assert``.

.. _ipcadapter_vs_udpadapter:

IpcAdapter vs UdpAdapter
~~~~~~~~~~~~~~~~~~~~~~~~

Should one be able to use both at the same time? Unlikely to work for
some things, e.g., both use ReceiveCommand internally.

.. _oncommand_vs_the_xml_configuration_definitions:

OnCommand vs the XML Configuration Definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the planner is not careful to make the number (and type) of the
parameters defined in the config file for a message/command and command
as it is used in the plan match, it can only be caught in a very heavy
handed way at run time using ``assert``. We have taken this approach in
the hope that bad plans will be caught during development and fixed
prior to run time deployment. This is not ideal, but at least it does
fit into the general Plexil approach.

At run time, the ``OnCommand`` node starts with only a single argument
passed to ``executeCommand``, regardless of the number of parameters
defined in the config file. This is of course one of the consequences of
the Plexil policy of "one parameter at a time for commands" in Plexil
which is further enforced by the ``ep2cp`` process, which translates the
``OnCommand`` node specified by the planner into a sequence of low level
Plexil nodes, including one for each parameter (the ``GetParameter``
nodes), the ``SendReturnValue`` node, and so on. As a consequence of
this, if the planner has used too many parameters for a particular
``OnCommand``, it can only be discovered at run time and handled by and
assert. It is also possible to detect too few parameters; the unused
parameters simply go unused.


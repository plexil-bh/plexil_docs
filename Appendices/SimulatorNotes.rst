.. _SimulatorNotes:

Simulator Notes
===================

*10/5/10*

This chapter provides detailed notes on the Plexil Simulator. It is
directed at software developers rather than users.

.. contents::

Introduction
------------

Although the objective of the Plexil simulator is to program it to mimic
a real-life application, in the interest of customizability and ease of
use, it is a good idea to identify and separate out the aspects of the
architecture that are common to all applications. In particular, the
design of the simulator consists of two parts;

-  An application (or domain) independent core
-  An application (or domain) dependent aspect that needs to be tweaked
   on a per application basis such as the inter-process communication
   protocol setup, code that parses the message received by the
   simulator constructs the response messages.

The core part of the simulator architecture comprises of specific
implementations as well as base classes whose concrete form needs to be
provided by the user. Specifically, the core consists of an
implementation of a reader for the non-customizable part of the
simulation script (i.e., *line 1*), a manager that keeps track of the
commands to be simulated and their corresponding responses, a timing
service the serves the purpose of setting timers and handling wakeup
calls and a scheduler that coordinates the entire simulation activity.
The core also provides several base classes that are implemented by the
application dependent peripheral.

.. _application_independent_core:

Application Independent Core
----------------------------

.. _script_reader:

Script Reader
~~~~~~~~~~~~~

Reads and parses the mandatory *line 1* of the simulator script files
and passes on the optional *line 2* to a parser implemented by the user
for to extract the return value information that is specific to a
command or data pertaining to a specific state being posted as telemetry
data.

.. _response_message_manager:

Response Message Manager
~~~~~~~~~~~~~~~~~~~~~~~~

Manages the response that are to be posted for a command. In particular,
it is responsible for customizing the response for a specific occurrence
of a command since the simulator design allows the user to customize the
behavior per command occurrence.

.. _timing_service:

Timing Service
~~~~~~~~~~~~~~

Performs the job of setting timers and provides the necessary handlers
that will be called with the scheduled time expires. The design makes
use of the Posix interval timer.

Simulator
~~~~~~~~~

Coordinates all the steps involved in the simulation activity such as
reading script files, maintaining a list of pending tasks, requesting
and handling wakeup calls and calling the appropriate entity to send a
response.

.. _response_base:

Response Base
~~~~~~~~~~~~~

A base class that encapsulates the actual response object as well as the
time delay for that response. It declares a pure virtual function

::

   virtual ResponseMessage* createResponseMessage() = 0;

that is to be implemented by the user which creates the actual response
message.

.. _response_factory_base:

Response Factory Base
~~~~~~~~~~~~~~~~~~~~~

A base class that has a pure virtual function that invokes the method to
parse *line 2* in the simulation script.

::

   ResponseBase* ResponseFactory::parse(const std::string& cmdName,
                                        timeval tDelay, std::istringstream& inStr) = 0;

The virtual method takes in three arguments, which are the name of the
command whose response is being parsed, the time delay for the response,
and the entire contents of *line 2*. The method subsequently returns a
pointer to response object it creates.

.. _communication_relay_base:

Communication Relay Base
~~~~~~~~~~~~~~~~~~~~~~~~

A base class that serves as the conduit for communication between the
simulator core and the user provided communication interface. The base
class declares a pure virtual function

::

    virtual void sendResponse(const ResponseMessage* respMsg) = 0;

that is to be implemented by the user. The virtual function will be
invoked by the simulator core when a response needs to be sent and takes
as its argument a pointer the the response message. The implementation
provided by the user is then responsible for publishing the response.

.. _application_dependent_peripheral:

Application Dependent Peripheral
--------------------------------

.. _response_factory_implementation:

Response Factory Implementation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Provides a concrete implementation of the parser for *line 2* entries if
the simulation script. The user is expected to construct the appropriate
response for each of the command entries specified in the simulation
script including all specific occurrences of a command.

.. _response_implementation:

Response Implementation
~~~~~~~~~~~~~~~~~~~~~~~

Provides the implementation of the response that the response factory
implementation instantiates.

.. _communication_relay_implementation:

Communication Relay Implementation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Provides the concrete implementation of the communication relay base
class that was described in the simulator core.

.. _communication_mechanism:

Communication Mechanism
-----------------------

Since the "simulator" part of the Plexil Simulator is a process in its
own right, we need some sort of a inter-process communication mechanism
to send it commands and also receive responses. We have chosen the
`IPC <http://www.cs.cmu.edu/~ipc/>`__ package from Carnegie Mellon
University. IPC handles both the communication and data marshalling
aspects of inter-process communication making it straightforward for the
user to program the simulator to send and receive appropriate messages.
This package is found in ``plexil/third-party/ipc`` in the PLEXIL
distribution.

.. _steps_to_construct_a_simulator:

Steps to Construct a Simulator
------------------------------

Let us now look at in greater detail the steps involved in building a
simulator. The process of building a simulator for a specific
application consists of two steps, the first of which is to build the
simulator core library as described above. The core library as we saw
earlier contains all the application independent parts. The second step
is the responsibility of the user and it involves providing concrete
implementations of all the base classes provided in the core. The
concrete implementation capture information that are specific to the
scenario being simulated such as the structure of values returned in
response to issued commands and the specific communication mechanism and
protocol. An example of the second has been provided as part of the
Plexil Simulator distribution in

::

    plexil/src/apps/StandAloneSimulator/PlexilSimulator

.. _concrete_implementations:

Concrete Implementations
~~~~~~~~~~~~~~~~~~~~~~~~

The classes *PlexilSimResponseFactory* and *PlexilSimResponse* provide
responses. The former establishes the correspondence between the command
being simulated and the response that has to be created while the latter
specifies the detail structure of each of the responses.

The class *PlexilCommRelay* provides communication. The file
*PlexilSimulator.cc* provides the ``main()`` function and creates the
simulator.

.. _built_in_default_message_structures:

Built-in Default Message structures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to facilitate ease of use, a default command and telemetry data
structure has been implemented in the simulator. The user will not be
required to provide any specializations for parsing either the command
or the telemetry script files provided the values posted by them is an
array of *integer* or *real* values. The command and telemetry files
given as examples in the earlier sections will be parsed by the built in
response parser. If however, one or more of the values being returned on
response to a command or as a telemetry state is a string, the user will
have to provide a customized parser.

--------------

Copyright (c) 2006-2014, Universities Space Research Association (USRA).
All rights reserved.

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

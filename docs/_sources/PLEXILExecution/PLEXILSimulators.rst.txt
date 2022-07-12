.. _PLEXILSimulators:

PLEXIL Simulators
=====================

*10 Apr 2021*

.. contents::


Introduction
------------

In order to be useful, |PLEXIL| plans need an external world, or
simulation thereof, to operate within and upon. The |PLEXIL| Executive
executes |PLEXIL| plans, but it is the executive's :ref:`external interface <InterfacingOverview>` that connects the executive to
external systems, thus realizing a *PLEXIL application*.

The |PLEXIL| distribution provides several example applications. This
chapter describes two of them: two different simulators for modeling an
external world with which a |PLEXIL| plan can interact.

The *Test Executive* is a simple simulator which interleaves events
specified in a script file with nodes coded in a |PLEXIL| plan. The
*PLEXIL Simulator* is an application that uses a more powerful
simulation script and has its simulator running as a separate process
from the |PLEXIL| Executive, providing a more realistic representation of
an external system. Additionally, it is possible to use a |PLEXIL|
executive itself as a simulator for another executive; this topic is
covered in the chapter on :ref:`Inter-Executive Communication <Inter-ExecutiveCommunication>`.

.. _test_executive:

Test Executive
--------------

The Test Executive, also called TestExec, found in
``plexil/src/apps/TestExec``, is essentially a plan execution simulator.
Given a plan, and simulation script representing the behavior of the
external world or system, the plan is executed against this script.

More precisely, *the script drives the execution of the plan*. The
script has two parts, an initial state and an event list. First, the
initial state (external variable values) is read and the executive
advanced one :ref:`step <micro_steps_macro_steps_and_the_quiescence_cycle>`. Then, events are
read from the script one at a time, advancing the executive one
:ref:`step <micro_steps_macro_steps_and_the_quiescence_cycle>` after each event. Note that if
the script runs out while there are still nodes waiting to execute, the
plan will terminate in an unfinished state.

The most flexible way to start the Test Executive is with the
``plexiltest`` command. Let's assume you have a Plexil plan filed as
``foo.plx``. The most basic way to execute this plan is as follows. (The
second line is an abbreviated form of the first).

::

    plexiltest -plan foo.plx
    plexiltest -p foo.plx

A simulation script (described in sections below) is required to run the
Test Executive. When not specified on the command line (as above), a
default is automatically chosen. The first choice is a script file whose
name is the same as the plan's filename, but having a ``.psx``
extension, and filed either in the same directory or a ``scripts``
subdirectory. Otherwise, the default script used is the empty simulation
script, ``plexil/examples/empty.psx``. This script is appropriate for
Plexil plans that do not interact with an external world (i.e. contain
no lookups, commands, or updates).

You can specify a simulation script file (let's call it ``world.psx``)
as follows.

::

    plexiltest -plan foo.plx -script world.psx
    plexiltest -p foo.plx -s world.psx

If your plan uses libraries (discussed in :ref:`Plexil Reference <PlexilReferences>`), they may be specified as well. In the
following example, assume two library files named ``lib1.plx`` and
``lib2.plx``.

::

     plexiltest -plan foo.plx -script world.psx -library lib1.plx -library lib2.plx
     plexiltest -p foo.plx -s world.psx -l lib1.plx -l lib2.plx

There are many more command line options available; type
``plexiltest -help`` to see a listing. We provide a few pointers here.

-  The ``-check`` (or ``-ch``) option runs Plexil's :ref:`static type checker <PlexilChecker>` on the plan prior to having it loaded.
-  The ``-debug`` (or ``-d``) option specifies a Debug Configuration
   file (see next section).
-  The ``-quiet`` (or ``-q``) option suppresses a leading printed
   summary and default debug messages during execution.

The :ref:`Plexil viewer <PLEXILViewer>` chapter describes options
that bring up a graphical plan viewer, which can be very useful.

Examples
~~~~~~~~

Here are a few examples of running the Test Executive in the
'plexil/test/TestExec-regression-test' directory. Note that in the first
example, the simulation script is found automatically because of its
filename.

::

   % plexiltest -p plans/site-survey.plx

   Running executive from /Users/fred/plexil
     Plan:      plans/site-survey.plx
     Script:    scripts/site-survey.psx
     Libraries:
     PORT:

   [Node:transition]Transitioning 'SiteSurveyWithEOF' from INACTIVE to WAITING
   [Node:transition]Transitioning 'SiteSurveyWithEOF' from WAITING to EXECUTING
   [Node:transition]Transitioning 'SiteSurveyWrapper' from INACTIVE to WAITING
   [Node:transition]Transitioning 'SignalEndOfPlan' from INACTIVE to WAITING
   [Node:transition]Transitioning 'MonitorAbortSignal' from INACTIVE to WAITING
   [Node:transition]Transitioning 'SendAbortUpdate' from INACTIVE to WAITING


     [ rest of output omitted ]


   %plexiltest -p plans/library-call6.plx -l plans/library6.plx -s scripts/library-call6-script.psx

   Running executive from /Users/fred/plexil
     Plan:      plans/library-call6.plx
     Script:    scripts/library-call6-script.psx
     Libraries:  -l plans/library6.plx
     PORT:

   [Node:transition]Transitioning 'root' from INACTIVE to WAITING
   [Node:transition]Transitioning 'root' from WAITING to EXECUTING
   [Node:transition]Transitioning 'library6' from INACTIVE to WAITING
   [Node:transition]Transitioning 'library6' from WAITING to EXECUTING
   [Node:transition]Transitioning 'root' from EXECUTING to FINISHING
   [Node:transition]Transitioning 'library6' from EXECUTING to ITERATION_ENDED
   [Node:transition]Transitioning 'library6' from ITERATION_ENDED to FINISHED
   [Node:outcome]Outcome of 'library6' is SUCCESS
   [Node:transition]Transitioning 'root' from FINISHING to ITERATION_ENDED
   [Node:transition]Transitioning 'root' from ITERATION_ENDED to FINISHED
   [Node:outcome]Outcome of 'root' is SUCCESS

.. _test_executive_simulation_script:

Test Executive Simulation Script
--------------------------------

The Test Executive operates on a script that encodes an initial world
state and a sequence of state change events and responses to commands
and updates. Execution of a plan is interleaved with the processing of
events and responses in the script. The initial state and/or event
sequence may be empty.

If the plan operates on time, the passing of time is simulated by
encoding values (as real numbers) for the state variable ``time`` in the
script.

Processing of the simulation script proceeds in "lock step" with
execution of the plan. If the script contains responses for commands or
updates that haven't occurred in the plan, a runtime error will result.
Conversely, if the script fails to acknowledge a command or update that
is executed, or provide a state value for a lookup, the plan may
terminate prematurely or be left "hanging", waiting for this external
feedback.

.. _simulation_script_files:

Simulation Script files
~~~~~~~~~~~~~~~~~~~~~~~

Simulation scripts are written in files that should have a ``.pst``
extension. These files must be translated into their XML representation
before running the Test Executive. To translate a script, e.g.
``world.pst`` into its XML form, ``world.psx``, type:

::

    plexilc world.pst

This command will overwrite any existing version of ``world.psx``.

The syntax for Test Executive simulation scripts is described in the
following sections.

.. _example_simulation_scripts:

Example Simulation Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is the file ``plexil/examples/SimpleDrive.pst``, which is a
simulation script for the plan
``plexil/plexil/examples/SimpleDrive.ple``:

::

   initial-state {
     state At ("Rock" : string) = false : bool;
   }

   script {
     command-success drive (1.0 : real);
     state At ("Rock" : string) = true : bool;
     command-success takeSample ();
   }

This script specifies an initial state named *At* whose value is false.
Note that this is a *parameterized* state, with a single parameter
valued "Rock". The script performs these events:

#. It changes the *At* state to true (i.e. the rover has reached the
   rock).
#. It acknowledges the *drive* command with the ``COMMAND_SUCCESS``
   handle.
#. It acknowledges the *takeSample* command with the ``COMMAND_SUCCESS``
   handle.

See the following section for important information about scripting
command handles.

There are more examples of simulation scripts in the following
directories of the |PLEXIL| distribution:

::

    plexil/examples/
    plexil/src/apps/TestExec/test/scripts

.. _scripting_commmand_handles_and_return_values:

Scripting Commmand Handles and Return Values
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The example in the previous section illustrates the scripting of
:ref:`command handles <command_handles>`.

There is an important aspect of scripting command handles when the
command *also* returns a value. Namely, the handle must occur *after*
the value. Example:

::

   script {
     command         get-input () = "yes" : string;
     command-success get-input ();
   }

This ordering requirement is purely an artifact of implementation, and
it's a common coding error to have them reversed.

.. _test_executive_script_syntax:

Test Executive Script Syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is the syntax for Test Executive simulation scripts:

::

   element =
       script        { <element> ... }
     | initial-state { <element> ... }
     | simultaneous  { <element> ... }
     | update-ack <name> ;
     | function-call <name> (<value> : <type>, ...) = <value> : <type> ;
     | command       <name> (<value> : <type>, ...) = <value> : <type> ;
     | command-abort <name> (<value> : <type>, ...) = <value> : <type> ;
     | command-ack   <name> (<value> : <type>, ...) = <value> : <type> ;
     | command-accepted       <name> (<value> : <type>, ...);
     | command-denied         <name> (<value> : <type>, ...);
     | command-sent-to-system <name> (<value> : <type>, ...);
     | command-rcvd-by-system <name> (<value> : <type>, ...);
     | command-success        <name> (<value> : <type>, ...);
     | command-failed         <name> (<value> : <type>, ...);

   value = true | false | -100 | 100 | 100.0 | -100.0
         | "hello" | <unknown> | (<value>, ...)

   type = bool | int | real | string
        | bool-array | int-array | real-array | string-array

.. note::

    **`<unknown>`** as included in the kinds of values above, is a literal. It
    extends all |PLEXIL| types, and allows the scripting of a lookup or
    command to return the UNKNOWN value to |PLEXIL|. Recall that UNKNOWN has
    no literal representation in |PLEXIL| itself -- a value can only be tested
    using the **isKnown** expression. For examples of the scripting of
    UNKNOWN values, see:

::

   plexil/examples/basic/TestUnknown.ple
   plexil/examples/basic/scripts/TestUnknown.pst

.. _plexil_simulator:

Plexil Simulator
----------------

The Plexil Simulator is a |PLEXIL| application that uses a simple,
stateless, non-graphical simulator. (Formerly, this simulator was called
the Standalone Simulator, or SAS). The Plexil Simulator can be used for
testing Plexil plans and capabilities of the Plexil Executive. This
simulator's objective is to mimic the rudimentary behavior of real
applications at a low fidelity level involving commands and their
response, thereby eliminating the necessity to interface with complex
systems during development, testing and validation. The simulator
accepts commands as the real system normally would and responds in a
pre-programmed manner as defined by the user in a simulation script. The
simulator can also post telemetry data as specified in a script file.
Such an approach provides an excellent way to develop and test the
coverage of off-nominal behaviors of systems.

For example, if you are developing a Plexil-based controller that
interacts with the navigation and instruments on-board a rover, it may
not be possible and sometimes not desirable to interface with the actual
rover throughout the development, debugging and validation process. By
using the Plexil Simulator, the user can instead easily simulate the
desired behavior such as responding with *Success* or *Failure* values
after a time delay for various navigation and science tasks.

The remainder of this section is a guide for using the Plexil Simulator.
A description of its architecture is given in :ref:`Appendix D <SimulatorNotes>`.

See the ``plexil/src/apps/StandaloneSimulator/PlexilSimulator/test``
directory and its ``README`` file for a simple example usage of the
Plexil Simulator.

.. _capabilities_of_the_plexil_simulator:

Capabilities of the Plexil Simulator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Plexil Simulator provides the following capabilities;

-  Ability to respond to specific commands as well as post telemetry
   data. The required behavior of the simulator is specified using two
   script files, one for command/response and another for telemetry
   behavior.
-  *Command/Response behavior*: The simulation script contains the names
   of commands that need to be simulated, the time delay after which the
   corresponding response messages are posted, and the contents of the
   response itself.
-  Ability to customize the behavior of specific commands as well as the
   specific occurrence of a command. For example, it is not only
   possible to specify the behavior of a Move command in the context of
   a rover, but it is also possible to specify the behavior of the *Nth*
   Move command that the simulator receives.
-  Ability to control the delay between the time when the simulator
   receives a command and the time when the response gets sent out for
   that command. The time delay can be specified at a microsecond
   resolution.
-  Responses for multiple instances of a command that are slated to be
   posted at the same time will be sent in a first-in, first-out order.
   Note that there are two possible ways in which this situation can
   arise. First, if the simulator receives multiple instances of the
   same command back-to-back and they all have the same time delay.
   Second, if multiple instances of a command are received by the
   simulator at various times but their simulation time delay is such
   that more than one command response is scheduled to be sent out at
   the same time.
-  The exact response value data structure can be defined by the user.
   For example, for some commands the response could be just a single
   boolean value but in other could be a data structure consisting of a
   heterogeneous types including strings, integers, reals and booleans.
-  *Telemetry behavior*: The telemetry data to be posted along with the
   time when it has to be posted (relative to the start time of the
   simulator) is captured in the form of a second script.
-  The simulator design is not tied to a specific inter-process
   communication protocol. The architecture provides the necessary hooks
   that allows the user to pick the preferred data transport mechanism.

.. _running_the_plexil_simulator:

Running the Plexil Simulator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most flexible way to start the Plexil Simulator is the ``plexilsim``
command. Let's assume your Plexil plan is filed as ``foo.plx``, and
script as ``foo-script.txt``. The most basic way to execute this plan is
as follows. (The second line is an abbreviated form of the first).

::

    plexilsim -plan foo.plx -script foo-script.txt
    plexilsim -p foo.plx -s foo-script.txt

Note that unlike the Test Executive (described above), a simulation
script is required for the Plexil Simulator -- there is no default.

Many other useful command line options are available. Type
``plexilsim -help`` for a listing. For a description of the most useful
options, please see the section above on the :ref:`plexiltest script <test_executive>`, which shares the same
options.

.. _advanced_usage:

Advanced usage
^^^^^^^^^^^^^^

The approach above is sufficient to run the Plexil Simulator in the most
basic way: executing a single plan using a single simulator. It general
it is possible to have multiple executives interacting with multiple
simulators. For such configurations, the executives and (standalone)
simulators must be started separately. In addition, a third component,
the IPC communications router, which is automatically started by
``plexilsim``, must also be started manually. The procedure is as
follows.

1. In one shell, start IPC first:

::

    ipc

2. In additional shells (each running its own simulator) start the
simulator as a standalone component:

::

    run-sas <script>


3. Also in their own shells, start the Plexil Executive(s) last, using
the ``plexilexec`` script. See the :ref:`PLEXIL Executive <PLEXILExecutive>` chapter for instructions on this script.
You'll need an interface configuration file that specifies the IPC
Adapter for commands and lookups; see
``/Users/kdalal/plexil/src/apps/StandAloneSimulator/PlexilSimulator/test/config.xml``
for an example.

.. _plexil_simulator_script:

Plexil Simulator Script
-----------------------

The behavior of the Plexil Simulator is dictated by script files that
specify command responses and lookup values (another term for lookup
values is *telemetry*). Each of these specifications has its own syntax,
and a simulation script may contain zero or more sections (prefixed by a
keyword) for each kind of specification. More than one simulation script
can be used, and multiple script files are the equivalent of their
concatenation.

.. note::

    Currently, command responses and telemetry values are
    restricted to numbers (integers or real).

A Plexil Simulator script is a text file, completely distinct from the
Test Executive scripts described above. There is no requirement for the
file's name.

Command/Response
~~~~~~~~~~~~~~~~

A simulation script can itemizes the commands that need to be simulated.
This specification begins with the keyword

::

    BEGIN_COMMANDS

and is followed by entries having the following format.

::

   Line 1 (required): <command_name : string> <command_index : integer>
          <response_needed? : boolean, 0 or 1> <time_delay : real>
   Line 2 (required only if response_needed == 1): <response values>

All the entries in *Line 1* are mandatory while *Line 2* is required
only if the *response_needed?* field in *Line 1* is 1. *Line 1* is
parsed by the script reader implemented in the core software while *Line
2* is parsed by the user. Therefore, no restriction is placed on the
data type or the ordering of the elements in *Line 2* and it is entirely
up to the user to define the structure and provide a parser for it. The
significance of each of the fields in *line 1* is the following;

::

   <command_name>     : Name of the command that the simulator is expected
                        to respond to.  If the simulator receives a command
                        that does not match (case sensitive) with any of those
                        specified in the simulation script file, it will be ignored.
   <command_index>    : Allows the user to customize the response for a
                        specific occurrence of a command. The index value count is
                        one-based with the behavior corresponding to 0 being the
                        default that applies to all instances of commands that do not
                        have a specific behavior.
   <response_needed?> : Specifies if a return value(s) has to be posted by
                        the simulator for the command.
   <time_delay>       : The time delay after which the simulator needs to
                        respond to a command.

Consider a rover that accepts four types of navigation commands (MoveUp,
MoveRight, MoveDown, MoveLeft) and a command that queries some sensors
(QueryEnergySensor). Also, the navigational commands expect an integer
response value of -1, 0 or 1 while the query command expects an array of
five real values. The following example simulates a specific behavior of
the rover wherein all the navigational commands except for the second
occurrence of the MoveDown command will return a value of 1. The second
occurrence of the MoveDown command will return a value of 0. All the
QueryEnergySensor commands will return an array of five real values 1.1,
1.2, 1.3, 1.4 and 1.5. The responses for all commands will be posted
after 2.0 seconds.

::

   MoveUp 0 1 2.0
   1

   MoveRight 0 1 2.0
   1

   MoveDown 0 1 2.0
   1

   MoveLeft 0 1 2.0
   1

   QueryEnergySensor 0 1 2.0
   1.1 2.2 3.3 4.4 5.5

   MoveDown 2 1 2.0
   0

Telemetry
~~~~~~~~~

In addition to responding to specific commands, the simulator can also
post telemetry data at predefined time instances. This specification
begins with the keyword

::

    BEGIN_TELEMETRY

which is followed by entries having the following format.

::

   Line 1: <telemetry_data_name: string> <time : real>
   Line 2: <telemetry data>

   where

   <telemetry_data_name> : Name of the state whose value is being posted as telemetry data.
   <time>                : The time when the telemetry data has to be
                           posted. This time is computed relative to the
                           start of the simulator.

   <telemetry_data>      : The actual data that needs to be posted.

Consider the following example where the state (*RobotState*) of the
rover position (*X, Y and Z*) is being posted at instances 3.0 to 7.0
seconds computed with respect to the start of the simulator.

::

   RobotState 3.0
   100.1 100.2 100.3

   RobotState 4.0
   200.1 200.2 200.3

   RobotState 5.0
   300.1 300.2 300.3

   RobotState 6.0
   400.1 400.2 400.3

   RobotState 7.0
   500.1 500.2 500.3


.. _ResourceModel:

Resource Model
==================

*16 Aug 2021*

This chapter describes the resource model of PLEXIL and its associated
language constructs. See the `Resource Arbiter <Resource_Arbiter>`__
chapter for execution-related aspects of this model.

.. contents::

Introduction
------------

**NOTE: A number of serious problems have been identified in the
resource model described herein. This section should be considered
deprecated; it will be superseded in some future release.**

Currently applicable only to *command* nodes, the PLEXIL resource model
allows resource requirements for commands to be specified in the plan,
and provides a mechanism to check and enforce these requirements during
plan execution.

Resources are entities of limited availability and can be classified
into several categories such as *consumable*, *renewable*, *unary*,
*non-unary*, etc. Consumable resources are those wherein a pool of a
predetermined size is available for use and each command grabs whatever
it requires from this pool and may or may not release it upon
completion. Renewable resources are types that can be *generated* and
once again as in the case of consumables, commands that generate
resource may or may not have their action persist after their
completion. The simplest kind of consumable and renewable resources are
unary, where a resource is allocated in its entirety to the command that
needs it and released (once again in its entirety) upon completion.
After a unary resource is grabbed by a command, no other command that
requires the resource can be accepted by the external system.

Consider the following simple example of a strictly consumable unary
resource. Let's say we wish to perform two activities, reading from and
writing to a specific block in memory. In order to eliminate any
possibility of data corruption the current activity (reading or writing)
should lock the entire resource (the specific memory block) during the
course of its execution.

A good example of a non-unary consumable is the memory stack. Multiple
commands (or function routines) can make use of the stack as long as
their cumulative sum does not exceed the maximum limit and when the
command is completed it relinquishes all the space it had grabbed
thereby freeing it up for the next command.

Storage memory is a good example of a non-unary resource that can be
either consumed or renewed and commands that make use of it can have
their effect persist after completion.

Approach
--------

Resource handling in PLEXIL has three aspects.

#. Specification of resource requirements in the PLEXIL plan. This
   includes all the necessary constructs needed to specify the resources
   that a command needs. Since a command might need more than one
   resource, we should be able to specify a list of resources.
#. A *resource arbiter* that decides which commands can be issued to the
   external systems and which ones should be rejected based on resource
   requirements and availability.
#. A mechanism using *command handles* that registers the decisions made
   by the resource arbiter and keeps track of the lifecycle of commands
   after they leave the Plexil Executive.

.. _resource_specification:

Resource Specification
~~~~~~~~~~~~~~~~~~~~~~

The resource requirements for a command are specified in a PLEXIL plan
in the following way.

::

   Resource
     // required
     Name = <String expr>,
     Priority = <Integer expr> // smallest value = first priority
     // optional
     , LowerBound = <Real expr> // default=1.0
     , UpperBound = <Real expr> // default=1.0
     , ReleaseAtTermination = <Boolean expr>  // default = true
     ; 

For example:

::

   Command c1();

   C1:
   {
     Resource Name = "left_arm", Priority = 10;
     Resource Name = "right_arm", Priority = 10;
     Resource Name = "memory",
       LowerBound = 10.0,
       UpperBound = 20.0,
       ReleaseAtTermination = false,
       Priority = 10;

     c1();
   }

This example states that the command ``c1()`` requires exclusive use of
the left arm and the right arm (unary resources) and memory within the
lower and upper bounds [10.0, 20.0]. The priority value will be used
during resource arbitration in case of resource contention within the
same execution cycle. Lower (numerical) priority commands will not be
preempted if contention occurs between two different execution cycles.

A resource without bounds is construed to be unary. In the case of
resources that are not unary, the lower and upper bound values need to
be passed by the Plexil Executive to the resource arbiter.

The field ResourceReleaseAtTermination specifies whether the command
releases the resource upon its completion. In the example given above
the command will not release the ``memory`` resource when it terminates.

Several sample plans are given in the Examples section below.

.. _resource_arbitration:

Resource Arbitration
~~~~~~~~~~~~~~~~~~~~

Commands are currently issued only at the end of
`quiescence <Detailed_Semantics#Macro_Steps_and_Quiescence>`__. All the
commands identified for execution at the end of the quiescence cycle
will be sent to the *resource arbiter* instead of the sub-system (i.e
external world)) directly. The arbiter will consider each of the
commands in the batch it receives and pick a subset that can be sent to
the sub-system based on resource availability. The commands accepted by
the arbiter will be forwarded to the external sub system by the external
interface while the ones that are rejected will be acknowledged as so by
the external interface by setting the appropriate value in the command
handle (enumerated in the next section). Note that the PLEXIL plan can
specify the next course of action in case a command gets rejected, for
example, the command could be reissued, etc.

The resource arbiter is explained in further detail in `Chapter
10 <Resource_Arbiter>`__.

.. _command_handles:

Command Handles
~~~~~~~~~~~~~~~

**UNDER REVISION 23 Oct 2015**

A *command handle* is the status of a particular command. They provide a
way to keep track of the particular stage a command node is in after it
transitions to the *Executing* state.

From the time a Command node becomes executable until the time the
external system completes executing the command, several things can
happen. First, the command's resource requirements are analyzed by the
resource arbiter, to determine whether the requirements of all
concurrent commands can be met. If the arbiter accepts the command, the
interface adapter sends the command (via whatever means) to the external
system. Finally, the external system can report whether the command
succeeded or failed.

It will be worthwhile to inform the executive as to what stage of its
journey the command is in for more reasons than just book-keeping. The
plan could, for instance, use a particular state of the command to start
or stop other nodes.

Command handles may take one of the following values:

-  COMMAND_DENIED
-  COMMAND_ACCEPTED
-  COMMAND_SENT_TO_SYSTEM
-  COMMAND_RCVD_BY_SYSTEM
-  COMMAND_SUCCESS
-  COMMAND_FAILED

When the resource arbiter rejects a command due to inadequate resources,
its command handle is set to COMMAND_DENIED.

When a command is issued by the executive, the interface may send any
legal command handle value as feedback. The PLEXIL Exec doesn't put any
interpretation on these values, with the exception of COMMAND_DENIED and
COMMAND_FAILED.

A hypothetical application could use them in the following ways:

-  COMMAND_ACCEPTED when the interface begins to process the command;
-  COMMAND_SENT_TO_SYSTEM when the interface has sent the command to the
   external system;
-  COMMAND_RCVD_BY_SYSTEM when the external system receives the command;
-  COMMAND_SUCCESS when the external system has successfully executed
   the command;
-  COMMAND_FAILED when the command issued to the external system fails
   to execute.

The default EndCondition of Command nodes is ``True``. If an
EndCondition is supplied, the effective expression used for the
EndCondition is:

::

   Self.command_handle == COMMAND_DENIED
    || Self.command_handle == COMMAND_FAILED
    || <user EndCondition>

Command handles are stored in a variable that can be used in any node as
a decision variable. This is illustrated in the sample plan shown below.
Here, there are two Command nodes, *Drive* and *NextWaypoint*. Using a
StartCondition, we can make the NextWaypoint node issue a command
*after* the Drive node receives the COMMAND_RCVD_BY_SYSTEM signal from
the external system.

::

   Integer Command drive();
   Command next_waypoint();

   SimpleDrive:
   Concurrence
   {
     Drive:
     {
       Integer returnValue = -1;
       EndCondition returnValue == 10;
       PostCondition Drive.command_handle == COMMAND_SUCCESS;
       returnValue = drive();
     }

     NextWaypoint:
     {
       StartCondition Drive.command_handle == COMMAND_RCVD_BY_SYSTEM;
       next_waypoint();
     }
   }

A command may be *aborted* when its Command node fails (i.e. the
InvariantCondition of the node or any of its direct ancestors becomes
``False``) or is interrupted (the ExitCondition of the node or its
direct ancestors becomes ``True``). How a command abort is actually
handled is determined by the interface adaptor for the given PLEXIL
application. See the the section on
`interfacing <Interfacing_Overview>`__ for more information.

The command handle is ignored in the event of an abort. A Boolean status
variable, the *abort acknowledgment variable*, is used to determine when
the abort is complete. The Command node transitions to *Iteration_Ended*
when the abort has been acknowledged by the interface adapter.

Examples
--------

-  Example 1: `Simple unary resources <Simple_unary_resources>`__
-  Example 2: `Non-unary resources <Non-unary_resources>`__ is a
   variation of the previous example with one of the resources
   represented as a non-unary resource.
-  Example 3: `Hierarchical resources <Hierarchical_resources>`__
   illustrates a command with interrelated resources.
-  Example 4: `Renewable resources <Renewable_resources>`__ illustrates
   a command that *generates* resources (as opposed to consuming them).

--------------

Copyright (c) 2006-2015, Universities Space Research Association (USRA).
All rights reserved.

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

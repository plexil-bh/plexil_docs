.. _ResourceArbiter:

ResourceArbiter
=================

*10/4/10*

This chapter describes in greater detail PLEXIL's *resource arbiter*,
which was introduced in the `Resource Model <Resource_Model>`__ chapter.

.. contents::

Design
------

The PLEXIL language provides constructs to list resource requirements
for a command. The Resource Arbiter, which is part of the PLEXIL
Executive, implements the necessary logic that keeps track of the
resources consumed and also performs the task of accepting or rejecting
commands based on the available resource level. This schematic shown
below gives an overall idea of the resource model implemented in the
executive. (Note: *UE*, which stands for *Universal Executive*, is an
outdated name for the *PLEXIL Executive*).

.. figure:: ../_static/images/Resourcemodel.jpg

Capabilities
------------

The resource model provides the following capabilities.

-  Implements consumable and renewable unary and non-unary resources.
-  Consumed resource levels are maintained by the resource arbiter. This
   assumes that the consumption per command is known ahead of time and
   fixed. Also tracks only resource consumption/production for commands
   issued by the executive.
-  No assumptions are made about the duration of command etc. it is
   assumed that whatever resource a command consumes or generates
   happens at its start and similarly and resource release happens when
   the command ends.

The resource model does *not* provide querying of the external system
for resource availability.

The bulk of resource model implementation is the resource arbiter.
Besides the language extension, the only other entity that is affected
is the external interface. Instead of sending the commands directly to
the external subsystem, the external interface has to first invoke the
arbitration process and then forward only the accepted commands to the
external sub-system.

The resource arbiter is handed a list of commands along with the
resources each of them require. The task of the arbiter then is to
identify all the commands that can be executed so that the available
maximum resource level is not exceeded. While doing so, the arbiter also
needs to pay attention to the priority levels of the commands for the
reason that when two or more commands vie for the same resource, the
command with the higher priority value wins.

The current implementation enforces the following restriction;

#. All the resources required by a command have the same priority (which
   is equal to the priority of the the command). Although the XML
   specification allows for a priority value per resource, the arbiter
   will pick the priority value of the first resource listed for the
   command and use it for all the other resources used by the command.
   NOTE: this could perhaps be enforced by the Standard Plexil compiler.
#. The lower bound of the resource is ignored.

.. _the_basic_algorithm:

The Basic Algorithm
-------------------

#. For a command to be accepted all its resource requirements have to be
   met.
#. Optimizes both on priority values and the total number of commands
   accepted.
#. A lower priority command will be accepted only if there is still some
   resource left over after the higher priority commands have taken
   their share or if a higher priority command gets rejected.
#. If two commands have the same priority, they will be prioritized in
   order in which they are batched at the end of the quiescence cycle.
#. When a subset of the commands is accepted, its worst case resource
   requirements should be not exceed the maximum allowable limits.

.. _resource_configuration_file:

Resource Configuration File
---------------------------

By default, the resource arbiter obtains the identity of resources from
the command itself and when commands that use a particular resource
completes, the resource arbiter purges the resource from its database.
The default (absolute) maximum consumable and renewable value is 1.0.

The user also has the option of gathering information about the
resources identified in the system as well as their availability in a
form of a configuration file that can then be read by the resource
arbiter. This file must be named ``resource.data`` and filed in the
directory from which the executive is run; see an example in
``plexil/exampes/resource.data``.

| Such a configuration file at the minimum can contain information such
  as the maximum consumable and renewable levels. In addition the
  configuration can also capture interdependencies between resources.
  Currently the resource arbiter can handle resource dependencies that
  can be represented in the form of a weighted Directed Acyclic Graph.
  The schematic shown below shows the general structure of such a graphs
  and the format of the configuration file. The weights represent the
  absolute value of the resource usage.
  
.. figure:: ../_static/images/Dagresources3.jpg

--------------

Copyright (c) 2006-2014, Universities Space Research Association (USRA).
All rights reserved.

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

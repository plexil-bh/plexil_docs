.. _RenewableResources:

Renewable Resources 
=======================

*12 May 2015*

It is possible for a command to *generate* resources, as opposed to
consuming them.

Consider the case when there are, say, 2 units of memory left out of a
maximum of 5 units. Consider the following resource requirement.

-  ``C1`` requires ``<arm, 10>`` and ``<sys_memory, 20, 0.8, 0.8>``
-  ``C2`` requires ``<sys_memory, 30, 0.3, 0.3>``
-  ``C3`` requires ``<sys_memory, 10, -0.1, -0.1>``

where the integer value in the resource pair denotes the priority (lower
number implies higher priority) and the last two real values are the
lower and upper bounds of the memory needed by the command. In our
example, ``C1`` and ``C2`` *consume* 0.8 and ``C2`` 0.3 units of memory
respectively, while ``C3`` *generates* 0.1 units of memory.

The resulting outcome will be to accept commands all the commands
since the cumulative memory requirements is less than 5.0.

.. figure:: ../../_static/images/Renewableresources4.jpg

The entire PLEXIL plan (filed in plexil/examples/resource4.ple) is
given below.

*PLEASE NOTE that this example does not seem to work as advertised in
the current version of the PLEXIL executive. We are investigating;
please drop us a line if you need this feature.*

::

   Integer Command c1;
   Integer Command c2;
   Integer Command c3;

   SimpleTask:
   Concurrence {
     C1: {
       Integer returnValue = -1;
       EndCondition returnValue == 10;
       PostCondition C1.command_handle == COMMAND_SUCCESS;
       Resource Name = "sys_memory",
         LowerBound = 0.8,
         UpperBound = 0.8,
         Priority = 20;
       returnValue = c1();
     }
     C2: {
       Integer mem_priority = 30;
       Integer returnValue = -1;
       PostCondition C2.command_handle == COMMAND_SUCCESS;
       EndCondition returnValue == 10;
       Resource Name = "sys_memory",
         LowerBound = 0.3,
         UpperBound = 0.3,
         Priority = mem_priority;
       returnValue = c2();
     }
     C3: {
       Integer returnValue = -1;
       PostCondition C3.command_handle == COMMAND_SUCCESS;
       EndCondition returnValue == 10;
       Resource Name = "sys_memory",
         LowerBound = -0.1,
         UpperBound = -0.1,
         Priority = 40;
       returnValue = c3();
     }
   }


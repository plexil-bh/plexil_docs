.. _NonUnaryResources:

Non-Unary Resources 
======================

*12 May 2015*

This example (filed in ``plexil/examples/resource2.ple``) is similar to the
unary resource example expect that the resource sys_memory is not
treated as a unary resource. This is because of the manner in which the
resource usage is specified, Including the lower and upper bounds
implies that the resource is not unary. In particular,

-  ``C1`` requires ``<arm, 10>`` and ``<sys_memory, 20, 0.5, 0.5>``
-  ``C2`` requires ``<sys_memory, 30, 0.3, 0.3>``
-  ``C3`` requires ``<vision_system, 10>``

where the integer value in the resource pair denotes the priority (lower
number implies higher priority) and the last two real values are the
lower and upper bounds of the memory needed by the command. In our
example, ``C1`` requires 0.5 units of memory and ``C2`` 0.3 units. By
default let us assume that the maximum value is 1.0.

The resulting outcome will be to accept commands all the commands
since the cumulative memory requirements of ``C1`` and ``C2`` is less
than 1.0.

.. figure:: ../../_static/images/Nonunaryresource3.jpg

The entire PLEXIL plan is given below;

::

   Integer Command c1;
   Integer Command c2;
   Integer Command c3;

   SimpleTask: 
   Concurrence
   {
     C1: {
       Integer returnValue = -1;
       EndCondition returnValue == 10;
       PostCondition C1.command_handle == COMMAND_SUCCESS;
       Resource Name = "sys_memory",
         LowerBound = 0.5,
         UpperBound = 0.5,
         Priority = 20;
       Resource Name = "arm",
         Priority = 20;
       returnValue = c1();
     }
     C2: {
       Integer mem_priority = 30;
       Integer returnValue = -1;
       RepeatCondition C2.command_handle == COMMAND_DENIED;
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
       Integer vision_priority = 10;
       PostCondition C3.command_handle == COMMAND_SUCCESS;
       EndCondition returnValue == 10;
       Resource Name = "vision_system",
         Priority = vision_priority;
       returnValue = c3();
     }
   }


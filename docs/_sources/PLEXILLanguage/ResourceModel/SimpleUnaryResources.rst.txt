.. _SimpleUnaryResources:

Simple Unary Resources 
=========================

*12 May 2015*

This example (filed in plexil/examples/resource1.ple) involves three
command nodes that are eligible for execution and three unary resources.
Two of the nodes vie for the same resource with different priorities.
The node that has the higher resource priority (lower numeric value)
gains access to the resource. A repeat condition in the lower priority
node ensures that it executes when the resource it needs frees up.

Specifically, we have three commands ``C1``, ``C2`` and ``C3`` scheduled
to run concurrently. There are also three resources, ``arm``,
``sys_memory`` and ``vision_system`` that are used by the commands. In
particular,

-  ``C1`` requires ``<arm, 20>`` and ``<sys_memory, 20>``
-  ``C2`` requires ``<sys_memory, 30>``
-  ``C3`` requires ``<vision_system, 10>``

where the integer value in the resource pair denotes the priority (lower
number implies higher priority).

The resulting outcome will first be to accept commands ``C1`` and
``C3`` and reject ``C2``. Then since ``C2`` is associated with a
repeat condition, ``C2`` will be accepted after the completion of
``C1``.
  
.. figure:: ../../_static/images/Unaryresources3.jpg

The entire PLEXIL plan is shown below. Notice that values for the
resource elements can be parameterized i.e., can either be variables
or values. Parameterizing the resource elements makes it possible to
determine them during the course of execution.

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
         Resource Name = "sys_memory", Priority = 20;
         Resource Name = "arm", Priority = 20;
         returnValue = c1();
     }
     C2: {
         Integer mem_priority = 30;
         Integer returnValue = -1;
         RepeatCondition C2.command_handle == COMMAND_DENIED;
         PostCondition C2.command_handle == COMMAND_SUCCESS;
         EndCondition returnValue == 10;
         Resource Name = "sys_memory", Priority = mem_priority;
         returnValue = c2();
     }
     C3: {
         Integer returnValue = -1;
         Integer vision_priority = 10;
         PostCondition C3.command_handle == COMMAND_SUCCESS;
         EndCondition returnValue == 10;
         Resource Name = "vision_system", Priority = vision_priority;
         returnValue = c3();
     }
    }

.. _ExamplePlans:

Example Plans
=============

*10 Apr 2021*

This chapter explains a few |PLEXIL| plan examples, which are found in the
directory ``plexil/examples/basic``.

.. contents::

.. _example_1_a_simple_assignment:

Example 1: A simple assignment
------------------------------

Our first example, ``SimpleAssignment.ple`` is very small: the plan is a
single assignment node. An integer variable is declared, the
postcondition is it's having a certain value, and the body is assigning
this value.

::

   SimpleAssignment:
   {
     Integer foo = 0;
     PostCondition foo == 3;
     foo = 3;
   }

.. _example_2_drive_to_target:

Example 2: Drive to Target
--------------------------

Our second |PLEXIL| plan, ``DriveToTarget.ple`` is one for controlling a
rover. In this contrived example, a rover is commanded to drive until
either a target is in view, or time has reached 10. If the target comes
into view, an image is taken using the Pancam. If the time limit is
reached, an image is taken using the Navcam. All the while temperature
is monitored, and the heater is turned on whenever temperature drops
below 0, and turned off when it exceeds 10.

The root node, ``DriveToTarget``, represents a starting point and is a
Concurrence that performs (concurrently) the actions described above.
This plan illustrates complex sequencing: note the specific start
conditions in each child node of the Concurrence. It also illustrates
*monitors*, which are nodes that actively await an enabling condition
for execution, or a condition (e.g. ``timeout``) that deactivates them
altogether.

::

   // A Plexil plan illustrating a simple rover driving task.

   Command rover_drive (Integer);
   Command rover_stop;
   Command take_navcam;
   Command take_pancam;
   Command turn_on_heater;

   Real Lookup time;
   Real Lookup temperature;
   Boolean Lookup target_in_view;


   DriveToTarget:
   Concurrence {
     Boolean drive_done = false, timeout = false;
     Drive:  rover_drive(10);
     StopForTimeout:
     {
       StartCondition Lookup(time) >= 10;
       Concurrence {
         Stop: rover_stop();
         SetTimeoutFlag: timeout = true;
       }
     }

     StopForTarget:
     Concurrence {
       StartCondition Lookup(target_in_view);
       SkipCondition timeout;
       Stop: rover_stop();
       SetDriveFlag: drive_done = true;
     }

     TakeNavcam:
     {
       StartCondition timeout;
       SkipCondition drive_done;
       take_navcam();
     }

     TakePancam:
     {
       StartCondition drive_done;
       SkipCondition timeout;
       take_pancam();
     }

     Heater:
     {
       SkipCondition timeout;
       StartCondition Lookup(temperature) < 0;
       EndCondition Lookup(temperature) >= 10;
       turn_on_heater();
     }
   }

.. _example_3_safedrive:

Example 3: SafeDrive
--------------------

This example is another rover plan and illustrates loops and sequencing.

::

   Command Drive (Integer);
   Command TakePicture;
   Command pprint(...);
   Boolean Lookup WheelStuck;

   SafeDrive:
   {
     Integer pictures = 0;
     EndCondition Lookup(WheelStuck) || pictures == 10;

     while (! Lookup(WheelStuck))
     {
       OneMeter: { Drive(1); }
       TakePic: {
         StartCondition pictures < 10;
         TakePicture();
       }
       Counter: {
         PreCondition pictures < 10;
         pictures = pictures + 1;
       }
       Print: { pprint ("Pictures taken:", pictures); }
     }
   }

This plan's root node, ``SafeDrive`` is a While loop that continues
execution so long as the rover's wheels are not stuck (as represented by
a state lookup). The body of the While loop is a Sequence of three
nodes: ``OneMeter``, which invokes a command that drives the rover one
meter, ``TakePic``, which invokes a command that takes a picture, and
``Counter``, which counts the number of pictures that have been taken.
The start condition of ``TakePic`` ensures that the node starts
execution only when local variable ``pictures`` is strictly smaller than
10. A precondition in ``Counter`` ensures that no more than 10 pictures
are taken.

.. _more_examples:

More Examples
-------------

More sophisticated plans, and their associated simulation scripts, are
found in the various subdirectories of ``plexil/examples``


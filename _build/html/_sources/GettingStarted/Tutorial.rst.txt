.. _Tutorial:

Tutorial
============

*10 Apr 2021*

.. contents::


`Plexil <http://plexil.sourceforge.net/wiki/index.php/Main_Page>`__
(Plan Execution Interchange Language) is a language used to represent
plans of automation. To help new users learn the language without having
a system to automate, we have created the Robosim simulator. This
tutorial will guide you through learning Plexil using this simulator.

Throughout this tutorial links to the `Plexil
Documentation <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference>`__
will be provided so you can read more about the topics we explore.  

.. _starting_the_simulation:

1. Starting the simulation
--------------------------

Once you have Plexil
`installed <http://plexil.sourceforge.net/wiki/index.php/Installation>`__
you can navigate to ``$PLEXIL_HOME/examples/robosim`` and run:

::

   $ ./robosim start

This command runs an IPC server and the Robosim simulation in the
background of your terminal. You should see the Robosim application
launch, this is the environment we'll be working with for this tutorial.

To restart the simulation or stop both the simulation and the IPC server
run

::

   $./robosim restart

or

::

   $./robosim stop

 

Troubleshooting
~~~~~~~~~~~~~~~

If the application fails to launch you may have to run

::

   $ make

in one of these two locations:

``$PLEXIL_HOME/`` or ``$PLEXIL_HOME/examples/robosim/2DRobotSimulator``

 

.. _your_first_node_and_command:

2. Your first node and command
------------------------------

Let's start by making a Plexil file. inside
``$PLEXIL_HOME/examples/robosim/plans``, create a new file called
``Drive.ple``.

A Plexil program is made up of nodes that consist of a pair of curly
braces: ``{ }``. Each Plexil plan should have one root node, let's
create it now.

::

   Drive:
   {

   }

We named this node ``Drive``. Now lets make our program do something!
Plexil programs interact with their environment using commands provided
by the system you are automating. Let's declare and use the utility
command ``pprint()``.

::

   Command pprint(...);

   Drive:
   {
       pprint("Hello World"); // Prints 'Hello World' to the unix terminal
   }

 

.. _compile_and_run_a_plexil_plan:

3. Compile and run a PLEXIL plan
--------------------------------

To run a Plexil program you must first compile it:

::

   $ plexilc Drive.ple

This will generate a compiled xml file called ``Drive.plx``. We can run
the program with:

::

   $ plexilexec -p Drive.plx

You should see ``'Hello World'`` printed to the terminal.

 

.. _moving_a_robot:

4. Moving a robot
-----------------

.. _robots_first_step:

Robots first step:
~~~~~~~~~~~~~~~~~~

Now let's start working with the
`robosim <http://plexil.sourceforge.net/wiki/index.php/Example_application:_RoboSim>`__.
Start by declaring and using the move command

::

   Integer Command Move(String name, Integer direction);
   Drive:
   {
       String robotName = "RobotYellow";
       Move(robotName, 1);
   }

Let's make sure we understand whats going on in each line:

::

   Integer Command Move(String name, Integer direction);

Here we declare the command that tells the simulator to move the robot.
You can read more about commands
`here <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#Command>`__,
and see the commands supported by the robosim
`here <http://plexil.sourceforge.net/wiki/index.php/Example_application:_RoboSim#Commanding_Robots>`__

::

   String robotName = "RobotYellow";

Next we declare and initialize a variable of type String to hold the
name of the robot we want to control. Plexil supports several data types
that can be found
`here <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#Variables>`__.

::

   Move(robotName, 1);

Finally we call the move command to move the Yellow Robot right
(direction 1) once. (We aren't using the return value of Move yet)

Compile the program with ``plexilc``. To run a program that uses the
robosim commands we have to provide the robosim interface like this:

::

   $ plexilexec -p Drive.plx -c ../interface-config.xml

.. _continuous_motion:

Continuous motion:
~~~~~~~~~~~~~~~~~~

So how do we move the robot more than once? There are two options; one
is to use a
`loop <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#While_Loop>`__,
the second is to use a
`RepeatCondition <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#Conditions>`__.
Lets look at a while loop.

::

   Integer Command Move(String name, Integer direction);
   Drive:
   {
       String robotName = "RobotYellow";
       Boolean run = true;
       while (run)
       {
           Integer moveResponse;
           Move:
           {
               EndCondition isKnown(moveResponse); // Wait for the response after moving the robot
               moveResponse = Move(robotName, 1);  // Direction 1 means move right
           }
           if (moveResponse != 1) // A response of 1 means the move succeeded
               run = false;       // If the move failed stop the run
       }
   }

We can see that the robot will continue to move until the move fails.
This can happen if the robot hits an obstacle, or runs out of battery.
Feel free to try both methods and learn about
`conditions <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#Conditions>`__
and
`loops <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#While_Loop>`__.

 

.. _adding_intelligence:

5. Adding Intelligence
----------------------

Now let's teach the robot to path to an energy source.

These are the commands we are going to need so we must declare them at
the top of out file first.

::

   Integer Command Move(String name, Integer direction);
   Real[5] Command QueryEnergySensor(String name);
   Integer[4] Command QueryVisibilitySensor(String name);

Next we have to call these commands to figure out which direction an
energy source is and in which directions we can move.

::

   Drive:
   {
       String robotName = "RobotYellow";
       Boolean run = true;
       while (run)
       Concurrence // this tells our nodes to execute concurrently allowing us to read both sensors at the same time
       {
           Real energyVals[5]; // Declare arrays to hold our query data
           Integer visibilityVals[4];
           Integer direction = 0; // The direction we will go

           ReadEnergySensor:
           {
               // end the node when the query has returned values
               EndCondition isKnown(energyVals[0]);
               energyVals = QueryEnergySensor(robotName);
           }

           ReadVisibilitySensor:
           {
               // end the node when the query has returned values
               EndCondition isKnown(visibilityVals[0]);
               visibilityVals = QueryVisibilitySensor(robotName);
           }
       }
   }

Now that we have the data from our sensors we must figure out which
direction to move in.

::

   FindDirection:
   {
       Real maxVal = 0; // initialize a variable to track the best direction

       // Make sure we only start this node after we have our sensor data
       StartCondition isKnown(energyVals[0]) && isKnown(visibilityVals[0]);

       // Loop through every direction we could go
       for(Integer i = 0; i < 4; i + 1>)
       {
           // If the direction is unoccupied and has the highest energy value we save it into the variable direction
           if(visibilityVals[i] == 1 && energyVals[i] > maxVal)
           {
               maxVal = energyVals[i];
               direction = i;
           }
           endif
       }
       // If our next move will take us to an energy source we can stop the run
       if(maxVal== 1)
           run = false;
   }

The only thing left to do is make the move:

::

   Move:
   {
       // only start once we know what direction to go
       StartCondition FindDirection.outcome == SUCCESS;

       Move(RobotName, direction)
   }

Now lets put it all together:

::

   Integer Command Move(String name, Integer direction);
   Real[5] Command QueryEnergySensor(String name);
   Integer[4] Command QueryVisibilitySensor(String name);

   Drive:
   {
       String robotName = "RobotYellow";
       Boolean run = true;
       while (run)
       Concurrence // this tells our nodes to execute concurrently allowing us to read both sensors at the same time
       {
           Real energyVals[5]; // Declare arrays to hold our query data
           Integer visibilityVals[4];
           Integer direction = 0; // The direction we will go

           ReadEnergySensor:
           {
               // end the node when the query has returned values
               EndCondition isKnown(energyVals[0]);
               energyVals = QueryEnergySensor(robotName);
           }

           ReadVisibilitySensor:
           {
               // end the node when the query has returned values
               EndCondition isKnown(visibilityVals[0]);
               visibilityVals = QueryVisibilitySensor(robotName);
           }

           FindDirection:
           {
               Real maxVal = 0; // initialize a variable to track the best direction

               // Make sure we only start this node after we have our sensor data
               StartCondition isKnown(energyVals[0]) && isKnown(visibilityVals[0]);

               // Loop through every direction we could go
               for(Integer i = 0; i < 4; i + 1)
               {
                   // If the direction is unoccupied and has the highest energy value we save it into the variable direction
                   if(visibilityVals[i] == 1 && energyVals[i] > maxVal)
                   {
                       maxVal = energyVals[i];
                       direction = i;
                   }
                   endif
               }
               // If our next move will take us to an energy source we can stop the run
               if(maxVal == 1)
                   run = false;
           }

           Move:
           {
               // only start once we know what direction to go
               StartCondition FindDirection.outcome == SUCCESS;

               Move(robotName, direction);
           }
       }
   }

Now you can watch your work in node! Compile and run the code on the
robosim and watch the robot path to the nearest energy source. If you
want you can show the detection range of the energy sources in the sim
by pressing 'e' the goal by pressing 'g' and the flags by pressing 'f'.

 

.. _library_calls:

6. Library calls
----------------

Abstraction is a key idea when programming that allows the creation of
more organized less error prone code. In Plexil abstraction is achieved
through `library
calls <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#Library_Call>`__.
In our example we would like our robot to move to several different
objects, the energy sources, the flags, and the goals. To avoid copied
code we will write a library node that determines the best direction to
go for any of these objects.

Because each Plexil file may only have one top level node we will write
our library node in a new file called ``BestDirection.ple``.

::

   BestDirection:
   {
       In Real directionVals[4];   // The value of moving in each direction (Read Only)
       In Integer visibilityVals[4];  // The visibility value of each direction (Read Only)
       InOut Integer maxDirection; // An out-parameter used to return the best direction to the calling node.

       Real maxVal = 0;    // Track the max val just like before

       StartCondition isKnown(directionVals[0]);

       for(Integer i = 0; i < 4; i + 1)
       {
           if(visibilityVals[i] == 1 && directionVals[i] > maxVal)
           {
               maxDirection = i;
               maxVal = directionVals[i];
           }
           endif
       }
   }

Our library node will accept an array of reals representing the goal,
energy, or flag values in each of the 4 directions and return through
the out parameter ``maxDirection`` the direction with the largest value.

Now lets use this library node in a plan that will path our robot to the
goal, picking up energy sources along the way if need be.

First we declare our robosim Commands and our library node:

::

   Real[5] Command QueryGoalSensor(String name);
   Real[5] Command QueryEnergySensor(String name);
   Integer[4] Command QueryVisibilitySensor(String name);
   Real[4] Command QueryRobotState(String name);
   Integer Command Move(String name, Integer direction);

   LibraryAction BestDirection (In Real directionVals[4],
       In Integer visibilityVals[4],
       InOut Integer maxDirection);

Next lets create our main loop and read the sensors:

::

   Drive:
   {
       String robotName = "RobotYellow";
       Boolean atGoal = false;
       Boolean noBattery = false;

       // Fail if we ever run out of battery
       InvariantCondition !noBattery;
       // Succeed if we ever reach the goal
       ExitCondition atGoal;

       while(!atGoal && !noBattery) {
           Real robotState[4];
           Real energyVals[5];
           Real goalVals[5];
           Integer visibilityVals[4];
           Integer direction;

           ReadRobotState:
           {
               EndCondition isKnown(robotState[0]);
               robotState = QueryRobotState(robotName);
           }

           ReadEnergySensor:
           {
               EndCondition isKnown(energyVals[0]);
               energyVals = QueryEnergySensor(robotName);
           }

           ReadGoalSensor:
           {
               EndCondition isKnown(goalVals[0]);
               goalVals = QueryGoalSensor(robotName);
           }

           ReadVisibilitySensor:
           {
               EndCondition isKnown(visibilityVals[0]);
               visibilityVals = QueryVisibilitySensor(robotName);
           }
       }
   }

Now we use the sensor data to determine which direction we should go:

::

   DetermineDirection:
   {
       // If we are below 50% energy then move to energy
       if(robotState[2] < .5)
           LibraryCall BestDirection(directionVals=energyVals,
               visibilityVals=visibilityVals,
               maxDirection=direction);
       endif
       // If we have energy or we are not in range to detect any move towards the goal.
       if(!isKnown(direction))
           LibraryCall BestDirection(directionVals=goalVals,
               visibilityVals=visibilityVals,
               maxDirection=direction);
       endif
       // If we still have no direction move down
       if(!isKnown(direction))
           direction = 2;

       // check if we are at the goal or if we are out of battery.
       if(robotState[2] == 0)
           noBattery = true;
       endif
       if(goalVals[4] == 1)
           atGoal = true;
       endif
   }

Finally we just need to move in the direction we found:

::

   Move:
   {
       StartCondition isKnown(direction);
       // if we haven't reached the goal continue moving
       if(goalVals[4] != 1)
           Move(robotName, direction);
   }

Lets put it all together:

::

   Real[5] Command QueryGoalSensor(String name);
   Real[5] Command QueryEnergySensor(String name);
   Integer[4] Command QueryVisibilitySensor(String name);
   Real[4] Command QueryRobotState(String name);
   Integer Command Move(String name, Integer direction);

   LibraryAction BestDirection (In Real directionVals[4],
       In Integer visibilityVals[4],
       InOut Integer maxDirection);

   Drive:
   {
       String robotName = "RobotYellow";
       Boolean atGoal = false;
       Boolean noBattery = false;

       // Fail if we ever run out of battery
       InvariantCondition !noBattery;
       // Succeed if we ever reach the goal
       ExitCondition atGoal;

       while(!atGoal && !noBattery) {
           Real robotState[4];
           Real energyVals[5];
           Real goalVals[5];
           Integer visibilityVals[4];
           Integer direction;

           ReadRobotState:
           {
               EndCondition isKnown(robotState[0]);
               robotState = QueryRobotState(robotName);
           }

           ReadEnergySensor:
           {
               EndCondition isKnown(energyVals[0]);
               energyVals = QueryEnergySensor(robotName);
           }

           ReadGoalSensor:
           {
               EndCondition isKnown(goalVals[0]);
               goalVals = QueryGoalSensor(robotName);
           }

           ReadVisibilitySensor:
           {
               EndCondition isKnown(visibilityVals[0]);
               visibilityVals = QueryVisibilitySensor(robotName);
           }

           DetermineDirection:
           {
               // If we are below 50% energy then move to energy
               if(robotState[2] < .5)
                   LibraryCall BestDirection(directionVals=energyVals,
                       visibilityVals=visibilityVals,
                       maxDirection=direction);
               endif
               // If we have energy or we are not in range to detect any move towards the goal.
               if(!isKnown(direction))
                   LibraryCall BestDirection(directionVals=goalVals,
                       visibilityVals=visibilityVals,
                       maxDirection=direction);
               endif
               // If we still have no direction move down
               if(!isKnown(direction))
                   direction = 2;

               // check if we are at the goal or if we are out of battery.
               if(robotState[2] == 0)
                   noBattery = true;
               endif
               if(goalVals[4] == 1)
                   atGoal = true;
               endif
           }

           Move:
           {
               StartCondition isKnown(direction);
               // if we haven't reached the goal continue moving
               if(goalVals[4] != 1)
                   Move(robotName, direction);
           }
       }
   }

Try running your code and watch your robot go! *Don't forget to compile
your library node*.

 

.. _further_exploration:

7. Further exploration
----------------------

Now it's time to explore the language on your own...

-  Can you make the robot pick up the flag and take it to the goal?
-  Can you control two robots at once?
-  Explore some more advanced feature of the language like
   `lookups <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#External_State_.28Lookups.29>`__
   or the other types of
   `nodes <http://plexil.sourceforge.net/wiki/index.php/Plexil_Reference#High-level_Nodes>`__
   not covered here.

 

Originally Written by Bryce Campbell (2020)

--------------

Copyright (c) 2020-2021, Universities Space Research Association (USRA).
All rights reserved.

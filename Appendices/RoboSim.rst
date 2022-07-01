.. _RoboSim:

RoboSim
=======

*7/15/2020*

.. contents::

Introduction
------------

RoboSim is a simple two-dimensional robot simulator that can be used to
demonstrate and study the behavior of planning, scheduling, and plan
execution systems.

Several instances of robots can be created and deployed in a 2D terrain
that contains obstacles, energy sources, goals, flags, and other robots.
Robots understand and respond to a small list of commands.

The RoboSim application is filed in the ``plexil/src/apps/robosim``
directory of the PLEXIL distribution. It has been tested on Linux and
MacOS systems.

This document describes the RoboSim application and its interface with
PLEXIL and the PLEXIL Executive.

Domain
------

.. figure:: ../_static/images/NewRobosim.png
   :alt: NewRobosim.PNG


Figure 1: A snapshot of the RoboSim terrain and its occupants.

This screenshot shows an example terrain occupied by many robots
(squares with small triangular appendages representing sensors). The red
angular lines are static obstacles, the yellow lightning bolts are
energy sources, the purple target is a goal, and there is one green flag
in center near the top of the simulation.

.. _terrain_and_obstacles:

Terrain and Obstacles
~~~~~~~~~~~~~~~~~~~~~

The RoboSim terrain is a 2D cell grid where each cell is represented by
its row and column (integers). Figure 1 depicts a square terrain with 32
cells along each direction.

There are two types of obstacles in a terrain, static and dynamic.
Static obstacles include the terrain boundary and walls (thick red lines
within the terrain). Dynamic obstacles are other robots. Obstacles block
a robot's movement, and static obstacles cannot be removed.

.. _energy_sources:

Energy Sources
~~~~~~~~~~~~~~

Distributed throughout the terrain are energy sources that robots can
“consume” to boost their energy level. Such energy sources can be viewed
as charging stations and are shown as yellow lightning bolts in Figure
1. (Note that energy sources occupy exactly one grid cell, though
visually they extend over several surrounding cells.)

The energy level of each robot, initialized to its maximum value of 1.0,
drops by a prefixed amount (currently 0.05 units) for every successful
step it takes. The energy level is boosted to the maximum level of 1.0
by visiting a cell containing an energy source.

The presence of an energy source can be “sensed” by a robot as will be
discussed in subsequent sections.

.. _goal_flags:

Goal & Flags
~~~~~~~~~~~~

The goal and flag components in the scene give the robots a purpose to
navigate the terrain. Figure 1 shows a purple target and a green flag,
either of which could be what the robot is tasked to find. The flag is
slightly different in that the robot will pick up a flag when they
occupy the same space. An advanced plan could task the robot to pick up
the flag and take it to the goal.

As with energy sources, goals and flags occupy exactly one grid cell,
and their presence can also be “sensed” by the robots.

.. _robots_and_their_sensors:

Robots and their Sensors
~~~~~~~~~~~~~~~~~~~~~~~~

Robots are display as colored squares with small triangular appendages
representing their sensors. A robot can be moved one cell at a time
along one of the four principal directions up, right, down and left,
provided the movement is not blocked by an obstacle. A robot is equipped
with four sensors along each principal direction that provide feedback
about energy source levels, goal and flag levels and visibility. See the
section below on the robot interface for details about these values.

.. _commanding_robots:

Commanding Robots
-----------------

The following commands can be issued to each of the robots.

Movement
~~~~~~~~

::

     Integer MoveUp(String robot_name);
     Integer MoveRight(String robot_name);
     Integer MoveDown(String robot_name);
     Integer MoveLeft(String robot_name);
     Integer Move(String robot_name, Integer direction);

As their names suggest, these commands attempt to move the controlled
robot in a named direction or by using an integer to represent the
direction: up (0), right (1), down (2), or left (3). The integer value
returned is interpreted as follows.

-  1 means the move succeeded
-  -1 means the move failed due to the presence of a dynamic obstacle
-  0 means the move failed due to the presence of a static obstacle

.. _sensing_energy_sources_and_goal:

Sensing Energy Sources and Goal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

     Real[5] QueryEnergySensor(String robot_name);
     Real[5] QueryGoalSensor(String robot_name);
     Real[5] QueryFlagSensor(String robot_name);

These commands query the energy, goal, and flag values, respectively, in
the neighborhood of the robot. They return an array of five real values
in the range [0, 1], corresponding to the interpolated energy/goal
values at the cell location in the directions up, right, down and left,
respectively, followed by the energy/goal value at the current location
of the robot. The ranges at which objects are visible can be seen by
pressing 'e' for energy sources, 'f' for flags, and 'g' for goals.

.. _sensing_obstacles:

Sensing Obstacles
~~~~~~~~~~~~~~~~~

::

   Integer[4] QueryVisibilitySensor(String robot_name);

This command is used to determine the mobility of the robot from its
current location. It returns an array of four integers ∈ {-1, 0, 1}
where each of the four values corresponding to cells in the directions
up, right, down and left, respectively. The values have the following
interpretation.

-  1 means the desired cell is currently unoccupied
-  -1 means there is a dynamic object in the desired cell
-  0 means there is a static object in the desired cell

.. _querying_location_and_energy_level:

Querying Location and Energy Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   Real[4] QueryRobotState(String robot_name);

This command returns an array of three real values corresponding to the
controlled robot's current row and column position, its current energy
level, and whether or not it has a flag (0 or 1), respectively. Note
that the robot's coordinates are returned as real numbers rather than
integers.

.. _using_robosim:

Using RoboSim
-------------

To use RoboSim, first start the IPC communication server:

::

    % ipc

Then, in a separate Unix shell, start the graphical simulator:

::

    % robosim

Both the IPC communication server and the graphical simulator can be
started and stopped in the background with the commands:

::

    % robosim start

and

::

    % robosim stop

The graphical simulator can be restarted in the background with:

::

    % robosim restart

Finally, cd to 'plans' and run the Plexil Executive with the plan of
your choice, e.g.

::

    % plexilexec -p CaptureTheFlag.plx

Note that other options to
`plexilexec <PLEXIL_Executive#Running_the_Executive>`__ are possible.

Currently, PLEXIL can control a yellow robot ("RobotYellow") and a blue
one ("RobotBlue3"). Other robots move around randomly until they run out
of energy.

.. _plexil_interface:

PLEXIL Interface
~~~~~~~~~~~~~~~~

PLEXIL plans can interact with RoboSim using commands only. At this time
no lookups (world state queries) have been implemented.

The commands available are exactly those listed above in the section
`Commanding Robots <Example_application:_RoboSim#Commanding_Robots>`__.

See the directories ``plexil/src/apps/robosim/plans`` and
``robosim/scripts`` for sample PLEXIL code, which contain examples of
the commands.

--------------

Copyright (c) 2006-2020, Universities Space Research Association (USRA).
All rights reserved.

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

This is the main page for the PLEXIL wiki. For PLEXIL's main project
page, which has downloads and other information, go here:
http:/sourceforge.net/projects/plexil

*Updated 18 Aug 2021*

.. contents::


.. _what_is_plexil:

What is PLEXIL?
===============

**PLEXIL** (**Pl**\ an **Ex**\ ecution **I**\ nterchange **L**\ anguage)
is a language for representing plans for automation, as well a
technology for executing these plans on real or simulated systems.
PLEXIL has been used in robotics, control of unmanned vehicles,
automation of operations in human habitats, and systems and simulations
involving intelligent software agents. Scroll down for some examples.

PLEXIL was designed initially to meet the requirements of flexible,
efficient and reliable plan execution in space mission operations. It is
compact, semantically clear, and deterministic given the same sequence
of events from the external world. At the same time, the language is
quite expressive and can represent branches, loops, time- and event-
driven activities, concurrent activities, sequences, and temporal
constraints. The core syntax of the language is simple and uniform,
making plan interpretation simple and efficient, while enabling the
application of validation and testing techniques.

Accompanying PLEXIL is an execution engine, or *executive*, which
implements efficiently the PLEXIL language and provides interfaces to
controlled systems as well as decision support systems from which plans
may be sent. The PLEXIL software suite also includes graphical plan
execution viewers, a static plan checker, and two different plan
simulators.

PLEXIL was originally developed as a collaborative effort between
researchers at NASA and Carnegie Mellon University, funded by NASA's
Mars Technology Program through the Research Institute for Advanced
Computer Science (RIACS) in the Universities Space Research Association
(USRA). Since then it has continually evolved through application on
NASA projects, which have included the control of prototype planetary
rovers and habitats, drilling equipment, and demonstration of adjustable
automation for International Space Station operations.

See below for a brief description of major PLEXIL applications to date.
See the `References <References>`__ chapter of the PLEXIL manual for
more information on the background and applications of PLEXIL.

See the sidebar (column to the left of this page) for links to many
PLEXIL topics.

If you have questions, please email us at
plexil-support@lists.sourceforge.net

.. _nasa_applications:

NASA Applications
=================

`OceanWATERS <http://github.com/nasa/ow_simulator>`__
-----------------------------------------------------

The open source Ocean Worlds Autonomy Testbed for Exploration Research
and Simulation (`OceanWATERS <http://github.com/nasa/ow_simulator>`__)
uses PLEXIL for onboard lander autonomy.

.. figure:: _static/images/Lander_europa.jpg
   :height: 290
   :width: 350

   Image Credit: NASA
   

.. _cockpit_hierarchical_activity_planning_and_execution_chap_e:

Cockpit Hierarchical Activity Planning and Execution (CHAP-E)
-------------------------------------------------------------

In this prototypical aircraft decision support system under development
at NASA, PLEXIL procedures for aviation are authored as well as
automatically generated from various higher level representations.


.. figure:: _static/images/Chape.jpg
   :height: 290
   :width: 350

   Image Credit: NASA

.. _autonomy_operating_system_aos:

`Autonomy Operating System (AOS) <http://mys5.org/Proceedings/2017/Day_1/2017-S5-Day1_1135_Lowry.pdf>`__
--------------------------------------------------------------------------------------------------------

This project focuses on automation of FAA piloting procedures towards
the goal of enabling Unmanned Aerial Vehicles (UAVs) to fly in the
national airspace. The navigation component of this software is a PLEXIL
application and has been flight-tested on autonomous rotorcraft drones.

.. figure:: _static/images/Drone.jpg
   :height: 290
   :width: 350

   Image Credit: B&H Photo

`ICAROUS <https://github.com/nasa/icarous>`__
---------------------------------------------

PLEXIL is a component of this formal methods based software suite for
unmanned aircraft, open-sourced by NASA Langley.


.. figure:: _static/images/Icarous-logo.jpg
   :height: 290
   :width: 350

   Image Credit: NASA

.. _lunar_atmosphere_and_dust_environment_explorer_ladee:

`Lunar Atmosphere and Dust Environment Explorer (LADEE) <http://www.nasa.gov/mission_pages/ladee/main>`__
---------------------------------------------------------------------------------------------------------

PLEXIL was incorporated into LADEE's flight software as an experiment in
automating the handling of various conditions sensed by a spacecraft
component. Although this experiment did not make it into space, it
motivated a port of the PLEXIL software to the
`VxWorks <http://www.windriver.com>`__ embedded operating system, and a
host of improvements to make PLEXIL more robust for flight applications.


.. figure:: _static/images/Ladee.jpg
   :height: 290
   :width: 350

   Image Credit: NASA
   
.. _edison_demonstration_of_smallsat_networks:

`Edison Demonstration of Smallsat Networks <http://www.nasa.gov/centers/ames/engineering/projects/edison.html#.U17Facd2M1Q>`__
------------------------------------------------------------------------------------------------------------------------------

PLEXIL was used as the executive in an early version of the EDSN
software architecture. Satellite operations were encoded in a PLEXIL
library that would reside and be executed onboard the craft. Although
this version of the software architecture was not the final flight
version, this project was PLEXIL's foray into small spacecraft
automation.


.. image:: _static/images/Edsn1.jpg
    :width: 45 %
.. image:: _static/images/Edsn2.jpg
    :width: 45 %
	
Image Credit: NASA

.. _habitat_demonstration_unit:

`Habitat Demonstration Unit <http://www.nasa.gov/exploration/analogs/hdu_project.html>`__
-----------------------------------------------------------------------------------------

PLEXIL ran onboard NASA's Deep Space Habitat and Habitat Demonstration
Unit (DSH/HDU), a functional living and working station designed to
accommodate a group of astronauts on deep space missions. For two
consecutive years, the DSH was field tested during the Desert Research
and Technologies Studies (Desert RATS), where PLEXIL was used to
demonstrate automated control of several DSH subsystems.


.. image:: _static/images/120px-Hdu1.jpg
    :width: 45 %
.. image:: _static/images/120px-Hdu2.jpg
    :width: 45 %
	
Image Credit: NASA

.. _international_space_station:

`International Space Station <http://www.nasa.gov/mission_pages/station/main/index.html>`__
-------------------------------------------------------------------------------------------

PLEXIL has been used to demonstrate automation for International Space
Station operations.

.. image:: _static/images/Iss.jpg
    :width: 45 %
.. image:: _static/images/Iss2.jpg
    :width: 45 %
	
Image Credit: NASA


.. _mars_drill:

`Mars Drill <http://www.nasa.gov/centers/ames/multimedia/images/2006/marsdrill.html>`__
---------------------------------------------------------------------------------------

PLEXIL has served as the executive for the Drilling Automation for Mars
Exploration (DAME) drilling application. Field tested at the Haughton
Crater on Devon Island in Canada's Nunavut Territory north of Ontario
and Quebec, this is perhaps the first fully automated drill rig.


.. image:: _static/images/150999main_marsdrill2032.png
    :width: 45 %
.. image:: _static/images/151000main_marsdrill2033.png
    :width: 45 %
	
Image Credit: NASA/Ames

.. _k10_rover:

`K10 Rover <http://ti.arc.nasa.gov/tech/asr/intelligent-robotics/>`__
---------------------------------------------------------------------

PLEXIL was used to operate the K10 rover in a coordinated demonstration
of Human Robot Interaction, Surface Handling and Surface Mobility
Systems. NASA Ames Research Center's K10 rovers are field work rovers
designed for human-paced operational tasks such as assembly and
inspection. Running software developed at Ames, the K10 rover performed
an autonomous 360-degree inspection of the larger SCOUT rover, taking a
series of high-resolution pictures at pre-determined locations.


.. image:: _static/images/K10-and-scout_v.jpg
    :width: 45 %
.. image:: _static/images/K10_v.jpg
    :width: 45 %
	
Image Credit: NASA/Ames

.. toctree::
   :caption: Getting Started
   :maxdepth: 1
   :hidden:

   Introduction <GettingStarted/Introduction>
   Requirements <GettingStarted/Requirements>
   Installation <GettingStarted/Installation>
   Tutorial <GettingStarted/Tutorial>
   Licensing <GettingStarted/Licensing>


.. toctree::
   :caption: PLEXIL Language
   :maxdepth: 1
   :hidden:

   PLEXIL Overview <PLEXILLanguage/PLEXILOverview>
   PLEXIL References <PLEXILLanguage/PLEXILReferences>
   PLEXIL Semantics <PLEXILLanguage/PLEXILSemantics>
   Resource Model <PLEXILLanguage/ResourceModel>

.. toctree::
   :caption: PLEXIL Execution
   :maxdepth: 1
   :hidden:

   PLEXIL Executive <PLEXILExecution/PLEXILExecutive>
   PLEXIL Simulators <PLEXILExecution/PLEXILSimulators>
   PLEXIL Viewer <PLEXILExecution/PLEXILViewer>
   Communication <PLEXILExecution/Communication>
   Inter-Executive Communication <PLEXILExecution/Inter-ExecutiveCommunication>
   UDP Adapter <PLEXILExecution/UDPAdapter>
   ResourceArbiter <PLEXILExecution/ResourceArbiter>
   Plan Persistence and Checkpoints <PLEXILExecution/PlanPersistenceandCheckpoints>

.. toctree::
   :caption: PLEXIL Tools
   :maxdepth: 1
   :hidden:

   PLEXILTools/PLEXILChecker
   PLEXILTools/XMLSchemaEmacs
   
.. toctree::
   :caption: Interfacing
   :maxdepth: 1
   :hidden:

   Interfacing Overview <Interfacing/InterfacingOverview>
   Interface Configuration File <Interfacing/InterfaceConfigurationFile>
   Standard Libraries <Interfacing/StandardLibraries>
   The Application Framework <Interfacing/TheApplicationFramework>
   Implementing Custom Interfaces <Interfacing/ImplementingCustomInterfaces>
   Implementing Custom Applications <Interfacing/ImplementingCustomApplications>
   API Reference <Interfacing/APIReference>
   
   
.. toctree::
   :caption: Release Notes
   :maxdepth: 1
   :hidden:

   PLEXIL-4 Release Notes <ReleaseNotes/PLEXIL4ReleaseNotes>
   
.. toctree::
   :titlesonly:
   :caption: Appendices
   :maxdepth: 1
   :hidden:

   Node State Diagrams <Appendices/NodeStateDiagrams>
   Example Plans <Appendices/ExamplePlans>
   RoboSim <Appendices/RoboSim>
   SimulatorNotes <Appendices/SimulatorNotes>
   References <Appendices/References>


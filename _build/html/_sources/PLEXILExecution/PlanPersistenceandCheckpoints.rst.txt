.. _PlanPersistenceandCheckpoints:

Plan Persistence and Checkpoints
===================================

*Apr 7, 2021*

This chapter describes |PLEXIL|'s Checkpoint Adapter, available for
inclusion in any |PLEXIL| plan.

.. contents::

Overview
--------

The Checkpoint Adapter enables plan persistence by allowing plans to set
and read "checkpoints" (identified by a String name), which contain the
state (true/false/Unknown (if not set)), the last modified time (as a
Real), and a user-set information String.

Checkpoints are associated with a boot - that is, if the plan were to
set a checkpoint "abc" to true, it would be associated with Boot 0. If
the executive were to crash or shut down, all checkpoints are "promoted"
by one boot - "abc" would now be true in Boot 1 and Unknown in Boot 0.

Metadata about each boot and the number of boots is also saved. This
includes time of boot, time of last save, and an is_ok flag. Boots (and
checkpoints associated with the boot) other than Boot 0 (current boot)
are not modifiable other than the is_ok variable.

Further, the total number of boots ever recorded, the number of boots
accessible to the executive, and the number of boots with is_ok=false
are available through Lookups. The number of accessible boots may not
always be the same as the total number of boots if using a SaveManager
other than SimpleSaveManager.

The save management strategy can be modified by extending SaveManager.
The only current implementation, SimpleSaveManager, stores all previous
boots and writes to disk on every command sent to CheckpointAdapter.

Any command or lookup which has an incorrect number of arguments prints
an error message to cerr and returns Unknown.

The adapter will use the same time adapter available to the plan. If
that adapter can't handle the time lookup or returns Unknown, any
time-dependent values will be set to Unknown.

Usage
-----

If the adapter is included in the configuration file, all lookups and
commands will be available to the plan, and startup/shutdown behavior is
enabled(as described below in :ref:`Options <Options>`).

An example configuration follows:

::

     <Adapter AdapterType="CheckpointAdapter">
       <SaveConfiguration Directory="./saves"
                  RemoveOldSaves="true"/>
       <AdapterConfiguration OKOnExit="true"
                 FlushOnExit="true"
                 FlushOnStart="true"
                 UseTime="true" />
     </Adapter>

.. _Options:

Options
-------

The adapter has multiple settings which can be modified in the interface
configuration file. See the example in examples/checkpoints for the
exact layout.

::

   AdapterConfiguration:
       OKOnExit: Whether is_ok is set to true for the current boot when the executive exit normally. It is not recommended to set this without also setting FlushOnExit.
           Default: "True"
       FlushOnStart: Whether a flush is called at startup to make sure every boot is logged
           Default: "True"
       FlushOnExit: Whether checkpoint data/is_ok/last_save_time is flushed to disk on exit
           Default: "True"
       UseTime: Whether the checkpoint system attempts to find and use a time adapter. This is useful because if an adapter registered to handle time isn't found, the PLEXIL interface manager returns the default adapter as the "time" adapter. Attempting to use this anyway will result in slightly reduced performance and potentially a large number of warnings (the CheckpointAdapter will continue to function, however).
           Default: "True"
   SaveConfiguration:
       Directory: The directory in which saves (in the form x_save.xml or x_save.part) are saved
           Default: "./saves"
       RemoveOldSaves: If false, older saves aren't deleted as new ones are added. If true, a maximum of two save files will be maintained. This can be useful in testing and debugging.
           Defailt: "True"
       NOTE: All attributes of SaveConfiguration are sent to the SaveManager, so different implementations may have a larger set of configuration options

.. _api_reference:

API Reference
-------------

The Lookups are as follows:

::

   Integer NumberOfTotalBoots () //Returns the total number of boots ever logged

   Integer NumberOfAccessibleBoots () //Returns number of boot entries available to the plan

   Integer NumberOfUnhandledBoots () //Returns number of boot entries which are not marked as is_ok

   Boolean DidCrash() // Returns true if IsOK(1)==false, false otherwise. 
                      // A true value indicates that the executive shut down without setting is_ok

    
   // If a lookup attempts to query a boot that doesn't exist, Unknown is returned

   Boolean IsBootOK ( Integer boot=0 ) //Returns the is_ok flag in the selected boot, which starts off as false when the boot begins and may be affected by the OKOnExit configuration

   Integer TimeOfBoot ( Integer boot=0 ) //Returns the time that CheckpointAdapter.start() is called by the interface adapter, Unknown if time not available

   Integer TimeOfLastSave ( Integer boot=0 ) //Returns the time of the last save to disk


   // If a lookup attempts to query a checkpoint that doesn't exist, Unknown is returned

   Boolean CheckpointState ( String name, Integer boot=0 ) // If checkpoint not set, Unknown

   Real CheckpointTime ( String name, Integer boot=0 ) //Returns last modified time,

   String CheckpointInfo ( String name, Integer boot=0 ) // User defined string

   Integer CheckpointWhen( String name ) //Return the most recent boot number where the checkpoint has been set, Unknown if none

The Commands are as follows:

::

   // All commands return handle RECEIVED_BY_SYSTEM when subsequent lookups will return the new value, and COMMAND_SUCCESS when the change is saved to persistent memory.

   // Note that as no guarantees are made about when the changes from set_boot_ok and set_checkpoint will be saved to persistent memory other than via the flush_checkpoints command, so using them with SynchronousCommand is highly discouraged
   // Also note that some characters aren't handled by pugixml which SimpleSaveManager uses - for example, using "|" (pipe) in the info string leads to runtime errors

   Integer set_boot_ok(Boolean state=True, Integer boot=0 ) //Sets IsOK, returns previous value

   Boolean set_checkpoint( String name, Boolean value=true, String info="") //Overrides any existing checkpoint, returns the previous state of the checkpoint (Unknown if checkpoint did not exist)

   Boolean flush_checkpoints() //Flushes all changes (including set_boot_ok) to disk

Structure
---------

The Checkpoint Adapter consists of three major components – the adapter
itself, the underlying checkpoint system, and the save manager. The
adapter receives commands from the |PLEXIL| executive, replaces missing
arguments with their default values, and calls the appropriate function
in the checkpoint system. The checkpoint system maintains a data
structure that tracks checkpoints and boots. After each command is
processed, it indicates to the adapter that the command has been
received and. The save manager can view the data structure maintained by
the checkpoint system and is responsible for writing to persistent
memory.

There are no constraints on when the save manager will write to
persistent memory other than during a Flush command, making the
checkpoint adapter flexible for use in different environments. A diagram
of the adapter is pictured below.

.. figure:: ../_static/images/Command_Flow.png
   :alt:  Command_Flow


Extending
---------

If a different saving strategy is desired, this can be done by extending
the SaveManager class and changing which SaveManager is used in
CheckpointSystem.hh

The checkpoint adapter currently resides in ``plexil/examples/checkpoint``

In the checkpoint adapter directory there are a number of tests,
including a general usage test described in the README, and a fuzz-like
test of the SaveManager implementation in the testing directory. It is
encouraged to run the fuzz-like test for a sufficiently long time if a
different SaveManager extension is implemented.


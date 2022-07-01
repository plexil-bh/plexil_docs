.. _NodeStateDiagrams:

Node State Diagrams
===================

*29 May 2015*

These illustrations describe the node state transition semantics of
PLEXIL nodes. The previous version of this page is found
`here <Original_Node_State_Transition_Diagrams>`__; it was written
before the addition of Exit Condition, uses a different notation, and
does not capture the exact behavior of the Plexil Executive in a few
areas.

.. contents::

.. _inactive_state:

INACTIVE state
--------------

Effective with the PLEXIL 3 release, INACTIVE nodes behave as shown:

.. figure:: ../_static/images/Inactive_revised_2013-03-06.png
   :alt: Inactive state for all node types

   Inactive state for all node types

.. _waiting_state:

WAITING state
-------------

Nodes in the WAITING state transition directly to FINISHED when their or
some ancestor's ExitCondition becomes true.

.. figure:: ../_static/images/Waiting_with_Exit_condition.png
   :alt: Waiting state for most node types

   Waiting state for most node types

For Assignment nodes, the assignment takes place when the node
transitions to EXECUTING:

.. figure:: ../_static/images/Waiting_Assignment_nodes_with_Exit_condition.png
   :alt: Waiting state for Assignment nodes

   Waiting state for Assignment nodes

.. _executing_state:

EXECUTING state
---------------

.. _executing___empty_nodes:

EXECUTING - Empty nodes
~~~~~~~~~~~~~~~~~~~~~~~

Empty nodes transition directly to ITERATION_ENDED when their
ExitCondition becomes true, or to FINISHED if some ancestor's becomes
true.

.. figure:: ../_static/images/Executing_Empty_nodes_with_Exit_condition.png
   :alt: Executing_Empty_nodes_with_Exit_condition.png

   Executing_Empty_nodes_with_Exit_condition.png

.. _executing___assignment_nodes:

EXECUTING - Assignment nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Assignment nodes transition to FAILING and restore the variable's
previous value if their or some ancestor's ExitCondition becomes true,
or their or some ancestor's InvariantCondition becomes false. Otherwise
Assignment nodes transition to ITERATION_ENDED when the ExitCondition
becomes true; the PostCondition determines the outcome.

.. figure:: ../_static/images/Executing_Assignment_nodes_with_Exit_condition.png
   :alt: Executing_Assignment_nodes_with_Exit_condition.png

   Executing_Assignment_nodes_with_Exit_condition.png

.. _executing___command_nodes:

EXECUTING - Command nodes
~~~~~~~~~~~~~~~~~~~~~~~~~

Command nodes transition to FINISHING in the nominal case. In the event
of an ExitCondition true or InvariantCondition false, they transition to
FAILING to abort the command.

Note that the supplied EndCondition is ORed with
``(command_handle == COMMAND_DENIED || command_handle == COMMAND_FAILED)``
. This allows the node to transition in the event the resource arbiter
rejects the command.

.. figure:: ../_static/images/Executing_Command_with_Exit_and_FINISHING.png
   :alt: Executing_Command_with_Exit_and_FINISHING.png

   Executing_Command_with_Exit_and_FINISHING.png

.. _executing___update_nodes:

EXECUTING - Update nodes
~~~~~~~~~~~~~~~~~~~~~~~~

Update nodes behave similarly to Assignment nodes.

Note that the supplied EndCondition is ANDed with update-complete.

.. figure:: ../_static/images/Executing_Update_with_Exit_condition.png
   :alt: Executing_Update_with_Exit_condition.png

   Executing_Update_with_Exit_condition.png

.. _executing___nodelist_and_librarynodecall_nodes:

EXECUTING - NodeList and LibraryNodeCall nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

NodeList and LibraryNodeCall nodes transition to FAILING if their or
some ancestor's ExitCondition becomes true.

Note that the default EndCondition for these node types is all children
in FINISHED state.

.. figure:: ../_static/images/Executing_List_nodes_with_Exit_condition.png
   :alt: Executing_List_nodes_with_Exit_condition.png

   Executing_List_nodes_with_Exit_condition.png

.. _finishing_state:

FINISHING state
---------------

In general, the FINISHING state waits for completion of actions that may
take an indeterminate time. If the node fails while waiting for
completion, it transitions to FAILING.

.. _finishing___command_nodes:

FINISHING - Command nodes
~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: ../_static/images/Finishing_Command_with_Exit_condition.png
   :alt: Finishing_Command_with_Exit_condition.png

   Finishing_Command_with_Exit_condition.png

.. _finishing___nodelist_and_librarynodecall_nodes:

FINISHING - NodeList and LibraryNodeCall nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: ../_static/images/Finishing_with_Exit_condition.png
   :alt: Finishing_with_Exit_condition.png

   Finishing_with_Exit_condition.png

.. _failing_state:

FAILING state
-------------

In general, FAILING is used to finish recovery from an abnormal
situation.

.. _failing___assignment_nodes:

FAILING - Assignment nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~

Assignment nodes simply transition to FINISHED or ITERATION_ENDED as
appropriate. The variable has already been restored to its previous
value on the transition into FAILING.

Note that the previous as-implemented behavior was to assign UNKNOWN in
the event of a failure. The entire team agrees that restoring the
previous value is preferable.

.. figure:: ../_static/images/Failing_Assignment_nodes_with_Exit_condtion.png
   :alt: Failing_Assignment_nodes_with_Exit_condtion.png

   Failing_Assignment_nodes_with_Exit_condtion.png

.. _failing___command_nodes:

FAILING - Command nodes
~~~~~~~~~~~~~~~~~~~~~~~

Command nodes abort the command, wait for the abort to complete, then
transition to FINISHED or ITERATION_ENDED as appropriate.

.. figure:: ../_static/images/Failing_Command_with_Exit_condition.png
   :alt: Failing_Command_with_Exit_condition.png

   Failing_Command_with_Exit_condition.png

.. _failing___update_nodes:

FAILING - Update nodes
~~~~~~~~~~~~~~~~~~~~~~

Update nodes simply wait for the update to complete, then transition to
FINISHED or ITERATION_ENDED as appropriate.

.. figure:: ../_static/images/Failing_Update_with_Exit_condition.png
   :alt: Failing_Update_with_Exit_condition.png

   Failing_Update_with_Exit_condition.png

.. _failing___nodelist_and_librarynodecall_nodes:

FAILING - NodeList and LibraryNodeCall nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

NodeList and LibraryNodeCall nodes wait for all children to achieve
either the WAITING or FINISHED state before transitioning to FINISHED or
ITERATION_ENDED.

.. figure:: ../_static/images/Failing_List_node_with_Exit_condition.png
   :alt: Failing_List_node_with_Exit_condition.png

   Failing_List_node_with_Exit_condition.png

.. _iteration_ended_state:

ITERATION_ENDED state
---------------------

ITERATION_ENDED transitions directly to FINISHED if an ancestor's
ExitCondition becomes true.

.. figure:: ../_static/images/Iteration_Ended_with_Exit_condition.png
   :alt: Iteration_Ended_with_Exit_condition.png

   Iteration_Ended_with_Exit_condition.png

.. _finished_state:

FINISHED state
--------------

The Finished state is unchanged from the specification and previous
implemented behavior.

.. figure:: ../_static/images/Finished_Revised.png
   :alt: Finished_Revised.png

   Finished_Revised.png

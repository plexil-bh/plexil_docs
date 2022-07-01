.. _PLEXILSemantics:

PLEXIL Semantics
=======================

*10 Apr 2021*

Clear semantics make it easier to verify and validate automated plans
and to implement a lightweight executive that conforms to the semantics
of execution. This chapter elaborates the `Plexil
Reference <Plexil_Reference>`__ by describing detailed semantics for
various aspects of PLEXIL and its execution. The formal semantics
presented here are based entirely on *Core PLEXIL*.

Recall that Core PLEXIL is a subset of PLEXIL into which all other
constructs are translated for execution. Core PLEXIL consists
essentially of 5 different types of *nodes*, which are structures
containing a variety of data, a set of possible execution states and
outcomes, and rules specifying the exact execution of these nodes and
transitions between their states. A formal semantic framework of Core
PLEXIL, together with operational, small-step semantics for its
execution, is presented in detail in `References <References>`__ 1. Much
of the information given here is a distillation of these papers.

.. contents::

.. _plexil_execution:

PLEXIL Execution
----------------

The execution of a PLEXIL plan is described formally using five
relations that constitute a *small-step semantics* for the language.

-  The *atomic* relation describes the state transition of an individual
   node.
-  The *micro* relation is the synchronous execution of the atomic
   relation (i.e. transitions for all active nodes in a plan).
-  The *quiescence* relation is the "run until completion" of the micro
   relation.
-  The *macro* relation describes how PLEXIL reacts to events in the
   external world.
-  The *execution* relation is the N-step iteration of the macro
   relation.

A PLEXIL plan executes in discrete *steps* within each of these
relations. The following sections describe some of these steps in more
detail. See `References <References>`__ 1 for a thorough description.

.. _atomic_steps:

Atomic Steps
------------

The execution semantics of an individual PLEXIL node is specified in
terms of node states and transitions between node states that are
triggered by condition changes.

.. _node_states:

Node States
~~~~~~~~~~~

Each node must be in one and only one of the following states at any
given time.

-  Inactive
-  Waiting
-  Executing
-  Finishing
-  Iteration_Ended
-  Failing
-  Finished

.. _node_transitions:

Node Transitions
~~~~~~~~~~~~~~~~

The set of condition changes that effect node state transitions are as
follows.

The first set are user-specified *gate conditions,* which initiate a
transition:

-  **``SkipCondition``** T : The skip condition changes from unknown or
   false to true.
-  **``StartCondition``** T : The start condition changes from unknown
   or false to true.
-  **``InvariantCondition``** F : Invariant condition changes from true
   or unknown to false.
-  **``ExitCondition``** T : Exit condition changes from unknown or
   false to true.
-  **``EndCondition``** T : End condition changes to unknown or false to
   true.
-  **``RepeatCondition``** T/F : the repeat condition changes from
   unknown to either true or false.

The second set are user-specified *check conditions,* which cannot
initiate a transition, but can affect which transition is taken:

-  **``PreCondition``** T : The precondition is checked at the
   transition out of the Waiting state, when the **``StartCondition``**
   becomes true.
-  **``PostCondition``** T : The postcondition is checked at the
   transition out of the Executing state, when the **``EndCondition``**
   becomes true.

The conditions below are internally generated and not directly
accessible to users:

-  **``Ancestor_invariant_condition``** F : The invariant condition of
   any ancestor changes to false.
-  **``Ancestor_exit_condition``** T : The exit condition of any
   ancestor changes to true
-  **``Ancestor_end_condition``** T : The end condition of any ancestor
   changes to true
-  **``All_children_waiting_or_finished``** T : This is true when all
   child nodes are in either in node state waiting or finished and no
   other states.
-  **``Command_abort_complete``** T : When the abort for a command node
   is completed.
-  **``Parent_waiting``** T : The parent of the node transitions to node
   state waiting.
-  **``Parent_finished``** T : The parent of the node transitions to
   node state finished.
-  **``Parent_executing``** T : The parent of the node transitions to
   node state executing.

The `Node State Transition Diagrams <Node_State_Transition_Diagrams>`__
document all the PLEXIL node transitions.

.. _nominal_execution:

Nominal Execution
~~~~~~~~~~~~~~~~~

At the beginning of plan execution, all the nodes in a plan are
initialized to state Inactive. An Inactive node does not affect the
external system at all.

From Inactive, root nodes transition directly to state Waiting. Interior
and leaf nodes transition to state Waiting when their parent enters
state Executing. A node remains in state Waiting until either its skip
condition or start condition is met.

If the start condition becomes True, and the precondition is not False,
the node enters state Executing. The default start condition is True,
which implies that the node may execute immediately upon activation. In
state Executing, a node or its children perform the specified actions.

Upon completion of the action (e.g. command sent or assignment
finished), leaf nodes transition to state Iteration_Ended, from which
they can transition either to state Waiting if the repeat condition is
True, or to state Finished.

List and library call nodes proceed similarly from state Executing to
state Iteration_Ended when their end conditions are satisfied. The
default end condition for these nodes is all children being in state
Finished. From Iteration_Ended list and library call nodes transition to
state Finishing. Only when all children of a node are in either state
Waiting or state Finished can the parent node transition to Finished.

If an ancestor node repeats, the node may transition from Finished to
Inactive. Otherwise the node remains in Finished and cannot be
reactivated.

.. _execution_in_the_presence_of_failures:

Execution in the Presence of Failures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Execution failure can be invoked in several ways:

-  If a precondition is False when the start condition is True, the node
   bypasses the Executing state.
-  If an invariant condition becomes False while the node is in state
   Executing, execution is aborted.
-  If a postcondition is False when the end condition is True, the node
   has already completed execution.

In each of these cases, the node's outcome is set to Failed.

The exit condition behaves similarly to the invariant condition, but
with a reversed logical sense and a distinct outcome value. If an exit
condition becomes True while the node is in state Executing, execution
is aborted with an outcome of Interrupted.

When a failure or interruption occurs, the next state transition and
execution action depends on the node type. Empty nodes transition
directly to either state Iteration_Ended or state Finished (see below
for the distinction).

All other node types go through the Failing state on the way to
Iteration_Ended or Finished:

-  Assignment nodes retract the assignment and enter state Failing, then
   transition immediately to Iteration_Ended or Finished.
-  Command nodes abort the command and enter state Failing, to wait for
   acknowledgement of the abort. Upon receipt they then transition to
   Iteration_Ended or Finished.
-  Update nodes enter state Failing and wait for acknowledgement of the
   update, at which point they transition to Iteration_Ended or
   Finished.
-  NodeList and LibraryNodeCall nodes cause the sub-tree below them to
   abort in a deterministic manner. Only in the case of a failure or
   interruption does a parent node abort a child node.

All nodes transition to state Iteration_Ended if the failure or
interruption was local, or to state Finished if the parent failed or was
interrupted. The cause of a node's failure can be determined from the
Failure Type field of the node.

.. _node_termination_and_outcome:

Node Termination and Outcome
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three main causes for node termination: completion of
execution, external events, and faults.

The default completion of a node depends on the type of node:

-  An Empty node ends immediately upon entering the Executing state.
-  An Assignment node ends immediately upon entering the Executing
   state.
-  A Command node ends when a *command handle* (a kind of
   acknowledgement) is returned from the external interface
-  An Update node ends when an acknowledgement of the update is returned
   from the external interface.
-  A NodeList node ends when all its child nodes have finished.
-  A LibraryNodeCall node is effectively a NodeList node with one child;
   it ends when its child has finished.

External events or cascading effects of external events may satisfy the
explicit end condition of a node. When the end condition of a NodeList
node is satisfied and some of its child nodes are still executing, the
children will terminate cleanly, in the sense they are not aborted but
rather continue executing and finish according to the node transition
rules. As one technique to aid in clean termination, a plan can allow
the running of “clean-up” nodes to ensure safe termination of processes.
The semantics of clean termination of a NodeList node with running
children are:

-  Only currently executing and just-activated child nodes continue to
   run
-  Pending child nodes whose start conditions are not satisfied do not
   run
-  Parent node waits for active child nodes to finish executing

Faults can also drive node termination. A node fails if its invariant
condition is violated or its pre- or post-conditions are not satisfied.
When a node fails it aborts its child nodes without clean-up. When a
node fails no more events are processed by the sub-tree rooted at that
node. All clean-up actions must be handled by sibling nodes.

*Outcome* is a node attribute that provides additional information about
the result of node execution. The outcome of a node records the cause of
termination. A node will have at most one of the following outcomes:

-  Unknown: the initial outcome value.
-  Success: the node completed execution normally.
-  Failure: the node or one of its ancestors had an InvariantCondition
   become False while the node was in Executing or Iteration_Ended
   state.
-  Interrupted: the node or one of its ancestors had an ExitCondition
   become True while the node was in Executing or Iteration_Ended state.
-  Skipped: the node's SkipCondition became true, or its parent had an
   EndCondition or ExitCondition become False, had an InvariantCondition
   become true, or otherwise transitioned to Finished, while the node
   was in Inactive or Waiting state.

The node outcome is initialized to Unknown. The outcome is set to
Skipped if the node did not run, and to Success if the current iteration
completed successfully. The outcome is set to Failure if a failure
happened, and to Interrupted if the node or any of its ancestors' exit
conditions became true. Outcomes provide information only for the
current iteration; they are reset for repeating nodes.

*Failure type* records the reason for a Failure or Interrupted outcome.
The failure type is initialized to Unknown, and upon failure or
premature exit can take one of the following values:

-  Precondition failed
-  Postcondition failed
-  Invariant condition failed
-  Parent failed
-  Exit condition became true
-  Parent exited

Note that all conditions are checked once upon transition to a state in
which they apply. Only the condition changes listed above cause node
state transitions, e.g. a start condition changing to true causes node
state transitions, but the start condition changing to false does not
cause any node state transitions. Once a condition is enabled it stays
enabled until it is explicitly reset. The conditions are only reset for
repeating nodes.

The complete set of node state transitions that govern the semantics of
the execution of a single node are provided in the `Node State
Transition Diagrams <Node_State_Transition_Diagrams>`__. In certain
states, e.g. state Waiting, all node types have the same semantics. In
other cases, such as state Executing, the semantics depend on the node
type (NodeList, Command, Assignment, …). For efficiency we represent
ancestor end conditions. These are easily computed from the immediate
parent and child nodes. In principle, a node only needs to know about
its single immediate parent and all child nodes.

.. _node_conditions:

Node Conditions
~~~~~~~~~~~~~~~

The default value of each node condition for each node type is given in
the following tables. All node types share common defaults for most of
the conditions:

============= ===== ===== ==== ========= ===== ====== ====
Condition     Start Skip  Pre  Invariant Exit  Repeat Post
Default Value True  False True True      False False  True
============= ===== ===== ==== ========= ===== ====== ====

End condition defaults vary by node type:

+-----------+-------+-----------+---------+-----------+-----------+
| Node Type | Empty | List,     | Command | A         | Update    |
|           |       | Library   |         | ssignment |           |
|           |       | Call      |         |           |           |
+-----------+-------+-----------+---------+-----------+-----------+
| End       | True  | all       | True    | True      | update    |
| Condition |       | children  |         |           | ack       |
|           |       | finished  |         |           | nowledged |
+-----------+-------+-----------+---------+-----------+-----------+

For any node condition that is not explicitly specified, the default
value from the tables above is used. When a condition is given
explicitly, it replaces the default value in all but the following
cases:

-  The actual End Condition of Command nodes is the disjunction of the
   given expression and the command handle value being either
   COMMAND_FAILED or COMMAND_REJECTED. This is to allow commands
   rejected by the resource arbitrator and failed commands to terminate.
-  The actual End Condition of Update nodes is the conjunction of the
   given expression and the update acknowledgement.

.. _micro_steps_macro_steps_and_the_quiescence_cycle:

Micro Steps, Macro Steps, And the Quiescence Cycle
--------------------------------------------------

Execution of a PLEXIL plan occurs in three classes of steps: *micro
steps*, *macro steps*, and *quiescence steps*.

A *quiescence step* or *quiescence cycle* begins when the executive is
notified of an external event. This could be the addition of a new plan,
a timer expiration, a command or update acknowledgement, or other
external change of state. The executive reads the current external state
and begins a *macro step*. The quiescence cycle ends when no nodes are
eligible to transition and all side effects (e.g. variable assignments,
external actions) have been performed, i.e. when the executive is
quiescent.

A *macro step* consists of zero or more successive *micro steps*. The
current state of the plan and the external world are examined to see if
some node(s) are eligible for transition. A macro step ends when either
no further node state transitions are possible without additional input,
one or more Assignment nodes are executed, or one or more commands
and/or updates needs to be sent. At the end of every macro step, any
external actions are executed. Then if more transitions are possible due
to node condition changes resulting from the assignments or external
actions, another macro step is begun.

*Micro steps* correspond to transitions that modify only the local data
in the executive, i.e. node states, outcomes, failure types, and the
values of the timepoint variables. A micro step is defined as the
synchronous parallel execution of one or more transitions, as defined by
the `Node State Transition Diagrams <Node_State_Transition_Diagrams>`__.
All eligible transitions are performed in lockstep,
pseudo-simultaneously. If additional transitions are possible at the end
of a micro step, another micro step is started immediately.

PLEXIL does not make any assumption about the duration of execution of a
micro step. An assumption that is commonly made for synchronous
languages is that a step (in our case, a micro step) takes zero time.
Or, alternatively, that the external world changes (and therefore the
occurrence of external events happens) less frequently than the
execution of steps. This assumption is not mandatory in PLEXIL. If a
micro step takes more than zero time, this means that the execution of a
macro step also takes more than zero time. Since the external world may
keep changing in the meantime, it is possible that some micro steps
within the macro step use stale data. A similar situation occurs if an
event is processed much later than it was produced. In this case, it is
possible that the current status of the external variables associated
with the event might have changed.

**NOTE:** If more than one node is eligible for execution in a given
quiescence cycle, the actual order of execution is *unspecified*. In any
context where the exact node execution order really matters, it should
be encoded explicitly in the plan.

.. _semantics_of_lookups:

Semantics of Lookups
--------------------

*Lookups* simply read external state values. In other words, looking up
these states should not have any side effects. (An implementation of the
executive or external interface may choose to cache state values without
violating this requirement.) It is legal, and occasionally useful, for a
lookup to return UNKNOWN.

There are two types of lookups in PLEXIL: immediate (**``LookupNow``**)
and continuous (**``LookupOnChange``**).

**``LookupNow``** is executed during the quiescence cycle. The executive
blocks until a value is returned by the external interface.

**``LookupOnChange``** returns a value immediately as in the case of
**``LookupNow``**, and also causes an internal PLEXIL event to be
generated when the value has changed (i.e, the previous value is
different from the current value by more than the specified tolerance,
if any). If no tolerance is specified, any change in the external value
provokes the internal PLEXIL event.

Any **``LookupNow``** which is active for more than one quiescence
cycle, e.g. in an invariant or end condition, may have its value updated
in subsequent cycles, particularly (but not exclusively) if there is
also a **``LookupOnChange``** for the same state.

Example:

::

   StartCondition (LookupOnChange(“Rover battery level”) > 10.0 &&
                   powerTrackingNode.state == EXECUTING);

In this example, an asynchronous event is triggered whenever the rover
battery level changes. The state of the node **``powerTrackingNode``**
is maintained internally and it triggers an event when it changes. The
start condition is checked whenever such events occur.

--------------

Copyright (c) 2006-2022, Universities Space Research Association (USRA).
All rights reserved.

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

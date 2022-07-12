.. _PLEXILOverview:

PLEXIL Overview
====================

*10 Apr 2021*

.. contents::

.. _what_is_plexil:

What is PLEXIL?
---------------

Plan execution is a cornerstone in autonomy applications, such as
robotics, unmanned vehicles and habitats, and systems or simulations
involving intelligent software agents. |PLEXIL| (*PL*\ an E\ *x*\ ecution
*I*\ nterchange *L*\ anguage) is a language for representing plans for
automation.

|PLEXIL| was designed to meet the requirements of flexible, efficient and
reliable plan execution in space mission operations. It is compact,
semantically clear, and deterministic given the same sequence of events
from the external world. At the same time, the language is quite
expressive and can represent branches, loops, time- and event- driven
activities, concurrent activities, sequences, and temporal constraints.
The core syntax of the language is simple and uniform, making plan
interpretation simple and efficient, while enabling the application of
validation and testing techniques.

Accompanying |PLEXIL| is an execution engine, or *executive*, which
implements efficiently the |PLEXIL| language and provides interfaces to
controlled systems as well as decision support systems from which plans
may be sent. The |PLEXIL| software suite also includes a graphical plan
execution viewer, a static plan checker, and two different plan
simulators.

|PLEXIL| was originally developed as a collaborative effort between
researchers at NASA and Carnegie Mellon University, funded by NASA's
Mars Technology Program through the Research Institute for Advanced
Computer Science (RIACS) in the Universities Space Research Association
(USRA). Since then it has continually evolved through application on
NASA projects, which have included the control of prototype planetary
rovers and habitats, drilling equipment, and demonstration of adjustable
automation for International Space Station operations. See the
:ref:`References <references>` for more information on the background and
applications of |PLEXIL|.

.. _plexil_overview:

Overview
---------------

This section is a short overview of the |PLEXIL| language. It describes
abstractly the main features of |PLEXIL|, and explains how one programs in
|PLEXIL|. The next chapter, :ref:`PLEXIL Reference <PLEXILReferences>`,
describes in detail the programming constructs of |PLEXIL|. The :ref:`Detailed Semantics <PLEXILSemantics>`
chapter covers |PLEXIL| execution in
greater depth.

Nodes
~~~~~

A |PLEXIL| plan is built from *nodes* 

.. note::
    *nodes* were once called *actions* 
    
A node specifies something to do, either within the plan
itself, or to the external world. Nodes are the composable building
blocks from which arbitrarily complex behaviors can be specified. There
are many kinds of nodes in |PLEXIL|, and each is a programming construct
specifying a certain behavior.

A |PLEXIL| plan is a tree of nodes with a single root node. This tree
represents a hierarchical decomposition of tasks. High level tasks are
closer to the root node, while leaf nodes represent primitive behaviors
such as assigning to a variable or sending a command to the external
system.

The following diagram exemplifies a simple hierarchical plan. Its
representation in |PLEXIL| would have a similar tree structure.

.. figure:: ../_static/images/Sample-plan.png

--------------

Let's meet the nodes. The following UML diagram illustrates the essence
of |PLEXIL|.

.. figure:: ../_static/images/Plexil-uml.jpg

Stacked in a column on the right side of the diagram are the kinds of
*compound nodes*, which specify high level behaviors.

-  The *Concurrence* node groups child nodes to be executed in parallel.
-  The *Sequence*, *Unchecked Sequence*, and *Try* nodes group child
   nodes to be executed in the order listed, in various different ways.
-  The *IfThenElse*, *Do*, *While*, and *For* nodes provide conditional
   and iterative execution of their child nodes as implied by their
   names.
-  The *OnMessage* and *OnCommand* nodes are used in multi-executive
   applications and specify response policies.

Nodes are described in greater detail in the :ref:`Plexil Reference <PLEXILReferences>` chapter.

.. _core_plexil:

Core PLEXIL
~~~~~~~~~~~

The bottom row of nodes in the diagram above are *simple nodes* and
constitute a subset of the language called *Core PLEXIL*. These six
nodes provide all the computational power of |PLEXIL|. In fact, all the
compound nodes described above and shown on the right side of the
diagram are translated into a tree of simple nodes prior to execution.
Only Core |PLEXIL| is directly executed by the |PLEXIL| executive; the final
plan executed is a single tree of simple nodes.

Historical note: Originally, Core |PLEXIL| was the entire |PLEXIL| language.
Core |PLEXIL| was too primitive to write non-trivial plans, so it was
extended with higher level constructs, essentially "macros" that
translated into Core |PLEXIL| for execution; this extended language was
called *Extended PLEXIL*. Today, the full language is simply called
PLEXIL (or Plexil).

There are six types of nodes comprising Core |PLEXIL|. The interior nodes
in a plan are *List nodes*. List nodes have *child nodes* (or
"children") which can be of any type. A List node is called the *parent
node* (or "parent") of its child nodes. The remaining nodes are leaf
nodes in a plan.

-  An *empty node* can contain only attributes and performs no action.
-  An *assignment node* performs a local computation, whose value is
   assigned to a variable.
-  A *command node* issues commands to the system being operated on.
-  An *update node* provides information to the planning and
   decision-support interface.
-  A *library call node* invoke nodes located in an external library.

Conditions
~~~~~~~~~~

A node can specify up to eight *conditions*, which determine its
execution and outcome. There are nominal control conditions that specify
when the node should start executing, when it should finish executing,
when it should be repeated, and when it can be "skipped". These are
referred to collectively as the node's *gate conditions*.

-  A *start condition* specifies when the node should start execution.

-  An *end condition* specifies when the node should finish its
   execution.

-  A *repeat condition* specifies when the node should be made eligible
   for a repeat execution.

-  A *skip condition* specifies when the node's execution should be
   bypassed altogether.

Next, there are failure conditions that identify when execution is not
successful, and these are referred to collectively as a node's *check
conditions*.

-  A *precondition* is checked immediately after the start condition
   becomes true. If this check fails, the node will be aborted and have
   an outcome of failure. Preconditions are often used to verify that it
   is "safe" to execute the node.

-  A *postcondition* is checked after the node has completed execution.
   If this check fails, the node has an outcome of failure.
   Postconditions are often used to verify that a node had the intended
   effect.

-  An *invariant condition* is checked during node execution, and if it
   fails at any point, the node will be aborted and have an outcome of
   failure. Invariant conditions are often used to monitor conditions
   that are needed for the safe continued execution of the node.

Finally, there is a condition that says when to terminate a node
"prematurely" (i.e. before its end condition is satisfied), though
intentionally.

-  An *exit condition* is checked during node execution, and if it is
   satisfied at any point, the node will be terminate with an outcome of
   INTERRUPTED. The Exit condition can be used to effect deliberate plan
   cancellation. It is effectively the dual of the Invariant condition,
   which when false has the same effect but with a failure outcome.

Each condition specifies a logical expression. |PLEXIL| employs a
*ternary* logic in which logical expressions evaluate to one of True,
False, or Unknown.

Variables
~~~~~~~~~

A node may declare variables, which have *lexical scope*, i.e. they are
accessible to the node and all its descendants, but not siblings or
ancestors. Access to variables (for reading or writing) can be
restricted by use of *interfaces* in nodes. Interfaces are described in
the next chapter.

.. _state_outcome_and_introspection:

State, Outcome, and Introspection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A node is always in one of seven states, and always terminates with one
of four outcomes. The exact semantics for this behavior in Core |PLEXIL|
is given in the :ref:`Node State Transition Diagrams <NodeStateDiagrams>`.
A node can (e.g. in one of
its conditions) query the state and outcome of itself, its parent,
children, and siblings, but no other nodes; it can also query the start
and end times of any given state of these nodes.

.. _external_world:

External World
~~~~~~~~~~~~~~

|PLEXIL| reads the state of the external world or system through
*lookups*, which come in several varieties. |PLEXIL| can affect the state
of the external world or system through *commands*, which are sent
asynchronously.

Time
~~~~

Time (durations, elapsed time, clock time, etc) is often an essential
concept in automated and simulated systems. |PLEXIL| has no native concept
of time, per se. Time in |PLEXIL| is just another external state, and
|PLEXIL| has predefined the state named ``time`` for its operations that
involve time. The expressions in |PLEXIL| that *imply* time, i.e. those
that give the start and end times of node states, rely on the ``time``
external variable, which at present is a unitless real number.

Resources
~~~~~~~~~

|PLEXIL| has a simple resource model, described in detail in :ref:`Chapter 5 <ResourceModel>`.
In short, resource requirements for commands
(only) can be specified in command nodes, and these requirements are
checked during execution via a resource arbitration mechanism. Simple
models of unary, non-unary, hierarchical, and renewable resources are
supported.

.. _programming_in_plexil:

Programming in PLEXIL
~~~~~~~~~~~~~~~~~~~~~

.. _standard_plexil:

Standard Plexil
^^^^^^^^^^^^^^^

There are several means of programming in |PLEXIL|. The executable form of
|PLEXIL| is its XML representation. While many external applications have
*generated* |PLEXIL| code in its XML form, the XML form is not practical
for authoring |PLEXIL| directly.

The standard programming syntax for |PLEXIL| is described in this manual.
It is simply called Plexil, or sometimes *standard Plexil*. A translator
for Plexil, which converts user programs into Core |PLEXIL| XML, is
described in the :ref:`next chapter <PLEXILReferences>`.

Note that the terms "PLEXIL" and "Plexil" can refer to the abstract
|PLEXIL| language (nodes) and/or to its standard programming syntax.

.. _plexil_and_xml:

PLEXIL and XML
~~~~~~~~~~~~~~

As mentioned in the previous section, the executable form of |PLEXIL| is
represented in XML (Extensible Markup Language), a widely used standard
for information modeling. Although a |PLEXIL| user does not normally need
to be concerned with XML, it is important to note that the formal
specification of |PLEXIL|'s syntax is given at the XML level. Core |PLEXIL|
is specified by *XML schemas*. More information about |PLEXIL| XML is
given in the chapter titled :ref:`PLEXIL, XML, and Emacs <XMLSchemaEmacs>`.

.. _the_plexil_community:

The PLEXIL Community
--------------------

We have two mailing lists:

-  *plexil-discussion* email will be sent to anyone interested in
   |PLEXIL|.
-  *plexil-support* email will be sent only to the |PLEXIL| developers.

Please feel free to add yourself to *plexil-discussion* using the
`Sourceforge mailing lists <https://sourceforge.net/projects/plexil/support>`_


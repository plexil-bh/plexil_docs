.. _PLEXILReferences:

PLEXIL References
======================

*16 Aug 2021*

.. contents::

.. _about_this_chapter:

About this chapter
------------------

This chapter documents the Standard PLEXIL programming language, and how
to create and compile Plexil files for execution.

We explain PLEXIL primarily though examples, using Standard PLEXIL
syntax.

This is not a rigorous treatment of PLEXIL, but hopefully -- in
combination with the `Example PLEXIL Plans <Example_PLEXIL_Plans>`__ --
provides enough to enable you to program in PLEXIL effectively.

The PLEXIL langauge is summarized briefly in the previous chapter,
`Overview <Overview>`__. Its execution semantics are explained more
deeply in the following chapter, `Detailed
Semantics <Detailed_Semantics>`__, which cites papers that provide a
rigorous formal definition of the language. The PLEXIL syntax is
formalized in XML schemas found in the software distribution.

NOTE: Although this chapter focuses on the Standard PLEXIL language and
its semantics, as independent from the exact behavior of a particular
implementation of Plexil, explanations of how the supplied Plexil
Executive behaves have been provided in places we felt this information
helpful.

Nodes
-----

A Standard PLEXIL plan consists of one *node*, which may have child
nodes. A node is a structure specifying a certain kind of behavior.

A node is notated by a pair of curly braces containing the node's
specification, preceded optionally by a name for the node. The simplest
node is the empty node.

::

    { }

We can give it a name:

::

    DoNothing : { }

The node's name (also called its *Id*), denoted by an identifier and
colon preceding the opening brace, is optional. An anonymous (nameless)
node is valid, though it cannot be referenced anywhere explicitly,
except within the node itself (by using the ``Self`` keyword, described
below). In practice, every node has a name; an anonymous node is
assigned a unique name when compiled into PLEXIL's XML form for
execution.

A node and its parent, immediate children, and siblings (this structure
will be explained later) must have unique names. Uniqueness of names
across more distant relationships in a plan is not required, especially
since these nodes cannot reference each other (more on referencing scope
below).

Standard PLEXIL is case-sensitive, but whitespace-insensitive, so the
``DoNothing`` plan above can also be written, for example, in either of
the following ways:

::

    DoNothing:
    {
    }

::

    DoNothing: {
    }

Through composition, nodes form a tree-shaped hierarchy. The root of the
tree is the *root* or *top level* node. A Plexil source file must
contain exactly one top level node.

Nodes have two components: a set of *attributes* that drive the
execution of the node, and a "body" which specifies what the execution
of the node accomplishes.

Nodes which have no attributes may omit the enclosing braces. Examples
will be provided below.

.. _node_attributes:

Node Attributes
---------------

Nodes may contain *attributes*, which include local variables, an
*interface*, *conditions*, and a comment. Attributes are optional, and
some have specific default values. When attributes are specified, they
must occur *first* in node's form, i.e. immediately following the
opening curly brace.

Variables
~~~~~~~~~

A node may declare variables, which are local to the node. Plexil
currently supports variables of type Boolean, integer, real, string, and
arrays of these four basic types. Examples of declarations of the basic
types are as follows.

::

    Boolean isReset;
    Integer n;
    Real pi;
    String message;

These examples of variable declarations do not specify initial values
for the variables. Uninitialized variables of all types except arrays
are given the value `Unknown <#Data_Types_and_Expressions>`__. Here are
the same variable declarations with initial values specified. Initial
values must be literals -- expressions are not allowed.

::

    Boolean isReset = true;
    Integer n = 123;
    Real pi = 3.14159;
    String message = "hello there";

Arrays are declared by following the variable name with square brackets
containing the size of the array. Array variables do not default to
Unknown, but rather to an allocated array, all of whose *elements* are
Unknown. The first example below declares an array of 100 integers. The
second declares a smaller array of real numbers, with the first three
elements initialized (the remaining seven are Unknown).

::

    Integer scores[100];
    Real defaults[10] = #(1.3 2.0 3.5);

Variables have *lexical scope*, which mean they are visible only within
the node and any descendants of the node. Scope can be explicitly
limited using the Interface clause described below. Here is an example
of an empty node that declares some variables.

::

    DoNothing1:
    {
    String name = "Fred";
    Real MaxTemp = 100.0;
    }

So far we've using empty nodes as examples simply because we haven't yet
introduced the other nodes. The example above is illustrative but would
serve no practical purpose, since its variables cannot be used in any
way.

*NOTE: Variable declarations and interface declarations (described in
the following section) must occur prior to any other kinds of attributes
in a node definition. They may be intermixed.*

Interface
~~~~~~~~~

A node's *interface* is the set of variables it can read and/or write
(assign) to. By default, the interface of a node N is the union of its
parent's interface and the variables declared in N. Interface clauses
impose a *restriction* on the set of variables inherited from the
parent's interface by specifying the *only* variables from the parent
that are accessible. There are two kinds of interface clauses. The
``In`` clause specifies variables that can be read. The ``InOut`` clause
specifies those that can be read or written. All stated variables must
be part of the parent's node's interface, otherwise the clause is in
error. Furthermore, read-only variables in the parent cannot be declared
``InOut``. Here's an example of an empty node with interface clauses.

::

   Test:
   {
     In Integer x, y;
     InOut String z;

     Integer a, b;
   }

Variables ``x`` and ``y`` are assumed to be readable, and ``z`` readable
and writable, in Test's parent node. No other variables in Test's
ancestors will be accessible. Variables ``a`` and ``b`` are local
variables in Test.

A node's interface variables are also called its *parameters*. It is an
error for a node to declare a variable having the same name as a
variable that appears in its interface.

*NOTE: Variable declarations (described in the previous section) and
interface declarations must occur prior to any other kinds of attributes
in a node definition. They can be intermixed.*

Conditions
~~~~~~~~~~

A node can specify up to eight *conditions* that govern precisely how
the node is executed. Exact details are described in the `Node State
Transition Diagrams <Node_State_Transition_Diagrams>`__ document.

::

    StartCondition      // Node won't begin until this is true
    EndCondition        // Node won't terminate until this is true
    ExitCondition       // Node will terminate (if executing) or be skipped (if waiting) if this is true
    RepeatCondition     // Node will repeat if this is true
    SkipCondition       // Node will be skipped if this is true when node begins
    PreCondition        // Node will fail if this is false when node begins
    PostCondition       // Node will fail if this is false when node ends
    InvariantCondition  // Node will fail if this is false while node is executing

A condition specifies a Plexil *Boolean expression*. Expressions are
described in a section `below <#Data_Types_and_Expressions>`__. Here are
some varied examples of conditions:

::

   StartCondition Node1.outcome == SUCCESS;

   EndCondition SignalEndOfPlan.state == FINISHED ||
                SendAbortUpdate.state == FINISHED ||
                abort_due_to_exception;

   PreCondition Request_Human_Consent.state == FINISHED &&
                Lookup(ZZZZCWEC5520J) == 1;

   PostCondition AtGoal;

   InvariantCondition Lookup(ZZZZCWEC5520J) == 1;

   RepeatCondition Count < 10;

Here is an example of an empty node with some declarations and
conditions:

::

   Step2:
   {
     Real temperature;
     Real MaxTemp = 100.0;

     StartCondition Step1.state == FINISHED;
     InvariantCondition temperature < MaxTemp;
   }

The conditions specify that this node should begin execution after node
Step1 finishes, and that the temperature should remain less than MaxTemp
throughout execution. (Note that Plexil does not provide *constants*,
only variables). Incidentally, this an an example of a potentially
useful empty node. Empty nodes are often used to *wait* for a condition
(expressed through the start condition) and/or to test or *verify* a
condition (expressed here through the invariant condition).

Comments
~~~~~~~~

There are two kinds of comments in a Standard PLEXIL plan. The source
code can include comments to help document the code but that are not
preserved in the translated Core PLEXIL XML output. These are notated in
the C/C++ style syntax for block and single line comments. Examples of
each are as follows.

::

   /*
    * Here is a block comment example which
    * allows for multiple lines.
    */

   // Here is a single-line comment example that extends to the end of the line.

Second, Plexil nodes have the option of including a single ``Comment``
clause, which must be the first item in the node's attribute section.
Here's an example:

::

   Comment "This node verifies the robot's camera is functioning.";

The ``Comment`` clause gets preserved in the compiled (XML) version of
the plan, unlike other comments.

.. _leaf_nodes:

Leaf Nodes
----------

As described in the `Overview <Overview>`__, PLEXIL has many kinds of
nodes. The type of a given node is identified by the node's *body*. A
node's body is what immediately follows its attributes (described in the
previous sections).

Nodes that do not contain or decompose into child nodes form the leaves
in a PLEXIL plan tree. These nodes are called *nodes* and are part of
Core PLEXIL, which is the subset of PLEXIL that is executed directly.

.. _empty_node:

Empty Node
~~~~~~~~~~

All the examples presented above are empty nodes (also called empty
nodes). Empty nodes contain only attributes. They have no external
behavior (i.e. no direct effect on an external system or a plan
variable). In practice, empty nodes are quite useful and common. A
typical use is for verification of a state in the external world. Here's
a node that verifies a temperature reading.

::

    VerifyTemp:
    {
    PostCondition Lookup(engine_temperature) > 100.0;
    }

Assignment
~~~~~~~~~~

An assignment to a declared variable has the following form:

::

     <variable> = <expression>;

The part of the assignment, referred to as its left-hand side (LHS),
must be a writable variable in the node's interface. The ``expression``,
referred to as the right-hand side (RHS) of the assignment, can be any
PLEXIL expression. Expressions are described below. The type of the
expression must match the type of the variable.

An *assignment node*, is one that contains only an expression of this
form in its body. The following are examples of assignment nodes. Note
that some context, in particular the variables' declarations, are not
shown.

::

   IncrementCounter:
   {
     ExecutionCount = 1 + ExecutionCount;
   }

   CopyEntry:
   {
     TemperatureReadings[i] = x;
   }

As with other nodes, Assignment nodes without attributes may omit the
braces and/or names. The preceding examples could be rewritten as:

::

   IncrementCounter: ExecutionCount = 1 + ExecutionCount;

   TemperatureReadings[i] = x;

.. _assignments_and_concurrency:

Assignments and Concurrency
^^^^^^^^^^^^^^^^^^^^^^^^^^^

If two nodes in a PLEXIL plan attempt to assign the same variable
simultaneously, this is an error condition. The PLEXIL compiler does not
detect the possibility of concurrent assignment, and unfortunately the
current PLEXIL executive behaves ungracefully when it is attempted: it
issues a message about the conflict and then aborts. If your plan
contains such nodes, this contention problem can be resolved with the
``Priority`` clause. Here's a trivial contrived example:

::

   ConcurrentAssignment: Concurrence
   {
     Integer x;

    A:
     {
       Priority 1;
       x = 0;
     }

    B:
     {
       Priority 2;
       x = 1;
     }
   }

Without the ``Priority`` clauses, a runtime error would result. The
``Priority`` clause *orders* the execution of nodes from the lowest
priority number to the highest. In this example, node A will execute
first, then B, and the final value of ``x`` will be 1. Though it's
possible that the ``Priority`` clause may have a legitimate application,
it is probably best to design your plans such that potentially
conflicting assignments are avoided.

Command
~~~~~~~

A *command* has the form:

::

    [<variable> =] <command_name> ([<argument_list>]);

where:

-  ``command_name`` is an identifier or a parenthesized string
   expression;
-  ``argument_list`` is an optional comma-separated list of zero or more
   arguments, which may be either literal values, variables, or array
   element references (other kinds of expressions are not supported).

The assignment of the command's return value (assuming it returns a
value) to a variable is optional, and if specified, must be a writable
variable in the node's interface.

A *command node*, is one that contains only an expression of this form
in its body. The following are examples of command nodes. Note that some
context is not shown, e.g. the declaration of the command (discussed
next) and that of the variable receiving the return value.

::

   StopRover: { stop(); }

   SetWaypoint: { set_waypoint (x,y,z); }

   GetSpeed: { speed = get_speed(); }

   PrintSpeed: { print("Got speed: ",speed); }

As with assignment nodes, if no attributes are required, the braces may
be omitted. Names may also be ommitted:

::

   StopRover: stop();

   SetWaypoint: set_waypoint (x,y,z);

   GetSpeed: speed = get_speed();

   print("Got speed: ",speed);

*Commands must be declared* at the top of the file in which they are
used. Here are declarations for the commands above, and a few more
examples:

::

   Command stop();

   Command set_waypoint(Real x, Real y, Real z);

   Real Command get_speed();

   // Parameter names are optional, though usually aid readability.
   Boolean Command set_speed (Real);

   String Command getMessage (Integer channel);

   // Ellipses specify that one or more arguments can be provided but don't restrict the types
   Command print(...);

By default, a command node finishes when the executive receives a
*command handle* for its command, via the PLEXIL external interface (see
the Interfacing section of this manual). See `Resource
Model <Resource_Model>`__ for a description of command handles.

Note that the finishing of the command node is distinct from the
finishing of the command itself; command execution may be ongoing even
after the node finishes. We elaborate further on this point.

.. _asynchronous_command_execution:

Asynchronous command execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Note that calls to commands do *not* wait, or block execution of the
plan. Rather, the command executes in the external system asynchronously
with the plan. To examine the progress of the command, the plan should
inspect its handle.

A command may take arbitrarily long to complete in the external system.
If the command returns a value, and this value is assigned to a variable
in the command node, the node should *wait* for the value, i.e. the
command's completion, before ending execution. This is accomplished with
an appropriate end condition. Here's an example:

::

    ConfirmProceed:
    {
      Boolean result;
      EndCondition isKnown(result);
      PostCondition result;
      result = QueryYesNo("Proceed with instructions?");
    }

In this example, the end condition makes the node wait for the command's
result (whose initial value is Unknown). It further stipulates, via the
postcondition, that *success* of this node requires a positive user
confirmation.

However, this idiom is cumbersome to code and difficult or impossible to
get right in the general case. For example, if a command assigns to a
variable that already has a value, the ``isKnown`` test is unhelpful.
Fortunately, Plexil provides a convenient form for *synchronous*
commanding -- see the `section <#Synchronous_Command>`__ below.

Optionally, command nodes may specify resource requirements for the
affected command. The syntax and semantics for this is described in the
`Resource Model <Resource_Model>`__ chapter.

.. _utility_commands:

Utility Commands
^^^^^^^^^^^^^^^^

Several convenient utilities, in the form of commands, are available in
Plexil. Currently there are two commands that print PLEXIL expressions
to the standard output stream (e.g. the Unix terminal).

-  ``print (exp1, exp2, ...)`` prints expressions without any added
   characters.
-  ``pprint (exp1, exp2, ...)``, short for "pretty print" is like
   ``print`` but adds spaces between the expressions and a final
   newline.

The utility commands are automatically available when running Plexil
through the `Test Executive <Executing_Plans#Test_Executive>`__.
Otherwise they are available by including the `Utility
Adapter <Standard_Interface_Libraries#Utility_Adapter>`__.

Update
~~~~~~

An Update node serves to relay information outside the executive. For
example, it can be used to update a planner or other system that has
invoked the executive, with status about execution of the plan. The
manner in which this information is sent is determined by the `external
interface <Interfacing_Overview>`__ for the executive. An update
consists of name/value pairs; an update should include one or more such
pairs. The **``Update``** keyword identifies an Update node, and has the
form:

::

     Update <name> = (<value> | <variable>) [, <name> = (<value> | <variable>) ]*;

where ``name`` is an identifier, and the right hand side is either a
``value`` which is a literal (e.g. 5, "foo"), or a ``variable`` which is
an identifier naming a declared variable that is visible to the node.
Any number of such name/value pairs can be given, separated by commas.

Here's an example:

::

    SendAbortUpdate:
    {
      StartCondition MonitorAbortSignal.state == FINISHED;
      Update taskId = taskTypeAndId[1], result = -2, message = "abort";
    }

As with assignment and command nodes, if no attributes are required, the
containing braces may be omitted.

.. _library_call:

Library Call
~~~~~~~~~~~~

A ``Library Call`` node has the following form as its body.

::

    LibraryCall <Id> [<alias_list>];

where is the ID of the invoked *library node*. The is an optional list
of *aliases*, which are pairs of the form

::

     <parameter> = <expression>

An alias allows one to rename/assign a node parameter (i.e. a variable
present in the interface of the library node) with an actual value or
declared variable.

Here's a contrived example of a call to trivial library node. The first
file defines the library node ``F``, and the second file contains a node
that calls ``F``. NOTE: These examples contain a declaration of the
library node using the historic

::

   LibraryAction

form. A newer version of PLEXIL (not yet released) aliases this with

::

   LibraryNode

.

::

   --- begin F.ple ---

   F:
   {
    In Integer i;
    InOut Integer j;
    j = j * j + i;
   }

   ---- end F.ple ----

   --- begin LibraryCallTest.ple ---

   LibraryAction F (In Integer i, InOut Integer j);

   LibraryCallTest:
   {
    Integer k = 2;
    LibraryCall F(i=12, j=k);
   }

A library node can be any type of node (e.g. Sequence, Command) but it
must be a *top level* node, that is, the outermost node in a file.
Conversely, any top level node can be used as a library node.

As with assignment, command, and update nodes, if no attributes are
required, the containing braces may be omitted:

::

   LibraryAction makePhoneCall(In Integer number);

   CallHome: LibraryCall makePhoneCall(number=5551212);

.. _library_call_execution:

Library Call execution
^^^^^^^^^^^^^^^^^^^^^^

Prior to execution of a Plexil plan, at every point of a library call, a
copy of the invoked library node is *statically* inserted in place of
the call. Hence, "call" is technically a misnomer, and the mechanism for
library execution is essentially a "macro" style code substitution. The
executed plan is a single monolithic node with all library calls
replaced by their invoked nodes. Thus, repeated "calls" to the same
library node can produce a large plan for execution.

.. _compound_nodes:

Compound Nodes
--------------

Compound nodes are translated into simple (Core PLEXIL) nodes prior to
execution. A Core PLEXIL plan is a tree consisting of the leaf nodes
described in the previous section, plus the List Node, described in this
section under Concurrence.

Sequence
~~~~~~~~

A Sequence executes its child nodes in the given order.

Because sequential execution is so often the intended and expected
behavior of a plan, the ``Sequence`` keyword is optional:

::

   {
     <node1>;
     ...
     <nodeN>;
   }

**Note:** Sequence is currently an alias for CheckedSequence. Because of
the overhead of checking for child node success, and the default
behavior in other sequential languages is to continue sequential
execution after a child node fails (e.g. shell scripting), future PLEXIL
release may instead alias it to UncheckedSequence. To ensure your plans
do not change behavior, please consider explicitly using either
CheckedSequence or UncheckedSequence.

CheckedSequence
~~~~~~~~~~~~~~~

A CheckedSequence executes its child nodes in the given order. If any
node fails (i.e. terminates with outcome ``FAILURE``), the
CheckedSequence also terminates with outcome ``FAILURE`` and failure
type ``INVARIANT_CONDITION_FAILED``. A CheckedSequence succeeds if and
only if all its nodes succeed. An empty CheckedSequence always succeeds.

A CheckedSequence is denoted as follows.

::

   CheckedSequence
   {
     <node1>;
     ...
     <nodeN>;
   }

.. _unchecked_sequence:

Unchecked Sequence
~~~~~~~~~~~~~~~~~~

An UncheckedSequence simply executes its child nodes in the given order.
An UncheckedSequence succeeds by default.

::

   UncheckedSequence
   {
     <node1>;
     ...
     <nodeN>;
   }

Concurrence
~~~~~~~~~~~

A Concurrence encloses zero or more child nodes, which are executed
*concurrently*. Precisely, there are no execution constraints on the
child nodes other than those imposed by explicit conditions (those found
in each child node as well as in the Concurrence form itself).

Concurrence translates directly to a Core PLEXIL List node.

::

   Concurrence
   {
     <node1>;
     ...
     <nodeN>;
   }

A Concurrence finishes when all its children have finished. If a
different behavior is desired, such as ordering constraints between
children, or finishing before all children have executed, this behavior
must be specified explicitly through conditions in the Concurrence and
its children. Here is a contrived example that illustrates a Concurrence
with a particular execution protocol:

::

   Command inform(String message);
   Boolean Command DoIt(Integer n);

   Root: Concurrence
   {
     Integer x;

     Inform:
      inform("Plan executing...");

     Init:
       x = GetX();

     Commence:
     {
       Boolean result;
       StartCondition Init.state == FINISHED;
       PostCondition result;
       SynchronousCommand result = DoIt(x);
     }

     InformSuccess:
     {
       StartCondition Commence.outcome == SUCCESS;
       inform("Operation succeeded!");
     }

     InformFailure:
     {
       StartCondition Commence.outcome == FAILURE;
       inform("Operation failed!");
     }
   }

In the example above, the Inform and Init nodes are unconstrained --
they can start immediately. The Commence node waits for Init to finish
before it can start. After it finishes, either InformSuccess or
InformFailure will execute, depending on the result.

NOTE: If more than one child node is eligible for execution at a given
moment, and PLEXIL is being executed on a sequential machine, the actual
order of execution is *unspecified*. In any context where the exact
execution order of nodes really matters, it must be encoded explicitly
in the plan.

Try
~~~

In a Try sequence, the child nodes are executed in sequence, *until* one
succeeds. The remaining nodes are skipped. A Try succeeds if and only if
one of its nodes succeed. An empty Try always fails.

The PLEXIL Try is distinct from the try-catch idiom found in many
popular programming languages.

::

   Try
   {
     <node1>;
     ...
     <nodeN>;
   }

.. _if_elseif_else:

If-Elseif-Else
~~~~~~~~~~~~~~

This is the traditional *if-then-else* construct, with optional "elseif"
and "else" parts. The ``if`` and optional ``elseif`` clauses each
specify a condition, and a node to execute if this condition is true;
they are evaluated in the order listed until one condition succeeds. The
optional ``else`` clause provides a default node which is executed if
none of the conditions evaluates to true.

**Note:** Previous versions of the PLEXIL compiler required an ``endif``
keyword to terminate the ``if`` node. This requirement has been
eliminated since PLEXIL 4.5. The ``endif`` keyword is still accepted by
the compiler for backwards compatibility.

Each clause may have multiple child nodes.

::

   if C1
     <node-1>
   [elseif C2
     <node-2> ]*
   [else
     <node-3> ]

where C1, C2 are `Boolean expressions <#Boolean_Expressions>`__.
Specifically, if C1 evaluates true, node-1 will be executed. If C1 is
false or *unknown*, C2 is then evaluated, etc. If an ``if`` statement
has no true conditions, and does not supply an ``else`` clause, it will
invoke no action.

Examples:

::

   if true
     {
       foo();
       bar();
     }
   elseif 2 == 2
     bar();
   else
     baz();

   if ( Lookup(raining) )
     Concurrence
     {
       Wipers: turn_on_wipers();
       Lights: turn_on_lights();
     }

.. _while_loop:

While Loop
~~~~~~~~~~

This is a traditional *while* loop.

::

   while C
     <node>

where C is a `Boolean expression <#Boolean_Expressions>`__. Example:

::

   while ! Lookup(RoverWheelStuck)
      RoverDriveOneMeter();

.. _do_loop_plexil_4.6_and_later_releases:

Do Loop (PLEXIL 4.6 and later releases)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is a traditional *do-while* loop.

::

   do <node>
   while C

where C is a `Boolean expression <#Boolean_Expressions>`__. Example:

::

   do
    RoverDriveOneMeter();
   while ! Lookup(RoverWheelStuck)

.. _for_loop:

For Loop
~~~~~~~~

This is a traditional For loop, limited to a numeric variant.

::

   for (T V = Z; C; E)
     <node>

where T is either ``Integer`` or ``Real``, V is a variable name, Z is a
numeric expression for the initial value of V, C is a `Boolean
expression <#Boolean_Expressions>`__ indicating when to continue the
loop, and E is a numeric expression for updating V after each iteration.
Examples:

::

   for (Integer i = 0; i <= 5; i + 1) pprint ("i: ", i);

   for (Integer i = 2; i <= n; i + 1)
   {
     result = s1 + s2;
     s1 = s2;
     s2 = result;
   }

OnCommand
~~~~~~~~~

``OnCommand`` implements a "handler" for an external command. It is used
in multiple executive settings where one executive receives a command
sent by another executive. It has the following syntax.

::

    OnCommand <command-name> [<parameter-declaration>]
      <node>

where:

-   is a string expression naming the command to be handled;
-   is an optional list of zero or more comma-separated variable
   declarations for parameters; and
-   is an action to be performed upon receiving the command.

Example:

::

    OnCommand "Sum" (Integer a, Integer b)
      Increment: { SendReturnValue(a + b); }

Requirements
^^^^^^^^^^^^

``OnCommand`` expects the following commands to be implemented:

-  ``String Command ReceiveCommand(String command_name)`` - waits for
   the named command to be received. When the command is received,
   returns a *handle* which is used to fetch the command's parameters,
   and a command acknowledgment value of ``COMMAND_SUCCESS``.
-  ``Any Command GetParameter(String handle, Integer index)`` - waits
   (if necessary) for the specified parameter to be published. When the
   parameter is received, returns the parameter value, and a command
   acknowledgement of ``COMMAND_SUCCESS``.
-  ``Command SendReturnValue(String handle, Any return_val)`` -
   publishes return_val as the result of the command referenced by the
   handle argument. (**Note:** The ``OnCommand`` macro automatically
   provides the handle value; it should not be supplied by the user.)
   The external system **must** respond with a command acknowledgement.

The *IpcAdapter* interface module provided with the PLEXIL distribution
implements these commands; see `Inter-Executive
Communication <Inter-Executive_Communication>`__. But any PLEXIL
application which implements these commands as specified here can use
``OnCommand``.

.. _failure_behavior:

Failure behavior
^^^^^^^^^^^^^^^^

In the event of an interface error in receiving the command or its
parameters, the ``OnCommand`` node will have an outcome of ``FAILURE``
and a failure type of ``INVARIANT_CONDITION_FAILED``.

.. _returning_a_value_from_the_command:

Returning a value from the command
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Important:** Every ``OnCommand`` node is required to call the command

::

     SendReturnValue(<value>)

where can be any legal PLEXIL expression with a known value.

If a ``SendReturnValue`` command is not present in the body, a
``SendReturnValue(true)`` command is automatically generated, and runs
after the body node has finished executing.

**If the ``SendReturnValue`` command is not acknowledged, the
``OnCommand`` node will never finish.**

The requirement to issue, and acknowledge, a ``SendReturnValue`` command
may be removed in a future release of the PLEXIL Executive.

For more information, see `Inter-Executive
Communication <Inter-Executive_Communication>`__.

OnMessage
~~~~~~~~~

``OnMessage`` is similar to ``OnCommand``, but only receives text sent
by the command ``SendMessage``, and may not have parameters.

::

    OnMessage <message>
      <node>

Where:

-   is a string expression; and
-   is an action to be performed upon receiving that message.

Example:

::
    OnMessage “ConnectionEstablished”
       BeginProcess();

This "handler" for messages can be invoked by the following command in
the IpcAdapter:

::

     SendMessage(<string>)

.. _requirements_1:

Requirements
^^^^^^^^^^^^

The ``OnMessage`` macro requires the application to provide the
following command:

-  ``Command ReceiveMessage(String msg)`` - Waits (if necessary) for a
   message equal to the supplied string to be published. Once the
   message is received, it returns a command acknowledgement value of
   ``COMMAND_SUCCESS``.

The IpcAdapter implements this command as described, as well as the
corresponding ``SendMessage`` command

.. _failure_behavior_1:

Failure behavior
^^^^^^^^^^^^^^^^

In the event of an interfacing error, the ``OnMessage`` node will have
an outcome of ``FAILURE`` and a failure type of
``INVARIANT_CONDITION_FAILED``.

For more information, see `Inter-Executive
Communication <Inter-Executive_Communication>`__.

.. _synchronous_command:

Synchronous Command
~~~~~~~~~~~~~~~~~~~

Plexil provides a convenient form for synchronous commands, which
automatically waits for a return value if expected, otherwise a command
handle indicating completion of the command.

Furthermore a *timeout* can be given, which specifies the maximum amount
of time (expressed as a unitless real number) to wait for the command to
complete. When timeout is given, a real-valued *tolerance* can also be
given; this specifies a minimum granularity for the time measurement.

Here are some examples. First, we call the command ``foo`` with no
arguments:

::

     SynchronousCommand foo();

Call ``foo`` with argument 1 and a timeout of 2.0 time units:

::

     SynchronousCommand foo(1) Timeout 2.0;

Call ``foo``, assigning its return value to ``x``.

::

     SynchronousCommand x = foo();

Call ``foo``, assigning its return value to ``x``, with a timeout of 2.0
with tolerance of 0.1 (i.e. check the time every 0.1 units if possible).

::

     SynchronousCommand x = foo() Timeout 2.0, 0.1;

This section begs elaboration of several aspects of PLEXIL not yet
discussed in detail.

-  Time. As mentioned in the `Overview <Overview>`__, time is not a
   special concept in PLEXIL -- it's just an external world state;
   specifically, a real-valued state variable named ``time``. This
   variable may be referenced explicitly, e.g. ``Lookup (time, 1)``,
   though in most cases it is used implicitly: the Plexil executive
   reads it from the external world at every cycle and uses it for
   time-related computations in a plan, such as the timeout in
   ``SynchronousCommand`` described here. The tolerance parameter to the
   timeout is simply the tolerance given to the Lookup that queries
   ``time`` for this node.

-  Command Handles. These are described in the `Resource
   Model <Resource_Model>`__ chapter, but we must note here that
   instances of SynchronousCommand without return values *require* that
   certain command handles are supported by the `Plexil
   application <PLEXIL_Executive>`__. Specifically, for
   SynchronousCommand to work, the application *must* return one of the
   following handles for the command invoked: COMMAND_SUCCESS,
   COMMAND_FAILED, COMMAND_DENIED.

Wait
~~~~

The ``Wait`` node does just that -- waits for a specified amount of time
to pass:

::

     Wait <time-units> [<tolerance>]

where is a unitless real number (the time unit this actually represents
is application-specific), and so is ``tolerance``. Tolerance, which is
optional and defaults to the , specifies the minimum amount of time that
is of significance in the wait. Real-valued variables can also be used.
Examples:

::

     Real rtol = 0.5;
     Real rdelay = 1.414;

::

     Wait 2.0;           // wait 2.0 units
     Wait 5.0, 0.1;      // wait 5.0 units with a tolerance of 0.1 units
     Wait rdelay, rtol;  // wait, using variables
     Delay1: Wait 3.10;  // a wait node named Delay1

.. _data_types_and_expressions:

Data Types and Expressions
--------------------------

PLEXIL supports the following data types: integer, real, string, Boolean
(logical expressions), and arrays (homogeneous arrays of any type except
array itself). PLEXIL provides a variety of operations on each of these
types.

An *expression* in PLEXIL is either a literal value, a variable, a
lookup, or a combination of any of these formed by operators. In
particular, expressions can contain expressions (i.e. they can be
arbitrarily complex). Expressions can occur within node conditions, the
target of assignments, and resource specifications.

Unknown/isKnown
~~~~~~~~~~~~~~~

Each PLEXIL type is extended by a special value ``UNKNOWN``, i.e. any
expression can evaluate to ``UNKNOWN``. The unknown value occurs in the
following cases.

-  It's the default initial value for variables, a node's outcome, and
   array elements.
-  It results when a lookup fails.
-  It results when a requested *node timepoint* has not occurred. Node
   timepoints are discussed below.
-  It is a valid value for Plexil logical expressions.

The ``UNKNOWN`` value is *not* a literal -- it may not be used in a
PLEXIL plan. It is tested solely through the ``isKnown`` operator, which
returns false if its argument evaluates to ``UNKNOWN``, and true
otherwise. An example of the use of ``isKnown`` is found in the section
above on Command nodes.

.. _numeric_expressions:

Numeric Expressions
~~~~~~~~~~~~~~~~~~~

Numeric expressions include literals (integers, real numbers), variables
(of type Integer or Real), lookups and node timepoint values (both
discussed below), and arithmetic operations: addition, subtraction,
multiplication, division, square root, minimum, maximum, and absolute
value. In addition, arrays have as numeric expressions their size,
element index, and, for arrays of numeric type, their elements.

Here are varied examples of each of the aforementioned types of numeric
expressions.

::

   234
   12.9
   X /* where X was declared Integer */
   Bar /* where Bar was declared Real */
   Lookup(ExternalTemperature)
   TakePicture.EXECUTING.START  /* a node timepoint */
   Bar + 4.5
   X - (30 + Lookup(x) )
   3 * X
   (3 * X)/(X - 20)
   sqrt(X)
   abs(X)
   Entries[X] /* where Entries is an array of Integer or Real */

Precedence and associativity rules for these operators are consistent
with the standard rules for C and C++. Parentheses can be used to make
explicit the intended semantics.

Integers and Reals may be mixed in Real-valued numeric expressions.
Integer values are automatically promoted to Real in mixed calculations,
so are legal in all contexts where a Real is expected. However, a Real
value cannot be used where an Integer is expected, e.g. as an array
index, nor can a Real value be assigned to an Integer variable or array
element.

.. _numeric_type_conversions:

Numeric Type Conversions
^^^^^^^^^^^^^^^^^^^^^^^^

Plexil offers the following type conversion operators for converting a
Real to an Integer:

::

   ceil(r) /* returns least positive integer greater than or equal to r */
   floor(r) /* returns most positive integer less than or equal to r */
   round(r) /* as defined in the C language standard */
   trunc(r) /* rounds toward 0 */
   real_to_int(r) /* For converting a Real that is known to be exactly integer-valued */

In each conversion function, if the supplied Real is out of range for an
Integer, UNKNOWN is returned. Additionally, ``real_to_int`` will return
UNKNOWN if the supplied Real is not exactly an integer value.

.. _boolean_expressions:

Boolean Expressions
~~~~~~~~~~~~~~~~~~~

PLEXIL employs a *ternary* logic, extending the usual Boolean logic with
a third value, Unknown, described in a section above. Though strictly a
misnomer, the term Boolean is used throughout this manual and PLEXIL
itself to describe operators, expressions, and values in this ternary
logic.

Logical expressions include the Boolean literals ``true`` and ``false``,
``Boolean``-typed variables, lookups, comparisons, logical operations,
array elements (of ``Boolean`` arrays), and the ``isKnown`` operator.

The logical connectives, their syntax in PLEXIL, and arity (number of
operands allowed) are as follows:

::

   Negation (Not)     !, NOT      1
   Conjunction (And)  &&, AND     2 or more
   Disjunction (Or)   ||, OR      2 or more
   Exclusive Or       XOR         2

When restricted to Boolean (``true`` or ``false``) values in their
constituents, logical expressions in PLEXIL follow the standard rules of
Boolean logic. Here is how PLEXIL handles the Unknown value, again a
standard interpretation.

::

   true && Unknown     = Unknown
   false && Unknown    = false
   Unknown && Unknown  = Unknown
   true || Unknown     = true
   false || Unknown    = Unknown
   Unknown || Unknown  = Unknown
   true XOR Unknown    = Unknown
   false XOR Unknown   = Unknown
   Unknown XOR Unknown = Unknown
   ! Unknown           = Unknown

The operators ``AND`` and ``OR`` are evaluated left to right in a
*short-circuit* fashion. Conjunctions have value ``true`` until an
operand evaluates to ``false`` or Unknown; this value becomes the value
of the expression. Similarly, disjunctions have value ``false`` until an
operand evaluates to ``true`` or Unknown.

The comparison operators, all of which take exactly two operands, are:

::

   Equality                 ==
   Inequality               !=
   Less than                <
   Greater than             >
   Less than or equal       <=
   Greater than or equal    >=

In these comparision expressions, if *any* operand evaluates to Unknown,
the entire expression yields Unknown.

Here are varied examples of logical expressions.

::

   true
   false
   CommandReceived /* where CommandReceived was declared Boolean */
   ! CommandReceived
   Lookup(RoverInitialized) /* where RoverInitialized is declared a Boolean lookup */
   count <= 30 /* where count was declared Integer */
   Lookup(RoverBatteryCharge) > 120.0 /* where RoverBatteryCharged is declared a Real lookup */
   Lookup(RoverInitialized) || CommandReceived
   Flags[3] /* where Flags is an array of Boolean */
   isKnown(val)  /* where val is any variable */
   node3.state == FINISHED && node3.outcome == SUCCESS

NOTE: Precedence and associativity rules for these operators are
consistent with the standard rules for C and C++. Parentheses can be
used to make explicit the intended semantics.

.. _string_expressions:

String Expressions
~~~~~~~~~~~~~~~~~~

String expressions include literal strings, variables (of type String),
lookups, and string concatenations. Examples of each are as follows.

::

   "foo"
   "Would you like to continue?"
   Username /* where Username was declared string */
   Lookup(username)
   "Hello, " + "Fred"    => "Hello, Fred"
   "Hello, " + Username

The only comparison operations currently defined on strings are ``==``
and ``!=``.

The ``strlen`` operator returns the length of a String as an Integer.

.. _dates_and_durations:

Dates and Durations
~~~~~~~~~~~~~~~~~~~

One may want to reason about time. PLEXIL provides basic support for
*date*, *time*, and *duration* types as defined by the ISO-8601
standard. See http://en.wikipedia.org/wiki/ISO_8601 for a detailed
description of this standard and the date/duration formats, as these are
covered only by example here.

.. _date_and_duration_expressions:

Date and Duration expressions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

PLEXIL expressions can have type ``Date`` or ``Duration``. The former
includes *time* and combined *date/time* expressions. Dates and
durations are encoded as *strings* in the ISO-8601 format. Here are some
examples of Date and Duration variable declarations.

::

    Duration dur1;  // uninitialized duration variable
    Date date1;     // uninitialized date variable
    Duration dur2 = Duration("PT60M");  // 60 minutes
    Date date2 = Date("2012-05-26T20:42:00.00Z");  // UTC time
    Date date3 = "2011-12-03T00:42:12.00";  // local time

Dates and Durations are expressed as literals using the ``Date`` and
``Duration`` constructor, respectively. These are exemplified in the
variable initializations shown above. Here are more examples:

::

   // subtract 1.5 seconds from the given date.
   date3 = date3 - Duration("PT1.5S");

   // Calculate the duration between two dates.
   dur2 = date3 - Date("2011-05-16T03:19:00");

Finally, here is a simple practical use of these types: a node that
starts on or after a given date, and runs for a specified duration:

::

   Date Lookup time;
   Date Lookup start;
   Duration Lookup duration;

   Test:
   {
       Start Lookup(time, 1) >= Lookup(start);
       End   Lookup(time, 1) >= Self.EXECUTING.START + Lookup(duration);
   }

Additional PLEXIL plans illustrating varied uses of dates and durations
may be found in the directory ``plexil/examples/temporal`` in the PLEXIL
source code distribution.

**CAVEATS:** At present, date and duration literals are not checked for
valid syntax. Also, unspecified behavior will result if an arithmetic
operation involving dates or durations yields a negative value.

.. _operations_using_dates_and_durations:

Operations using Dates and Durations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following arithmetic operations involving dates and durations are
supported.

::

    date       -   date     =  duration
    date       +-  duration =  date
    duration   +-  duration =  duration
    duration   *   number   =  duration
    duration   /   number   =  duration
    duration   /   duration =  duration
    duration   mod duration =  duration
    duration   mod number   =  duration
    abs duration            =  duration

Dates can be compared with the operators <, >, <=, >=, ==, and !=, as
can Durations. Dates and Durations cannot be directly compared.

.. _date_and_duration_representation:

Date and Duration representation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

At present, dates and durations are not defined in Core PLEXIL. Recall
that in Core PLEXIL, time is represented as a unitless real number,
whose actual unit is application defined.

Expressions of type Date in the full PLEXIL language are translated into
Core PLEXIL for execution, where they are converted to real numbers
representing absolute time as *seconds* since the Unix epoch of Jan 1,
1970 (1970-01-01T00:00:00Z to be precise). This is a highly standard
convention. At present, PLEXIL does not support the use of alternate
epochs.

Similarly, Duration expressions are converted into real numbers
representing seconds.

**CAVEAT:** A key limitation in the current Plexil executive is that it
does not recognize dates and durations as distinct from other real
numbers. Therefore, for example, if date or duration values are
inspected or printed in a running plan, a unitless real number will be
shown. The PLEXIL team hopes to remedy this and make dates and durations
better supported in general.

Arrays
~~~~~~

PLEXIL provides just one aggregate data type, the *array*. At present,
the array type in PLEXIL is somewhat limited compared to what's found in
modern programming languages. PLEXIL arrays are homogenous and
one-dimensional: a sequence of values of a single scalar data type,
indexed by integers beginning with 0. Specifically, arrays may of type
Integer, Real, String, or Boolean only.

PLEXIL provides both variables and literals of array type. Like other
variables, array variables must be declared prior to use. An array
declaration specifies its name, type, maximum size (number of elements),
and, optionally, initial values for some or all of the array's elements.
The memory needed by an array is allocated (for its maximum size) when
the array is declared. Unlike scalar variables, array variables are
*not* initialized to the Unknown value by default; rather, each element
of the array is initialized to Unknown. Array indices start with 0.

The following examples illustrate the key properties of PLEXIL arrays.

::

     Boolean flags[10];

This an array of ten booleans. Each element has the value Unknown (i.e.
each element will fail the **isKnown** test).

::

    Integer X[6] = #(1 3 5);

This example illustrates initialization of elements and the array
literal. This array of 6 integers is initialized with an array
containing 1,3, and 5 as its first three elements. That is, X[0] = 1,
X[1] = 3, and X[2] = 5. The last three elements of X are Unknown. It is
an error to initialize an array variable with an array containing more
elements than its maximum size. (As an aside, the syntax for the array
literal is taken from Common Lisp).

Arrays support the following operations. Assume an array named X.

-  Read an element: ``X[``\ \ ``]``, where can be any integral
   expression. Array elements are a kind of expression, and thus may be
   used in any place where expressions are allowed.
-  Assign an element: ``X[``\ \ ``] =``\ . Assignments can occur only in
   assignment nodes.
-  Assign an entire array: ``X = Y``, where Y is either an array
   variable or an array literal. It is an error if ``Y`` represents an
   array larger than ``X``. If ``Y`` is smaller than ``X`` then the
   remaining elements in ``X`` will be filled with ``Unknown``.
-  Get the size of an array as an Integer: ``arraySize(Y)``, where Y is
   an array-valued expression.
-  Get the maximum size of an array as an Integer: ``arrayMaxSize(Y)``,
   where Y is an array-valued expression.

.. _node_state:

Node State
~~~~~~~~~~

A PLEXIL node can access its own internal state, or the internal state
of other nodes, but only those nodes which are its siblings, children,
or parent. (The internal state of more distant relatives is not
accessible).

Node state consists of:

-  The current execution state of a node
-  The start and end times of each state a node has encountered
-  The outcome value of a node, if it has terminated
-  The failure type of a node, if it has failed
-  For command nodes, the last *command handle* received.

Each of these values is a unique type, with the exception of start and
end times, which are of type ``Date``. The only operations that can be
performed with these values are comparison for equality or inequality
with each other, or against a literal value.

The syntax for referencing these types of information is the following,
where is the node's identifier.

::

     <Id>.state

returns one of INACTIVE, WAITING, EXECUTING, FINISHED, ITERATION_ENDED,
FAILING, FINISHING.

::

     <Id>.<state>.<timepoint>

where is one of the seven states listed above, and is one of START, END,
will return the time elapsed (as a real number) since the given state
started or ended (respectively) for the given node. If the requested
timepoint has not occurred, the value of this variable is Unknown. For
an explanation of time in PLEXIL, see the `Overview <Overview>`__.

::

     <Id>.outcome

returns one of SUCCESS, FAILURE, or SKIPPED, if the given node has
terminated (else it will return Unknown).

::

     <Id>.failure

returns one of INVARIANT_CONDITION_FAILED, POST_CONDITION_FAILED,
PRE_CONDITION_FAILED, PARENT_FAILED, if the node has terminated with
failure (else it will return Unknown).

::

    <Id>.command_handle

returns one of COMMAND_ACCEPTED, COMMAND_SUCCESS,
COMMAND_RCVD_BY_SYSTEM, COMMAND_SENT_TO_SYSTEM, COMMAND_FAILED, or
COMMAND_DENIED, if the node is executing (else it will return Unknown).

.. _external_state_lookups:

External State (Lookups)
~~~~~~~~~~~~~~~~~~~~~~~~

External state is read through *lookups*. Lookups access states using
domain-specific measurement names. The syntax for a lookup is:

::

     Lookup(<state_name> [(<param>*)] [, <tolerance>])

where is either an identifier or a string expression that evaluates to
the desired state name. States can have parameters, which are specified
by a comma-separated list of literal state names or string expressions
that follow the state name. Tolerance, which is optional, must be a real
number or real-valued variable; it specifies the granularity of accuracy
for the lookup, and defaults to 0.0. Lookups may not be *overloaded* --
only one Lookup with a given name may be used.

NOTE: For the state name, literal names are unquoted, while string
expressions are parenthesized. For state parameters, literal names are
double-quoted, while string expressions are given no special treatment.

Here are some basic examples:

::

   Lookup(time)                       // queries the state named "time"

   Lookup((pressureSensorName), 1.0)  // queries the state named by the
                                      // pressureSensorName variable

   Lookup(At("rock1"))                  // queries the parameterized state At("rock1")

String expressions used for state names can include Lookup themselves.
For example, here an external query is used to get the name of a sensor
for a Lookup:

::

     Lookup((Lookup(ModuleVoltageSensorName("Crew Habitat"))), 0.1)

The tolerance parameter is optional and defaults to 0.0. If given, it
must be a real number and specifies the minimum value by which the state
must have changed since its last reading in order to be read again. The
example above says to read the module voltage sensor when it changes at
least 0.1. Tolerances are unitless in Plexil; the unit of measure they
represent is specified by the queried external system. The tolerance
parameter is meaningless and ignored in certain contexts. See the
following section for an explanation.

.. _execution_of_lookups:

Execution of Lookups
^^^^^^^^^^^^^^^^^^^^

There are two contexts for lookups that are important to distinguish.
One is the asynchronous context implied by a node's gate conditions
(Start, Skip, End, Repeat). These conditons passively "wait" to become
true. Lookups found in these conditions are processed as *subscriptions*
to the external system for updates to the requested states. It is only
in this context that *tolerance* is meaningful. These Lookup forms are
compiled into *LookupOnChange* in Core PLEXIL's XML representation.

The second context for lookups is the synchronous context implied by a
node's check conditions (Pre, Post, Invariant) and its body. In these
contexts, a lookup is processed on demand, that is, its value is
*fetched* at specific points in execution of the node. Tolerance is
meaningless in this context, and ignored if specified. These Lookup
forms are compiled into *LookupNow* in Core PLEXIL's XML representation.

.. _the_time_state:

The "time" state
^^^^^^^^^^^^^^^^

The state name ``time`` is predefined in the Plexil executive. It
returns the system time as a real number, which is compatible with the
Date type. The units and epoch of the returned value are system
dependent. On the typical platforms that support PLEXIL, they would be
in POSIX/Unix time, i.e. the number of seconds since January 1, 1970
midnight UTC (1970-01-01T00:00:00.000Z).

Even though ``time`` is predefined, it must still be declared in the
plan, in one of the following ways.

::

   Date Lookup time;
   Real Lookup time;

NOTE: Due to how the PLEXIL executive's interface to the system clock is
implemented, a tolerance parameter is required for time lookups. E.g. to
specify a tolerance of one time unit:

::

   Lookup(time, 1)

.. _global_declarations:

Global Declarations
-------------------

If a plan contains calls to system commands (i.e. command nodes), uses
library nodes, or queries world state using lookups, these *must* be
pre-declared.

These *global declarations* must occur first in Plexil files, before the
top-level node. They can occur in any order; declarations for commands,
lookups, and library nodes can be intermixed freely.

Including global declarations as a standard practice has several
advantages. First, it allows you to define and view your plan's entire
external interface in one place, rather than having it scattered
throughout the plan. Second, it enables `static
checking <Plexil_Checker>`__ of your plan. Static checks will insure
that your declarations are consistent and that all uses of declared
items in the plan are correct.

The following are examples of global declarations.

::

   // simple command
   Command Stop();

   // command with parameter (name is optional)
   Command Drive(Real meters);

   // command with return value
   Boolean Command TakePicture(Integer, Integer, Real);

   // state
   Real Lookup Temperature;

   // state with parameter
   Boolean Lookup At (String location);

   // library node
   LibraryAction LibTest(In Real i, In vals[10], InOut Integer j);

.. _programming_in_plexil:

Programming in Plexil
---------------------

.. _plexil_files:

Plexil Files
~~~~~~~~~~~~

A file containing Plexil code can have any name, though its extension
must be ``.ple``. A Plexil file must contain exactly one construct, i.e.
a single node. Your application may comprise many Plexil files; in this
case, one file will contain the *top level* node, and the rest will
contain library nodes.

We strongly recommend that the top level node in a Standard PLEXIL file
be named the same as the file, e.g. file HaltAndCatchFIre.ple should
contain the top level node named ``HaltAndCatchFire``.

.. _plexilc_the_plexil_translator:

``plexilc``, the Plexil translator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Plexil plans and Plexilscript scripts must be translated into XML for
execution by the PLEXIL Executive. The ``plexilc`` utility performs this
translation for several different PLEXIL syntaxes.

E.g. given a Plexil file ``foo.ple``, translate it with the following
command:

::

     plexilc foo.ple

If ``foo.ple`` is free of errors, this command will create the Core
PLEXIL XML file ``foo.plx``.

``plexilc`` chooses the translator for its inputs based on the file
name's extension. Input languages supported by ``plexilc`` are:

-  .ple - Standard PLEXIL
-  .plp - Standard PLEXIL with preprocessing (see below)
-  .pst - Plexilscript, the scripting language for the Test Executive
-  .pli - Plexilisp (deprecated), a Lisp-like syntax used prior to the
   development of the Standard PLEXIL language.

``plexilc`` supports the following command-line options (this list is
obtainable by calling ``plexilc`` with no arguments):

::

    -c, -check              Run static checker on output (only valid for plan files)
    -d, -debug <logfile>    Print debug information to <logfile>
    -h, -help               Print this help and exit
    -o, -output <outfile>   Write translator output to <outfile>
    -q, -quiet              Parse files quietly
    -v, -version            Print translator version and exit

Some options are not supported by all source file formats.

If there are errors in your Plexil code, ``plexilc`` will report them,
along with their line numbers and character positions. No output file is
created. Often, when there are many errors, correcting one of them will
take care of subsequent errors.

If ``plexilc`` outputs only *warnings* about your Plexil code, the
translated output file is still created. Warnings usually indicate
potentially serious errors in the program's logic, so they should be
inspected and dealt with.

.. _preprocessing_plexil_plans:

Preprocessing PLEXIL Plans
~~~~~~~~~~~~~~~~~~~~~~~~~~

The Standard PLEXIL compiler as of release 4.5 now accepts C
preprocessor statements such as ``#include`` and ``#define``. This is a
convenient way to share (e.g.) Command, Lookup, and LibraryAction
declarations, and constant definitions, across several PLEXIL source
files.

``plexilc`` automatically invokes the preprocessor when the input file
name ends in ``.plp``

.. _executing_plexil_plans:

Executing PLEXIL Plans
~~~~~~~~~~~~~~~~~~~~~~

See the `PLEXIL Executive <PLEXIL_Executive>`__ page for details on
executing PLEXIL plans.

--------------

**Copyright (c) 2006-2021, Universities Space Research Association
(USRA). All rights reserved.**

`Category:PLEXIL REFERENCE MANUAL <Category:PLEXIL_REFERENCE_MANUAL>`__

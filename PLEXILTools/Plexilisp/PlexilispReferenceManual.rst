.. _PlexilispRefernceManual:

Plexilisp Reference Manual
===========================

*This wiki was automatically generated from "plexil.el" on Mon Feb 17
15:47:11 2014 for version 3.2. Do not edit!*

This is a complete reference for the Plexilisp language. It assumes a
basic understanding of the |PLEXIL| language. Each construct in Plexilisp
has at least two aliases (e.g. ``CommandNode`` and ``command-node``).
You may use whichever you prefer, or mix and match them.

.. contents::

.. _plexil_plan:

PLEXIL Plan
===========

This section describes the forms (constructs) that comprise |PLEXIL|
plans.

--------------

::

   (PlexilPlan form [ form ] ...)
   (plexil-plan form [ form ] ...)

The top level form for a plan. A Plexilisp file must contain exactly one
of these, and nothing else. A ``PlexilPlan`` form must contain forms in
the following order. The first (optional) can be a
``GlobalDeclarations``. The second (required) is the plan's root node.
Additional forms can only be ``Comment``'s.

.. _global_declarations:

Global Declarations
=======================

::

   (GlobalDeclarations [ decl ] ...)
   (global-declarations [ decl ] ...)
   (Declarations [ decl ] ...)
   (declarations [ decl ] ...)

The plan's global declarations.

--------------

::

   (LibraryNodeDeclaration name [ interface-declaration ] ...)
   (library-node-declaration name [ interface-declaration ] ...)

Declare a library node (call). Following the name may be any number of
interface declarations, which are either ``In`` or ``InOut`` forms.

--------------

::

   (StateDeclaration name type [ param ] ...)
   (state-declaration name type [ param ] ...)

Declare a state (lookup), specifying its type (as a Return form) and
parameters (as Parameter forms).

--------------

::

   (CommandDeclaration name [ returns-then-parameters-then-resource-lis ] ...)
   (command-declaration name [ returns-then-parameters-then-resource-lis ] ...)

Declare a command. Following the name should be zero or more Return
forms, then zero or more Parameter forms, then an optional ResourceList
form. They cannot be intermixed.

--------------

::

   (Return type)
   (return type)

Specify a return type. Argument must be one of 'string', 'integer',
'array', 'boolean', 'real' (case insensitive).

--------------

::

   (Parameter type)
   (parameter type)

Specify a parameter type. Argument must be one of 'string', 'integer',
'array', 'boolean', 'real' (case doesn't matter).

.. _nodes_and_node_types:

Nodes and Node Types
--------------------

These are the forms for defining |PLEXIL| nodes. It takes at least two to
fully define a node. An outer form declares the node. (e.g.
``CommandNode``), and an inner form defines the action (e.g.
``(command ...)``). These forms must be compatible (more specifics in
each entry below)

--------------

::

   (ListNode [ name ] [ &rest ] [ node-clauses ] )
   (list-node [ name ] [ &rest ] [ node-clauses ] )

Defines a List Node. Must contain a ``List`` form.

--------------

::

   (List [ node ] ...)
   (list [ node ] ...)

Required inside a ``ListNode``, this form wraps its list of nodes.

--------------

::

   (CommandNode [ name ] [ &rest ] [ clauses ] )
   (command-node [ name ] [ &rest ] [ clauses ] )

Defines a Command Node. Must contain either a ``Command`` or
``CommandWithReturn`` form.

--------------

::

   (Command command-name [ arg ] ...)
   (command command-name [ arg ] ...)

Required inside a ``CommandNode``, this form calls the specified
command. command-name may be any string expression (literal, variable,
concatenation, or lookup). If resources are specified, they must follow
the command name.

--------------

::

   (CommandWithReturn var command-name [ arg ] ...)
   (command-with-return var command-name [ arg ] ...)

This is just like ``Command`` above, but a value returned from the
command is assigned to the given variable, which must be declared in
this node or one of its ancestors.

--------------

::

   (UpdateNode [ name ] [ &rest ] [ clauses ] )
   (update-node [ name ] [ &rest ] [ clauses ] )

Defines an Update Node. Must contain an ``Update`` form.

--------------

::

   (Update [ pair ] ...)
   (update [ pair ] ...)

Required inside an ``UpdateNode``, this form defines the plan update.It
must contain one or more ``Pair`` forms.

--------------

::

   (Pair name value)
   (pair name value)

Required inside an ``Update``, this form defines a name/value pair.The
``name`` must be a string and the ``value`` may be any |PLEXIL| type.

--------------

::

   (AssignmentNode [ name ] [ &rest ] [ clauses ] )
   (assignment-node [ name ] [ &rest ] [ clauses ] )

Defines an Assignment Node. Must contain an ``Assignment`` form.

--------------

::

   (Assignment var val)
   (assignment var val)

Required inside an ``AssignmentNode``, this form assigns a value (any
|PLEXIL| type) to a variable that must be declared in this node or one of
its ancestors.

--------------

::

   (LibraryCallNode [ name ] [ &rest ] [ node-clauses ] )
   (library-call-node [ name ] [ &rest ] [ node-clauses ] )

A Library Call Node. Must contain exactly one ``call`` form.

--------------

::

   (Call nodeid [ aliase ] ...)
   (call nodeid [ aliase ] ...)

A call to a library node.

--------------

::

   (Alias parameter value)
   (alias parameter value)

In a library node call, this pairs a parameter of the node with a value.
The parameter is an ncName, and the value must be either a literal or
declared variable.

--------------

::

   (EmptyNode [ name ] [ &rest ] [ clauses ] )
   (empty-node [ name ] [ &rest ] [ clauses ] )

An Empty Node.

.. _variable_declaration:

Variable Declaration
==========================

::

   (VariableDeclarations [ decl ] ...)
   (variable-declarations [ decl ] ...)
   (Variables [ decl ] ...)
   (variables [ decl ] ...)
   (DeclareVariables [ decl ] ...)
   (declare-variables [ decl ] ...)

The node's variable declarations. Must contain one or more of the
declaration forms that follow.

--------------

::

   (Integer name [ val ] )
   (integer name [ val ] )

Declare an integer variable, with optional initial value.

--------------

::

   (Real name [ val ] )
   (real name [ val ] )

Declare a real variable, with optional initial value.

--------------

::

   (Boolean name [ val ] )
   (boolean name [ val ] )

Declare a boolean variable, with optional initial value.

--------------

::

   (String name [ val ] )
   (string name [ val ] )

Declare a string variable, with optional initial value.

--------------

::

   (Duration name [ val ] )
   (duration name [ val ] )

Declare an ISO 8601 duration variable, with optional initial value.

--------------

::

   (Date name [ val ] )
   (date name [ val ] )

Declare an ISO 8601 date variable, with optional initial value.

--------------

::

   (IntArray name size [ value ] ...)
   (int-array name size [ value ] ...)

Declare an integer array with given name, size, and initial values.

--------------

::

   (StringArray name size [ value ] ...)
   (string-array name size [ value ] ...)

Declare a string array with given name, size, and initial values.

--------------

::

   (BooleanArray name size [ value ] ...)
   (boolean-array name size [ value ] ...)

Declare a boolean array with given name, size, and initial values.

--------------

::

   (RealArray name size [ value ] ...)
   (real-array name size [ value ] ...)

Declare a real number array with given name, size, and initial values.

.. _node_conditions:

Node Conditions
======================

::

   (Postcondition exp)
   (postcondition exp)

--------------

::

   (PostCondition exp)
   (post-condition exp)

--------------

::

   (EndCondition exp)
   (end-condition exp)

--------------

::

   (ExitCondition exp)
   (exit-condition exp)

--------------

::

   (SkipCondition exp)
   (skip-condition exp)

--------------

::

   (Precondition exp)
   (precondition exp)

--------------

::

   (PreCondition exp)
   (pre-condition exp)

--------------

::

   (RepeatCondition exp)
   (repeat-condition exp)

--------------

::

   (StartCondition exp)
   (start-condition exp)

--------------

::

   (InvariantCondition exp)
   (invariant-condition exp)

.. _variable_reference:

Variable Reference
------------------

All variable references must take one of the following forms, which
specifies the name of the variable as a string. The variable is assumed
to be legally declared.

--------------

::

   (BooleanVariable name)
   (boolvar name)

--------------

::

   (IntegerVariable name)
   (intvar name)

--------------

::

   (RealVariable name)
   (realvar name)

--------------

::

   (StringVariable name)
   (stringvar name)

--------------

::

   (ArrayVariable name)
   (arrayvar name)

--------------

::

   (DateVariable name)
   (datevar name)

--------------

::

   (DurationVariable name)
   (durvar name)
   (durationvar name)

--------------

::

   (ArrayElement name index)
   (array-element name index)

Reference a single array element by index (beginning with 0). Name must
be a string (XML NCName precisely). Index must be a numeric expression.

.. _interface_declaration:

Interface Declaration
---------------------

Plexilisp does not automatically generate any ``Interface``
declarations. They must be created explicitly with these forms.

--------------

::

   (Interface [ decl ] ...)
   (interface [ decl ] ...)

The Node's interface. This must contain only ``In`` and ``InOut`` forms.
They can be intermixed.

--------------

::

   (In [ var ] ...)
   (in [ var ] ...)

Declare input variables. Your must use the variable declaration forms
defined above.

--------------

::

   (InOut [ var ] ...)
   (inout [ var ] ...)

Declare input/ouput variables. Your must use the variable declaration
forms defined above.

.. _boolean_comparisons:

Boolean Comparisons
-------------------

These return true or false.

--------------

::

   (= x y)
   (eq x y)

--------------

::

   (!= x y)
   (ne x y)

The following work with all numeric types.

--------------

::

   (> x y)
   (gt x y)

--------------

::

   (>= x y)
   (ge x y)

--------------

::

   (< x y)
   (lt x y)

--------------

::

   (<= x y)
   (le x y)

.. _logical_connectives:

Logical Connectives
-------------------

These return true, false, or unknown.

--------------

::

   (Or [ disjunct ] ...)
   (or [ disjunct ] ...)

Permits 0 or more disjuncts. ``(Or)`` = ``false``.

--------------

::

   (And [ conjunct ] ...)
   (and [ conjunct ] ...)

Permits 0 or more conjuncts. ``(And)`` = ``true``.

--------------

::

   (Not x)
   (not x)

.. _numeric_operators:

Numeric Operators
-----------------

These should be self-explanatory. They work with integer or real values.

--------------

::

   (+ x y)
   (add x y)

--------------

::

   (- x y)
   (sub x y)

--------------

::

   (* x y)
   (mul x y)

--------------

::

   (/ x y)
   (div x y)

--------------

::

   (% x y)
   (mod x y)

--------------

::

   (Max x y)
   (max x y)

--------------

::

   (Min x y)
   (min x y)

--------------

::

   (Abs x)
   (abs x)

--------------

::

   (Sqrt x)
   (sqrt x)

Lookups
-------

The new form Lookup (and its variant LookupWithTolerance) is a
convenient substitute for the Core |PLEXIL| forms LookupNow and
LookupOnChange. It can be used anywhere, though note that 'tolerance' is
valid only in gate conditions (Start, End, Repeat, Skip) and otherwise
ignored.

--------------

::

   (Lookup state [ arg ] ...)
   (lookup state [ arg ] ...)

Queries for the value of the given state with given arguments.

--------------

::

   (LookupWithTolerance state tolerance [ arg ] ...)
   (lookup-with-tolerance state tolerance [ arg ] ...)

Like the above, but uses the specified tolerance. The tolerance is a
real number or (name of a) real variable.

--------------

::

   (LookupNow state [ arg ] ...)
   (lookup-now state [ arg ] ...)

Queries for the value of the given state with given arguments. Valid
only in node bodies and check conditions (Pre, Post, Invariant, Exit).

--------------

::

   (LookupOnChange state [ arg ] ...)
   (lookup-on-change state [ arg ] ...)

Subscribes for updates to the given state with given arguments.Valid
only in gate conditions (Start, End, Repeat, Skip).

--------------

::

   (LookupOnChangeWithTolerance state tolerance [ arg ] ...)
   (lookup-on-change-with-tolerance state tolerance [ arg ] ...)

Like the above, but uses the specified tolerance. The tolerance is a
real number or real variable.

Literals
--------

Most of these are not needed, because Plexilisp automatically infers
types of literals. For example, 5.5 would be a real, 5 would be an
integer, "foo" a string, ``true`` and ``false`` a boolean. Date and
Duration literals are a strong exception.

--------------

::

   (IntegerValue val)
   (intval val)

Integer value

--------------

::

   (RealValue val)
   (realval val)

Real value

--------------

::

   (BooleanValue val)
   (boolval val)

Valid arguments are 0, 1, and UNKNOWN. More simply, the symbols
``true``, ``false``, and ``unknown`` may be used instead of this form.

--------------

::

   (StringValue str [ str ] ...)
   (stringval str [ str ] ...)

Concatenates its arguments into one string. For long strings, this makes
your file more readable.

--------------

::

   (DateValue val)
   (dateval val)

ISO-8601 date value

--------------

::

   (DurationValue val)
   (durationval val)
   (durval val)

ISO-8601 duration value

.. _node_state_outcome_and_failure_type:

Node State, Outcome, and Failure Type
-------------------------------------

Predicates for querying the state, outcome, and failure type of actions.

--------------

::

   (Finished id)
   (finished id)
   (isFinished id)
   (is-finished id)

Is the given action in state FINISHED?

--------------

::

   (IterationEnded id)
   (iteration-ended id)
   (isIterationEnded id)
   (is-iteration-ended id)

Is the given node in state ITERATION_ENDED?

--------------

::

   (Executing id)
   (executing id)
   (isExecuting id)
   (is-executing id)

Is the given action in state EXECUTING?

--------------

::

   (Waiting id)
   (waiting id)
   (isWaiting id)
   (is-waiting id)

Is the given action in state WAITING?

--------------

::

   (Inactive id)
   (inactive id)
   (isInactive id)
   (is-inactive id)

Is the given action in state INACTIVE?

--------------

::

   (Successful id)
   (successful id)
   (isSuccessful id)
   (is-successful id)

Did the given action finish successfully?

--------------

::

   (IterationSuccessful id)
   (iteration-successful id)

Did the last iteration of the given action finish successfully?

--------------

::

   (IterationFailed id)
   (iteration-failed id)

Did the last iteration of the given action fail?

--------------

::

   (Failed id)
   (failed id)
   (isFailed id)
   (is-failed id)

Did the given action fail?

--------------

::

   (Skipped id)
   (skipped id)
   (isSkipped id)
   (is-skipped id)

Was the given action skipped?

--------------

::

   (InvariantFailed id)
   (invariant-failed id)

Did the invariant condition of the given action fail?

--------------

::

   (PostConditionFailed id)
   (PostconditionFailed id)
   (postcondition-failed id)
   (post-condition-failed id)

Did the postcondition of the given action fail?

--------------

::

   (PreConditionFailed id)
   (PreconditionFailed id)
   (precondition-failed id)
   (pre-condition-failed id)

Did the precondition of the given action fail?

--------------

::

   (ParentFailed id)
   (parent-failed id)

Did the parent of the given action fail?

.. _conditionals_and_loops:

Conditionals and Loops
----------------------

These are high level syntax extensions of |PLEXIL| (syntactic sugar). They
expand into node structures.

--------------

::

   (If condition then-part [ else-part ] )
   (if condition then-part [ else-part ] )

If-then-else. The ``then-part`` and ``else-part`` may be nodes or other
actions. The ``else-part`` is optional.

--------------

::

   (While condition action)
   (while condition action)

While loop.

--------------

::

   (for declaration condition update action)
   (For declaration condition update action)

For Loop. The declaration should look like a variable declaration. i.e
``(type name [init])``, where ``type`` must be either ``integer`` or
``real`` and the initial value ``init`` is optional (though generally
useful). ``condition`` is a boolean expression that will terminate the
loop when it is false. ``update`` is a numeric expression that expresses
a new value for the declared variable.

--------------

::

   (Sequence [ name-or-first-form ] [ &rest ] [ forms ] )
   (sequence [ name-or-first-form ] [ &rest ] [ forms ] )

Each action starts after the previous succeeds. If an action fails, the
sequence terminates immediately with failure.

--------------

::

   (UncheckedSequence [ name-or-first-form ] [ &rest ] [ forms ] )
   (unchecked-sequence [ name-or-first-form ] [ &rest ] [ forms ] )

Each action starts after the previous finishes, regardless of success or
failure.

--------------

::

   (Concurrence [ name-or-first-form ] [ &rest ] [ forms ] )
   (concurrence [ name-or-first-form ] [ &rest ] [ forms ] )
   (Concurrently [ name-or-first-form ] [ &rest ] [ forms ] )
   (concurrently [ name-or-first-form ] [ &rest ] [ forms ] )

Executes forms concurrently. Basically a List node.

--------------

::

   (Try [ name-or-first-form ] [ &rest ] [ forms ] )
   (try [ name-or-first-form ] [ &rest ] [ forms ] )

Executes actions sequentially, stopping after the an action succeeds.
Fails if and only if no action succeeds.

--------------

::

   (OnMessage message [ action ] )
   (on-message message [ action ] )

Specifies an action for responding to a given message (string).

--------------

::

   (OnCommand command-name arg-decls [ action ] )
   (on-command command-name arg-decls [ action ] )

Specifies an action for responding to a given command. command-name must
be a string, arg-decls a list of variable declarations (e.g., (real "x")
(integer "y") (boolean "z") (real-array "m" 4), etc.). If this is action
should return a value, send it using the command SendReturnValue. See
the Plexil manual (plexil.sourceforge.net) for more information.

.. _special_purpose_nodes:

Special Purpose Nodes
---------------------

These forms expand into nodes that perform convenient functions.

--------------

::

   (Action name [ form ] ...)
   (action name [ form ] ...)

Specifies any kind of action. The specified forms can include any node
clauses (except NodeId, which is given by ``name``, as well as any
number of actions. The actions form the body of the generated List Node.

--------------

::

   (Nothing ``)
   (nothing ``)

An action that does nothing. This becomes an anonymous empty node.

--------------

::

   (When condition action [ action ] ...)
   (when condition action [ action ] ...)

Executes actions (concurrently) when condition becomes true. This is
essentially a *monitor*.

--------------

::

   (Wait units [ name ] )
   (wait units [ name ] )

Waits given number of time units.

--------------

::

   (WaitWithTolerance units tolerance [ name ] )
   (wait-with-tolerance units tolerance [ name ] )

Waits given number of time units with given tolerance.

--------------

::

   (SynchronousCommand [ name-or-first-form ] [ &rest ] [ forms ] )
   (synchronous-command [ name-or-first-form ] [ &rest ] [ forms ] )

The Synchronous Command action, which waits for its return value or
status handle

--------------

::

   (timeout exp)
   (Timeout exp)

Specify a timeout clause, whose argument should be a numeric expression.

--------------

::

   (tolerance exp)
   (Tolerance exp)

Specify a tolerance value, whose argument should be a real number or
variable.

--------------

::

   (let vars form [ form ] ...)
   (Let vars form [ form ] ...)

Declares variables that are lexically scoped to the enclosing forms,
similar to LET in Lisp.

.. _resource_clauses:

Resource Clauses
======================

::

   (ResourceList [ resource ] ...)
   (resource-list [ resource ] ...)
   (Resources [ resource ] ...)
   (resources [ resource ] ...)

List of Resource specifications.

--------------

::

   (Resource name priority [ clause ] ...)
   (resource name priority [ clause ] ...)

A Resource specification. Name and priority are required. The remaining
clauses can be ``ResourceUpperBound`` or ``ResourceLowerBound``

--------------

::

   (ResourceUpperBound x)
   (resource-upper-bound x)

A resource upper bound.

--------------

::

   (ResourceLowerBound x)
   (resource-lower-bound x)

A resource lower bound.

.. _miscellaneous_node_clauses:

Miscellaneous Node Clauses
===============================

::

   (NodeComment [ sentence ] ...)
   (node-comment [ sentence ] ...)

A comment for the node. (The sentences will be concatenated.)

--------------

::

   (Permissions p)
   (permissions p)

The node's permissions.

--------------

::

   (Comment [ sentence ] ...)
   (comment [ sentence ] ...)

This creates a comment in the XML, and may occur in any number within a
node. It is useful for commenting your plan in a way that will also be
reflected in the XML.

.. _miscellaneous_expressions:

Miscellaneous Expressions
============================

::

   (Concat [ expr ] ...)
   (concat [ expr ] ...)

A string expression is either a string literal, StringValue expression,
string variable, lookup, or another Concat expression. The Concat form
takes 0 or more of these and returns a string that is the concatenation
of the evaluated expressions. Concat() is the empty string. Concat(x) =
x

--------------

::

   (TimepointValue nodeid node-state-value timepoint)
   (timepoint-value nodeid node-state-value timepoint)

Returns the amount of time since the specified state of the specified
node has either started or ended. node-state-value must be one of
INACTIVE, WAITING, FINISHED, ITERATION_ENDED, EXECUTING, FAILING,
FINISHING. Timepoint must be one of START, END.

--------------

::

   (StartTime nodeid)
   (start-time nodeid)

Time when given node started executing.

--------------

::

   (EndTime nodeid)
   (end-time nodeid)

Time when given node stopped executing.

--------------

::

   (PendingStart nodeid)
   (pending-start nodeid)

Did the given node started executing?

--------------

::

   (PendingEnd nodeid)
   (pending-end nodeid)

Did the given node finish?

--------------

::

   (NodeState nodeid)
   (node-state nodeid)

Specifies the state of the node with the given ID.

--------------

::

   (NodeOutcome nodeid)
   (node-outcome nodeid)

Specifies the outcome of the node with the given ID.

--------------

::

   (isKnown v)
   (IsKnown v)
   (is-known v)

Returns true or false depending on whether the value of the given
declared variable, node state variable, node outcome variable, or node
timepoint value is known.

--------------

::

   (starts-after-start d1 d2 id)
   (StartsAfterStart d1 d2 id)

A boolean expression stating a start time that is [d1 d2] after the
start of action named ID.

--------------

::

   (current-time ``)
   (CurrentTime ``)

Lookup the time.

.. _plexil_simulation_script:

PLEXIL Simulation Script
========================

This section describes the forms (constructs) that comprise simulation
.scripts, which are used to test plans with the |PLEXIL| Test Executive.

--------------

::

   (PlexilScript [ form ] ...)
   (plexil-script [ form ] ...)

The top level form for a script. A script must contain exactly one of
these, and nothing else.

--------------

::

   (InitialState [ form ] ...)
   (initial-state [ form ] ...)

Defines the initial state section of the script, which is optional. It
consists of ``State`` forms.

--------------

::

   (Script [ form ] ...)
   (script [ form ] ...)

Defines the "script" section of the script, which is required. It
consists of any of the following forms.

NOTE: In the following 5 forms that have a ``value`` or ``result``
argument, the argument may be either a single value or a list of values.
The list must not be quoted, e.g. ``(1 2 3)``. A list indicates that
that return value or result is an array. Its elements must all be of the
same type. For boolean arrays, you must use 0 and 1 for true and false,
respectively.

--------------

::

   (State name type value [ param ] ...)
   (state name type value [ param ] ...)

Sets the state of given name and type and parameters to the given
value(s).

--------------

::

   (CommandAck name type result [ param ] ...)
   (command-ack name type result [ param ] ...)

Acknowledges the named command with given parameters, returning the
given result(s) of given type.

--------------

::

   (Command name type result [ param ] ...)
   (command name type result [ param ] ...)

Simulates the completion of the named command with given parameters
returning the given result(s) of given type.

--------------

::

   (CommandAbort name type result [ param ] ...)
   (command-abort name type result [ param ] ...)

Simulates the abort of the named command with given parameters returning
the given result(s) of given type.

--------------

::

   (Param value [ type ] )
   (param value [ type ] )

Defines a parameter *value*, with optional type.

--------------

::

   (ParamString str [ str ] ...)
   (param-string str [ str ] ...)

Like the above, but useful for long strings (they are concatenated).

--------------

::

   (UpdateAck name)
   (update-ack name)

Acknowledges a plan update.

--------------

::

   (Simultaneous [ form ] ...)
   (simultaneous [ form ] ...)

Wraps script actions that should occur simultaneously.

--------------

::

   (Comment [ sentence ] ...)
   (comment [ sentence ] ...)

This creates a comment in the XML, and may occur in any number within a
script. It is useful for commenting your script in a way that will also
be reflected in the XML.

.. _simulation_script_shortcuts:

Simulation Script Shortcuts
---------------------------

This section describes various shortcuts for simulation scripts. They
each expand into some combination of the forms in the previous section.

--------------

::

   (CommandSuccess name [ param ] ...)
   (command-success name [ param ] ...)

Sends a COMMAND_SUCCESS handle for the given command invocation. NOTE:
If your plan is awaiting a return value from the command itself, you
must return this (using the ``Command`` form) *before* this handle.

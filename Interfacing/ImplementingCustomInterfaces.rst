.. _ImplementingCustomInterfaces:

.. contents::

Implementing Custom Interfaces
=================================

*30 Apr 2015*

This chapter outlines how the |PLEXIL| Executive can be interfaced to
arbitrary external systems through the addition of custom *interface
adapters* and *exec listeners*.

Overview
--------

Interface adapters allow plans to query state, perform required actions,
and provide feedback to the outside world. Exec listeners allow external
observers to monitor changes in plan state.

.. _implementing_interface_adapters:

Implementing Interface Adapters
-------------------------------

Interface adapters handle three types of operations: lookups, commands,
and planner updates. Each is (usually) independent of the others, though
this is a design choice left to the interface implementor. In addition,
they are required to implement the lifecycle API below to manage
initialization, activation, deactivation, and shutdown.

The abstract base class
:ref:`InterfaceAdapter <InterfaceAdapter>`
declares the API which interface developers must implement.

Lifecycle
~~~~~~~~~

Please see :ref:`The Lifecycle Model <the_lifecycle_model>`
for an overview of the phases of application execution.

Interface adapter classes must implement the following lifecycle
management methods, whose prototypes are declared in the
:ref:`InterfaceAdapter <InterfaceAdapter>`
abstract base class:

::

    bool initialize(); // set up internal data structures
    bool start(); // start communicating
    bool stop(); // stop communicating
    bool reset(); // reset to initialized state
    bool shutdown(); // release any resources and prepare to be deleted

The return value for each method is a ``bool`` value representing
whether the method succeeded or failed. Success is indicated by a
``true`` return value. So a minimal ``initialize()``\ method would look
like this example:

::

   bool MyAdapter::initialize()
   {
     return true;
   }

.. _data_flow_and_threading:

Data Flow and Threading
~~~~~~~~~~~~~~~~~~~~~~~

In the :ref:`PLEXIL Executive <PLEXILExecutive>`, there is a thread
dedicated to running the execution engine. Adapter methods called by the
exec are called from this thread.

Results of execution (LookupOnChange data, command and update
acknowledgments, command return values) are often (but not always)
delivered by other threads. These results are stored temporarily in a
*queue* inside the interface manager, and delivered to the execution
engine when its thread is active again.

The exec thread sleeps between execution cycles. Wakeups are delivered
by calling ``InterfaceAdapter::notifyOfExternalEvent()``. Enqueued data
are processed immediately after the wakeup is delivered.

Application developers can choose between waking the exec after every
datum is delivered, waking up after a batch of data is delivered (e.g.
enqueuing a telemetry packet's worth of data at a time), or running the
exec thread on a strict tick-based schedule. The choice is a tradeoff
between responsiveness requirements, CPU utilization, scheduler
overhead, and other application dependent factors.

Lookups
~~~~~~~

The |PLEXIL| language defines two external state query operators,
LookupNow and LookupOnChange. LookupNow largely means what it says:
"give me the current value of this external state." LookupOnChange means
"give me the current value, AND tell me when the value changes."

Lookups in the |PLEXIL| exec are implemented with the aid of a *state
cache*, a table of state names and their most recent values with
timestamps. Putting a value in the state cache when there is no active
lookup for that state name is permitted.

.. _adapter_methods:

Adapter Methods
^^^^^^^^^^^^^^^

The following member function is called for both LookupNow and
LookupOnChange:

-  void lookupNow(State const &state, StateCacheEntry &cacheEntry);

``lookupNow()`` returns the requested value by updating the
:ref:`StateCacheEntry <StateCacheEntry>`
object supplied in the call.

.. important::

    This member function is called synchronously inside
    the |PLEXIL| exec's main decision-making loop. Blocking (e.g. waiting for
    another process, or another system, to respond) during a ``lookupNow()``
    call should be avoided whenever possible, as it delays plan execution.

LookupOnChange can be implemented by the following member functions:

::

    void subscribe(const State& state);

Tells the interface adapter to notify the exec whenever a new value for
the state arrives.

::

    void unsubscribe(const State& state);

Tells the interface adapter to stop notifying the exec of new values for
this state.

``LookupOnChange`` of rapidly varying numeric quantities can be optimized
with the following optional member functions:

::

    void setThresholds(const State& state, double hi, double lo);
    void setThresholds(const State& state, int32_t hi, int32_t lo);

These tell the adapter to ignore changes in the state's value until they
meet or exceed the supplied thresholds.

.. note::

    Any adapter for the ``time`` state MUST implement
    ``setThresholds()``.

.. _statecacheentry_member_functions:

StateCacheEntry member functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The
:ref:`StateCacheEntry <StateCacheEntry>`
class provides the following member functions for implementing return
values from LookupNow:

::

    void setUnknown();

Returns an **UNKNOWN** value.

::

    void update(bool const &val);
    void update(int32_t const &val);
    void update(double const &val);
    void update(std::string const &val);

These return a Boolean, Integer, Real, or String value respectively.

::

    void updatePtr(std::string const *valPtr);

This returns a String value.

::

    void updatePtr(BooleanArray const *valPtr);
    void updatePtr(IntegerArray const *valPtr);
    void updatePtr(RealArray const *valPtr);
    void updatePtr(StringArray const *valPtr);

These are used to return array values.

::

    void update(Value const &val);

This is the generic version for returning a
:ref:`Value <Value>` instance.

.. _adapterexecinterface_member_functions:

AdapterExecInterface member functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These member functions are used to return values from LookupOnChange, or
to post values to the state cache asynchronously, independent of any
queries.

::

    void handleValueChange(State const &state, const Value& value);

Enqueues a state value update entry in the interface manager's queue.

::

    void notifyOfExternalEvent();

Tells the interface manager to wake up the exec and process the entries
in the queue. Several ``handleValueChange()`` and other calls can be
batched and processed by a single ``notifyOfExternalEvent()`` call.

Commands
~~~~~~~~

.. _adapter_methods_1:

Adapter methods
^^^^^^^^^^^^^^^

Adapter classes must implement the following methods to support
commands.

::

    void executeCommand(Command *cmd);

Send the command to the external system. At the very least this method
must cause a *command handle value* to be sent back to the exec.

::

    void invokeAbort(Command *cmd);

Abort the specified command. Called when a Command node is prematurely
ended. This method must at minimum cause an abort acknowledgment to be
sent back.

.. _command_member_functions:

Command member functions
^^^^^^^^^^^^^^^^^^^^^^^^

These are functions for extracting information from a Command object.

::

    std::string const &getName() const;

Get the command's name as a string.

::

    std::vector<Value> const &getArgValues() const;

Get a vector of the argument values.

.. _adapterexecinterface_member_functions_1:

AdapterExecInterface member functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These are the interface manager member functions for responding to a
command.

::

    void handleCommandAck(Command * cmd, CommandHandleValue value);

Enqueues the command handle (acknowledgment) value in the interface
manager's queue.

::

    void handleCommandReturn(Command * cmd, Value const& value);

Enqueues the command's return value in the interface manager's queue.

::

    void handleCommandAbortAck(Command * cmd, bool ack);

Enqueues the abort acknowledgment in the interface manager's queue.

::

    void notifyOfExternalEvent();

As with lookups, tells the interface manager to wake up the exec and
process the queue. Also as with lookups, several enqueued responses can
be processed with a single call.

.. _update_nodes:

Update Nodes
~~~~~~~~~~~~

Comprehensive documentation on interfacing the Update Node (or *planner
update*) is yet to be supplied. The following is a quick guide. Let's
assume your interface adapter class, derived from ``InterfaceAdapter``
is named ``MyAdapter``.

1. Don't use the virtual function
``InterfaceAdapter::sendPlannerUpdate(Update*)``, which is deprecated.
Instead, define a *static* member function in ``MyAdapter`` that has the
following signature, using any function/argument names you like:

::

   static void myPlannerUpdate (PLEXIL::Update* update, PLEXIL::AdapterExecInterface* exec);

2. This function, the Update node handler, must contain the following
line in order to transition the Update node to the Finished state. Your
application logic will determine where to place this line.

::

   exec->handleUpdateAck (update, true);

3. In ``MyAdapter::initialize()``, the following line is needed to
register your Update handler:

::

   g_configuration->registerPlannerUpdateHandler(OwAdapter::myPlannerUpdate);

Note that ``g_configuration`` is a global variable defined in the |PLEXIL|
header ``AdapterConfiguration.hh``, which you may need to include.

4. In your interface configuration (``.xml`` file), in the ``<Adapter AdapterType="...">`` element, add
the following (empty) element:

::

   <PlannerUpdate/>

Registration
~~~~~~~~~~~~

Interface adapters must be registered with the
:ref:`AdapterConfiguration <AdapterConfiguration>`
object. There are several ways to handle this.

*more to be supplied*

.. _implementing_exec_listeners_and_filters:

Implementing Exec Listeners and Filters
---------------------------------------

''to be supplied'

.. _integrating_interface_code_as_a_shared_library:

Integrating Interface Code as a Shared Library
----------------------------------------------

*to be supplied*


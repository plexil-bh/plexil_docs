.. _APIReference:

API Reference
===============

*28 April 2015*

This chapter summarizes the C++ variables, functions, and classes that a
developer might need to implement custom interfaces or applications for
a |PLEXIL| executive.

.. contents::

Overview
--------

This is a quick reference for the API of the |PLEXIL| Application
Framework. It is intended as a narrative overview. Where it differs from
the source code, the source code should always be considered current and
authoritative.

All symbols are in the ``PLEXIL`` namespace unless otherwise specified.

.. _application_framework:

Application Framework
---------------------

The |PLEXIL| application framework is intended to facilitate
implementation of custom interfaces and applications for the |PLEXIL|
executive.

.. _global_variables:

Global Variables
~~~~~~~~~~~~~~~~

These global variables replace the Singleton accessors used in previous
releases.

-  ``ExecConnector *g_exec`` - Pointer to the
   :ref:`PlexilExec <PlexilExec>` instance, cast to a pointer to its
   abstract base class :ref:`ExecConnector <ExecConnector>`.
-  ``AdapterConfiguration *g_configuration`` - Pointer to the
   :ref:`AdapterConfiguration <AdapterConfiguration>` instance.
-  ``InterfaceManager *g_manager`` - Pointer to the
   :ref:`InterfaceManager <InterfaceManager>` instance.
-  ``ExternalInterface *g_interface`` - Pointer to the
   :ref:`InterfaceManager <InterfaceManager>` instance, cast to a pointer
   to its base class ``ExternalInterface``.

Note that ``g_manager`` and ``g_interface`` should point to the same
``InterfaceManager`` instance at all times.

Functions
~~~~~~~~~

.. _plan_parsing:

Plan parsing
^^^^^^^^^^^^

|PLEXIL| plans are loaded in two stages. First, the XML text is read and
parsed into the :ref:`pugixml <pugixml>` representation. Then the pugixml
representation is used as a blueprint for constructing the plan.

The two stages are separate because the library node facility uses the
XML DOM to represent uninstantiated library node templates.

::

    extern pugi::xml_document *loadXmlFile(std::string const &filename)
      throw (ParserException);

Parses the named XML file. Returns a ``pugi::xml_document`` pointer. The
caller is responsible for disposing of the return value when finished. A
``ParserException`` is thrown if the input is not valid XML.

::

    extern Node *parsePlan(pugi::xml_node const xml)
      throw (ParserException);

Uses ``xml`` to construct a tree of |PLEXIL| Node instances, and returns a
pointer to the node at the root of the tree. The caller is responsible
for disposing of the return value when finished. A ``ParserException``
is thrown if the input cannot be parsed into a |PLEXIL| plan.

Classes
~~~~~~~

.. _ExecConnector:

ExecConnector
^^^^^^^^^^^^^

An abstract base class defining the API of the |PLEXIL| executive engine.
The existence of this class facilitates unit testing.

Relevant virtual member functions are:

::

    /**
     * @brief Add the plan under the node named by the parent.
     * @param root The internal representation of the plan.
     * @return True if succesful, false otherwise.
     */
    virtual bool addPlan(Node *root) = 0;

    /**
     * @brief Begins a single "macro step" i.e. the entire quiescence cycle.
     */
    virtual void step(double startTime) = 0;

    /**
     * @brief Returns true if the Exec needs to be stepped.
     */
    virtual bool needsStep() const = 0;
    virtual void setExecListener(ExecListenerBase * l) = 0;
    virtual void deleteFinishedPlans() = 0;
    virtual bool allPlansFinished() const = 0;

PlexilExec
^^^^^^^^^^

The concrete realization of the :ref:`ExecConnector <ExecConnector>` API
as a plan execution engine.

The default constructor and the destructor are the only relevant
additions to the ExecConnector members above.

ExecApplication
^^^^^^^^^^^^^^^

A concrete class which ties together the executive and its interfaces
into a runnable application, manages the life cycle of interfaces, and
optionally provides support for threading.

The constructor and destructor:

::

    ExecApplication();
    virtual ~ExecApplication();

These are the application life cycle members:

::

    /**
      * @brief Initialize all internal data structures and interfaces.
      * @param configXml Configuration data to use.
      * @return true if successful, false otherwise.
      * @note The caller must ensure that all adapter and listener factories
      *       have been created and registered before this call.
      */
    virtual bool initialize(pugi::xml_node const configXml);

    /**
      * @brief Start all the interfaces prior to execution.
      * @return true if successful, false otherwise.
      */
    virtual bool startInterfaces();

    /**
      * @brief Runs the initialized Exec.
      * @return true if successful, false otherwise.
      */
    virtual bool run();

    /**
      * @brief Suspends the running Exec.
      * @return true if successful, false otherwise.
      * @note Can only be suspended from APP_RUNNING state.
      */
    virtual bool suspend();

    /**
      * @brief Resumes a suspended Exec.
      * @return true if successful, false otherwise.
      * @note Can only resume from suspended state, i.e. 
      *   application state is APP_READY and isSuspended() is true.
      */
    virtual bool resume();

    /**
      * @brief Stops the Exec.
      * @return true if successful, false otherwise.
      */
    virtual bool stop();
      
    /**
      * @brief Resets a stopped Exec so that it can be run again.
      * @return true if successful, false otherwise.
      */
    virtual bool reset();

    /**
      * @brief Shuts down a stopped Exec.
      * @return true if successful, false otherwise.
      */
    virtual bool shutdown();

Plan and library loading:

::

    /**
      * @brief Add a plan as an XML document.
      * @return true if successful, false otherwise.
      */
    virtual bool addPlan(pugi::xml_document* planXml);

    /**
      * @brief Add the specified directory name to the end of the library node loading path.
      * @param libdir The directory name.
      */
    void addLibraryPath(const std::string& libdir);

    /**
      * @brief Add the specified directory names to the end of the library node loading path.
      * @param libdirs The vector of directory names.
      */
    void addLibraryPath(const std::vector<std::string>& libdirs);

    /**
      * @brief Add a library as an XML document.
      * @return true if successful, false otherwise.
      */
    virtual bool addLibrary(pugi::xml_document* libXml);

    /**
      * @brief Load the named library from the library path.
      * @param name The name of the library.
      * @return true if successful, false otherwise.
      */
    virtual bool loadLibrary(std::string const &name);

Plan execution:

::

       /**
        * @brief Step the Exec until the queue is empty.
        * @return true if successful, false otherwise.
        * @note Can only be called in APP_READY state.
        * @note Can be called when application is suspended.
        * @note Acquires m_execMutex and holds until done.  
        */
       virtual bool stepUntilQuiescent();

.. _AdapterConfiguration:

AdapterConfiguration
^^^^^^^^^^^^^^^^^^^^

A class which provides registration and lookup services for external
interfacing objects.

The application life cycle API:

::

    /**
     * @brief Performs basic initialization of the interface and all adapters.
     * @return true if successful, false otherwise.
     */
    bool initialize();

    /**
     * @brief Prepares the interface and adapters for execution.
     * @return true if successful, false otherwise.
     */
    bool start();

    /**
     * @brief Halts all interfaces.
     * @return true if successful, false otherwise.
     */
    bool stop();

    /**
     * @brief Resets the interface prior to restarting.
     * @return true if successful, false otherwise.
     */
    bool reset();

    /**
     * @brief Shuts down the interface.
     * @return true if successful, false otherwise.
     */
    bool shutdown();

Member functions which may be useful to application developers:

::

    /**
     * @brief Add an externally constructed interface adapter.
     * @param adapter The adapter ID.
     */
    void addInterfaceAdapter(InterfaceAdapter *adapter);

    /**
     * @brief Add an externally constructed ExecListener.
     * @param listener Pointer to the listener
     */
    void addExecListener(PlexilListener *listener);

Member functions which are useful to application and interface
developers:

::

    /**
     * @brief Register the given interface adapter.
     * @param adapter The interface adapter to be registered.
     */
    void defaultRegisterAdapter(InterfaceAdapter *adapter);

    /**
     * @brief Register the given interface adapter for this command.
     Returns true if successful.  Fails and returns false
     iff the command name already has an adapter registered
              or setting a command interface is not implemented.
     * @param commandName The command to map to this adapter.
     * @param intf The interface adapter to handle this command.
     */
    bool registerCommandInterface(std::string const &commandName,
                                  InterfaceAdapter *intf);

    /**
     * @brief Register the given interface adapter for lookups to this state.
     Returns true if successful.  Fails and returns false
     if the state name already has an adapter registered
              or registering a lookup interface is not implemented.
     * @param stateName The name of the state to map to this adapter.
     * @param intf The interface adapter to handle this lookup.
     */
    bool registerLookupInterface(std::string const &stateName,
                                 InterfaceAdapter *intf);

    /**
     * @brief Register the given interface adapter for planner updates.
              Returns true if successful.  Fails and returns false
              iff an adapter is already registered
              or setting the default planner update interface is not implemented.
     * @param intf The interface adapter to handle planner updates.
     */
    bool registerPlannerUpdateInterface(InterfaceAdapter *intf);

    /**
     * @brief Register the given interface adapter as the default for all lookups and commands
     which do not have a specific adapter.  Returns true if successful.
     Fails and returns false if there is already a default adapter registered
              or setting the default interface is not implemented.
     * @param intf The interface adapter to use as the default.
     */
    bool setDefaultInterface(InterfaceAdapter *intf);

    /**
     * @brief Register the given interface adapter as the default for lookups.
             This interface will be used for all lookups which do not have
          a specific adapter.
     * @param intf The interface adapter to use as the default.
     * @return True if successful, false if there is already a default adapter registered.
     */
    bool setDefaultLookupInterface(InterfaceAdapter *intf);

    /**
     * @brief Register the given interface adapter as the default for commands.
              This interface will be used for all commands which do not have
          a specific adapter.
          Fails and returns false if there is already a default command adapter registered.
     * @param intf The interface adapter to use as the default.
     * @return True if successful, false if there is already a default adapter registered.
     */
    bool setDefaultCommandInterface(InterfaceAdapter *intf);

Plan and library loading path access:

::

    /**
     * @brief Get the search path for library nodes.
     * @return A reference to the library search path.
     */
    const std::vector<std::string>& getLibraryPath() const;

    /**
     * @brief Get the search path for plan files.
     * @return A reference to the plan search path.
     */
    const std::vector<std::string>& getPlanPath() const;

    /**
     * @brief Add the specified directory name to the end of the library node loading path.
     * @param libdir The directory name.
     */
    void addLibraryPath(const std::string& libdir);

    /**
     * @brief Add the specified directory names to the end of the library node loading path.
     * @param libdirs The vector of directory names.
     */
    void addLibraryPath(const std::vector<std::string>& libdirs);

    /**
     * @brief Add the specified directory name to the end of the plan loading path.
     * @param libdir The directory name.
     */
    void addPlanPath(const std::string& libdir);

    /**
     * @brief Add the specified directory names to the end of the plan loading path.
     * @param libdirs The vector of directory names.
     */
    void addPlanPath(const std::vector<std::string>& libdirs);

.. _ExternalInterface:

ExternalInterface
^^^^^^^^^^^^^^^^^

An abstract base class defining the API used by the |PLEXIL| executive
engine to talk to the outside world.

Application developers should have no need to interact directly with
this class; use the member functions on
:ref:`InterfaceManager <InterfaceManager>` or
:ref:`AdapterExecInterface <AdapterExecInterface>` instead.

.. _AdapterExecInterface:

AdapterExecInterface
^^^^^^^^^^^^^^^^^^^^

An abstract base class defining the API used by interface adapters to
talk to the :ref:`InterfaceManager <InterfaceManager>`. Its existence as a
separate class is intended to hide knowledge of the interface manager
internals from adapters.

Member functions relevant to interface adapters:

::

    /**
     * @brief Notify of the availability of a new value for a lookup.
     * @param state The state for the new value.
     * @param value The new value.
     */
    virtual void handleValueChange(State const &state, const Value& value) = 0;

    /**
     * @brief Notify of the availability of a command handle value for a command.
     * @param cmd Pointer to the Command instance.
     * @param value The new value.
     */
    virtual void handleCommandAck(Command * cmd, CommandHandleValue value) = 0;

    /**
     * @brief Notify of the availability of a return value for a command.
     * @param cmd Pointer to the Command instance.
     * @param value The new value.
     */
    virtual void handleCommandReturn(Command * cmd, Value const& value) = 0;

    /**
     * @brief Notify of the availability of a command abort acknowledgment.
     * @param cmd Pointer to the Command instance.
     * @param ack The acknowledgment value.
     */
    virtual void handleCommandAbortAck(Command * cmd, bool ack) = 0;

    /**
     * @brief Notify of the availability of a planner update acknowledgment.
     * @param upd Pointer to the Update instance.
     * @param ack The acknowledgment value.
     */
    virtual void handleUpdateAck(Update * upd, bool ack) = 0;

    /**
     * @brief Notify the executive of a new plan.
     * @param planXml The pugixml representation of the new plan.
     */
    virtual void handleAddPlan(pugi::xml_node const planXml)
      throw (ParserException)
      = 0;

    /**
     * @brief Notify the executive of a new library node.
     * @param planXml Pointer to the XML document containing the new library node
     * @note Deletion of the XML document will be handled by the interface.
     */
    virtual void handleAddLibrary(pugi::xml_document *planXml)
      throw (ParserException)
      = 0;

    /**
     * @brief Notify the executive that it should run one cycle.  This should be sent after
     each batch of lookup and command return data.
    */
    virtual void notifyOfExternalEvent() = 0;

    /**
     * @brief Run the exec and wait until all events in the queue have been processed.
     * @note Only implemented when thread support is enabled, i.e. the preprocessor macro PLEXIL_WITH_THREADS is defined.
     */
    virtual void notifyAndWaitForCompletion() = 0;

InterfaceManager
^^^^^^^^^^^^^^^^

The concrete realization of the
:ref:`ExternalInterface <ExternalInterface>` and
:ref:`AdapterExecInterface <AdapterExecInterface>` APIs.

.. _InterfaceAdapter:

InterfaceAdapter
^^^^^^^^^^^^^^^^

An abstract base class defining the API of external interfaces as seen
by the :ref:`InterfaceManager <InterfaceManager>`.

The constructors require a reference to the
:ref:`AdapterExecInterface <AdapterExecInterface>` instance, which is
always an ``InterfaceManager``:

::

    /**
      * @brief Constructor.
      * @param execInterface Reference to the parent AdapterExecInterface object.
      */
    InterfaceAdapter(AdapterExecInterface& execInterface);

    /**
      * @brief Constructor from configuration XML.
      * @param execInterface Reference to the parent AdapterExecInterface object.
      * @param xml The XML element describing this adapter
      * @note The instance maintains a shared reference to the XML.
      */
    InterfaceAdapter(AdapterExecInterface& execInterface, 
                      pugi::xml_node const xml);

The base class provides these accessors:

::

    /**
     * @brief Get the configuration XML for this instance.
     */
    pugi::xml_node const getXml();

    /**
     * @brief Get the AdapterExecInterface for this instance.
     */
    AdapterExecInterface& getExecInterface()

The base class also implements this method for registering the adapter
with the AdapterConfiguration. Derived classes may override it if
desired:

::

    /**
     * @brief Register this adapter based on its XML configuration data.
     * @note The adapter is presumed to be fully initialized and working at the time of this call.
     * @note This is a default method; adapters are free to override it.
     */
    virtual void registerAdapter();

**Below this line are the methods which must be implemented for any
class derived from ``InterfaceAdapter``.**

The lifecycle API must be implemented in any derived class. The methods
can be stubs, but they must return ``true`` as a minimum.

::

    /**
      * @brief Initializes the adapter, possibly using its configuration data.
      * @return true if successful, false otherwise.
      */
    virtual bool initialize() = 0;

    /**
      * @brief Starts the adapter, possibly using its configuration data.  
      * @return true if successful, false otherwise.
      */
    virtual bool start() = 0;

    /**
      * @brief Stops the adapter.  
      * @return true if successful, false otherwise.
      */
    virtual bool stop() = 0;

    /**
      * @brief Resets the adapter.  
      * @return true if successful, false otherwise.
      * @note Adapters should provide their own methods.
      */
    virtual bool reset() = 0;

    /**
      * @brief Shuts down the adapter, releasing any of its resources.
      * @return true if successful, false otherwise.
      * @note Adapters should provide their own methods.
      */
    virtual bool shutdown() = 0;

The lookup API:

::

    /**
     * @brief Perform an immediate lookup on an existing state.
     * @param state The state.
     * @return The current value for the state.
     * @note Adapters should provide their own methods. The default method warns and sets the cache entry to unknown.
     */
    virtual void lookupNow(State const &state, StateCacheEntry &cacheEntry);

    /**
     * @brief Inform the interface that it should report changes in value of this state.
     * @param state The state.
     * @note Adapters should provide their own methods.  The default method raises an assertion.
     */
    virtual void subscribe(const State& state);

    /**
     * @brief Inform the interface that a lookup should no longer receive updates.
     * @note Adapters should provide their own methods.  The default method raises an assertion.
     */
    virtual void unsubscribe(const State& state);
    
    /**
     * @brief Advise the interface of the current thresholds to use when reporting this state.
     * @param state The state.
     * @param hi The upper threshold, at or above which to report changes.
     * @param lo The lower threshold, at or below which to report changes.
     * @note Adapters should provide their own methods as appropriate.  The default methods do nothing.
     */
    virtual void setThresholds(const State& state, double hi, double lo);
    virtual void setThresholds(const State& state, int32_t hi, int32_t lo);

The command API:

::

    /**
     * @brief Execute a command with the requested arguments.
     * @param cmd The Command instance.
     */
    virtual void executeCommand(Command *cmd);

    /**
     * @brief Abort the pending command.
     * @param cmd Pointer to the command being aborted.
     * @note Derived classes may implement this method.
     */
    virtual void invokeAbort(Command *cmd);

The planner update API:

::

    /**
     * @brief Send the name of the supplied node, and the supplied value pairs, to the planner.
     * @param update Pointer to the Update object.
     * @note Derived classes may implement this method.
     */
    virtual void sendPlannerUpdate(Update *update);

.. _ExecListenerBase:

ExecListenerBase
^^^^^^^^^^^^^^^^

An abstract base class defining the API used by the
:ref:`PlexilExec <PlexilExec>` to notify the outside world of plan state
changes.

.. _PlexilListener:

PlexilListener
^^^^^^^^^^^^^^

An abstract base class derived from
:ref:`ExecListenerBase <ExecListenerBase>`, adding APIs used by the
:ref:`InterfaceManager <InterfaceManager>` to notify the outside world
when new plans or libraries are loaded.

ExecListener
^^^^^^^^^^^^

An abstract base class derived from
:ref:`PlexilListener <PlexilListener>`, which |PLEXIL| application
developers may extend to notify the outside world of events inside the
executive. Optionally uses an instance of
:ref:`ExecListenerFilter <ExecListenerFilter>` to perform event filtering.

The constructors:

::

    /**
     * @brief Default constructor.
     */
    ExecListener();

    /**
     * @brief Constructor from configuration XML
     * @param xml Pointer to the (shared) configuration XML describing this listener.
     */
    ExecListener(pugi::xml_node const xml);

The ``setFilter()`` accessor may be used to install a custom
:ref:`ExecListenerFilter <ExecListenerFilter>` instance to select events
for notification. Only applications which do not use the :ref:`interface configuration file <InterfaceConfigurationFile>` approach need to
call this accessor.

::

    /**
     * @brief Set the filter of this instance.
     * @param fltr Pointer to the filter.
     */
    void setFilter(ExecListenerFilter *fltr);

The ``getXml()`` accessor can be used to extract configuration
information:

::

    pugi::xml_node const getXml() const;

**Below this line are the member functions which a derived class may
override if desired. All have default methods.**

The lifecycle member functions:

::

    /**
     * @brief Perform listener-specific initialization.
     * @return true if successful, false otherwise.
     */
    virtual bool initialize();

    /**
     * @brief Perform listener-specific startup.
     * @return true if successful, false otherwise.
     */
    virtual bool start();

    /**
     * @brief Perform listener-specific actions to stop.
     * @return true if successful, false otherwise.
     */
    virtual bool stop();

    /**
     * @brief Perform listener-specific actions to reset to initialized state.
     * @return true if successful, false otherwise.
     */
    virtual bool reset();

    /**
     * @brief Perform listener-specific actions to shut down.
     * @return true if successful, false otherwise.
     */
    virtual bool shutdown();

The notification implementation member functions have ``protected``
access:

::

    /**
     * @brief Notify that nodes have changed state.
     * @param Vector of node state transition info.
     * @note Current states are accessible via the node.
     */
    virtual void implementNotifyNodeTransitions(std::vector<NodeTransition> const & /* transitions */) const;

    /**
     * @brief Notify that a node has changed state.
     * @param prevState The old state.
     * @param node The node that has transitioned.
     * @note The current state is accessible via the node.
     */
    virtual void implementNotifyNodeTransition(NodeState /* prevState */,
                                               Node * /* node */) const;

    /**
     * @brief Notify that a plan has been received by the Exec.
     * @param plan The intermediate representation of the plan.
     */
    virtual void implementNotifyAddPlan(pugi::xml_node const /* plan */) const;

    /**
     * @brief Notify that a library node has been received by the Exec.
     * @param libNode The intermediate representation of the plan.
     */
    virtual void implementNotifyAddLibrary(pugi::xml_node const /* libNode */) const;

    /**
     * @brief Notify that a variable assignment has been performed.
     * @param dest The Expression being assigned to.
     * @param destName A string naming the destination.
     * @param value The value being assigned.
     */
    virtual void implementNotifyAssignment(Expression const * /* dest */,
                                           std::string const & /* destName */,
                                           Value const & /* value */) const;

.. _ExecListenerFilter:

ExecListenerFilter
^^^^^^^^^^^^^^^^^^

An abstract base class which |PLEXIL| application developers may extend to
select the events reported by an instance of a class derived from
:ref:`ExecListener <ExecListener>`.

ParserException
^^^^^^^^^^^^^^^

Used to report an error in plan or script parsing.

Constructors:

::

    ParserException() throw();
    ParserException(const char* msg) throw ();
    ParserException(const char* msg, const char* filename, int offset) throw();
    ParserException(const char* msg, const char* filename, int line, int col) throw();

Several ways to invoke the constructor.

-  ``msg`` is the message to be displayed.
-  ``filename`` is the name of the file in which the error was found, if
   available.
-  ``offset`` is the number of bytes into the file at which the error
   was found. (This is provided because the :ref:`pugixml <pugixml>`
   doesn't maintain a line counter.)
-  ``line`` and ``col`` are the line number and column number,
   respectively, of the error location.

::

    virtual ~ParserException() throw();

Destructor.

::

    ParserException& operator=(const ParserException&) throw();

Assignment operator.

::

    virtual const char *what() const throw();

Returns the error message.

.. _internal_representations:

Internal representations
------------------------

.. _basic_data_types:

Basic Data Types
~~~~~~~~~~~~~~~~

These are the representations used inside the |PLEXIL| Exec.

ValueType
^^^^^^^^^

This is an enumeration which identifies the type of a |PLEXIL| value.

Boolean
^^^^^^^

Represented by the C++ ``bool`` type, effectively a one-byte integer.
``false`` is 0, and ``true`` is any other value, but by convention 1 is
preferred.

Integer
^^^^^^^

A 32-bit signed two's complement integer. Implemented as the C99 type
``int32_t``, defined in standard header file stdint.h.

Real
^^^^

A 64-bit floating point number, as defined by the C type ``double``.
Expected to conform to the IEEE 754 standard.

String
^^^^^^

A character string, implemented as the C++ Standard Library class
``std::string``.

.. _internal_node_values:

Internal Node Values
^^^^^^^^^^^^^^^^^^^^

The |PLEXIL| types ``NodeStateValue``, ``NodeOutcomeValue``,
``NodeFailureValue``, and ``NodeCommandHandleValue`` are implemented as
C enumerated types with non-overlapping ranges.

In the |PLEXIL| implementation, internal node values are stored and passed
as the C99 type ``uint16_t``, defined in standard header file stdint.h.

.. _classes_1:

Classes
~~~~~~~

Array
^^^^^

A base class for homogeneous one-dimensional arrays of |PLEXIL| language
data types ``Boolean``, ``Integer``, ``Real``, or ``String``.

Each ``Array`` has a maximum size, and an element value type. Each
individual element can be unknown or an actual value.

These query methods are generic:

::

    size_t size() const;
    bool elementKnown(size_t index) const;
    bool allElementsKnown() const;
    bool anyElementsKnown() const;
    std::vector<bool> const &getKnownVector() const;

These pure virtual accessors are implemented by derived classes of
specific element types:

::

    virtual ValueType getElementType() const = 0;
    virtual Value getElementValue(size_t index) const = 0;

These pure virtual accessors should be used only after the array's
element type has been checked. Incompatible types may result in a failed
assertion.

::

    virtual bool getElement(size_t index, bool &result) const = 0;
    virtual bool getElement(size_t index, int32_t &result) const = 0;
    virtual bool getElement(size_t index, double &result) const = 0;
    virtual bool getElement(size_t index, std::string &result) const = 0;

These accessors are only implemented on String arrays.

::

    virtual bool getElementPointer(size_t index, std::string const *&result) const = 0;
    virtual bool getMutableElementPointer(size_t index, std::string *&result) = 0;

These accessors give a const reference to the vector actually storing
the array elements.

::

    virtual void getContentsVector(std::vector<bool> const *&result) const = 0;
    virtual void getContentsVector(std::vector<int32_t> const *&result) const = 0;
    virtual void getContentsVector(std::vector<double> const *&result) const = 0;
    virtual void getContentsVector(std::vector<std::string> const *&result) const = 0;

Arrays can be set to all-unknown by the reset method:

::

    virtual void reset();

To resize an array, use this method:

::

    /**
     * @brief Expand the array to the requested size. 
     *        Mark the new elements as unknown.
     *        If already that size or larger, does nothing.
     * @param size The requested size.
     */
    virtual void resize(size_t size);

Elements of the array can be set via the setElement methods:

::

    virtual void setElement(size_t index, bool const &newVal) = 0;
    virtual void setElement(size_t index, int32_t const &newVal) = 0;
    virtual void setElement(size_t index, double const &newVal) = 0;
    virtual void setElement(size_t index, std::string const &newVal) = 0;

.. note::

    A call to ``setElement`` with a numeric value that is not one
    of the specific types above (e.g. ``int``) may result in the ``bool``
    method being selected by the compiler. We recommend explicitly
    converting or casting values to one of the above types to be safe.

.. _Value:

Value
^^^^^

A concrete class representing a typed value in the |PLEXIL| language.
``Value`` instances can represent any legal value in the |PLEXIL|
language.

The :ref:`AdapterExecInterface <AdapterExecInterface>` members
``handleValueChange()`` and ``handleCommandReturn()`` take ``Value``
instances as arguments.

Constructors:

::

    Value();                                      // creates an unknown value of unknown type
    Value(Value const &);                         // copy constructor

    Value(bool val);                              // Boolean
    Value(uint16_t enumVal, ValueType typ);       // internal values, typed UNKNOWN
    Value(int32_t val);                           // Integer
    Value(double val);                            // Real
    Value(std::string const &val);                // String
    Value(char const *val);                       // String; convenience constructor
    Value(BooleanArray const &val);               // Array types
    Value(IntegerArray const &val);
    Value(RealArray const &val);
    Value(StringArray const &val);

    // Constructs the appropriate array type.
    Value(std::vector<Value> const &vals);

.. note::

    A call to a ``Value`` constructor with a numeric value that is
    not one of the specific types above (e.g. ``int``) may result in the
    ``bool`` constructor being selected by the compiler. We recommend
    explicitly converting or casting values to one of the above types to be
    safe.

Assignment operators:

::

    Value &operator=(Value const &);
    Value &operator=(bool val);
    Value &operator=(uint16_t enumVal); // Node-internal values only
    Value &operator=(int32_t val);
    Value &operator=(double val);
    Value &operator=(std::string const &val);
    Value &operator=(char const *val);
    Value &operator=(BooleanArray const &val);
    Value &operator=(IntegerArray const &val);
    Value &operator=(RealArray const &val);
    Value &operator=(StringArray const &val);
    void setUnknown();

..note::

    A call to a ``Value`` assignment operator with a numeric value
    that is not one of the specific types above (e.g. ``int``) may result in
    the ``bool`` method being selected by the compiler. We recommend
    explicitly converting or casting values to one of the above types to be
    safe.

You can ask a ``Value`` what its type is, and if its value is known:

::

    ValueType valueType() const;
    bool isKnown() const;

``getValue()`` and ``getValuePointer`` accessors all take a reference to
the result variable as an argument, and return ``true`` if the value was
known and valid for the variable, ``false`` if it was unknown or of a
type incompatible with the reference.

::

    bool getValue(bool &result) const;
    bool getValue(uint16_t &result) const;
    bool getValue(int32_t &result) const;
    bool getValue(double &result) const;
    bool getValue(std::string &result) const;

    bool getValuePointer(std::string const *&ptr) const;
    bool getValuePointer(Array const *&ptr) const; // generic Array
    bool getValuePointer(BooleanArray const *&ptr) const;
    bool getValuePointer(IntegerArray const *&ptr) const;
    bool getValuePointer(RealArray const *&ptr) const;
    bool getValuePointer(StringArray const *&ptr) const;

Two comparison operators are defined as member functions:

::

    bool equals(Value const &) const;
    bool lessThan(Value const &) const; // for (e.g.) std::map

And several variations are defined as overloaded operators:

::

    bool operator==(Value const &a, Value const &b);
    bool operator!=(Value const &a, Value const &b);
    bool operator<(Value const &a, Value const &b);
    bool operator<=(Value const &a, Value const &b);
    bool operator>(Value const &a, Value const &b);
    bool operator>=(Value const &a, Value const &b);

Print methods:

::

    void print(std::ostream &s) const;
    std::string valueToString() const;

And the overloaded operator ``<<``:

::

    std::ostream &operator<<(std::ostream &, Value const &);

.. _StateCacheEntry:

StateCacheEntry
^^^^^^^^^^^^^^^

A concrete class representing the current value of a Lookup in the
|PLEXIL| language. They are managed by the executive.

The accessors (setters) below are the only member functions an interface
developer should need:

::

    void setUnknown();

    void update(Value const &val);  // convenience

    void update(bool const &val);
    void update(int32_t const &val);
    void update(double const &val);
    void update(std::string const &val);

.. note::

    A call to ``update`` with a numeric value that is not one of
    the specific types above (e.g. ``int``) may result in the ``bool``
    method being selected by the compiler. We recommend explicitly
    converting or casting values to one of the above types to be safe.

::

    void updatePtr(std::string const *valPtr);
    void updatePtr(BooleanArray const *valPtr);
    void updatePtr(IntegerArray const *valPtr);
    void updatePtr(RealArray const *valPtr);
    void updatePtr(StringArray const *valPtr);

.. _other_utilities:

Other utilities
---------------

.. _debug_logging:

Debug logging
~~~~~~~~~~~~~

The ``debugMsg`` and ``condDebugMsg`` macros allow a developer to
selectively print information from inside the Exec.

The debug facility is enabled by default. Use the
``--disable-debug-logging`` option to the ``configure`` script to
disable it. See the ``README`` file in the top level of the
distribution.

Each message has an associated *marker*, a string identifying one or
more related messages, as well as the name of the source file in which
the message appears. The list of active file names and marker patterns
is in the *debug configuration file*, by default the file named
``Debug.cfg`` in the working directory from which the program was
launched.

.. _debug_macros:

Debug macros
^^^^^^^^^^^^

These macros, when inserted into a C++ source file, will print messages
when the appropriate marker and file are enabled. In each case
``marker`` should be a quoted string.

``#define debugMsg(marker, data) ...``

When ``marker`` is enabled, prints the ``data`` to the debug log stream.
Note that ``data`` is expanded into a statement like:

``stream << data ;``

``#define condDebugMsg(cond, marker, data) ...``

Like ``debugMsg`` above, but the message is only printed when the
``marker`` is enabled and the ``cond`` expression is true.

``#define debugStmt(marker, stmt)``

Executes ``stmt`` when ``marker`` is enabled.

``#define condDebugStmt(cond, marker, stmt)``

Executes ``stmt`` when ``marker`` is enabled and the ``cond`` expression
is true.

.. _debug_logging_initialization:

Debug logging initialization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The |PLEXIL| Executive executables already enable debug logging (unless
configured without it during the build). By default the pattern list is
read from the file named ``Debug.cfg`` in the working directory from
which the program was launched.

Applications which implement their own ``main()`` routine can initialize
the facility via the following C++ API. Note that these functions are in
the global namespace, not ``PLEXIL``.

The debug output stream defaults to standard output. If redirecting the
debug output, the stream must be set before any other debug logging
operations are performed:

::

    extern bool setDebugOutputStream(std::ostream &os);

Debug markers are all disabled at startup. Marker patterns can be read
with this function.

::

    extern bool readDebugConfigStream(std::istream &is);

All, none, or a subset of the message markers can be enabled via the
following functions:

::

    extern void enableAllDebugMessages();
    extern void disableAllDebugMessages();
    extern void enableMatchingDebugMessages(char const *file, char const *marker);

The ``file`` and ``marker`` arguments to ``enableMatchingDebugMessages``
may not be ``NULL``.

Macros
~~~~~~

.. _functions_1:

Functions
~~~~~~~~~

.. _finalization_utility:

Finalization utility
^^^^^^^^^^^^^^^^^^^^

Where a cleanup upon program completion is required, but the order in
which the cleanup is performed can't be determined statically, you can
use this utility, which is written in standard C. It allows cleanup
functions to be registered, and executes them in last-in, first-out
order.

::

    typedef void (*lc_operator)() ;

Cleanup functions take no arguments and return no value.

::

    void addFinalizer(lc_operator op);

Registers one cleanup function.

::

    void runFinalizers();

Run all the registered cleanup functions, and free the finalizer list.
This function is most useful if it is the very last function called by a
program before exit.

.. _classes_2:

Classes
~~~~~~~

.. _third_party_code:

Third-party code
----------------

|PLEXIL| uses additional open-source libraries. For documentation of those
libraries, please use the links below.

.. _pugixml:

pugixml
~~~~~~~

`pugixml website <http://pugixml.org/>`_ is a lightweight, high-performance,
cross-platform, DOM-style XML package coded in C++. pugixml is actively
developed and maintained at this writing.


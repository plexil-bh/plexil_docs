.. _ImplementingCustomApplications:

.. contents::

Implementing Custom Applications
======================================

*27 April 2015*

This chapter outlines the reasons one might need a custom application of
the |PLEXIL| Executive, how to implement such an application, and
considerations for the design of that application.

Overview
--------

The :ref:`PLEXIL Executive <PLEXILExecutive>` is a general purpose plan
execution engine, with provisions for loading user-specified interfacing
code at startup time. But some applications have requirements which the
|PLEXIL| Executive cannot meet. Such requirements may include:

-  A need to launch the Exec as a subroutine of a larger program;
-  Lack of run-time facilities for loading interface shared libraries;
-  A need to "lock down" the application's interface code for
   configuration control purposes;
-  Operation as a single thread (process) in a real-time operating
   system environment.

.. _anatomy_of_the_universalexec_application:

Anatomy of the universalExec Application
----------------------------------------

The ``universalExec`` main program simply does the following:

#. Parse command line arguments.
#. Construct the framework objects.
#. Get interface configuration data.
#. Construct interfaces.
#. Initialize interfaces.
#. (optional) Load the specified libraries.
#. (optional) Load the specified plan.
#. Start the interfaces.
#. Start the top-level loop thread.
#. Wait for execution to finish.
#. Destroy interface objects.
#. Destroy framework objects.
#. Exit.

Embedded |PLEXIL| executive applications will need to perform many of
these same steps.

.. _plexil_application_framework_lifecycle_model:

PLEXIL Application Framework Lifecycle Model
--------------------------------------------

For an overview of the lifecycle states, please see :ref:`The Lifecycle Model <the_lifecycle_model>`.

.. _interface_setup:

Interface Setup
---------------

The |PLEXIL| application is responsible for setting up the
AdapterConfiguration instance to route interface requests to the
appropriate objects. AdapterConfiguration has the capability to read
user-supplied XML, load the appropriate libraries, and construct the
interface objects. However, if required AdapterConfiguration can simply
route to interface objects explicitly constructed by the application, in
ways explicitly directed by the application.

.. _interface_configuration_via_xml:

Interface Configuration via XML
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The :ref:`AdapterConfiguration <AdapterConfiguration>` member function
``AdapterConfiguration::constructInterfaces(pugi::xml_node const)``
parses configuration XML and constructs the interface adapters and exec
listeners specified in it. The application developer only needs to open
and read the file, e.g. via ``pugi::xml_document::load()`` or
``pugi::xml_document::load_file()``, extract the top level element from
the XML document via ``pugi::xml_document::document_element()``, and
call ``AdapterConfiguration::constructInterfaces()`` on the element.

.. _hard_coding_interface_configuration:

Hard-Coding Interface Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*to be supplied*

.. _the_top_level_loop:

The Top-Level Loop
------------------

*to be supplied*

.. _multi_threaded_versus_single_threaded_operation:

Multi-Threaded versus Single-Threaded Operation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The |PLEXIL| Application Framework can run as a multi-threaded
application, or as a single thread.

*more to be supplied*


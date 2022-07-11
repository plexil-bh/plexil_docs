.. _Plexilisp:

Plexilisp
=============

Introduction
------------

Plexilisp is a Lisp-like syntax for Plexil that is designed for use
within the Emacs editor. It allows you to author Plexil plans and
simulation scripts in Emacs, and interactively compile them into their
XML representation for execution.

Here are the main features of Plexilisp:

Emacs-based

-  Works with Emacs (every version we've tried so far, including Gnu, X,
   and Aquamacs).

Lisp-like

-  Easy to edit in Emacs (``plexilisp`` mode provided).
-  Only PLEXIL constructs are supported; arbitrary Lisp is not allowed.

Plexil language support

-  Implements the :ref:`Extended PLEXIL <PlexilReferences>` syntax, and provides
   additional convenience forms and shortcuts.
-  Provides a syntax for :ref:`simulation scripts <test_executive_simulation_script>` needed
   for testing PLEXIL plans with the :ref:`Test Executive <test_executive>`.
-  Generates formatted XML in new Emacs buffers.
-  Instant validation of XML in Emacs is possible with :ref:`supporting tools <XMLSchemaEmacs>`.

.. _using_plexilisp:

Using Plexilisp
---------------

To get started, see the :ref:`tutorial <PlexilispTutorial>`.

For details, see the :ref:`reference manual <PlexilispRefernceManual>`.

To enable automatic validation of the generated PLEXIL XML, you'll need
the freely available :ref:`nXML utilities <XMLSchemaEmacs>`.

.. toctree::
   :maxdepth: 1
   :hidden:
   
   Plexilisp/PlexilispTutorial
   Plexilisp/PlexilispReferenceManual
   


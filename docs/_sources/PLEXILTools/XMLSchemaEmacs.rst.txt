.. _XMLSchemaEmacs:

XML/Schema, Emacs
====================

*10/14/10*

.. contents::

Introduction
------------

Core |PLEXIL|, the executable form of |PLEXIL|, is specified in the
*Extensible Markup Language*, or XML. This XML representation of |PLEXIL|
is hidden from the user in general use. This chapter provides
information about |PLEXIL|'s XML definition, and tools for inspecting and
editing Plexil XML files, if the need should arise.

Plexil may be coded in any text editor. The *Emacs* editor in particular
has features suitable for |PLEXIL|, namely its powerful XML editing and
validation capabilities, as well as being the platform for the
alternative |PLEXIL| syntax :ref:`Plexilisp <Plexilisp>`.

This chapter describes helpful XML and Emacs resources for |PLEXIL|.

.. _plexil_xml_schema:

PLEXIL XML Schema
-----------------

An XML Schema is a file that defines the format for a particular kind of
XML file. |PLEXIL| XML is defined by schema files found in the |PLEXIL|
distribution, under ``plexil/schema``. See the README file in that
directory for detailed information.

.. _schema_validation:

Schema Validation
-----------------

An XML file may be *validated* against an XML schema. There are many
tools for this. Both Linux and Mac OS provide ``xmllint``, a Unix
command line schema validator. Several commercial graphical tools, such
as *Oxygen* are also available. Schema validation can also be done
interactively using Emacs; this is described below.

Emacs
-----

For decades Emacs has been a popular text editor for programming. It is
a featureful and highly customizable editor and integrated work
environment, available for every common computer platform.

This document does not provide detailed instructions for Emacs, and
largely assumes the reader to be already fairly comfortable with Emacs.
If you are just starting out with Emacs, or interested, please consult
the abundance of introductory material available on the Web. Emacs
usually comes standard on Unix-based systems such as Linux and Mac OS,
however, there are variants of Emacs available that are more featureful
than these built-in versions.

Emacs is helpful for viewing, validating, and, if desired, editing Core
|PLEXIL| XML. Most versions of Emacs have some built-in support for
editing XML files. When you open or create a file with the extension
``.xml``, the resulting *buffer* will usually be in an XML-specific
editing mode.

.. _emacs_setup:

Emacs Setup
~~~~~~~~~~~

When Emacs starts, it loads the file ``~/.emacs``, which is where user
customizations are generally placed. The following code will make Emacs
more Plexil-friendly. You'll need to customize the pathnames for your
environment.

::

     

   ;;; Set the required PLEXIL_HOME environment variable with Emacs. (Although it ;;; may already be set in your Unix environment, Emacs does not always read this ;;; setting.
   (setenv "PLEXIL_HOME" "/users/fred/plexil")

   ;;; Edit .plx files as XML.
     (setq auto-mode-alist
       (cons '("\\(plx\\)\\'" . xml-mode) auto-mode-alist))

NOTE: if you install **nXML** (described below), you'll want to
substitute ``nxml-mode`` for ``xml-mode``.

.. _schema_validation_using_nxml_mode:

Schema Validation Using nXML Mode
---------------------------------

It is possible to validate XML files against a schema within Emacs,
using a freely available Emacs utility for XML called :ref:`nxml mode <http://www.thaiopensource.com/nxml-mode/>`. It provides not only
an easy means to edit XML (e.g. automatic indentation), but also a means
for validating your XML against an XML schema interactively.

.. _getting_nxml:

Getting nXML
~~~~~~~~~~~~

First, check if your Emacs already has nXML. Open or create an XML file
and see if the *mode line* (text bar at bottom of the buffer) contains
"(nXML ...)". If not, see if the following command is available:

::

    M-x nxml-mode

If not, you'll need to download nXML. Information can be found at the
following web sites.

::

    http://www.thaiopensource.com/nxml-mode/
    http://www.emacswiki.org/cgi-bin/wiki/NxmlMode

.. _installing_nxml:

Installing nXML
~~~~~~~~~~~~~~~

To automate the loading of nXML and ease its use, the following code can
be added to your .emacs file. Edit the file path as needed for your
system.

::

   ;;; Load NXML mode
    (load "/Applications/nxml-mode-20041004/rng-auto.el")

    ;;; Make standard XML file extensions engage nxml-mode
    (setq auto-mode-alist
    (cons '("\\(xml\\|xsd\\|xsl\\|rng\\|xhtml\\)\\'" . nxml-mode)
    auto-mode-alist))

    ;;; Make nxml-mode override Emacs's native xml-mode
    (push '("\\`<\\?xml" . nxml-mode) magic-mode-alist)

.. _using_nxml:

Using nXML
~~~~~~~~~~

Once installed, any XML file you edit in Emacs will be done so in nXML
mode.

To validate against a schema using nXML, you'll need to have a schema in
*Relaxed Compact NG* (RNC) format. The |PLEXIL| schemas are provided in
this format. The "XML" menu in Emacs provides various methods of
validation, one of which requires you to specify a schema file. However,
to have any XML file in a given directory be automatically validated by
the same schema file, create a file called ``schemas.xml`` in this
directory, with the following contents (change the schema file pathname
as needed).

::

   <?xml version="1.0"?>
   <locatingRules xmlns="http://thaiopensource.com/ns/locating-rules/1.0">

    <default uri="../../schema/extended-plexil.rnc"/>

   </locatingRules>

There is more information about nXML in ``plexil/schema/Makefile``.


.. _Communication:

Communication
===============

*15 May 2015*

The |PLEXIL| software system includes two facilities that allow a |PLEXIL|
executive to communicate with other |PLEXIL| executives or arbitrary
external entities.

#. A :ref:`mechanism for inter-executive communication <Inter-ExecutiveCommunication>` based on the `CMU IPC <http://www.cs.cmu.edu/~IPC/>`_ cross-platform message-passing
   middleware.
#. A :ref:`UDP-based adapter <UDPAdapter>` useful for inter-executive
   communication as well as general communication.

Using |PLEXIL|'s :ref:`interfacing framework <InterfacingOverview>`, you can
also develop your own custom communication interfaces.


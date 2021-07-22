.. _Tf2Main:

tf2 Tutorials
=============

Many of the tf2 tutorials are available for both C++ and Python.
The tutorials are streamlined to complete either the C++ track or the Python track.
If you want to learn both C++ and Python, you should go through the tutorials
once for C++ and once for Python.

.. contents:: Contents
   :depth: 2
   :local:

.. toctree::
   :hidden:

   Introduction-To-Tf2
   Writing-A-Tf2-Static-Broadcaster-Py
   Writing-A-Tf2-Broadcaster-Py
   Writing-A-Tf2-Listener-Py
   Adding-A-Frame-Py

Workspace Setup
---------------

If you have not yet created a workspace in which to complete the tutorials,
:ref:`follow this tutorial <ROS2Workspace>`.

Learning tf2
------------

#. :ref:`Introduction to tf2 <IntroToTf2>`.

   This tutorial will give you a good idea of what tf2 can do for you. It
   shows off some of the tf2 power in a multi-robot example using turtlesim.
   This also introduces using ``tf2_echo``, ``view_frames``, and ``rviz``.

#. :ref:`Writing a tf2 static broadcaster (Python) <WritingATf2StaticBroadcasterPy>`.

   This tutorial teaches you how to broadcast static coordinate frames to tf2.

#. :ref:`Writing a tf2 broadcaster (Python) <WritingATf2BroadcasterPy>`.

   This tutorial teaches you how to broadcast the state of a robot to tf2.

#. :ref:`Writing a tf2 listener (Python) <WritingATf2ListenerPy>`.

   This tutorial teaches you how to use tf2 to get access to frame transformations.

#. :ref:`Adding a frame (Python) <AddingAFramePy>`.

   This tutorial teaches you how to add an extra fixed frame to tf2.

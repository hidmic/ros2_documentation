.. _Tutorials:

Tutorials
=========

The tutorials are a collection of step-by-step instructions meant to steadily build skills in ROS 2.

The best way to approach the tutorials is to walk through them for the first time in order, as they build off of each other and are not meant to be comprehensive documentation.

For quick solutions to more specific questions, see the :ref:`Guides`.

Beginner
--------

Beginner: CLI Tools
^^^^^^^^^^^^^^^^^^^

.. toctree::
   :maxdepth: 1

   Tutorials/Configuring-ROS2-Environment
   Tutorials/Turtlesim/Introducing-Turtlesim
   Tutorials/Understanding-ROS2-Nodes
   Tutorials/Topics/Understanding-ROS2-Topics
   Tutorials/Services/Understanding-ROS2-Services
   Tutorials/Parameters/Understanding-ROS2-Parameters
   Tutorials/Understanding-ROS2-Actions
   Tutorials/Rqt-Console/Using-Rqt-Console
   Tutorials/Launch-Files/Creating-Launch-Files
   Tutorials/Ros2bag/Recording-And-Playing-Back-Data

Beginner: Client Libraries
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. toctree::
   :maxdepth: 1

   Tutorials/Workspace/Creating-A-Workspace
   Tutorials/Creating-Your-First-ROS2-Package
   Tutorials/Writing-A-Simple-Cpp-Publisher-And-Subscriber
   Tutorials/Writing-A-Simple-Py-Publisher-And-Subscriber
   Tutorials/Writing-A-Simple-Cpp-Service-And-Client
   Tutorials/Writing-A-Simple-Py-Service-And-Client
   Tutorials/Custom-ROS2-Interfaces
   Tutorials/Single-Package-Define-And-Use-Interface
   Tutorials/Using-Parameters-In-A-Class-CPP
   Tutorials/Using-Parameters-In-A-Class-Python
   Tutorials/Getting-Started-With-Ros2doctor

Intermediate
------------

.. toctree::
   :maxdepth: 1

   Tutorials/Actions/Creating-an-Action
   Tutorials/Actions/Writing-a-Cpp-Action-Server-Client
   Tutorials/Actions/Writing-a-Py-Action-Server-Client
   Tutorials/Launch-system
   Tutorials/Composition
   Tutorials/Colcon-Tutorial

Advanced
--------

.. toctree::
   :maxdepth: 1

   Tutorials/Topics/Topic-Statistics-Tutorial
   Tutorials/Discovery-Server/Discovery-Server
   Tutorials/Allocator-Template-Tutorial

Miscellaneous
-------------

.. toctree::
   :maxdepth: 1

   Tutorials/Deploying-ROS-2-on-IBM-Cloud
   Tutorials/Eclipse-Oxygen-with-ROS-2-and-rviz2
   Tutorials/Building-ROS-2-on-Linux-with-Eclipse-Oxygen
   Tutorials/Building-Realtime-rt_preempt-kernel-for-ROS-2
   Tutorials/Ros2bag/Recording-A-Bag-From-Your-Own-Node

Demos
-----

.. toctree::
   :hidden:

   Tutorials/Quality-of-Service
   Tutorials/Managed-Nodes
   Tutorials/Intra-Process-Communication
   Tutorials/Rosbag-with-ROS1-Bridge
   Tutorials/tf2
   Tutorials/URDF/Using-URDF-with-Robot-State-Publisher
   Tutorials/Real-Time-Programming
   Tutorials/dummy-robot-demo
   Tutorials/Logging-and-logger-configuration

* `Use quality-of-service settings to handle lossy networks <Tutorials/Quality-of-Service>`.
* `Management of nodes with managed lifecycles <Tutorials/Managed-Nodes>`.
* `Efficient intra-process communication <Tutorials/Intra-Process-Communication>`.
* `Bridge communication between ROS 1 and ROS 2 <https://github.com/ros2/ros1_bridge/blob/foxy/README.md>`__.
* `Recording and playback of topic data with rosbag using the ROS 1 bridge <Tutorials/Rosbag-with-ROS1-Bridge>`.
* `Turtlebot 2 demo using ROS 2 <https://github.com/ros2/turtlebot2_demo>`__.
* `TurtleBot 3 demo using ROS 2 <http://emanual.robotis.com/docs/en/platform/turtlebot3/ros2_setup/>`__. [community-contributed]
* `Simulate the TurtleBot 3 on ROS [community-contributed] <https://ubuntu.com/blog/simulate-the-turtlebot3>`__.
* `Navigate TurtleBot 3 in simulation <https://github.com/cyberbotics/webots_ros2/wiki/Navigate-TurtleBot3>`__. [community-contributed]
* `SLAM with TurtleBot3 in simulation <https://github.com/cyberbotics/webots_ros2/wiki/SLAM-with-TurtleBot3>`__. [community-contributed]
* `MoveIt 2 arm motion planning demo <http://moveit2_tutorials.picknik.ai/>`__.
* `Using tf2 with ROS 2 <Tutorials/tf2>`.
* `Using URDF with robot_state_publisher <Tutorials/URDF/Using-URDF-with-Robot-State-Publisher>`.
* `Write real-time safe code that uses the ROS 2 APIs <Tutorials/Real-Time-Programming>`.
* `Use the robot state publisher to publish joint states and TF <Tutorials/dummy-robot-demo>`.
* `Use DDS-Security <https://github.com/ros2/sros2/blob/foxy/README.md>`__.
* `Logging and logger configuration <Tutorials/Logging-and-logger-configuration>`.

Examples
--------

* `Python and C++ minimal examples <https://github.com/ros2/examples/tree/foxy>`__.

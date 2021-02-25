Installing ROS 2 via Debian Packages
====================================

.. contents:: Table of Contents
   :depth: 2
   :local:

Debian packages for ROS 2 Foxy Fitzroy are available for Ubuntu Focal.

Resources
---------

* Status Page:

  * ROS 2 Foxy (Ubuntu Focal): `amd64 <http://repo.ros2.org/status_page/ros_foxy_default.html>`__\ , `arm64 <http://repo.ros2.org/status_page/ros_foxy_ubv8.html>`__
* `Jenkins Instance <http://build.ros2.org/>`__
* `Repositories <http://repo.ros2.org>`__

Set locale
----------

.. include:: ../_Linux-Set-Locale.rst

Setup Sources
-------------

.. include:: ../_Apt-Repositories.rst

.. _Foxy_linux-install-debians-install-ros-2-packages:

Install ROS 2 packages
----------------------

Update your apt repository caches after setting up the repositories.

.. code-block:: bash

   sudo apt update

Desktop Install (Recommended): ROS, RViz, demos, tutorials.

.. code-block:: bash

   sudo apt install ros-foxy-desktop

ROS-Base Install (Bare Bones): Communication libraries, message packages, command line tools.
No GUI tools.

.. code-block:: bash

   sudo apt install ros-foxy-ros-base
   
Install extensions for colcon
-----------------------------

.. code-block:: bash

   sudo apt install python3-colcon-common-extensions

Environment setup
-----------------

Sourcing the setup script
^^^^^^^^^^^^^^^^^^^^^^^^^

Set up your environment by sourcing the following file.

.. code-block:: bash

   source /opt/ros/foxy/setup.bash

Install argcomplete (optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ROS 2 command line tools use argcomplete to autocompletion.
So if you want autocompletion, installing argcomplete is necessary.

.. code-block:: bash

   sudo apt install -y python3-pip
   pip3 install -U argcomplete

Try some examples
-----------------

If you installed ``ros-foxy-desktop`` above you can try some examples.

In one terminal, source the setup file and then run a C++ ``talker``\ :

.. code-block:: bash

   source /opt/ros/foxy/setup.bash
   ros2 run demo_nodes_cpp talker

In another terminal source the setup file and then run a Python ``listener``\ :

.. code-block:: bash

   source /opt/ros/foxy/setup.bash
   ros2 run demo_nodes_py listener

You should see the ``talker`` saying that it's ``Publishing`` messages and the ``listener`` saying ``I heard`` those messages.
This verifies both the C++ and Python APIs are working properly.
Hooray!

Next steps after installing
---------------------------
Continue with the `tutorials and demos </Tutorials>` to configure your environment, create your own workspace and packages, and learn ROS 2 core concepts.

Using the ROS 1 bridge
----------------------
The ROS 1 bridge can connect topics from ROS 1 to ROS 2 and vice-versa. See the dedicated `documentation <https://github.com/ros2/ros1_bridge/blob/master/README.md>`__ on how to build and use the ROS 1 bridge.

Additional RMW implementations (optional)
-----------------------------------------
The default middleware that ROS 2 uses is ``Fast-RTPS``, but the middleware (RMW) can be replaced at runtime.
See the `tutorial </Tutorials/Working-with-multiple-RMW-implementations>` on how to work with multiple RMWs.

Troubleshooting
---------------

Troubleshooting techniques can be found `here </Troubleshooting>`.

Uninstall
---------

If you need to uninstall ROS 2 or switch to a source-based install once you
have already installed from binaries, run the following command:

.. code-block:: bash

  sudo apt remove ros-foxy-* && sudo apt autoremove

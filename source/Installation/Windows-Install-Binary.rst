Installing ROS 2 on Windows
===========================

.. contents:: Table of Contents
   :depth: 2
   :local:

This page explains how to install ROS 2 on Windows from a pre-built binary package.

.. note::

    The pre-built binary does not include all ROS 2 packages.
    All packages in the `ROS base variant <https://ros.org/reps/rep-2001.html#ros-base>`_ are included, and only a subset of packages in the `ROS desktop variant <https://ros.org/reps/rep-2001.html#desktop-variants>`_ are included.
    The exact list of packages are described by the repositories listed in `this ros2.repos file <https://github.com/ros2/ros2/blob/master/ros2.repos>`_.

System requirements
-------------------

Only Windows 10 is supported.

.. _windows-install-binary-installing-prerequisites:

.. include:: _Windows-Install-Prerequisites.rst

Downloading ROS 2
-----------------

Binary releases of Rolling Ridley are not provided.
Instead you may download nightly `prerelease binaries <Prerelease_binaries>`.

* Download the latest package for Windows, e.g., ``ros2-package-windows-AMD64.zip``.

.. note::

    There may be more than one binary download option which might cause the file name to differ.

.. note::

    To download the ROS 2 debug libraries you'll need to download ``ros2-package-windows-debug-AMD64.zip``

* Unpack the zip file somewhere (we'll assume ``C:\dev\ros2_rolling``\ ).

Environment setup
-----------------

Start a command shell and source the ROS 2 setup file to set up the workspace:

.. code-block:: bash

   call C:\dev\ros2_rolling\local_setup.bat

It is normal that the previous command, if nothing else went wrong, outputs "The system cannot find the path specified." exactly once.

Try some examples
-----------------

In a command shell, set up the ROS 2 environment as described above and then run a C++ ``talker``\ :

.. code-block:: bash

   ros2 run demo_nodes_cpp talker

Start another command shell and run a Python ``listener``\ :

.. code-block:: bash

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
The default middleware that ROS 2 uses is ``Cyclone DDS``, but the middleware (RMW) can be replaced at runtime.
See the `guide <../Guides/Working-with-multiple-RMW-implementations>` on how to work with multiple RMWs.

Troubleshooting
---------------

Troubleshooting techniques can be found :ref:`here <windows-troubleshooting>`.

Uninstall
---------

1. If you installed your workspace with colcon as instructed above, "uninstalling" could be just a matter of opening a new terminal and not sourcing the workspace's ``setup`` file.
   This way, your environment will behave as though there is no Rolling install on your system.

2. If you're also trying to free up space, you can delete the entire workspace directory with:

   .. code-block:: bash

    rmdir /s /q \ros2_rolling

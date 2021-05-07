.. redirect-from::

  Installation/Rolling/OSX-Install-Binary

Installing ROS 2 on macOS
=========================

.. contents:: Table of Contents
   :depth: 2
   :local:

This page explains how to install ROS 2 on macOS from a pre-built binary package.

.. note::

    The pre-built binary does not include all ROS 2 packages.
    All packages in the `ROS base variant <https://ros.org/reps/rep-2001.html#ros-base>`_ are included, and only a subset of packages in the `ROS desktop variant <https://ros.org/reps/rep-2001.html#desktop-variants>`_ are included.
    The exact list of packages are described by the repositories listed in `this ros2.repos file <https://github.com/ros2/ros2/blob/master/ros2.repos>`_.

System requirements
-------------------

We currently support macOS Mojave (10.14).
The Rolling Ridley distribution will change target platforms from time to time as new platforms become available.
Most people will want to use a stable ROS distribution.

.. _Rolling_osx-install-binary-installling-prerequisites:

Installing prerequisites
------------------------

You need the following things installed before installing ROS 2.


*
  **brew** *(needed to install more stuff; you probably already have this)*:


  * Follow installation instructions at http://brew.sh/
  *
    *Optional*: Check that ``brew`` is happy with your system configuration by running:

    .. code-block:: bash

        brew doctor


      Fix any problems that it identifies.

*
  Use ``brew`` to install more stuff:

  .. code-block:: bash

       brew install python@3.8
       # Unlink in case you have python@3.7 installed already
       brew unlink python
       # Make the python command be Python 3.8
       brew link --force python@3.8

       # install asio and tinyxml2 for Fast-RTPS
       brew install asio tinyxml2

       # install dependencies for robot state publisher
       brew install tinyxml eigen pcre poco

       # OpenCV isn't a dependency of ROS 2, but it is used by some demos.
       brew install opencv

       # install OpenSSL for DDS-Security
       brew install openssl
       # if you are using ZSH, then replace '.bashrc' with '.zshrc'
       echo "export OPENSSL_ROOT_DIR=$(brew --prefix openssl)" >> ~/.bashrc

       # install Qt for RViz
       brew install qt freetype assimp

       # install console_bridge for rosbag2
       brew install console_bridge

       # install dependencies for rcl_logging_log4cxx
       brew install log4cxx spdlog

       # install CUnit for Cyclone DDS
       brew install cunit

*
  Install ros2 doctor dependencies

  ``python3 -m pip install rosdistro``

*
  Install rqt dependencies

  ``brew install sip pyqt5``

  Fix some path names when looking for sip stuff during install (see `ROS 1 wiki <http://wiki.ros.org/kinetic/Installation/OSX/Homebrew/Source#Qt_naming_issue>`__):

  ``ln -s /usr/local/share/sip/Qt5 /usr/local/share/sip/PyQt5``

  ``brew install graphviz``

  ``python3 -m pip install pygraphviz pydot matplotlib psutil``

  .. note::

      You may run into an issue installing ``pygraphviz``, "error: Error locating graphviz".
      Try the following install command instead:

      .. code-block:: bash

         python3 -m pip install --install-option="--include-path=/usr/local/include/" --install-option="--library-path=/usr/local/lib/" pygraphviz

*
  Install SROS2 dependencies

  ``python3 -m pip install lxml``

*
  Install additional runtime dependencies for command-line tools:

  .. code-block:: bash

       python3 -m pip install argcomplete catkin_pkg empy ifcfg importlib-metadata lark-parser lxml netifaces numpy pyparsing pyyaml setuptools

Downloading ROS 2
-----------------

Binary releases of Rolling Ridley are not provided.
Instead you may download nightly `prerelease binaries <Prerelease_binaries>`.

* Download the latest package for macOS; let's assume that it ends up at ``~/Downloads/ros2-package-osx-x86_64.tar.bz2``.

  * Note: there may be more than one binary download option which might cause the file name to differ.

*
  Unpack it:

  .. code-block:: bash

       mkdir -p ~/ros2_rolling
       cd ~/ros2_rolling
       tar xf ~/Downloads/ros2-package-osx-x86_64.tar.bz2

System Integrity Protection (SIP)
---------------------------------

macOS/OS X versions >=10.11 have System Integrity Protection enabled by default, which will prevent all ros2 libraries from executing.

Run the following command on the folder where you unpacked ros2 to remove quarantine flag:

.. code-block:: bash

   sudo xattr -r -d com.apple.quarantine ~/ros2_rolling


Install additional DDS implementations (optional)
-------------------------------------------------

If you would like to use another DDS or RTPS vendor besides the default, Eclipse Cyclone DDS, you can find instructions `here <DDS-Implementations>`.

Environment setup
-----------------

Source the ROS 2 setup file:

.. code-block:: bash

   . ~/ros2_rolling/ros2-osx/setup.bash

Try some examples
-----------------

In one terminal, set up the ROS 2 environment as described above and then run a C++ ``talker``:

.. code-block:: bash

   ros2 run demo_nodes_cpp talker

In another terminal, set up the ROS 2 environment and then run a Python ``listener``:

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

Troubleshooting techniques can be found :ref:`here <macOS-troubleshooting>`.

Uninstall
---------

1. If you installed your workspace with colcon as instructed above, "uninstalling" could be just a matter of opening a new terminal and not sourcing the workspace's ``setup`` file.
   This way, your environment will behave as though there is no Rolling install on your system.

2. If you're also trying to free up space, you can delete the entire workspace directory with:

   .. code-block:: bash

    rm -rf ~/ros2_rolling

.. _sros2:
.. _ROS-2-Security-Tutorials:

Introducing ROS 2 Security
==========================

**Goal:** Set up security with sros2

**Tutorial level:** Intermediate

**Time:** 15 minutes

.. contents:: Contents
   :depth: 2
   :local:


Background
----------

The ``sros2`` package provides the tools and instructions to use ROS2 on top of DDS-Security.
The security features have been tested across platforms (Linux, macOS, and Windows) as well as across different languages (C++ and Python).
The SROS2 has been designed to work with any secure middleware, although not all middleware is open source and support varies depending on the ROS distribution in use.
Please reach out to the `ROS 2 Security Working Group <https://docs.ros.org/en/foxy/Governance.html#security>`_ if you encounter any support issues.


Installation
------------

Typically security is available following installation using the :ref:`ROS 2 Installation Guide <InstallationGuide>` and the :ref:`configuration guide <ConfigROS2>`.
However, if you intend to install from source or switch middleware implementations, consider the following caveats:


Installing from source
^^^^^^^^^^^^^^^^^^^^^^

Before installing from source, you will need to have a recent version openssl (1.0.2g or later) installed:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: bash

      sudo apt update
      sudo apt install libssl-dev

  .. group-tab:: MacOS

    .. code-block:: bash

      brew install openssl

    You will need to have OpenSSL on your library path to run DDS-Security demos.
    Run the following command, and consider adding to your ``~/.bash_profile``:

    .. code-block:: bash

      export DYLD_LIBRARY_PATH=`brew --prefix openssl`/lib:$DYLD_LIBRARY_PATH
      export OPENSSL_ROOT_DIR=`brew --prefix openssl`


  .. group-tab:: Windows

    If you don't have OpenSSL installed, please follow :ref:`these instructions <windows-install-binary-installing-prerequisites>`

Fast-DDS requires an additional CMake flag to build the security plugins, so the colcon invocation needs to be modified to pass:

.. code-block:: bash

  colcon build --symlink-install --cmake-args -DSECURITY=ON


Selecting an alternate middleware
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you choose not to use the default middleware implementation, be sure to `change your DDS implementation <../../Installation/DDS-Implementations>` before proceeding.

ROS2 allows you to change the DDS implementation at runtime.
See `how to work with mulitple RMW implementations <../../Guides/Working-with-multiple-RMW-implementations>` to explore different middleware implementations.

Note that secure communication between vendors is not supported.



Run the demo
------------

1\. Create a folder for the security files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  Begin by creating folder to store all the files necessary for this demo:

  .. tabs::

    .. group-tab:: Linux

      .. code-block:: bash

        mkdir ~/sros2_demo

    .. group-tab:: MacOS

      .. code-block:: bash

        mkdir ~/sros2_demo

    .. group-tab:: Windows

      .. code-block:: bat

        md C:\dev\ros2\sros2_demo

2\. Generate a keystore
^^^^^^^^^^^^^^^^^^^^^^^

Use the ``sros2`` utilities to create the keystore.
Files in the keystore will be used to enable security for all the participants in the ROS 2 graph.

.. tabs::

  .. group-tab:: Linux

    .. code-block:: bash

      cd ~/sros2_demo
      ros2 security create_keystore demo_keystore

  .. group-tab:: MacOS

    .. code-block:: bash

      cd ~/sros2_demo
      ros2 security create_keystore demo_keystore

  .. group-tab:: Windows

    .. code-block:: bat

      cd sros2_demo
      ros2 security create_keystore demo_keystore

3\. Generate keys and certificates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once the keystore is created, create keys and certificates for each node with security enabled.
For our demo, that includes the talker and listener nodes.
This command uses the ``create_enclave`` feature which is covered in more detail in the next tutorial.

.. tabs::

  .. group-tab:: Linux

    .. code-block:: bash

      ros2 security create_enclave demo_keystore /talker_listener/talker
      ros2 security create_enclave demo_keystore /talker_listener/listener

  .. group-tab:: MacOS

    .. code-block:: bash

      ros2 security create_enclave demo_keystore /talker_listener/talker
      ros2 security create_enclave demo_keystore /talker_listener/listener

  .. group-tab:: Windows

    .. code-block:: bat

      ros2 security create_enclave demo_keystore /talker_listener/talker
      ros2 security create_enclave demo_keystore /talker_listener/listener


    If ``unable to write 'random state'`` appears then set the environment variable ``RANDFILE``.

    .. code-block:: bat

      set RANDFILE=C:\dev\ros2\sros2_demo\.rnd

    Then re-run the commands above.


4\. Configure environment variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Three environment variables allow the middleware to locate encryption materials and enable (and possibly enforce) security.
These and other security-related environment variables are described in the `ROS 2 DDS-Security Integration design document <https://design.ros2.org/articles/ros2_dds_security.html>`_.

.. tabs::

  .. group-tab:: Linux

    .. code-block:: bash

      export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keystore
      export ROS_SECURITY_ENABLE=true
      export ROS_SECURITY_STRATEGY=Enforce

  .. group-tab:: MacOS

    .. code-block:: bash

      export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keystore
      export ROS_SECURITY_ENABLE=true
      export ROS_SECURITY_STRATEGY=Enforce

  .. group-tab:: Windows

    .. code-block:: bat

      set ROS_SECURITY_KEYSTORE=%cd%/demo_keystore
      set ROS_SECURITY_ENABLE=true
      set ROS_SECURITY_STRATEGY=Enforce

These variables need to be defined in each terminal used for the demo.
For convenience you can add them to your boot environment.


5\. Run the ``talker/listener`` demo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Begin the demo by launching the talker node.

.. code-block:: bash

  ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker

In another terminal, do the same to launch the ``listener`` node.
The environment variables in this terminal must be properly set as described in step 4 above.

.. code-block:: bash

  ros2 run demo_nodes_py listener --ros-args --enclave /talker_listener/listener

These nodes will be communicating using authentication and encryption!
If you look at the packet contents (for example, using ``tcpdump`` or ``Wireshark`` as covered in another tutorial), you can see that the messages are encrypted.

Note: You can switch between the C++ (demo_nodes_cpp) and Python (demo_nodes_py) packages arbitrarily.

These nodes are able to communicate because we have created the appropriate keys and certificates for them.

Leave both nodes running as you answer the questions below.


Take the Quiz!
--------------

.. tabs::

  .. group-tab:: Question 1

    Open another terminal session, but **do not** set the environment variables so that security is not enabled.
    Start the listener.
    What do you expect to happen?

  .. group-tab:: Answer 1

    The listener launches but does not receive any messages.
    All traffic is encrypted, and without security enabled the listener does not receive anything.


.. tabs::

  .. group-tab:: Question 2

    Stop the listener, set the environment variable ``ROS_SECURITY_ENABLE`` to ``true`` and start the listener again.
    What results do you expect this time?

  .. group-tab:: Answer 2

    The listener still launches but does not receive messages.
    Although security has now been enabled, it is not been configured properly since ROS is unable to locate the key files.
    The listener launches, but in non-secure mode since security is not enforced, which means that although the properly configured talker is sending encrypted messages, this listener is unable to decrypt them.

.. tabs::

  .. group-tab:: Question 3

    Stop the listener and set ``ROS_SECURITY_STRATEGY`` to ``Enforce``.
    What happens now?

  .. group-tab:: Answer 3

    The listener fails to launch.
    Security has been enabled and is being enforced.
    Since it still is not properly configured, an error is thrown rather than launching in non-secure mode.


Learn More!
-----------

Are you ready to go further with ROS Security?
Take a look at the `Secure Turtlebot2 Demo <https://github.com/ros-swg/turtlebot3_demo>`_.
You'll find a functioning and complex implementation of ROS 2 security, ready to try out your own custom scenarios.
Be sure to create pull requests and issues here so we can continue improving security support in ROS!

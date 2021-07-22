.. _Security-on-Two:

Security Across Two Machines
============================

**Goal:** Make two different machines communicate securely

**Tutorial level:** Intermediate

**Time:** 7 minutes

.. contents:: Contents
  :depth: 2
  :local:


Background
----------

The previous tutorials have used two ROS nodes on the same machine sending all network communications over the localhost interface.
Let's extend that scenario to involve multiple machines, since the benefits of authentication and encryption then become more obvious.

Suppose that the machine with the keystore created in the previous demo has a hostname ``Alice``, and that we want to also use another machine with hostname ``Bob`` for our multi-machine ``talker/listener`` demo.
We need to move some keys from ``Alice`` to ``Bob`` to allow SROS 2 to authenticate and encrypt the transmissions.


Create the second keystore
--------------------------

Begin by creating an empty keystore on ``Bob``; the keystore is actually just an empty directory:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: bash

      ssh Bob
      mkdir ~/sros2_demo
      exit

  .. group-tab:: MacOS

    .. code-block:: bash

      ssh Bob
      mkdir ~/sros2_demo
      exit

  .. group-tab:: Windows

    .. code-block:: bat

      ssh Bob
      md C:\dev\ros2\sros2_demo
      exit


Copy files
----------

Next copy the keys and certificates for the ``talker`` program from ``Alice`` to ``Bob``.
Since the keys are just text files, we can use ``scp`` to copy them.

.. tabs::

  .. group-tab:: Linux

    .. code-block:: bash

      cd ~/sros2_demo/demo_keystore
      scp -r talker USERNAME@Bob:~/sros2_demo/demo_keystore

  .. group-tab:: MacOS

    .. code-block:: bash

      cd ~/sros2_demo/demo_keystore
      scp -r talker USERNAME@Bob:~/sros2_demo/demo_keystore

  .. group-tab:: Windows

    .. code-block:: bat

      cd C:\dev\ros2\sros2_demo\demo_keystore
      scp -r talker USERNAME@Bob:/dev/ros2/sros2_demo/demo_keystore

That will be very quick, since it's just copying some very small text files.
Now, we're ready to run a multi-machine talker/listener demo!


Launch the nodes
----------------

Once the environment is set up, run the talker on ``Bob``:

.. code-block:: bash

  ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker

and launch the listener on ``Alice``:

.. code-block:: bash

  ros2 run demo_nodes_py listener --ros-args --enclave /talker_listener/listener

Alice will now be receiving encrypted messages from Bob.

With two machines successfully communicating using both encryption and authentication, you can use the same procedure to add more machines to your ROS graph.

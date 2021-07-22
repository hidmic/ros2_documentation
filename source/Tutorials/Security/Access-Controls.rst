.. _Access-Controls:

Security Access Controls
========================

**Goal:** Limit the topics a node can use

**Tutorial level:** Advanced

**Time:** 20 minutes

.. contents:: Contents
   :depth: 2
   :local:


Background
----------

Permissions are quite flexible and can be used to control many behaviors within the ROS graph.

For this tutorial, we demonstrate a policy which only allows publishing messages on the default ``chatter`` topic.
This would prevent, for instance, remapping the topic when launching the listener or using the same security enclaves for another purpose.

In order to enforce this policy, we need to update the ``permissions.xml`` file and re-sign it before launching the node.
This can be done by modifying the permissions file by hand, or by using XML templates.


Modify ``permissions.xml``
^^^^^^^^^^^^^^^^^^^^^^^^^^

Begin by making a backup of your permissions files, and open ``permissions.xml`` for editing:

.. code-block:: bash

  cd ~/sros2_demo/demo_keys/enclaves/talker_listener/talker
  mv permissions.p7s permissions.p7s~
  mv permissions.xml permissions.xml~
  vi permissions.xml

We will be modifying the ``<allow_rule>`` for ``<publish>`` and ``<subscribe>``.
The topics in this XML file use the DDS naming format, not the ROS name.
Find details on mapping topic names between ROS and DDS in the `Topic and Service Names design document <https://design.ros2.org/articles/topic_and_service_names.html#mapping-of-ros-2-topic-and-service-names-to-dds-concepts>`_.

Paste the following XML content into ``permission.xml``, save the file and exit the text editor.
This shows the ``chatter`` and ``rosout`` ROS topics renamed to the DDS ``rt/chatter`` and ``rt/rosout`` topics, respectively:

.. code-block:: xml
  :emphasize-lines: 15,16,17,18,23,24

  <dds xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.omg.org/spec/DDS-SECURITY/20170901/omg_shared_ca_permissions.xsd">
    <permissions>
      <grant name="/talker_listener/talker">
        <subject_name>CN=/talker_listener/talker</subject_name>
        <validity>
          <not_before>2021-06-01T16:57:53</not_before>
          <not_after>2031-05-31T16:57:53</not_after>
        </validity>
        <allow_rule>
          <domains>
            <id>0</id>
          </domains>
          <publish>
            <topics>
              <topic>rt/chatter</topic>
              <topic>rt/rosout</topic>
              <topic>rt/parameter_events</topic>
              <topic>*/talker/*</topic>
            </topics>
          </publish>
          <subscribe>
            <topics>
              <topic>rt/parameter_events</topic>
              <topic>*/talker/*</topic>
            </topics>
          </subscribe>
        </allow_rule>
        <allow_rule>
          <domains>
            <id>0</id>
          </domains>
          <publish>
            <topics>
              <topic>ros_discovery_info</topic>
            </topics>
          </publish>
          <subscribe>
            <topics>
              <topic>ros_discovery_info</topic>
            </topics>
          </subscribe>
        </allow_rule>
        <default>DENY</default>
      </grant>
    </permissions>
  </dds>

This policy allows the talker to publish on the ``chatter`` and the ``rosout`` topics.
It also allows includes publish and subscribe permissions needed for the talker node to manage parameters (a requirement for all nodes).
Discovery permissions remain unchanged from the original template.


Sign the policy file
^^^^^^^^^^^^^^^^^^^^

This next command creates the new S/MIME signed policy file ``permissions.p7s`` from the updated XML file ``permissions.xml``.
The file must be signed with the Permissions CA certificate, **which requires access to the Permission CA private key**.
If the private key has been protected, additional steps may be required to unlock and use it accoring to your security plan.

.. code-block:: bash

  openssl smime -sign -text -in permissions.xml -out permissions.p7s \
    --signer permissions_ca.cert.pem \
    -inkey ~/sros2_demo/demo_keys/private/permissions_ca.key.pem


Launch the node
^^^^^^^^^^^^^^^

With the updated permissions in place, we can launch the node successfully using the same command used in prior tutorials:

.. code-block:: bash

  ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker

However, attempting to remap the ``chatter`` topic prevents the node from launching (note that this requires the ``ROS_SECURITY_STRATEGY`` set to ``Enforce``).

.. code-block:: bash

  ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker \
    --remap chatter:=not_chatter


Use the templates
^^^^^^^^^^^^^^^^^

Security policies can quickly become confusing, so the ``sros2`` utilities add the ability to create policies from templates.
Do this by using the `sample policy file <https://github.com/ros2/sros2/blob/master/sros2/test/policies/sample.policy.xml#L1>`_ provided in the ``sros2`` repository.
Let's creates a policy for both the ``talker`` and the ``listener`` to only use the ``chatter`` topic.

Begin by downloading the ``sros2`` repository with the sample policy files:

.. code-block:: bash

  git clone https://github.com/ros2/sros2.git /tmp/sros2

Then use the ``create_permission`` verb while pointing to the sample policy to generate the XML permission files:

.. code-block:: bash

  ros2 security create_permission demo_keystore \
    /talker_listener/talker \
    /tmp/sros2/sros2/test/policies/sample.policy.xml
  ros2 security create_permission demo_keystore \
    /talker_listener/listener \
    /tmp/sros2/sros2/test/policies/sample.policy.xml

These permission files allow nodes to only publish or subscribe to the ``chatter`` topic, and enable communications required for parameters.

In one terminal with security enabled as in previous security tutorials, run the ``talker`` demo program:

.. code-block:: bash

  ros2 run demo_nodes_cpp talker --ros-args -e /talker_listener/talker

In another terminal do the same with the ``listener`` program:

.. code-block:: bash

  ros2 run demo_nodes_py listener --ros-args -e /talker_listener/listener

At this point, your ``talker`` and ``listener`` nodes will be communicating securely using explicit access control lists.
However, the following attempt for the ``listener`` node to subscribe to a topic other than ``chatter`` will fail:

.. code-block:: bash

  ros2 run demo_nodes_py listener --ros-args --enclave /talker_listener/listener \
    --remap chatter:=not_chatter

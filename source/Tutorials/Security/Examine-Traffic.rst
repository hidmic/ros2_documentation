.. _Examine-Traffic:

Examine Network Traffic
=======================

**Goal:** Capture and examine raw ROS 2 network traffic

**Tutorial level:** Advanced

**Time:** 20 minutes

.. contents:: Contents
  :depth: 2
  :local:


Background
----------

ROS 2 communications security is all about protecting communications between nodes.
Prior tutorials enabled security, but how can you **really** tell if traffic is being encrypted?
In this tutorial we'll take a look at capturing live network traffic to show the difference between encrypted and unencrypted traffic.


Run the demo
------------

Install ``tcpdump``
^^^^^^^^^^^^^^^^^^^

Begin in a new terminal window by installing `tcpdump <https://www.tcpdump.org/manpages/tcpdump.1.html>`_, a command-line tool for capturing and displaying network traffic.
Although this tutorial describes ``tcpdump`` commands, you can also use `Wireshark <https://www.wireshark.org/>`_, a similar graphical tool for capturing and analyzing traffic.

.. code-block:: bash

  sudo apt update
  sudo apt install tcpdump

Run following commands on a single machine through multiple ``ssh`` sessions.

Start the talker and listener
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Start both the talker and the listener again, each in its own terminal.
The security environment variables are not set so security is not enabled for these sessions.

.. code-block:: bash

  # In terminal 1:
  ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker

  # In terminal 2:
  ros2 run demo_nodes_cpp listener --ros-args --enclave /talker_listener/listener


Display unencrypted discovery packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With the talker and listener running, open another terminal and start ``tcpdump`` to look at the network traffic.
You need to use ``sudo`` since reading raw network traffic is a privileged operation.

The command below uses the ``-X`` option to print packet contents, the ``-i`` option to listen for packets on any interface, and captures only `UDP <https://en.wikipedia.org/wiki/User_Datagram_Protocol>`_ port 7400 traffic.

.. code-block:: bash

  sudo tcpdump -X -i any udp port 7400

You should see packets like the following::

  20:18:04.400770 IP 8_xterm.46392 > 239.255.0.1.7400: UDP, length 252
    0x0000:  4500 0118 d48b 4000 0111 7399 c0a8 8007  E.....@...s.....
    0x0010:  efff 0001 b538 1ce8 0104 31c6 5254 5053  .....8....1.RTPS
    ...
    0x00c0:  5800 0400 3f0c 3f0c 6200 1c00 1800 0000  X...?.?.b.......
    0x00d0:  2f74 616c 6b65 725f 6c69 7374 656e 6572  /talker_listener
    0x00e0:  2f74 616c 6b65 7200 2c00 2800 2100 0000  /talker.,.(.!...
    0x00f0:  656e 636c 6176 653d 2f74 616c 6b65 725f  enclave=/talker_
    0x0100:  6c69 7374 656e 6572 2f74 616c 6b65 723b  listener/talker;
    0x0110:  0000 0000 0100 0000                      ........

This is a discovery datagram--the talker looking for subscribers.
As you can see, the node name (``/talker_listener/talker``) and the enclave (also ``/talker_listener/talker``) are passed in plain text.
You should also see similar discovery datagrams from the ``listener`` node.
Some other features of a typical discovery packet:

- The destination address is 239.255.0.1, which is a multicast IP address; ROS 2 uses multicast traffic for discovery by default.
- UDP 7400 is the destination port, as per the `DDS-RTPS specification <https://www.omg.org/spec/DDSI-RTPS/About-DDSI-RTPS/>`_.
- The packet contains the "RTPS" tag, also as defined to the DDS-RTPS specification.


Display unencrypted data packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use ``tcpdump`` to capture non-discovery RTPS packets by filtering on UDP ports above 7400:

.. code-block:: bash

  sudo tcpdump -i any -X udp portrange 7401-7500

You will see few different types of packets, but watch for something like the following which is obviously data being sent from a talker to a listener::

  20:49:17.927303 IP localhost.46392 > localhost.7415: UDP, length 84
    0x0000:  4500 0070 5b53 4000 4011 e127 7f00 0001  E..p[S@.@..'....
    0x0010:  7f00 0001 b538 1cf7 005c fe6f 5254 5053  .....8...\.oRTPS
    0x0020:  0203 010f 010f 4874 e752 0000 0100 0000  ......Ht.R......
    0x0030:  0901 0800 cdee b760 5bf3 5aed 1505 3000  .......`[.Z...0.
    0x0040:  0000 1000 0000 1204 0000 1203 0000 0000  ................
    0x0050:  5708 0000 0001 0000 1200 0000 4865 6c6c  W...........Hell
    0x0060:  6f20 576f 726c 643a 2032 3133 3500 0000  o.World:.2135...

Some features to note about this packet:

- The message contents, "Hello World: 2135", are sent in clear text
- The source and destination IP address is ``localhost``: since both nodes are running on the same machine, the nodes discovered each other on the ``localhost`` interface


Enable encryption
^^^^^^^^^^^^^^^^^

Stop both the talker and the listener nodes.
Enable encryption for both by setting the security environment variables and run them again.

.. code-block:: bash

  # In terminal 1:
  export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keys
  export ROS_SECURITY_ENABLE=true
  export ROS_SECURITY_STRATEGY=Enforce
  ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker

  # In terminal 2:
  export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keys
  export ROS_SECURITY_ENABLE=true
  export ROS_SECURITY_STRATEGY=Enforce
  ros2 run demo_nodes_cpp listener --ros-args --enclave /talker_listener/listener


Display encrypted discovery packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Run the same ``tcpdump`` command used earlier to examine the output of discovery traffic with encryption enabled:

.. code-block:: bash

  sudo tcpdump -X -i any udp port 7400

The typical discovery packet looks somewhat like the following::

  21:09:07.336617 IP 8_xterm.60409 > 239.255.0.1.7400: UDP, length 596
    0x0000:  4500 0270 c2f6 4000 0111 83d6 c0a8 8007  E..p..@.........
    0x0010:  efff 0001 ebf9 1ce8 025c 331e 5254 5053  .........\3.RTPS
    0x0020:  0203 010f bbdd 199c 7522 b6cb 699f 74ae  ........u"..i.t.
    ...
    0x00c0:  5800 0400 3f0c ff0f 6200 2000 1a00 0000  X...?...b.......
    0x00d0:  2f74 616c 6b65 725f 6c69 7374 656e 6572  /talker_listener
    0x00e0:  2f6c 6973 7465 6e65 7200 0000 2c00 2800  /listener...,.(.
    0x00f0:  2300 0000 656e 636c 6176 653d 2f74 616c  #...enclave=/tal
    0x0100:  6b65 725f 6c69 7374 656e 6572 2f6c 6973  ker_listener/lis
    0x0110:  7465 6e65 723b 0000 0110 c400 1400 0000  tener;..........
    0x0120:  4444 533a 4175 7468 3a50 4b49 2d44 483a  DDS:Auth:PKI-DH:
    0x0130:  312e 3000 0400 0000 0c00 0000 6464 732e  1.0.........dds.
    ...
    0x0230:  1100 0000 6464 732e 7065 726d 5f63 612e  ....dds.perm_ca.
    0x0240:  616c 676f 0000 0000 0d00 0000 4543 4453  algo........ECDS
    0x0250:  412d 5348 4132 3536 0000 0000 0000 0000  A-SHA256........
    0x0260:  0510 0800 0700 0080 0600 0080 0100 0000  ................

This packet is much larger and includes information which can be used to set up encryption among ROS nodes.
As we will see shortly, this actually includes some of the security configuration files that were created when we enabled security.
Interested in learning more? Take a look at the excellent paper `Network Reconnaissance and Vulnerability Excavation of Secure DDS Systems <https://arxiv.org/abs/1908.05310>`_ to understand why this matters.


Display encrypted data packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now use ``tcpdump`` to capture data packets:

.. code-block:: bash

  sudo tcpdump -i any -X udp portrange 7401-7500

A typical data packet looks like the following::

  21:18:14.531102 IP localhost.54869 > localhost.7415: UDP, length 328
    0x0000:  4500 0164 bb42 4000 4011 8044 7f00 0001  E..d.B@.@..D....
    0x0010:  7f00 0001 d655 1cf7 0150 ff63 5254 5053  .....U...P.cRTPS
    0x0020:  0203 010f daf7 10ce d977 449b bb33 f04a  .........wD..3.J
    0x0030:  3301 1400 0000 0003 492a 6066 8603 cdb5  3.......I*`f....
    0x0040:  9df6 5da6 8402 2136 0c01 1400 0000 0000  ..]...!6........
    0x0050:  0203 010f daf7 10ce d977 449b bb33 f04a  .........wD..3.J
    ...
    0x0130:  7905 d390 3201 1400 3ae5 0b60 3906 967e  y...2...:..`9..~
    0x0140:  5b17 fd42 de95 54b9 0000 0000 3401 1400  [..B..T.....4...
    0x0150:  42ae f04d 0559 84c5 7116 1c51 91ba 3799  B..M.Y..q..Q..7.
    0x0160:  0000 0000                                ....

The data in this RTPS packet is all encrpyted.

In addition to this data packet, you should see additional packets with node and enclave names; these support other ROS features such as parameters and services.
Encryption options for these packets can also be controlled by security policy.

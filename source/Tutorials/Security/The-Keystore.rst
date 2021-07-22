.. _The-Keystore:

Understanding the ROS 2 Security Keystore
=========================================

**Goal:** Explore files located in the ROS 2 Security Keystore

**Tutorial level:** Intermediate

**Time:** 15 minutes

.. contents:: Contents
   :depth: 2
   :local:


Background
----------

The ``sros2`` package can be used to create keys, certificates and policies necessary to enable ROS 2 security.
However, the security configuration is extrememly flexible.
A basic understanding of the ROS 2 Security Keystore will allow integration with an existing PKI (Public Key Infrastructure) and managment of sensitive key materials consistent with organizational policies.


Security Artifact Locations
---------------------------

With communications security enabled in the prior tutorial, let's take a look at the files which were created when security was enabled.
These are the files which make encryption possible.

The ``sros2`` utilities (``ros2 security ...``) separate files into public, private and enclave key materials.

ROS uses the directory defined by the environmental variable ``ROS_SECURITY_KEYSTORE`` as the keystore.
For this tutorial, we use the directory ``~/sros2_demo/demo_keystore``.


Public Key Materials
^^^^^^^^^^^^^^^^^^^^

You will find three encryption certificates in the public directory at ``~/sros2_demo/demo_keys/public``; however, the identity and permissions certificates are actually just a link to the Certificate Authority (CA) certificate.

In a public key infrastructure, the `Certificate Authority <https://en.wikipedia.org/wiki/Certificate_authority>`_ acts as a trust anchor: it validates the identities and permissions of participants.
For ROS, that means all the nodes that participate in the ROS graph (which may extend to an entire fleet of individual robots).
By placing the Certificate Authority's certificate (``ca.cert.pem``) in the proper location on the robot, all ROS nodes can establish mutual trust with other nodes using the same Certificate Authority.

Although in our tutorials we create a Certificate Authority on-the-fly, in a production system this should be done according to a pre-defined security plan.
Typically the Certificate Authority for a production system will be created off-line, and placed on the robot during initial setup.
It may be unique for each robot, or shared across a fleet of robots all intended to trust each other.

DDS (and ROS, by extension) supports separation of identity and permission trust chains, so each function has its own certificate authority.
In most cases a ROS system security plan does not require a separation between these duties, so the security utilities generate a single Certificate Authority which is used for both identity and permissions.

Use ``openssl`` to view this x509 certificate and display it as text:

.. code-block:: bash

  cd ~/sros2_demo/demo_keys/public
  openssl x509 -in ca.cert.pem -text -noout

The output should look similar to the following::

  Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            02:8e:9a:24:ea:10:55:cb:e6:ea:e8:7a:c0:5f:58:6d:37:42:78:aa
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN = sros2testCA
        Validity
            Not Before: Jun  1 16:57:37 2021 GMT
            Not After : May 31 16:57:37 2031 GMT
        Subject: CN = sros2testCA
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:71:e9:37:d7:32:ba:b8:a0:97:66:da:9f:e3:c4:
                    08:4f:7a:13:59:24:c6:cf:6a:f7:95:c5:cd:82:c0:
                    7f:7f:e3:90:dd:7b:0f:77:d1:ee:0e:af:68:7c:76:
                    a9:ca:60:d7:1e:2c:01:d7:bc:7e:e3:86:2a:9f:38:
                    dc:ed:39:c5:32
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:1
    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:d4:fc:d8:45:ff:a4:51:49:98:4c:f0:c4:3f:
         e0:e7:33:19:8e:31:3c:d0:43:e7:e9:8f:36:f0:90:18:ed:d7:
         7d:02:20:30:84:f7:04:33:87:bb:4f:d3:8b:95:61:48:df:83:
         4b:e5:92:b3:e6:ee:3c:d5:cf:30:43:09:04:71:bd:dd:7c

Some things to note about this CA certificate:
 - The certificate subject name ``sros2testCA`` is the default provided by the ``sros2`` utilities.
 - This certificate is valid for ten years from time of creation
 - Like all certificates, this contains a public key used for public-private key encryption
 - As a Root Certificate Authority, this is a `self-signed certificate <https://en.wikipedia.org/wiki/Self-signed_certificate>`_; i.e., it is signed using its own private key.

Since this is a public certificate, it can be freely copied as needed to establish trust throughout your ROS system.


Private Key Materials
^^^^^^^^^^^^^^^^^^^^^

Private key materials can be found in the keystore directory ``~/sros2_demo/demo_keys/private``.
Similar to the ``public`` directory, this contains one certificate authority key ``ca.key.pem`` and symbolic links to it to be used as both an Identity and a Permissions CA private key.

.. warning::

  Protect this private key and create a secure backup of it!

This is the private key associated with the public Certificate Authority which serves as the anchor for all security in your ROS system.
You will use it to modify encryption policies for the ROS graph and to add new ROS participants.
Depending upon your robot's security needs, the key can be protected with access permissions and locked down to another account, or it can be moved off the robot entirely and onto another system or device.
If the file is lost, you will be unable to change access permissions and add new participants to the system.
Similarly, any user or process with access to the file has the ability to modify system policies and participants.

This file is only required for configuring the robot, but is not needed for the robot to run.
It can safely be stored offline in another system or removable media.

The ``sros2`` utilities use `elliptic curve cryptograpy <https://en.wikipedia.org/wiki/Elliptic-curve_cryptography>`_ rather than RSA for improved security and reduced key size.
Use the following command to show details about this elliptic curve private key:


.. code-block:: bash

  cd ~/sros2_demo/demo_keys/private
  openssl ec -in ca.key.pem -text -noout

Your output should look similar to the following::

  read EC key
  Private-Key: (256 bit)
  priv:
      93:da:76:b9:e3:91:ab:e9:42:76:f2:38:f1:9d:94:
      90:5e:b5:96:7b:7f:71:ee:13:1b:d4:a0:f9:48:fb:
      ae:77
  pub:
      04:71:e9:37:d7:32:ba:b8:a0:97:66:da:9f:e3:c4:
      08:4f:7a:13:59:24:c6:cf:6a:f7:95:c5:cd:82:c0:
      7f:7f:e3:90:dd:7b:0f:77:d1:ee:0e:af:68:7c:76:
      a9:ca:60:d7:1e:2c:01:d7:bc:7e:e3:86:2a:9f:38:
      dc:ed:39:c5:32
  ASN1 OID: prime256v1
  NIST CURVE: P-256

In addition to the private key itself, note that the public key is listed, and it matches the public key listed in the Certificate Authority ``ca.cert.pem``.


Domain Governance Policy
^^^^^^^^^^^^^^^^^^^^^^^^

Find the domain governance policy in the enclave directory within the keystore, ``~/sros2_demo/demo_keys/enclaves``.
The ``enclave`` directory contains XML governance policy document ``governance.xml``, as well as a copy of the document which has been signed by the Permissions CA as ``governance.p7s``.

The ``governance.p7s`` file contains domain-wide settings such as how to handle unauthenticated participants, whether to encrypt discovery, and default rules for access to topics.

Use the following command to validate the `S/MIME signature <https://en.wikipedia.org/wiki/S/MIME>`_ of the governance file:

.. code-block:: bash

  openssl smime -verify -in governance.p7s -CAfile ../public/permissions_ca.cert.pem

This command will print out the XML document, and the last line will be ``Verification successful`` to show that the document was properly signed by the Permissions CA.


Security Enclaves
^^^^^^^^^^^^^^^^^

Secure processes (typically ROS nodes) run within a security enclave.
In the simplest case, all the processes can be consolidated into the same enclave, and all processes will then use the same security policy.
However, to apply different policies to different processes, the processes can use different security enclaves when starting.
For more details about security enclaves, see the `design document <https://design.ros2.org/articles/ros2_security_enclaves.html>`_.
The security enclave is specifed by using the ROS argument ``--enclave`` when running a node.

**Each security enclave requires six files** in order to enable security.
Each file **must** be named as defined below, and as outlined in the `DDS Security standard <https://www.omg.org/spec/DDS-SECURITY/1.1/About-DDS-SECURITY/>`_.
In order to avoid having mulitple copies of the same files, the ``sros2`` utilities create links for each enclave to the single governance policy, the Identity CA and Permissions CA descibed above.

See the following six files within the ``listener`` enclave.
Three are specific to this enclave, while three are generic to this ROS system:

 - ``key.pem``, the private key used to encrypt and decrypt within this enclave
 - ``cert.pem``, the public certificate for this enclave; this certificate has been signed by the Identity CA
 - ``permissions.p7s``, the permissions for this enclave; this file has been signed with the Permissions CA
 - ``governance.p7s``, a link to the signed security policy file for this domain
 - ``identity_ca.cert.pem``, a link to the Identity CA for this domain
 - ``permissions_ca.cert.pem``, a link to the Permissions CA for this domain

The private encryption key ``key.pem`` should be protected according to your security plan.
This key encrypts, decrypts and validates communications within this specific enclave.
Should the key be lost or stolen, revoke the key and create a new identity for this enclave.

The file ``permissions.xml`` has also been created in this directory and can be used to recreate the signed permissions file.
However, this file is not required to enable security since DDS uses the signed version of the file instead.


Take the quiz!
--------------

See if you can answer these questions about the ROS security keystore.
Begin with a new terminal session and enable security with the keystore created in the prior tutorial:

.. code-block:: bash

  export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keys
  export ROS_SECURITY_ENABLE=true
  export ROS_SECURITY_STRATEGY=Enforce

  cd ~/sros2_demo/demo_keys/enclaves/talker_listener/listener

Make a backup copy of ``permissions.p7s`` before beginning.

.. tabs::

  .. group-tab:: Question 1

    Open ``permissions.p7s`` in a text editor. Make a negligible change to the XML content (e.g., add a space or a blank line) and save the file.
    Launch the listener node:

    .. code-block:: bash

      ros2 run demo_nodes_cpp listener --ros-args --enclave /talker_listener/listener

    What do you expect to happen?

    Can you launch the talker node?

    .. code-block:: bash

      ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker

    What is the difference between launching the listener and launching the talker?

  .. group-tab:: Answer 1

    The listener fails to launch and throws an error.
    When the ``permissions.p7s`` file was modified--however minor--the file's signature became invalid.
    A node will not launch with security enabled and enforced when the permissions file is invalid.

    The talker will start as expected.
    It uses the ``permissions.p7s`` file in a different enclave, and the file is still valid.

.. tabs::

  .. group-tab:: Question 2

    What command lets you check to see if the signature on the modified ``permissions.p7s`` file is valid?

  .. group-tab:: Answer 2

    Check that ``permissions.p7s`` has been properly signed by the Permissions CA using the ``openssl smime`` command:

    .. code-block:: bash

      openssl smime -verify -in permissions.p7s -CAfile permissions_ca.cert.pem

Restore your original, properly signed ``permissions.p7s`` file before proceeding to the next tutorial.

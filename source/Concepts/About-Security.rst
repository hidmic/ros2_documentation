.. _ROS-2-Security:

About ROS 2 Security
====================

.. contents:: Table of Contents
   :local:


Overview
--------

Built-in ROS 2 security features enable control over communications throughout the ROS graph.
This not only allows for encrypting data in transit between ROS domain participants, but also enables authentication of participants sending data, ensures the integrity of data being sent, and enables domain-wide access controls.

ROS 2 security services are provided by the underlying `Data Distribution Service (DDS) <https://www.omg.org/spec/DDS/>`_ which is used for communications between nodes.
DDS vendors provide open source and commercial DDS implementations that work with ROS.
However, in order to create a specification-compliant implementation of DDS, all vendors must include security plugins as outlined in the `DDS Security Specification <https://www.omg.org/spec/DDS-SECURITY/About-DDS-SECURITY/>`_.
ROS security features take advantage of these DDS security plugins to provide policy-based encryption, authentication and access control.
DDS and ROS security is enabled through predefined configuration files and environment variables.


The Security Enclave
--------------------

A security enclave encapsulates a single policy for protecting ROS communications.
The enclave may set policy for multiple nodes, for an entire ROS graph, or any combination of protected ROS processes and devices.
Security enclaves can be flexibly mapped to processes, users, or devices at deployment.
Adjusting this default behavior becomes important for optimizing communications and for complex systems.
See the ROS 2 Security Enclaves `design document <https://design.ros2.org/articles/ros2_security_enclaves.html>`_ for additional details.


Security Files
--------------

A `ROS 2 security enclave <https://design.ros2.org/articles/ros2_security_enclaves.html>`_ is established with six files as outlined by the DDS specification.
Three of these files define an enclave's identity, while three other files define the permissions to be granted to the enclave.
All six files reside in a single directory, and nodes launched without a qualified enclave path use files in the default root level enclave.

Enclave Identity
^^^^^^^^^^^^^^^^

The Identity Certificate Authority file ``identity_ca.cert.pem`` acts as the trust anchor used to identify participants.
Each enclave also holds its unique identifying certificate in the file ``cert.pem``, and the associated private private key in the file ``key.pem``.
Because the ``cert.pem`` certificate has been signed by identity certificate, when a participant presents this certificate to other domain members, they are able to validate the participant's identity using their own copy of the identity certificate.
This valid certificate exchange allows the enclave to securely establish trusted communications with other participants.
The enclave does not not share the ``key.pem`` private key, but only uses it for decryption and message signing.

Enclave Permissions
^^^^^^^^^^^^^^^^^^^

The Permissions Certificate Authority file ``permissions_ca.cert.pem`` serves as the trust anchor to grant permissions to security enclaves.
This certificate is used to create the signed file ``governance.p7s``, an XML document which defines domain-wide protection policies.
Similarly the XML file ``permissions.p7s`` outlines permissions of this particular enclave and has been signed by the Permissions CA.
Domain members use a copy of the permissions CA to validate these signed files and grant the requested access.

Although these two certificate authorities enable separate workflows for identity and permissions, often the same certificate serves as both the identity and the permissions authority.

Private Keys
^^^^^^^^^^^^

The identity and permissions certificates also have associated private key files.
Add new enclaves to the domain by signing their Certificate Signing Request (CSR) with the identity certificate's private key.
Similarly, grant permissions for a new enclave by signing a permissions XML document with the permission certificate's private key.


Security Environment Variables
------------------------------

The environment variable ``ROS_SECURITY_ENABLE`` acts as the enclave's master "on/off" switch for ROS 2 security features.
Security has been turned off by default, so security features will not be enabled even when the proper security files are present.
In order to enable ROS 2 security, set this environment variable to ``true`` (case sensitive).

Once security has been enabled, the environment variable ``ROS_SECURITY_STRATEGY`` defines how domain participants handle problems when launching participants.
Security features depend on certificates and properly signed configuration files, yet by default, an improperly configured participant will still launch successfully but without security features.
In order to enforce strict compliance with security settings and fail to launch non-compliant enclaves, set this environment variable to ``Enforce`` (case sensitive).

Additional security-related environment variables can be found in the `ROS 2 DDS-Security Integration design document <https://design.ros2.org/articles/ros2_dds_security.html>`_.
These variables generally assist ROS in managing enclaves and locating the security files.


Learn More
----------

For more information and hands-on exercises enabling ROS 2 communications security, see the :ref:`ROS 2 Security Tutorials <ROS-2-Security-Tutorials>`.

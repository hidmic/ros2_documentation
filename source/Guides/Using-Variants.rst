Using variants
==============

Variants, referred to as "metapackages" in ROS 1 are packages which provide no functionality directly but depend on a specified set of ROS packages in order to provide a convenient installation mechanism for the entire group of packages at once.

The different variants in ROS 2 are specified in `REP-2001 <https://ros.org/reps/rep-2001.html>`_.

In addition to the official variants, there may be metapackages for specific institutions or robots as described in `REP-108 <(https://www.ros.org/reps/rep-0108.html#institution-specific>`_.

Adding variants
---------------

Additional variants that are of general use to the ROS community can be proposed by contributing an update to `REP-2001 via pull request <https://github.com/ros-infrastructure/rep/blob/master/rep-2001.rst>`_ describing the packages included in the new variant.
Institution and robot specific variants can be published directly by their respective maintainers and no update to REP-2001 is required.

Creating project-specific variants
----------------------------------

If you are creating ROS packages to use privately in your own projects, you can create variants specific to your projects using the official variants as examples.
To do so you need only create two files:

#. A minimal variant package is created as a package with the ``ament_cmake`` build type, a ``buildtool_depend`` on ``ament_cmake`` and ``exec_depend`` entries for each package you want to include in the variant.


.. code-block:: xml

  <?xml version="1.0"?>
  <?xml-model href="http://download.ros.org/schema/package_format2.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
  <package format="2">
    <name>my_project_variant</name>
    <version>1.0.0</version>
    <description>A package to aggregate all packages in my_project.</description>
    <maintainer email="maintainer-email">Maintainer Name</maintainer>
    <license>Apache License 2.0</license>
    <!-- packages in my_project -->
    <exec_depend>my_project_msgs</exec_depend>
    <exec_depend>my_project_services</exec_depend>
    <exec_depend>my_project_examples</exec_depend>

    <export>
      <build_type>ament_cmake</build_type>
    </export>
  </package>

#. A minimal ament_cmake package includes a ``CMakeLists.txt`` which registers the package.xml as an ament package for use in ROS 2.

.. code-block:: cmake

  cmake_minimum_required(VERSION 3.5)

  project(my_project_variant NONE)
  find_package(ament_cmake REQUIRED)
  ament_package()

#. You can then build and install your variant package alongside your other private packages.

Creating custom variants with platform-specific tools
*****************************************************

Some platforms have tools for creating basic packages that do not require a full ROS build farm environment or equivalent infrastructure.
It is possible to use these tools to create platform-dependent variants.
This approach does not include support for ROS packaging tools and is platform dependent but requires much less infrastructure to produce if you are creating collections of existing packages rather than a mix of public and private ROS packages.
For example, on Debian or Ubuntu systems you can use the ``equivs`` utilities.
The Debian Administrator's handbook has a `Section on meta-packages <https://www.debian.org/doc/manuals/debian-handbook/sect.building-first-package.en.html#id-1.18.5.2>`_.


ament_python user documentation
===============================

ament_python is the name of the Python build type for ROS 2.
ament_python packages are "pure" Python packages which create resources for the ament package index.

Unlike in ROS 1, most Python packages in ROS 2 can use the standard python build tools.
Colcon will use setuptools to build and install ament_python packages and bloom will create templates that use the
respective standard platform-specific tools for building python packages when generating package metadata for the ROS build farm.

Although ament_python packages use the standard ``setup.py`` for build and installation, ament_python packages must also include a
``package.xml`` file.
An unfortunate effect of this is that both ``package.xml`` and ``setup.py`` may contain some duplicated information if you want both package systems to accurately report information about installed packages.
You may wish to create a minimal setup.py and describe package metadata and dependency information only in your ``package.xml`` file.
That will be sufficient for building with Colcon and on the ROS build farm but may not be sufficient to build individual Python packages directly.

Compiled extensions to Python packages are not supported by Colcon and are untested on the ROS build farm.
When building Python packages with native extensions it is recommended to switch to the Ament CMake build system and use the ``ament_python_install_package`` helper.

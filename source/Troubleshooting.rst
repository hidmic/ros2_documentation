.. _Troubleshooting:

Troubleshooting
===============

Troubleshooting techniques are sorted by the platform they apply to.

.. contents:: Platforms
   :depth: 1
   :local:

General
-------

General troubleshooting techniques apply to all platforms.

Enable multicast
^^^^^^^^^^^^^^^^

In order to communicate successfully via DDS, the used network interface has to be multicast enabled.
We've seen in past experiences that this might not necessarily be enabled by default (on Ubuntu or OSX) when using the loopback adapter.
See the `original issue <https://github.com/ros2/ros2/issues/552>`__ or a `conversation on ros-answers <https://answers.ros.org/question/300370/ros2-talker-cannot-communicate-with-listener/>`__.
You can verify that your current setup allows multicast with the ROS 2 tool:

In Terminal 1:

.. code-block:: bash

   ros2 multicast receive

In Terminal 2:

.. code-block:: bash

   ros2 multicast send

If the first command did not return a response similar to:

.. code-block:: bash

   Received from xx.xxx.xxx.xx:43751: 'Hello World!'

then you will need to update your firewall configuration to allow multicast using `ufw <https://help.ubuntu.com/community/UFW>`__.

.. code-block:: bash

   sudo ufw allow in proto udp to 224.0.0.0/4
   sudo ufw allow in proto udp from 224.0.0.0/4


You can check if the multicast flag is enabled for your network interface using the :code:`ifconfig` tool and looking for :code:`MULITCAST` in the flags section:

.. code-block:: bash

   eno1: flags=4163<...,MULTICAST>
      ...

.. _linux-troubleshooting:

Linux
-----

Internal compiler error
^^^^^^^^^^^^^^^^^^^^^^^

If you experience an ICE when trying to compile on a memory constrained platform like a Raspberry PI you might want to build single threaded (prefix the build invocation with ``MAKEFLAGS=-j1``).

Out of memory
^^^^^^^^^^^^^

The ``ros1_bridge`` in its current form requires 4Gb of free RAM to compile.
If you don't have that amount of RAM available it's suggested to use ``COLCON_IGNORE`` in that folder and skip its compilation.

Multiple host interference
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you're running multiple instances on the same network you may get interference.
To avoid this you can set the environment variable ``ROS_DOMAIN_ID`` to a different integer, the default is zero.
This will define the DDS domain id for your system.
Note that if you are using the OpenSplice DDS implementation you will also need to update the OpenSplice configuration file accordingly.
The location of the configuration file is referenced in the ``OSPL_URI`` environment variable.

Exception sourcing setup.bash
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. only relevant to Eloquent and Foxy

If you encounter exceptions when trying to source the environment after building from source, try to upgrade ``colcon`` related packages using

.. code-block:: bash

   colcon version-check  # check if newer versions available
   sudo apt install python3-colcon* --only-upgrade  # upgrade installed colcon packages to latest version

.. _macOS-troubleshooting:

macOS
-----

Segmentation fault when using ``pyenv``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``pyenv`` seems to default to building Python with ``.a`` files, but that causes issues with ``rclpy``, so it's recommended to build Python with Frameworks enabled on macOS when using ``pyenv``:

https://github.com/pyenv/pyenv/wiki#how-to-build-cpython-with-framework-support-on-os-x

Library not loaded; image not found
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are seeing library loading issues at runtime (either running tests or running nodes), such as the following:

.. code-block:: bash

   ImportError: dlopen(.../ros2_<distro>/ros2-osx/lib/python3.7/site-packages/rclpy/_rclpy.cpython-37m-darwin.so, 2): Library not loaded: @rpath/librcl_interfaces__rosidl_typesupport_c.dylib
     Referenced from: .../ros2_<distro>/ros2-osx/lib/python3.7/site-packages/rclpy/_rclpy.cpython-37m-darwin.so
     Reason: image not found

Then you probably have System Integrity Protection enabled.
Follow `these instructions <https://developer.apple.com/library/content/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html>`__ to disable System Integrity Protection (SIP).

Qt build error: ``unknown type name 'Q_ENUM'``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you see build errors related to Qt, e.g.:

.. code-block:: bash

   In file included from /usr/local/opt/qt/lib/QtGui.framework/Headers/qguiapplication.h:46:
   /usr/local/opt/qt/lib/QtGui.framework/Headers/qinputmethod.h:87:5: error:
         unknown type name 'Q_ENUM'
       Q_ENUM(Action)
       ^

you may be using qt4 instead of qt5: see https://github.com/ros2/ros2/issues/441

Qt build error: ``"mkspecs/macx-clang" but this file does not exist``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. only on Crystal

To fix this error:

.. code-block:: bash

  CMake Error at /usr/local/lib/cmake/Qt5Core/Qt5CoreConfig.cmake:15 (message):
    The imported target "Qt5::Core" references the file

       "/usr/local/.//mkspecs/macx-clang"

    but this file does not exist. Possible reasons include:

    * The file was deleted, renamed, or moved to another location.

    * An install or uninstall procedure did not complete successfully.

    * The installation package was faulty and contained

       "/usr/local/lib/cmake/Qt5Core/Qt5CoreConfigExtras.cmake"

    but not all the files it references.

link ``mkspecs`` and ``plugins`` folders to ``/usr/local/``:

.. code-block:: bash

  export HOMEBREW_QT5_VERSION=5.12.3 # Specify appropriate Qt5 version here
  sudo ln -s /usr/local/Cellar/qt/$HOMEBREW_QT5_VERSION/mkspecs /usr/local/mkspecs
  sudo ln -s /usr/local/Cellar/qt/$HOMEBREW_QT5_VERSION/plugins /usr/local/plugins

If you are on a previous version of Homebrew, the ``qt`` formula could still be called ``qt5``, so make corresponding changes to the paths above.

Missing symbol when opencv (and therefore libjpeg, libtiff, and libpng) are installed with Homebrew
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you have opencv installed you might get this:

.. code-block:: bash

   dyld: Symbol not found: __cg_jpeg_resync_to_restart
     Referenced from: /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO
     Expected in: /usr/local/lib/libJPEG.dylib
    in /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO
   /bin/sh: line 1: 25274 Trace/BPT trap: 5       /usr/local/bin/cmake

If so, to build you'll have to do this:

.. code-block:: bash

   $ brew unlink libpng libtiff libjpeg

But this will break opencv, so you'll also need to update it to continue working:

.. code-block:: bash

   $ sudo install_name_tool -change /usr/local/lib/libjpeg.8.dylib /usr/local/opt/jpeg/lib/libjpeg.8.dylib /usr/local/lib/libopencv_highgui.2.4.dylib
   $ sudo install_name_tool -change /usr/local/lib/libpng16.16.dylib /usr/local/opt/libpng/lib/libpng16.16.dylib /usr/local/lib/libopencv_highgui.2.4.dylib
   $ sudo install_name_tool -change /usr/local/lib/libtiff.5.dylib /usr/local/opt/libtiff/lib/libtiff.5.dylib /usr/local/lib/libopencv_highgui.2.4.dylib
   $ sudo install_name_tool -change /usr/local/lib/libjpeg.8.dylib /usr/local/opt/jpeg/lib/libjpeg.8.dylib /usr/local/Cellar/libtiff/4.0.4/lib/libtiff.5.dylib

The first command is necessary to avoid things built against the system libjpeg (etc.) from getting the version in /usr/local/lib.
The others are updating things built by Homebrew so that they can find the version of libjpeg (etc.) without having them in /usr/local/lib.

Xcode-select error: tool ``xcodebuild`` requires Xcode, but active developer directory is a command line instance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. only relevant to Eloquent and Foxy

If you recently installed Xcode, you may encounter this error:

.. code-block:: bash

   Xcode: xcode-select: error: tool 'xcodebuild' requires Xcode,
   but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance

To resolve this error, you will need to:

1. Double check that you have the command line tool installed:

.. code-block:: bash

   $ xcode-select --install

2. Accept the terms and conditions of Xcode by typing in terminal:

.. code-block:: bash

   $ sudo xcodebuild -license accept

3. Ensure Xcode app is in the ``/Applications`` directory (NOT ``/Users/{user}/Applications``)

4. Point ``xcode-select`` to the Xcode app Developer directory using the following command:

.. code-block:: bash

   $ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer

qt_gui_cpp error: SIP binding generator NOT available
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. only relevant to Eloquent and Foxy

When building qt_gui_cpp there may be errors look like the following:

.. code-block:: bash

   --- stderr: qt_gui_cpp

   CMake Error at src/CMakeLists.txt:10 (message):
     No Python binding generator found.

   ---
   Failed   <<< qt_gui_cpp [ Exited with code 1 ]

To fix this issue, follow `these steps <../../Tutorials/RQt-Source-Install-MacOS>` to install dependencies for RQt.

.. _windows-troubleshooting:

Windows
-------

Import failing even with library present on the system
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sometimes ``rclpy`` fails to be imported because of some missing DLLs on your system.
If so, make sure to install all the dependencies listed in the "Installing prerequisites" sections of the `installation instructions <Foxy_windows-install-binary-installing-prerequisites>`).

If you are installing from binaries, you may need to update your dependencies: they must be the same version as those used to build the binaries.

If you are still having issues, you can use the `Dependencies <https://github.com/lucasg/Dependencies>`_ tool to determine which dependencies are missing on your system.
Use the tool to load the corresponding ``.pyd`` file, and it should report unavailable ``DLL`` modules.
Be sure that the current workspace is sourced before you execute the tool, otherwise there will be unresolved ROS DLL files.
Use this information to install additional dependencies or adjust your path as necessary.

CMake error setting modification time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you run into the CMake error ``file INSTALL cannot set modification time on ...`` when installing files it is likely that an anti virus software or Windows Defender are interfering with the build. E.g. for Windows Defender you can list the workspace location to be excluded to prevent it from scanning those files.

260 character path limit
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   The input line is too long.
   The syntax of the command is incorrect.

Depending on your directory hierarchy, you may see path length limit errors when building ROS 2 from source or your own libraries.

To allow deeper path lengths:

Run ``regedit.exe``, navigate to ``Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem``, and set ``LongPathsEnabled`` to 0x00000001 (1).

Hit the windows key and type ``Edit Group Policy``.
Navigate to Local Computer Policy > Computer Configuration > Administrative Templates > System > Filesystem.
Right click ``Enable Win32 long paths``, click Edit.
In the dialog, select Enabled and click OK.

Close and open your terminal to reset the environment and try building again.

CMake packages unable to find asio, tinyxml2, tinyxml, eigen, or log4cxx
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We've seen that sometimes the chocolatey packages for ``asio``, ``tinyxml2``, etc. do not add important registry entries and CMake will be unable to find them when building ROS 2.
We've not yet been able to identify the root cause, but uninstalling the chocolatey packages (with ``-n`` if the uninstall fails the first time), and then reinstalling them will fix the issue.

patch.exe opens a new command window and asks for administrator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This will also cause the build of packages which need to use patch to fail, even you allow it to use administrator rights.

The solution, for now, is to make sure you're building in a Visual Studio command prompt which has been run as administrator.
On some machines canceling the prompt without selecting "Yes" will also work.

Failed to load Fast RTPS shared library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. does not apply to Crystal

Fast RTPS requires ``msvcr20.dll``, which is part of the ``Visual C++ Redistributable Packages for Visual Studio 2013``.
Although it is usually installed by default in Windows 10, we know that some Windows 10-like versions don't have it installed by default (e.g.: Windows Server 2019).
In case you don't have it installed, you can download it from `here <https://www.microsoft.com/en-us/download/details.aspx?id=40784>`_.

Binary installation specific
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* If your example does not start because of missing dll's, please verify that all libraries from external dependencies such as OpenCV are located inside your ``PATH`` variable.
* If you forget to call the ``local_setup.bat`` file from your terminal, the demo programs will most likely crash immediately.

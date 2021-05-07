Installing prerequisites
------------------------

Install Chocolatey
^^^^^^^^^^^^^^^^^^

Chocolatey is a package manager for Windows, install it by following their installation instructions:

https://chocolatey.org/

You'll use Chocolatey to install some other developer tools.

Install Python
^^^^^^^^^^^^^^

Open a Command Prompt and type the following to install Python via Chocolatey:

.. code-block:: bash

   choco install -y python --version 3.8.3

Install Visual C++ Redistributables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Open a Command Prompt and type the following to install them via Chocolatey:

.. code-block:: bash

   choco install -y vcredist2013 vcredist140

Install OpenSSL
^^^^^^^^^^^^^^^

Download the *Win64 OpenSSL v1.1.1k* OpenSSL installer from `this page <https://slproweb.com/products/Win32OpenSSL.html>`__.
Scroll to the bottom of the page and download *Win64 OpenSSL v1.1.1k*.
Don't download the Win32 or Light versions.

Run the installer with default parameters, as the following commands assume you used the default installation directory.

This command sets an environment variable that persists over sessions:

* ``setx -m OPENSSL_CONF "C:\Program Files\OpenSSL-Win64\bin\openssl.cfg"``

You will need to append the OpenSSL-Win64 bin folder to your PATH.
You can do this by clicking the Windows icon, typing "Environment Variables", then clicking on "Edit the system environment variables".
In the resulting dialog, click "Environment Variables", then click "Path" on the bottom pane, finally click "Edit" and add the path below.

* ``C:\Program Files\OpenSSL-Win64\bin\``

Install Visual Studio
^^^^^^^^^^^^^^^^^^^^^

Install Visual Studio 2019.

If you already have a paid version of Visual Studio 2019 (Professional, Enterprise), skip this step.

Microsoft provides a free of charge version of Visual Studio 2019, named Community, which can be used to build applications that use ROS 2:

   https://visualstudio.microsoft.com/downloads/

Make sure that the Visual C++ features are installed.

An easy way to make sure they're installed is to select the ``Desktop development with C++`` workflow during the install.

   .. image:: https://i.imgur.com/2h0IxCk.png

Make sure that no C++ CMake tools are installed by unselecting them in the list of components to be installed.

Install additional DDS implementations (optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you would like to use another DDS or RTPS vendor besides the default, Eclipse Cyclone DDS, you can find instructions `here <DDS-Implementations>`.

Install OpenCV
^^^^^^^^^^^^^^

Some of the examples require OpenCV to be installed.

You can download a precompiled version of OpenCV 3.4.6 from https://github.com/ros2/ros2/releases/download/opencv-archives/opencv-3.4.6-vc16.VS2019.zip .

Assuming you unpacked it to ``C:\opencv``, type the following on a Command Prompt (requires Admin privileges):

.. code-block:: bash

   setx -m OpenCV_DIR C:\opencv

Since you are using a precompiled ROS version, we have to tell it where to find the OpenCV libraries.
You have to extend the ``PATH`` variable to ``C:\opencv\x64\vc16\bin``.

Install dependencies
^^^^^^^^^^^^^^^^^^^^

There are a few dependencies not available in the Chocolatey package database.
In order to ease the manual installation process, we provide the necessary Chocolatey packages.

As some chocolatey packages rely on it, we start by installing CMake

.. code-block:: bash

   choco install -y cmake

You will need to append the CMake bin folder ``C:\Program Files\CMake\bin`` to your PATH.

Please download these packages from `this <https://github.com/ros2/choco-packages/releases/latest>`__ GitHub repository.

* asio.1.12.1.nupkg
* bullet.2.89.0.nupkg
* cunit.2.1.3.nupkg
* eigen-3.3.4.nupkg
* tinyxml-usestl.2.6.2.nupkg
* tinyxml2.6.0.0.nupkg
* log4cxx.0.10.0.nupkg

Once these packages are downloaded, open an administrative shell and execute the following command:

.. code-block:: bash

   choco install -y -s <PATH\TO\DOWNLOADS\> asio cunit eigen tinyxml-usestl tinyxml2 log4cxx bullet

Please replace ``<PATH\TO\DOWNLOADS>`` with the folder you downloaded the packages to.

You must also install some additional python dependencies:

.. code-block:: bash

   python -m pip install -U catkin_pkg cryptography empy ifcfg importlib-metadata lark-parser lxml matplotlib netifaces numpy opencv-python PyQt5 pip pillow psutil pycairo pydot pyparsing pyyaml rosdistro setuptools


Install Qt5
^^^^^^^^^^^

This section is only required if you are building rviz, but it comes with our default set of sources, so if you don't know, then assume you are building it.

First get the installer from Qt's website:

https://www.qt.io/download

Select the Open Source version and then the ``Qt Online Installer for Windows``.

Run the installer and install Qt5.

We recommend you install it to the default location of ``C:\Qt``, but if you choose somewhere else, make sure to update the paths below accordingly.
When selecting components to install, the only thing you absolutely need is the appropriate MSVC 64-bit component under the ``Qt`` -> ``Qt 5.15.0`` tree.
We're using ``5.15.0`` as of the writing of this document and that's what we recommend since that's all we test on Windows, but later Qt5 versions will probably work too.
Be sure to select ``MSVC 2019 64-bit``.
After that, the default settings are fine.

Finally, set the ``Qt5_DIR`` environment variable in the ``cmd.exe`` where you intend to build so that CMake can find it:

.. code-block:: bash

   set Qt5_DIR=C:\Qt\5.15.0\msvc2019_64
   set QT_QPA_PLATFORM_PLUGIN_PATH=C:\Qt\5.15.0\msvc2019_64\plugins\platforms

You could set it permanently with ``setx -m Qt5_DIR C:\Qt\5.15.0\msvc2019_64`` and ``setx -m QT_QPA_PLATFORM_PLUGIN_PATH C:\Qt\5.15.0\msvc2019_64\plugins\platforms`` instead, but that requires Administrator.

.. note::

   This path might change based on which MSVC version you're using or if you installed it to a different directory.

RQt dependencies
^^^^^^^^^^^^^^^^

To run rqt_graph you need to `download <https://graphviz.gitlab.io/_pages/Download/Download_windows.html>`__ and install `Graphviz <https://graphviz.gitlab.io/>`__.
The installer will ask if to add graphviz to PATH, choose to either add it to the current user or all users.

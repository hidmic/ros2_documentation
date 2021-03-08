You will need to add the ROS 2 apt repositories to your system.
To do so, first authorize our GPG key with apt like this:

.. code-block:: bash

   sudo apt update && sudo apt install curl gnupg2 lsb-release
   curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | gpg --dearmor | sudo tee /usr/share/keyrings/ros.gpg > /dev/null

And then add the repository to your sources list:

.. code-block:: bash

   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2-latest.list > /dev/null

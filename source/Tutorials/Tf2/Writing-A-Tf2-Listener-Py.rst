.. _WritingATf2ListenerPy:

Writing a tf2 listener (Python)
===============================

**Goal:** Learn how to use tf2 to get access to frame transformations.

**Tutorial level:** Intermediate

**Time:** 10 minutes

.. contents:: Contents
   :depth: 2
   :local:

Background
----------

In the previous tutorials we created a tf2 broadcaster to publish the pose of a turtle to tf2. In this tutorial we'll create a tf2 listener to start using tf2.

Prerequisites
-------------

This tutorial assumes you have completed the writing a :ref:`tf2 broadcaster tutorial (Python) <WritingATf2BroadcasterPy>`.
In previous tutorial, we created a ``learning_tf2_py`` package, which is where we will continue working from.

Tasks
-----

1 Write the listener node
^^^^^^^^^^^^^^^^^^^^^^^^^

Let's first create the source files. Go to the ``learning_tf2_py`` package we created in the previous tutorial.
Inside the ``src/learning_tf2_py/learning_tf2_py`` directory download the example listener code by entering the following command:

.. tabs::

    .. group-tab:: Linux

        .. code-block:: console

            wget https://github.com/ros/geometry_tutorials/blob/ros2/turtle_tf2_py/turtle_tf2_py/turtle_tf2_listener.py

    .. group-tab:: macOS

        .. code-block:: console

            wget https://github.com/ros/geometry_tutorials/blob/ros2/turtle_tf2_py/turtle_tf2_py/turtle_tf2_listener.py

    .. group-tab:: Windows

        In a Windows command line prompt:

        .. code-block:: console

                curl -sk https://github.com/ros/geometry_tutorials/blob/ros2/turtle_tf2_py/turtle_tf2_py/turtle_tf2_listener.py -o turtle_tf2_listener.py

        Or in powershell:

        .. code-block:: console

                curl https://github.com/ros/geometry_tutorials/blob/ros2/turtle_tf2_py/turtle_tf2_py/turtle_tf2_listener.py -o turtle_tf2_listener.py

Open the file using your preferred text editor.

.. code-block:: python

    import math

    from geometry_msgs.msg import Twist

    import rclpy
    from rclpy.duration import Duration
    from rclpy.node import Node

    from tf2_ros import LookupException
    from tf2_ros.buffer import Buffer
    from tf2_ros.transform_listener import TransformListener

    from turtlesim.srv import Spawn


    class FrameListener(Node):

        def __init__(self):
            super().__init__('turtle_tf2_frame_listener')

            # Declare and acquire `target_frame` parameter
            self.declare_parameter('target_frame', 'turtle1')
            self.target_frame = self.get_parameter(
                'target_frame').get_parameter_value().string_value

            self._tf_buffer = Buffer()
            self._tf_listener = TransformListener(self._tf_buffer, self)

            # Create a client to spawn a turtle
            self.client = self.create_client(Spawn, 'spawn')

            # Check if the service is available
            while not self.client.wait_for_service(timeout_sec=5.0):
                self.get_logger().info('service not available, waiting again...')

            # Initialize request with turtle name and coordinates
            # Note that x, y and theta are defined as floats in turtlesim/srv/Spawn
            request = Spawn.Request()
            request.name = 'turtle2'
            request.x = float(4)
            request.y = float(2)
            request.theta = float(0)
            # Call request
            self.client.call_async(request)

            # Create turtle2 velocity publisher
            self.turtle_vel_ = self.create_publisher(Twist, 'turtle2/cmd_vel', 1)

            # Call on_timer function every second
            self._output_timer = self.create_timer(1.0, self.on_timer)

        def on_timer(self):
            # Store frame names in variables that will be used to
            # compute transformations
            from_frame_rel = self.target_frame
            to_frame_rel = 'turtle2'

            # Look up for the transformation between target_frame and turtle2 frames
            # and send velocity commands for turtle2 to reach target_frame
            try:
                now = rclpy.time.Time()
                trans = self._tf_buffer.lookup_transform(
                    to_frame_rel,
                    from_frame_rel,
                    now,
                    timeout=Duration(seconds=1.0))
            except LookupException:
                self.get_logger().info('transform not ready')
                return

            msg = Twist()
            msg.angular.z = 1.0 * math.atan2(
                trans.transform.translation.y,
                trans.transform.translation.x)

            msg.linear.x = 0.5 * math.sqrt(
                trans.transform.translation.x ** 2 +
                trans.transform.translation.y ** 2)

            self.turtle_vel_.publish(msg)


    def main():
        rclpy.init()
        node = FrameListener()
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass

        rclpy.shutdown()

1.1 Examine the code
~~~~~~~~~~~~~~~~~~~~

Now, let's take a look at the code that is relevant to get access to frame transformations.
The ``tf2_ros`` package provides an implementation of a ``TransformListener`` to help make the task of receiving transforms easier.

.. code-block:: python

    from tf2_ros.transform_listener import TransformListener

Here, we create a ``TransformListener`` object. Once the listener is created, it starts receiving tf2 transformations over the wire, and buffers them for up to 10 seconds.

.. code-block:: python

    self._tf_listener = TransformListener(self._tf_buffer, self)

Finally, we query the listener for a specific transformation. We call ``lookup_transform`` method with following arguments:

#. Target frame

#. Source frame

#. The time at which we want to transform

Providing ``rclpy.time.Time()`` will just get us the latest available transform.
All this is wrapped in a try-except block to catch possible exceptions.

.. code-block:: python

    now = rclpy.time.Time()
    trans = self._tf_buffer.lookup_transform(
        to_frame_rel,
        from_frame_rel,
        now,
        timeout=Duration(seconds=1.0))

2 Build and run
^^^^^^^^^^^^^^^

With your text editor, open the launch file called ``turtle_tf2_demo.launch.py``, and add the following lines after your first ``turtle1`` broadcaster node:

.. code-block:: python

    from launch import LaunchDescription
    from launch_ros.actions import Node

    def generate_launch_description():
        return LaunchDescription([
            ...,
            DeclareLaunchArgument(
                'target_frame', default_value='turtle1',
                description='Target frame name.'
            ),
            Node(
                package='learning_tf2_py',
                executable='turtle_tf2_broadcaster',
                name='broadcaster2',
                parameters=[
                    {'turtlename': 'turtle2'}
                ]
            ),
            Node(
                package='learning_tf2_py',
                executable='turtle_tf2_listener',
                name='listener',
                parameters=[
                    {'target_frame': LaunchConfiguration('target_frame')}
                ]
            ),
        ])

This will declare a ``target_frame`` launch argument, start a broadcaster for second turtle that we will spawn and listener that will subscribe to those transformations.
Now you're ready to start your full turtle demo:

.. code-block:: console

    ros2 launch learning_tf2_py turtle_tf2_demo.launch.py

You should see the turtle sim with two turtles.
In the second terminal window type the following command:

.. code-block:: console

    ros2 run turtlesim turtle_teleop_key

3 Checking the results
^^^^^^^^^^^^^^^^^^^^^^

To see if things work, simply drive around the first turtle using the arrow keys (make sure your terminal window is active, not your simulator window), and you'll see the second turtle following the first one!

Summary
-------

In this tutorial you learned how to use tf2 to get access to frame transformations.
You also have finished writing your own turtlesim demo that you have tried in the  :ref:`Introduction to tf2 <IntroToTf2>` tutorial.

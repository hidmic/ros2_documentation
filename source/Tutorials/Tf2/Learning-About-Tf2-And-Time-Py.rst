.. _LearningAboutTf2AndTimePy:

Learning about tf2 and time (Python)
====================================

**Goal:** Learn to use the ``timeout`` in ``lookup_transform`` function to wait for a transform to be available on the tf2 tree.

**Tutorial level:** Intermediate

**Time:** 10 minutes

.. contents:: Contents
   :depth: 2
   :local:

Background
----------

In previous tutorials, we recreated the turtle demo and learned to add a new frame to the transformation tree.
This tutorial will teach you how use the timeout in ``lookup_transform`` function to wait for a transform to be available on the tf2 tree.

Tasks
-----

1 Update the listener node
^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit ``turtle_tf2_listener.py`` and remove the ``timeout=Duration(seconds=1.0)`` parameter that is passed to the ``lookup_transform()`` call on line 76.
It should look like shown below:

.. code-block:: python

   trans = self._tf_buffer.lookup_transform(
      to_frame_rel,
      from_frame_rel,
      now)

Moreover, import additional exceptions that we will handle in the beggining of the file:

.. code-block:: python

   from tf2_ros import LookupException, ConnectivityException, ExtrapolationException

Edit the exception handling on line 81 by adding newly imported exceptions and ``raise`` statement to see the exception:

.. code-block:: python

   except (LookupException, ConnectivityException, ExtrapolationException):
      self.get_logger().info('transform not ready')
      raise
      return

Now run the launch file:

.. code-block:: console

    ros2 launch learning_tf2_py turtle_tf2_demo.launch.py

2 Fix the listener node
^^^^^^^^^^^^^^^^^^^^^^^

So, all of the sudden ``lookup_transform()`` is failing, telling you that the frame does not exist or that the data is in the future.
To fix this, edit your code on line 76 as shown below (return the ``timeout`` parameter):

.. code-block:: python

   trans = self._tf_buffer.lookup_transform(
      to_frame_rel,
      from_frame_rel,
      now,
      timeout=Duration(seconds=1.0))

The ``lookup_transform`` can take four arguments, where the last one is an optional timeout.
It will block for up to that duration waiting for it to timeout.

.. note::

   Once this change is made, remove the ``raise`` line from the ``except()`` block that we added above or the code will continue to fail.

You can now run the launch file.

.. code-block:: console

   ros2 launch learning_tf2_py turtle_tf2_demo.launch.py

You should notice that ``lookup_transform()`` will actually block until the transform between the two turtles becomes available (this will usually take a few milli-seconds).
Once the timeout has been reached (1 second in this case), an exception will be raised only if the transform is still not available.

Summary
-------

In this tutorial you learned more about the ``lookup_transform`` function and its timeout features.
You also learned how to catch and handle additional exceptions that can be thrown by tf2.

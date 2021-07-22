**Note**: Even if `Linux Mint 20 <https://linuxmint.com/download_all.php>`__ bases on Ubuntu Focal, the ``lsb_release -cs`` command returns the codename of Linux Mint. Thus you've to edit the ros2.list file and replace the codename of Linux Mint 20 with ``focal``.

.. code-block:: bash

   sudo nano /etc/apt/sources.list.d/ros2.list


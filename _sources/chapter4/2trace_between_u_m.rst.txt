内核态与用户态间方便的切换跟踪
=====================================

本项目解决的一个调试操作系统时面临的难点是内核态与用户态间的切换跟踪。通过我们编写的调试工具，用户只需要根据自己计算机上的环境调整一个配置文件launch.json（图4.2），用户通过这个配置文件指定需要调试的操作系统的位置以及QEMU虚拟机的启动参数。配置完毕后，用户即可开始进行内核态与用户态间的切换跟踪：

.. image:: 4-2.png
   :align: center
   :name: 图4.2 launch.json配置文件

配置launch.json并保存后，按F5键，即可启动调试进程。

首先按下removeAllCliBreakpoints按钮清除所有断点：

.. image:: 4-3.png
   :align: center
   :name: 图4.3  清除所有断点。右下角的通知显示清除成功

其次，设置内核入口（setKernelInBreakpoints按钮）、出口断点（setKernelOutBreakpoints按钮）。

最后，设置内核代码和用户程序代码的断点。这两种断点可以同时设置，调试插件会自动进行断点组的缓存与切换。

.. image:: 4-4.png
   :align: center
   :name: 图4.4  设置内核代码断点

.. image:: 4-5.png
   :align: center
   :name: 图4.5  设置用户程序代码的断点。右下角的通知显示，这个断点被缓存到了断点组中

断点设置完毕后，按continue按钮开始运行rCore-Tutorial。当运行到位于内核出口的断点时，插件会自动切换到用户态的断点：

.. image:: 4-6.png
   :align: center
   :name: 图4.6  边界断点触发，右下角的通知显示插件自动进行了断点组切换

.. image:: 4-7.png
   :align: center
   :name: 图4.7  触发用户态程序的断点

在用户态程序中如果想观察内核内的执行流，可以点击gotokernel按钮，然后点击继续按钮，程序会停在内核的入口断点，这时，可以先把内核出口断点设置好（点击setKernelOutBreakpoints按钮），接下来，可以在内核态设置断点，点击继续，即可触发内核断点：

.. image:: 4-8.png
   :align: center
   :name: 图4.8  回到内核，再次触发内核断点

如果运行到内核的出口断点，又会回到用户态。

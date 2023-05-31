当前特权级检测
========================================================

调试操作系统与调试一般应用程序的一大区别是，调试操作系统时用户经常需要关注当前运行在什么特权级上。因此，操作系统调试工具需要有检测当前特权级的功能。我们以特权级检测为例，展示这个操作系统调试工具的典型处理流程。

当 Extension Frontend 监听到 GDB 触发断点、用户手动暂停、或 Debug Adapter 发送了 stopped Event 时，Extension Frontend 发送一个 customRequest 请求 Debug Adapter 返回当前特权级、寄存器数据、内存数据、断点列表等信息。

接着 Debug Adapter 响应这些请求，向 GDB 发送命令。RISC-V 处理器没有寄存器可以透露当前的特权级，因此不能直接通过 info registers 这个 GDB 命令获得当前特权级。Debug Adapter 会尝试获取当前执行的代码的内存地址和文件名，进而判断当前的特权级。

在得到当前所在的特权级后，Debug Adapter 向 Extension Frontend 返回 Responses。Extension Frontend 接收并解析 Responses 和 Events，将信息传递到 Debug UI。Debug UI 收到信息后更新界面。（图4.1）

.. image:: 4-1.jpg
   :align: center
   :name: 图4.1  更新特权级后的调试器界面。在右侧窗口的上方，可以发现此时操作系统处于内核态


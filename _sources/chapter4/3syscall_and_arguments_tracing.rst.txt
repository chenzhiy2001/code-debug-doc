系统调用跟踪及系统调用参数的获取
========================================================

我们以系统调用跟踪及系统调用参数的获取为例，展示这套调试工具的使用流程。

首先，在 rCore-Tutorial-v3 中，每个系统调用都会被分发到对应的内核函数中进行处理，这些函数最多只有三个参数。根据RISC-V的函数调用规范（calling conventions），它的函数调用过程通常分为以下6个阶段[7]：

1.	将参数存储到函数能够访问到的位置；
2.	跳转到函数开始位置(使用 RV32I 的 jal 指令)；
3.	获取函数需要的局部存储资源，按需保存寄存器；
4.	执行函数中的指令；
5.	将返回值存储到调用者能够访问到的位置，恢复寄存器，释放局部存储资源；
6.	返回调用函数的位置(使用 ret 指令)。

在第三个阶段结束后获取函数的参数是比较容易的。查阅函数调用规范（图4.9）后可知，我们需要的三个函数调用参数会分别被放在a0, a1 和 a2 寄存器上。

.. image:: 4-9.png
   :align: center
   :name: 图4.9  RISC-V函数调用规范

因此，若要获取系统调用的参数，我们只需要编写一个能获取寄存器信息的 eBPF 程序，并在需要跟踪的系统调用对应的内核函数上插桩即可。

eBPF 程序可以用多种语言编写，在这里我们选用 C 语言。eBPF 程序的代码如下： 

.. code-block:: c

   #include "bpf.h"
   #include "kprobe.h"
   int bpf_prog(struct kprobe_bpf_ctx *ctx) {
   bpf_trace_printk("%", 0, 0, 0); //字符串是rust 格式的
   bpf_trace_printk("R", 0, 0, 0); //用于标识消息类别
   for (int i = 0; i < 3; ++i) {
      bpf_trace_printk("{}", ctx->tf.regs[i], 0, 0);//发送寄存器数据
   }
   bpf_trace_printk("#", 0, 0, 0);
   bpf_trace_printk("00", 0, 0, 0);
   return 0;
   }

需要注意的是，由于 rCore-Tutorial-v3 的eBPF模块的字符串处理例程直接调用了 Rust 语言的字符串处理函数，因此字符串是 Rust 格式的。编写后，用 Clang 编译器编译到 eBPF 目标。
接着，将编译出的目标文件以硬编码的形式存储在eBPF Server中。至此代码的修改完成，我们可以编译、启动整套调试工具。
我们在在线 IDE 中点击“运行 - 调试”，调试工具就会自动连接到虚拟机提供的 gdbserver。然后我们可以设置 main-stub 的断点：

.. image:: 4-10.png
   :align: center
   :name: 图4.10  开始调试，启动gdbserver

.. image:: 4-11.png
   :align: center
   :name: 图4.11  设置断点，从下方终端可以看到Qemu虚拟机已经启动

.. image:: 4-12.png
   :align: center
   :name: 图4.12  位于第58行的main-stub的断点触发

在设置完main-stub的断点后，我们按“continue”启动虚拟机。启动之后，在终端中打开 eBPF Server，此时 eBPF Server 可以通过串口和 GDB 中对应的子模块通信：

.. image:: 4-13.png
   :align: center
   :name: 图4.13  在终端中打开eBPF Server。可以看到eBPF Server正在从串口接收消息

我们让GDB 中对应的子模块连接到 eBPF Server。在调试控制台中输入下列命令启用GDB中的eBPF子模块：

so ~/rCore-Tutorial-v3-eBPF/rCore-Tutorial-v3/side-stub.py

调试控制台输出以下信息，表示成功启用该模块。

{"token":14,"outOfBandRecord":[],"resultRecords":{"resultClass":"done","results":[]}}

接着令这个模块通过串口连接到eBPF Server：

-side-stub target remote /dev/pts/4

调试控制台输出以下信息，这说明连接成功：

{"token":18,"outOfBandRecord":[],"resultRecords":{"resultClass":"done","results":[]}}

最后输入要跟踪的系统调用。此处我们以 sys_open 为例：

	-side-stub break sys_open then-get register-info

调试控制台再次响应，表示插桩成功：

{"token":19,"outOfBandRecord":[],"resultRecords":{"resultClass":"done","results":[]}}

接下来可以看到，当我们跟踪的内核函数被触发时，eBPF程序运行并收集信息，返回了我们需要的三个参数：

.. image:: 4-14.png
   :align: center
   :name: 图4.14  操作系统输出eBPF相关的日志，表示我们跟踪的内核函数被触发

.. image:: 4-15.png
   :align: center
   :name: 图4.15  寄存器数据以高亮字体输出



服务器部分
===========================

.. toctree::
   :hidden:
   :maxdepth: 5

在线 VSCode
-------------------------------

OpenVSCode Server 是 VS Code 的一个分支，它在 VSCode 原有的五层架构的基础上增加了服务器层，使其可以提供一个和 VSCode 功能相近的，通过浏览器即可访问的在线IDE。这个在线IDE可以和服务器上的开发环境、调试环境通信。

用户可以在在线 IDE 上编辑项目源代码，同时可以远程连接到服务器上的终端。我们在服务器里配置好了 Qemu 虚拟机和 GDB、Rust 工具链。用户可以自行通过终端命令使用 Qemu、GDB 等工具手动调试自己编写的操作系统，也可以通过在线 IDE 中的操作系统调试模块进行更便利的调试。

如果用户选择用操作系统调试模块进行调试，操作系统调试模块做的第一步是编译内核并获取操作系统镜像文件和调试信息文件。接下来我们以 rCore-Tutorial-v3 [3]操作系统为例，阐述如何获取这两类文件。


编译
-----------------------------

在使用默认编译参数的情况下，rCore-Tutorial-v3 编译出的操作系统镜像和调试信息文件难以用于操作系统调试。这是因为 rCore-Tutorial-v3 操作系统基于rust语言编写，使用rustc编译器。在默认情况下，rustc编译器会对代码进行比较激进的优化，例如内连函数，删除大量有助于调试的符号信息。因此，我们需要修改编译参数，以尽量避免编译器的优化操作。

rCore-Tutorial-v3 是用 cargo 工具创建的。一般而言，用 cargo 工具创建的 rust 项目可用release, debug 两种模式编译、运行。在这两种模式中， release 模式对代码进行较高等级的优化，删除较多调试相关的信息，而 debug 模式则对代码进行较弱等级的优化并保留了更多调试相关的信息，比较符合我们的需求。但是由于 rCore-Tutorial-v3 项目本身的设计缺陷，这个项目不支持使用 debug 模式进行编译。因此，我们需要修改 release 模式的配置文件，让编译器在 release 模式下也像在 debug 模式下一样关闭代码优化，保留调试信息。

此外，rCore-Tutorial-v3 为了提升性能，修改了用户态程序的链接脚本，使得 .debug_info 等包含调试信息的DWARF 段[4]在链接时被忽略。这些段对调试用户态程序非常重要，因此我们需要修改链接脚本，移除这种忽略。在修改了链接脚本后，为了让链接脚本生效，需要用 cargo clean 命令清空缓存。

在修改了编译参数、链接脚本后，编译出的可执行文件占用的磁盘空间显著增加，导致 rCore-Tutorial-v3 操作系统的 easy-fs 文件系统无法正常运作，例如在加载文件时崩溃，栈溢出等。因此，我们调整了这个文件系统的 easy-fs-fuse 磁盘打包程序的磁盘大小等参数。此外，由于可执行文件中保留了大量符号信息，用户程序在运行时占用的内存也显著增加，因此需要调整操作系统的用户堆栈大小和内核堆栈大小。

我们将这些对于配置文件、链接脚本、操作系统源代码的修改整理成一个 diff 文件，用户只需要在远程终端中通过 git 命令应用这个 diff 文件即可完成上述修改。我们同时也维护一个已经修改好的 rCore-Tutorial-v3 GitHub 仓库供用户直接下载使用。

Qemu 和 GDB
-----------------------------

在编译完成后，服务器上的 Qemu 会加载操作系统镜像，并开启一个 gdbserver。接着，GDB 加载编译时生成的符号信息文件并连接到 Qemu 提供的 gdbserver。如果用户开启了 eBPF 跟踪功能，Qemu中运行的操作系统会启动基于 eBPF 的调试服务器（即 eBPF server）。这个基于 eBPF 的调试服务器会通过其专属的调试用串口连接到GDB上的 eBPF 调试处理模块。

GDB 与 gdbserver、eBPF server 通过 GDB 远程串行协议 (RSP) [5]进行通信。RSP 是一个通用的、高层级的协议，用于将 GDB 连接到任何远程目标。 只要远程目标的体系结构（例如在本项目中是RISC-V）已经被 GDB 支持，并且远程目标实现了支持 RSP 协议的服务器端，那么 GDB 就能够远程连接到该目标。

Debug adapter
-----------------------------

Debug Adapter 是一个独立的进程，负责协调在线 IDE 和 GDB。在 GDB 准备就绪后，Debug Adapter 进程会启动，并开始监听在线 IDE 中 Extension Frontend 模块发送来的各种调试请求。

如下图所示，一旦 Debug Adapter 接收到一个请求，它就会将请求（Debug Adapter Requests）转换为符合 GDB/MI 接口规范（GDB/MI 是一个基于行的面向机器的 GDB 文本接口，它专门用于支持将调试器用作大型系统的一个小组件的系统的开发。）的文本并发送给 GDB。GDB 在解析、执行完 Debug Adapter 发来的命令后，返回符合 GDB/MI 规范的文本信息。Debug Adapter 将 GDB 返回的信息解析后，向 Extension Frontend 返回 Debug Adapter Protocol 协议的 Respond 消息。此外，调试过程中发生的特权级切换、断点触发等事件会通过 Debug Adapter Protocol 协议的 Event 消息发送给 Extension Frontend。

.. image:: 2-2.png
   :align: center
   :name: 图2.2  Debug Adapter 和GDB、Extension Frontend的通信机制
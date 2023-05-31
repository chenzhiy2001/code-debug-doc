面向rCore-Tutorial操作系统的调试工具
==========================================


摘    要
--------------------------

方便的源代码级调试工具，对监测程序运行状态和理解程序的逻辑十分重要，尤其是相对复杂的内核代码以及用户态、内核态的系统调用交互；高效的 Rust 语言跟踪能力，是 Rust 操作系统内核开发的必要工具，对基于 Rust的操作系统教学和实验很有帮助。然而现有 RISC-V、Rust 实验环境搭建成本高，上手难度大，不利于初学者的内核学习与开发工作。

我们实现了一种基于 VSCode 以及云服务器的内核源代码远程调试工具：在云服务器中部署 QEMU 虚拟机并运行Rust 操作系统，通过操作系统的 eBPF 模块和 QEMU 提供 GDB 接口与用户本地的网页或安装版 VSCode 进行连接，实现远程单步断点调试能力，提供一种对用户友好的 Rust 内核代码、用户态代码以及系统调用代码的调试方法。

**关键词：QEMU；GDB；eBPF；RISC-V；操作系统调试**

Abstract
-------------------------------------------------------

A convenient source code level debugging tool is very important for monitoring the running status of the program and understanding the logic of the program, especially the relatively complex kernel code and the system call interaction between the user mode and the kernel mode; the efficient Rust language tracking ability is the Rust operating system Necessary tools for kernel development, helpful for teaching and experimenting with Rust-based operating systems. However, the existing RISC-V and Rust experimental environments are expensive to build and difficult to get started, which is not conducive to the kernel learning and development work of beginners.

We have implemented a remote debugging tool for kernel source code based on VSCode and cloud servers: deploy QEMU virtual machines in cloud servers and run Rust operating systems, and provide GDB interfaces with users' local webpages or installations through eBPF modules of the operating systems and QEMU Version of VSCode to connect to realize remote single-step breakpoint debugging capability, and provide a user-friendly debugging method for Rust kernel code, user mode code and system call code.

**Key words: QEMU; GDB; BPF; RISC-V; Operating System Debugging**


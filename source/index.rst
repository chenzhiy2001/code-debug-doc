.. rCore-Tutorial-Book-v3 documentation master file, created by
   sphinx-quickstart on Thu Oct 29 22:25:54 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

code-debug 操作系统调试器文档
==================================================

.. toctree::
   :maxdepth: 2
   :caption: 论文
   :hidden:
   
   chapter0/index
   chapter1/index
   chapter2/index
   chapter3/index
   chapter4/index
   chapter5/index


.. toctree::
   :maxdepth: 2
   :caption: 相关文档
   :hidden:

   final-lab
   appendix-a/index
   appendix-b/index
   appendix-c/index


.. toctree::
   :maxdepth: 2
   :caption: 开发注记
   :hidden:

   setup-sphinx
   rest-example
   log

欢迎来到 code-debug 文档！



.. note::

   :doc:`/log` 



项目简介
---------------------

本文档是code-debug的文档
.. chenzhiy2001 这本教程旨在一步一步展示如何 **从零开始** 用 **Rust** 语言写一个基于 **RISC-V** 架构的 **类 Unix 内核** 。值得注意的是，本项目不仅支持模拟器环境（如 Qemu/terminus 等），还支持在真实硬件平台 Kendryte K210 上运行。


导读
---------------------

请大家先阅读第零章，对于项目的开发背景和操作系统的概念有一个整体把控。
 
在正式进行实验之前，请先按照代码仓库中的说明完成环境配置，再从第一章开始阅读正文。

.. chyyuu 如果已经对 RISC-V 架构、Rust 语言和内核的知识有较多了解，第零章章末的 :doc:`/chapter0/6hardware` 提供了我们采用的真实硬件平台 Kendryte K210 的一些信息。

项目协作
----------------------

- :doc:`/setup-sphinx` 介绍了如何基于 Sphinx 框架配置文档开发环境，之后可以本地构建并渲染 html 或其他格式的文档；
- :doc:`/rest-example` 给出了目前编写文档才用的 ReStructuredText 标记语言的一些基础语法及用例；
- `项目的源代码仓库 <https://github.com/chenzhiy2001/code-debug>`_ && `文档仓库 <https://github.com/chenzhiy2001/code-debug-doc>`_
- 时间仓促，本项目还有很多不完善之处，欢迎大家积极在每一个章节的评论区留言，或者提交 Issues 或 Pull Requests，让我们一起努力让文档变得更好！

项目进度
-----------------------

- 2023-5-30：环境搭建完成，上传了论文。

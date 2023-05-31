.. _term-trap-handle:

内核态用户态的断点冲突
===========================

.. toctree::
   :hidden:
   :maxdepth: 5

要实现操作系统调试功能的关键问题在于同时设置内核态、用户态的断点，但是用户态、内核态的断点设置是冲突的。这是由于 GDB 根据内存地址设置断点，但是内核态切换到用户态时 TLB 会刷新。例如：rCore-Tutorial-v3操作系统运行在内核态时，如果令 GDB 设置用户态程序的断点，这个用户态的断点不会被触发。原因是特权级切换时执行了 risc-v 处理器的 sfence.vma 指令，使得 TLB 刷新成用户进程的页表，导致之前在内核地址空间设置的断点失效。

解决这个问题的核心思路是，缓存设置后会造成异常情况的断点，待时机合适再令 GDB 设置这些断点。在用户态运行时，缓存内核态断点；在内核态运行时，缓存用户态断点。为此，我们在 Debug Adapter 中新增了一个断点组管理模块。

断点组管理模块用一个词典缓存了用户要求设置的（包括内核态和用户态）所有断点。词典中某个元素的键是内存地址空间的代号，元素的值是这个代号对应的断点组，即这个内存地址空间里的所有断点。当任何一个断点被触发时，Debug Adapter 都会检测当前触发的这个断点属于哪个断点组。我们将这些包含了最新触发断点的断点组称为当前断点组（Current Breakpoint Group）。

.. image:: 2-8.png
   :align: center
   :name: 图2.8  断点组

当用户在 VSCode 编辑器中设置新断点时，Extension Frontend 会向Debug Adapter 发送一个请求设置断点的 Request：

.. code-block:: javascript

    onDidSendMessage: (message) => {
        if (message.command === "setBreakpoints"){
    //如果Debug Adapter设置了一个断点 
            vscode.debug.activeDebugSession?.customRequest("update");
        }
        if (message.type === "event") {
            //如果（因为断点等）停下
            if (message.event === "stopped") {
                //更新寄存器和断点信息
                vscode.debug.activeDebugSession?.customRequest("update");   
            }

Debug Adapter中的断点组管理模块会先将这个断点的信息存储在对应的断点组中，然后判断这个断点所在的断点组是不是当前断点组，如果是的话，就令 GDB 当即设置这个断点。反之，如果不是，那么这个断点暂时不会令 GDB 设置（由于API名比较冗长，我们用截图来展示这份关键代码）：

.. image:: 2-9.png
   :align: center
   :name: 图2.9  断点组缓存代码

在这样的缓存机制下，GDB 不会同时设置内核态和用户态断点，因此避免了内核态用户态的断点冲突。接下来需要一个机制，在合适的时机进行断点组的切换，保证某个断点在可能被触发之前就令 GDB 设置下去。显然，利用特权级切换的时机是理想的选择。因此，我们令Debug Adapter 自动在内核态进入用户态以及用户态返回内核态处，设置断点。我们称这两个断点为边界断点。如果边界断点被触发，就意味着特权级发生了切换，进而内存地址空间也会发生切换，因此断点组也应当切换。我们令 Debug Adapter 每次断点被触发时都检测这个断点是否是边界断点。如果是的话，先移除旧断点组中的所有断点，再设置新断点组的断点（图2.10）：

.. image:: 2-10.png
   :align: center
   :name: 图2.10  断点组切换

断点组切换的代码如下：

.. code-block:: javascript

    protected handleBreakpoint(info: MINode) {
        if (this.addressSpaces.pathToSpaceName(
            info.outOfBandRecord[0].output[3][1][4][1])
            ==='kernel'
            ){//如果是内核即将trap入用户态处的断点
                        this.addressSpaces.updateCurrentSpace('kernel');
                        this.sendEvent({ event: "inKernel" } as DebugProtocol.Event);
                        if (info.outOfBandRecord[0].output[3][1][3][1] === "src/trap/mod.rs" 
            && info.outOfBandRecord[0].output[3][1][5][1] === '135') {
                            this.sendEvent({ event: "kernelToUserBorder" }
            as DebugProtocol.Event);//发送event
                        }
                    }
                }

为了保证相关功能正常运作，断点组切换时，符号表文件也应随着断点组的切换而切换：

.. code-block:: javascript

    //extension.ts    
    else if (message.event === "kernelToUserBorder") {
        //到达内核态->用户态的边界
        // removeAllCliBreakpoints();
        vscode.window.showInformationMessage("will switched to " + userDebugFile + " breakpoints");
        vscode.debug.activeDebugSession?.customRequest("addDebugFile", {
            debugFilepath:
                os.homedir() +
                "/rCore-Tutorial-v3/user/target/riscv64gc-unknown-none-elf/release/" +
                userDebugFile,
        });
        vscode.debug.activeDebugSession?.customRequest(
            "updateCurrentSpace",
            "src/bin/" + userDebugFile + ".rs"
        );

目前，rCore-Tutorial-v3 的主线版本只支持在单核处理器上运行，因此我们没有做多核处理器的适配工作。不过，从调试工具的角度来讲，多核处理器的适配是比较简单的，只需要根据进程的CPU号进行断点组的细分即可。
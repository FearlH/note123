### gdb 调试coredump，设置符号表的位置
使用 `ulimit -c` 查看系统对于coredump文件大小的设置，如果是0的话可以使用 `ulimit -c unlimited`设置coredump文件的大小为unlimited。

`gdb executable-file core-file` 调试coredump文件。 `info sharedlibrary`查看有哪些符号被加载，哪些没有被加载。`set solib-search-path path-to-library`设置寻找符号表的路径，只需要设置到文件夹即可。不同的路径之间使用:隔开。

### ubuntu的默认coredump的位置
/var/lib/apport/coredump

### 使用lldb在remote android上进行调试
1.在NDK中找到对应平台的lldb-server，将lldb传入android
2.在启动docker或者android emulator的时候进行端口映射。docker下的端口映射 -p host:container,比如-p 1111:1112 或者 -p 31200-31200:31200-31200
3.在docker中启动lldb-server `lldb-server platform --listen "*:11111" --server --min-gdbserver-port 31200 --max-gdbserver-port 31300`。在命令中设置了 `--listen "*:11111"`表示lldb-server监听端口。在监听之后，可以启动调试的程序。但是在调试的时候，lldb-server会产生一个新的端口和host互动，所以后面需要设置`--min-gdbserver-port 31200 --max-gdbserver-port 31300`，限制新的端口的值，方便docker进行端口的映射。
4.启动lldb 执行`platform select remote-linux`选择远程平台的类型，`platform connect connect://127.0.0.1:11111`，之后可以执行`file path_to_debug`去设置文件的路径（相对于lldb中显示WorkingDir的位置，默认应该是lldb-server的位置），后面就可以执行了。

### 在vscode里面进行lldb的设置
可以使用codeLLDB插件进行设置,设置的文件类似于：
```json
{
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "lldb launch",
            "program": "program_to_debug",
            // "preLaunchTask": "lldb build task",
            "initCommands": [
                "platform select remote-linux",
                "platform connect connect://localhost:11111",
                "settings set target.inherit-env false",
                // "platform settings -w /data/local/tmp/",
                "platform status"
            ],
            "sourceMap": {"code/at/build/time/folder" : "code/at/now/folder",
                          "code/at/build/time/folder/file":"code/at/now/file"}
        },
    ]
}
```
sourceMap是编译时期的代码的位置和现在代码位置的映射。

### gdb设置观察断点
使用 `watch *<address>`设置观察点。这个观察点只会去观察对地址的写入。可以使用强制转换来改变指针的类型，从而监控不同大小的范围。比如：
`watch *(int *)<address>`设置观察int大小的内存，`watch *(char[100]*)<address>`设置观察一个范围内的内存。

同时也可以使用 `rwatch *<address>`设置读观察点，使用`awatch *<address>`设置读写观察点。这两种观察点也和上面一样可以使用强制转换来观察范围。但是这两种观察点依赖于硬件的支持（x86只有有限个（4个）寄存器用来支持硬件观察点），设置的时候可能会出现一些问题，比如不能观察太大的范围等。

### 使用valgrind结合gdb设置观察点
由于单独的GDB无法设置较大范围的硬件内存观察点，可以结合valgrind用于模拟CPU去设置硬件观察点。具体的方法是：
`valgrind --vgdb=yes --vgdb-error=0 <file to be debuged>`
其中 `--vgdb=yes` 或 `--vgdb=full` 用来确保激活 Valgrind gdbserver 。辅助命令行选项 `--vgdb-error=number` 可用于告诉 gdbserver 仅在显示指定数量的错误后才变为活动状态。因此，值为0将导致 gdbserver 在启动时变为活动状态，这允许在开始运行之前插入断点。

同时，valgrind 将给出连接的提示。
`gdb <file to be debuged>` 启动gdb。 `(gdb) target remote | vgdb`连接到valgrind上面，同时可以使用-p去指定连接的进程。如果只有一个进程的话则可以不指定。

### GDB是如如何设置断点以及调试程序的
GDB主要通过信号以及更改指令来实现程序的调试。在linux上面存在一个系统调用`ptrace()`可以用来跟踪进程的执行。通过这个系统调用GBD可以去访问被调试程序的指令空间，数据空间， 调用栈和寄存器。同时GDB可以接管目标程序的所有事件，这些信息通过信号被发送给GDB。GDB可以`fork()`出一个进程，之后子进程调用`ptrace()`来使得GDB监控自己，之后子进程可以调用`exec()`系列的函数执行待调试的程序。在attach时GDB也是通过调用`ptrace()`来监控进程的执行。

需要设置断点的时候，GDB会去查询当前断点的位置以及对应的机器码的位置，并将对应机器码上面的代码替换为`INT 3`。被调试的进程在执行到断点处的代码时就会执行`INT 3`。从而引发中断，操作系统的中断处理程序会发出信号给GDB，这时断点命中。GDB同时会调用`ptrace()`函数去恢复原来的机器码并且回退PC指针到之前的位置。在进行C/C++单步调试的时候GDB会分析下一个C/C++代码对应的机器码的位置，替换为`INT 3`进行相同的流程来调试。
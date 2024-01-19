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
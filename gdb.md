### gdb 调试coredump，设置符号表的位置
使用 `ulimit -c` 查看系统对于coredump文件大小的设置，如果是0的话可以使用 `ulimit -c unlimited`设置coredump文件的大小为unlimited。

`gdb executable-file core-file` 调试coredump文件。 `info sharedlibrary`查看有哪些符号被加载，哪些没有被加载。`set solib-search-path path-to-library`设置寻找符号表的路径，只需要设置到文件夹即可。不同的路径之间使用:隔开。

### ubuntu的默认coredump的位置
/var/lib/apport/coredump

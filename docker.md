### docker里面的pid和linux的pid的映射关系
在 `/proc/PID/status`里面可以看到(`PID`就是在linux里面的进程号)
`NSpid:  28878   2986`
其中`28878`就是在linux里面的pid,`2986`就是在docker里面的pid
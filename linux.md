### wc
wc可以统计文件信息 wc [选项] [文件]
选项常用的有 -c 文件里面的字节数，-w 文件里面的单词数, -l 文件里面的行数。

### 2>&1
0标准输入，1标准输出，2标准错误。在>后面的&表示重定向的不是一个文件而是一个文件描述符。上面就表明把标准错误2重定向到标准输出1里面。

### tee
读取标准输入然后写入一个文件，tee file.

### file
可以使用file命令查看一个文件的格式

### 修改内核的启动顺序
在加载内核动态加载库的时候 modprobe ...发现加载不了，查找之后发现uname -r输出为50的版本里面没有这个库，`uname -r` 输出为79的/lib/module里面有库。
切换内核启动版本
`grep menuentry /boot/grub/grub.cfg` 查看内核的启动顺序 <从0开始>

`sudo gedit /etc/default/grub`启动编辑器

修改GRUB_DEFAULT=0为GRUB_DEFAULT="1> <上面看到的版本顺序>"
其中 1表示启动界面的第几个选项，版本顺序表示选项中的第几号
保存

之后`sudo update-grub`

重启`sudo shutdown -r now`

### ubuntu下安装同一个软件的多个版本
```shell
update-alternatives --install <link> <name> <path> <priority>
```
\<link\>：这个命令的符号引用，如/usr/bin/gcc；

\<name\>：这个链接组的名称，如gcc；

\<path\>：这个命令对应的可执行文件的实际路径，如/usr/bin/gcc-4.8；

\<priority\>： 优先级，数值越大，优先级越高；
比如 
```shell
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.7 200
```
之后使用
```
update-alternatives --config <name>
```
切换版本
如：```sudo update-alternatives --config gcc```  在里面选择需要的版本

查看命令已注册的版本信息
```shell
update-alternatives --list <name>
update-alternatives --display <name>
```

在alternatives组里删除一项符号链接
```shell
update-alternatives --remove <name> <path>
```
比如在alternatives组里删除gcc-5这一项，输入命令：
```shell
update-alternatives --remove gcc /usr/bin/gcc-5
```

### ulimit
`ulimit -c` 设置core文件的最大值，单位blocks。  
`ulimit -d` 设置数据段的最大值，单位kbytes。  
`ulimit -f` 设置创建文件的最大值，单位blocks。  
`ulimit -l` 设置内存中锁定物理内存的最大值，单位kbytes。  
`ulimit -m` 设置可以使用的常驻内存的最大值，单位kbytes。  
`ulimit -n` 设置进程可以打开的文件描述符的最大值，单位n。  
`ulimit -p` 设置管道缓冲区的最大值，单位kbytes。  
`ulimit -s` 设置堆栈的最大值，单位kbytes。  
`ulimit -t` 设置CPU使用时间的最大上限，单位seconds。  
`ulimit -v` 设置虚拟内存的最大值，单位kbytes。  

### top 查看进程和线程
使用`top -p pid`可以查看继承的信息。之后输入H可以查看进程中的线程信息。
%CPU： CPU Usage，自上次屏幕更新以来任务占用的CPU时间份额
%MEM： Memory Usage，进程使用的物理内存百分比
COMMAND：Command Name or Command Line，用于显示输入的命令行或者程序名称
PID：Process Id，任务独立的ID，即进程ID
PPID：Parent Process Id，父进程ID
UID：User Id，任务所有者的用户ID
USER：User Name，用户名
RUSER：Real User Name，实际的用户名
TTY：Controlling Tty，控制终端名称
TIME：CPU TIME，该任务CPU总共运行的时间
TIME+：同TIME，其粒度更细
OOMa：Out of Memory Adjustment Factor，内存溢出调整机制，这个字段会被增加到当前内存溢出分数中，来决定什么任务会被杀掉，范围是-1000到+1000。
OOMs：Out of Memory Score，内存溢出分数，这个字段是用来选择当内存耗尽时杀掉的任务，范围是0到+1000。0的意思是绝不杀掉，1000的意思是总是杀掉。
S：Process Status，表示进程状态信息
    D：不可中断休眠（多为阻塞在IO上，较难在top中看到）
    I：空闲
    R：运行中
    S：休眠
    T：被任务控制信号停止
    t：在跟踪期间被调试器停止
    Z：僵尸

### 查看和添加自己的组
可以使用`groups`来查看自己的组，使用`usermod -a -G <group name> <user name>`来为用户添加组，添加之后需要重启才能生效。

### 查看进程打开的文件描述符
`lsof -a -p <pid>`
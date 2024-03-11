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
使用`top -p pid`可以查看进程的信息。之后输入H可以查看进程中的线程信息。
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

### 查看pcie的信息
`cd /sys/bus/pci/devices` 在文件夹下面找到对应的设备，查询各种信息。

### 查看io的速率
`iostat -c -d -x -t -y <device> <time1> <time2>`
-c 显示CPU信息, -d 显示设备的利用率信息, -x 显示额外的统计信息, -t 显示时间, -y 如果在给定的时间间隔内显示多条记录则忽略自系统启动以来的第一条信息。如果不加时间间隔参数那么`iostat`显示的就是从开机以来的IO统计.
device 设备, time1 每隔几秒显示一次, time2 显示的总时间.

avg-cpu:  %user   %nice %system %iowait  %steal   %idle

%user：CPU处在用户模式下的时间百分比。
%nice：CPU处在带NICE值的用户模式下的时间百分比。
%system：CPU处在系统模式下的时间百分比。
%iowait：CPU等待输入输出完成时间的百分比。
%steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比，虚拟机监视程序偷走的时间。
%idle：CPU空闲时间百分比。

其中的%iowait表示的是CPU因为等待IO而空闲的时间占比。也就是此时的CPU是空闲的时候才会统计进入%iowait。当进程等待IO从而挂起时，如果切换到另外一个进程去执行命令的话，这个时候CPU没有空闲，所以不会统计进入%iowait里面。

后面的输出内容有：
Device：/dev 目录下的磁盘（或分区）名称
tps：该设备每秒的传输次数。一次传输即一次 I/O 请求，多个逻辑请求可能会被合并为一次 I/O 请求。一次传输请求的大小是未知的
kB_read/s：每秒从磁盘读取数据大小，单位KB/s
kB_wrtn/s：每秒写入磁盘的数据的大小，单位KB/s
kB_dscd/s: 每秒磁盘的丢块数，单数KB/s
kB_read：从磁盘读出的数据总数，单位KB
kB_wrtn：写入磁盘的的数据总数，单位KB
kB_dscd: 磁盘总的丢块数量

添加-x后可以输出更加详细的内容：

读指标： - r/s：每秒向磁盘发起的读操作数 - rkB/s：每秒读K字节数 - rrqm/s：每秒这个设备相关的读取请求有多少被Merge了（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）； - %rrqm: 合并读请求占的百分比 - r_await：每个读操作平均所需的时间；不仅包括硬盘设备读操作的时间，还包括了在kernel队列中等待的时间 - rareq-sz:平均读请求大小

写指标： - w/s：每秒向磁盘发起的写操作数 - wkB/s:每秒写K字节数 - wrqm/s：每秒这个设备相关的写入请求有多少被Merge了。 - %wrqm：合并写请求占的百分比 - w_await：每个写操作平均所需的时间；不仅包括硬盘设备写操作的时间，还包括了在kernel队列中等待的时间 - wareq-sz:平均写请求大小 

抛弃指标: - d/s：每秒设备完成的抛弃请求数（合并后）。 - dkB/s：从设备中每秒抛弃的kB数量 - drqm/s: 每秒排队到设备中的合并抛弃请求的数量 - %drqm:抛弃请求在发送到设备之前已合并在一起的百分比。 - d_await: 发出要服务的设备的抛弃请求的平均时间（以毫秒为单位）。 这包括队列中的请求所花费的时间以及为请求服务所花费的时间。 - dareq-sz: 发出给设备的抛弃请求的平均大小（以千字节为单位）。

其他指标: - aqu-sz:平均请求队列长度 - %util: 一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比，向设备发出I/O请求的经过时间百分比（设备的带宽利用率）。当串行服务请求的设备的该值接近100％时，将发生设备饱和。 但是对于并行处理请求的设备（例如RAID阵列和现代SSD），此数字并不反映其性能限制。这个指标高说明IO基本上就到瓶颈了，但是低也不一定IO就不是瓶颈。一般%util大于70%,I/O压力就比较大. 同时可以结合vmstat查看查看b参数(等待资源的进程数)和wa参数(I/O等待所占用的CPU时间的百分比,高过30%时I/O压力高)

具体的信息需要根据man page去查看。

### /usr/bin/time 统计进程的执行信息
可以使用 `/usr/bin/time <command>` 统计命令的执行情况。使用 `-v`选项可以输出更加详细的信息。   
Command being timed: 执行的命令   
User time (seconds): 在用户模式中运行所消耗的CPU时间 (只统计CPU时间，sleep/等待在IO上的时间是不计算的)   
System time (seconds): 在内核模式中运行所消耗的CPU时间 (只统计CPU时间，sleep/等待在IO上的时间是不计算的)   
Percent of CPU this job got: 这个任务获得的CPU时间百分比   
Elapsed (wall clock) time: 实际过去的时间   
Average shared text size: 平均共享文本区域大小（单位：KB）   
Average unshared data size: 平均非共享数据区域大小（单位：KB）   
Average stack size: 平均栈大小（单位：KB）   
Average total size: 平均总大小（单位：KB）   
Maximum resident set size: 最大常驻集大小，即进程在内存中占用的最大物理内存空间（单位：KB）   
Major (requiring I/O) page faults: 需要进行I/O操作的主要缺页次数   
Minor (reclaiming a frame) page faults: 不需要进行I/O操作的次要缺页次数（比如一块内存交换到swap，但是并没有被污染，可以继续使用原来的页）   
Voluntary context switches: 自愿上下文切换的次数 (主要是CPU时间片到期)   
Involuntary context switches: 非自愿上下文切换的次数 （如等待IO阻塞，交出时间片）   
Swaps: 交换次数   
File system inputs: 文件系统输入操作的数量   
File system outputs: 文件系统输出操作的数量   
Socket messages sent: 发送的套接字消息数量   
Socket messages received: 接收的套接字消息数量   
Signals delivered: 交付的信号数量   
Page size (bytes): 页面大小   
Exit status: 命令退出状态   

其中的User time 和 System time统计CPU时间，如果进程有多个线程，那么在一个实际的墙上始终周期内，进程可以执行多个CPU的时钟周期。

### 使用perf(performance)进行性能检测，以及绘制火焰图
<https://github.com/brendangregg/FlameGraph> 介绍了绘制火焰图的方法。
关于perf的使用有需要注意的地方：
`/proc/sys/kernel/perf_event_paranoid`指定了perf监控的访问权限，默认值为2。

|值|含义|
| ------ | -------- |
| -1 | 允许所有用户使用（几乎）所有事件。在没有CAP_IPC_LOCK的情况下，忽略perf_event_mlock_kb之后的mlock限制 |
| > = 0 | 没有CAP_SYS_ADMIN的用户不允许ftrace函数跟踪点。禁止没有CAP_SYS_ADMIN的用户进行原始跟踪点访问 |
| > = 1 | 禁止没有CAP_SYS_ADMIN的用户访问CPU事件 |
| > = 2	| 禁止没有CAP_SYS_ADMIN的用户进行内核配置  |

可以通过编辑`/etc/sysctl.conf`在其中设置`kernel.perf_event_paranoid = <setting>`.

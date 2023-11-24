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
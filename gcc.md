### gcc -v 可以查看编译的时候的include路径

### gcc -I 添加include的搜索路径
gcc -Iinclude_path/path

### gcc -l 指定链接哪些库，但是默认寻找动态库
gcc ... --static -lsome 指定链接静态库

### 大型项目里面gcc到编译那一步也是单独文件编译的

### gcc编译生成动态库
gcc -c -o sun.o sun.c
gcc -c -o moon.o moon.c
gcc -c -o earth.o earth.c 
gcc -fPIC -shared -o libstar.so sun.o moon.o earth.o
先编译生成.o文件，之后使用命令生成动态库，PIC表示 Position Independent Code 位置无关的代码

### linux共享库的位置
Linux动态共享库的一般位置是，/lib;/usr/lib/;usr/local/lib。不会从当前目录里面寻找，可以编辑。
/etc/ld.so.conf 是配置文件。里面会使用/etc/ld.so.conf.d/下的.conf文件可以自创一个比如xxx.conf的文件里面添加共享库的位置。
sudo ldconfig 使用这个命令刷新配置。 

**注意**：以上只是添加了共享库的加载位置。在链接的时候还需要指定共享库的位置

### gcc指定连接时共享库的搜索位置
gcc -o zeus zeus.o -L/usr/local/star/lib -lstar
使用-L指定搜索位置

### gcc -WL 后面的参数
-WL 后面的参数都是传递给连接器的
-WL,-Bstatic -laaa -lbbb -lccc -WL,-Bdynamic -lddd -leee 
-Bstatic 表示后面的库是静态链接，-Bdynamic表示后面的库是动态链接

### gcc 在静态库中添加静态库
先把原来的静态库解压，之后再去添加自己编译好的文件，重新生成静态库

### gcc静态库生成动态库
也可以先把静态库解压之后把里面的内容提取生成动态库

### gcc混合使用动态库和静态库
gcc -c -o theseus.o theseus.c
gcc -o theseus theseus.o -Wl,-Bstatic -lcook -Wl,-Bdynamic -lstar -L/usr/local/star/lib

就是和上面一样使用-Wl传递信息给编译器

### gcc预处理、编译、汇编的命令
预处理 gcc -E hello.c -o hello.i 处理#include、#ifdef等
编译 gcc -S hello.i -o hello.s 编译为汇编语言
汇编 as hello.s -o hello.o 把汇编语言翻译为机器语言
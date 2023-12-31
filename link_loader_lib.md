### 南桥和北桥
高速的设备连接到北桥芯片(PCI bridge，north birdge),低速的设备先连接到南桥（south bridge）再连接到北桥和CPU，内存等高速设备通信。

### 为什么初始化的全局变量初始化的static变量放在data段，而未初始化的全局变量和未初始化的static变量放在bss段
因为已经初始化的全局变量和已经初始化的static是有值的，把这些值放在磁盘里面保存。同时虽然未初始化的全局变量和未初始化的static变量在运行的时候会加载到内存里面但是，这些变量没有必要把值存放在磁盘的里面，只需要记录总体的大小就可以了，加载的时候再在内存里面开辟空间初始化这些变量（一般初始化为0）。

### 磁盘里面保存的程序
一般就是保存代码段和数据段（数据一般就是全局数据和static数据）。其他的属于栈或者堆的数据都是在程序加载进入内存之后才会生成的。

### 强符号和弱符号，强引用弱引用
在C语言里面，函数和初始化了的全局变量是强符号，其他的是弱符号。链接的时候只能有一个同名的强符号。弱符号在链接的时候可以有多个。如果链接的时候有一个强符号和多个弱符号那么就使用强符号进行链接。否则如果只有多个弱引用那么选取弱符号里面占用空间大的那个去链接。
同时强引用是指在最终的链接时必须进行符号决议的引用，就是需要找到该符号的定义。但是弱引用就是不需要一定找到这个符号的定义。连接器会把引用设置为0或者一个特殊的值。
弱符号和弱引用方便对于库进行管理。比如可以在程序里面把一些扩展功能函数的引用设置为弱引用，这样在链接的时候如果找得到扩展功能函数的引用就加载，找不到就不加载，方便程序的裁剪。同时可以在库里面把一些符号设置为弱符号，这样方便库的使用者自己定义自己的函数等，链接的时候就会用库的使用者自己定义的符号进行链接。

### 调试信息
在elf里面的调试信息包括目标程序和原始程序之间的映射关系，目标程序里面还可能添加结构体定义函数参数等。
在Linux下可以使用strip来除去ELF文件里面的调试信息

### 链接过程的空间和地址分配
在编译阶段，每一个编译单元都会分配自己的段在段内含有空间。在链接的时候会把这些段合并起来（比如不同编译单元的.text段会被合并到最终的.test段里面）。合并之后段内的地址会被重新计算为合并后段的地址。

### 链接过程的地址重定位
在链接的时候链接器会统计不同编译单元里面的符号，形成符号表。对于需要重定位的符号会在对应的段形成一个rela的表（如rela.text）来指示段内需要重回定位的地方。之后连接器会对于需要重定位的地方根据符号表进行符号决议和重定位。

### COMMON块
COMMON可以用于对弱符号使用。多个同名的弱符号都可以加上COMMON块，在链接的时候选择占用空间最大的COMMON块进行符号决议。

### 函数级别链接
visual C/C++支持函数级别的链接，从而减少输出文件的大小。具体的是把不同的函数单独的放在不同的段里面，链接的时候只选取需要的函数（段）进行合并。

### C++重复代码消除
C++有模板、虚函数表、外部内联函数等。以模板为例，不同的编译单元里面可能同时初始化了同一套模板。这样就会导致在链接的时候同名符号有很多个，而且都占据了段中的位置（同时也就都会被加载进内存占用空间）。这时编译器会给出一个提示，指示这些重复生成的模板是可以消除的，这样链接器就可以只保留一份模板的实现。

### C++重复代码消除存在的问题
不同的编译单元使用的编译器版本，优化等级可能不同，同一个符号对应的底层代码可能不同。这样在进行重复代码消除的过程中，连接器可能就会随机选择一份符号保留，并且给出警告信息。

### 全局的构造和析构
在linux里面程序的入口一般为_start这个是glibc提供的。同时在glibc完成程序的初始化之后就会调用程序的main函数。需要在main函数之前或者之后运行的代码可以放在.init段（初始化，在main函数之前）和.fini段里面（main函数结束之后运行）

### ABI
在链接过程中，ABI不同将导致问题。ABI为二进制层面的接口主要包括：
1.内置类型(int,float，double等)的大小和在内存中的分布方式（大端小端、对齐方式等）。
2.组合类型（数组，struct，union等）储存方式和内存分布。
3.函数调用方式（调用指令、参数入栈顺序、返回值如何保存等）。
4.堆栈的分布方式（参数和局部变量在堆栈中的位置、参数的传递方法等）。
5.寄存器的使用约定（函数调用时的哪些寄存器可以修改，哪些需要保存等）。
此外C++还有继承体系的内存分布、虚函数和虚表的分布、template的初始化、指向成员函数的指针、外部符号修饰、全局对象的构造和析构、异常处理、标准库的实现细节、内嵌函数的访问等。
这些都导致不同编译单元之间不能很好的相互链接。
现在的C++ABI标准主要有Microsoft的visual C++和GNU的GCC标准。

### 可以使用连接控制脚本来控制链接的过程
包括程序的入口、段的写入规则、生成哪些段、丢弃哪些段、设置虚拟地址等。

### GNU的BFD库
抽象了不同平台的可执行文件，处理不同的目标格式。GNU汇编器、连接器调试器通过操作BFD来控制文件的输出。不同的平台只需要在BFD里面注册自己的格式就可以了。

### 可执行文件的加载
可执行文件在磁盘里面按照段的方式存储，在加载到内存中的时候不同的段按照其性质（只读、可读可写、可读可执行等）组合到一起形成segment加载到内存，从而减少内存页的分配。linux下的VMA可segment比较类似，都是组织在一起的一些段。

### GCC编译形成共享库
`gcc -fPIC -shared -o lib.so lib.c`
链接共享库使用
`gcc -o program1 program1.c ./lib.so`

### gcc编译时动态链接的处理
在最终的符号决议的时候，如果发现一个符号是定义在一个动态共享对象里里面的符号，那么就会把符号标记为动态符号。符号决议就会在动态链接的时候去做。

### 动态链接的程序文件
在静态链接的时候，程序只有一个文件。动态链接的时候程序被分为若干个文件，可执行文件和所依赖的共享对象，很多时候可以称之为模块。动态库在加载的时候操作系统会寻找一块未使用的内存加载模块。这样模块的地址是不确定的就需要进行重定位。操作系统会先将程序的控制权限交给程序的连接器，连接器进行操作之后，再把控制权交给程序main或其他初始化代码运行。

### 使用装载时重定位
使用装载时重定位可以让在加载的时候，当找到了动态库的加载位置之后，就可以根据实际的加载地址和编译时期已知的偏移量进行重定位。数据区域每个进程都有一个副本。但是这样指令也需要每个进程具有一个副本（指令在每个进程中的加载位置也是不一样的，重定位后也就是不一样的了）。数据段因为每个进程都有一个副本所以方便装载时重定位（本来内存就不能共享）。

### 地址无关的代码
因为在动态加载的时候如果使用装载时重定位，那么指令就不能在不同的进程之间共享了。可以使用让编译器生成与地址无关的指令的方法。对于同一个模块中的数据和指令，不同模块中的数据和指令有着不同的寻址方法。具体的同一个模块里面使用相对地址寻址的方法，不同的模块之间的指令和数据可以使用全局唯一表（GOT）间接寻址。当需要引用数据地址/指令地址时，可以先去GOT里面寻找数据/指令的间接地址，随后使用简介地址去寻找目标地址。连接器寻找每个变量/函数的地址去填充GOT表。

### 装载时重定位和地址无关代码编译器指令
-fPIC就是指示编译器采用地址无关的方式编译代码。只使用-shared就是使用装载时重定位的方式。

### 动态链接的延迟绑定
动态链接可以在函数被调用的时候再进行绑定，而不是程序一开始的时候。

### 动态链接过程
操作系统先读取文件头，检查合法性，读取每个segment的虚拟地址、文件地址和属性，进行加载映射到虚拟空间的相应位置。这个和加载静态文件时一样的。动态加载随后把控制交给ld动态连接器进行动态链接，随后把控制权限交给可执行程序的入口。ELF的.interp段里面保存动态链接器的位置的字符串。动态链接器是公用的，操作系统按照特定的方式把动态链接器加载到程序地址空间中。

### 动态链接的同名符号
动态链接器会把可执行文件和连接器本身的符号都合并到一个全局符号表的里面。如果有多个相同名称的符号，连接器只会把第一个遇到的同名符号加入全局符号表里面。

### 共享库和可执行文件
共享库和可执行文件除了头表标识(可执行位等)和扩展名不一样之外，其他都是一样的。linux其实不关心ELF是否可以执行，它只是按照ELF头表信息装载，之后把控制权交给ELF入口地址（动态链接器或者程序的entry）。

### 动态装载库
显示运行时链接，在程序运行的时候让程序去加载指定的模块，同时可以卸载不需要的模块。和动态链接不同，动态链接是指在程序启动之前由动态链接器负责装载和链接。动态加载库则是在程序运行时期通过动态加载器提供的API进行的加载和链接。

### 共享库和共享对象
共享对象是共享库的一个很好的存在形式，现在基本上混用这两个词语。

### 共享库常见的修改方式和对ABI的影响
| 更改类型 | 兼容性 |
| ------------------------------------- | ------------------------------------- |
| 在共享库libfoo.so中添加一个导出符号foo2 | 兼容 |
| 删除共享库libfoo.so里面一个原有的导出符号foo | 不兼容 |
| 将libfoo.so给一个导出函数添加一个参数，比如原来的foo(int a)变成了foo(int a,int b) | 不兼容 |
| 删除一个导出函数的参数,如原来的foo(int a,int b)变成了foo(int a) | 不兼容 |
| 如果一个结构类型用于一个导出函数或导出全局变量，那么改变结构体类型的长度、内容、成员类型，如libfoo.so有导出函数foo(struct bar b)，而b的结构被改变 | 不兼容 |
| 修正一个导出函数的bug，或者改进某个导出函数的性能，但是不改变导出函数的语义、功能、行为和接口类型 | 兼容 |
| 修正导出函数的bug，或者改进某个导出函数的性能，但是同时改变了导出函数的语义、功能、行为或接口类型 | 不兼容 |

导致C语言的共享库ABI的行为主要有
1.导出函数的行为发生改变，也就是调用这个函数之后产生的结果和以前不一样，不在满足旧版本规定的函数行为准则。
2.导出函数被删除
3.导出的数据结构发生变化，比如共享库定义的结构体变量和结构发生改变：结构体成员删除、顺序改变或者其他引起结构体内存布局变化的行为(不过通常来讲，往结构体的尾部添加成员不会导致不兼容，当然这个结构体必须是在共享库的内部分配的，如果是外部分配的，在分配该结构体时必须成员添加的情况)。
4.导出函数的接口发生变化，如函数返回值、参数被更改。

如果可以保证上述的四种情况不发生，那么绝大多数情况下的C语言库将会保持ABI兼容。但是破坏一个库的ABI兼容性非常容易，如不同版本的编译器，操作系统和硬件平台等。使用不同版本的编译器或者系统库可能会导致结构体的成员对齐方式不一致，从而引起ABI的变化。

同时C++更为复杂，模板、虚函数、多重继承等加大的ABI兼容的复杂性。

### linux下面的共享库的命名方式
libname.so.x.y.z 前缀lib+共享库名name+.so后缀和.x.y.z
其中X表示主版本号，不同主版本号的库之间是不兼容的。y次版本号，表示对当前版本的一次增量更新，会添加一些接口符号，且原来的符号保持不变。在主版本号相同的情况下，高次版本号的库向后兼容低此版本号的库。新版的库保留原有的接口且不改变原有接口的定义和含义。如libfoo.so.1.2.x后面升级之后新增了一个函数，版本号变为libfoo.so.1.3.x那么依赖于1.2.x的程序也可以在1.3.x上继续运行。z表示发布版本号，表示对于库的一些错误的修正、性能的改进等，并不添加任何接口，也不对接口进行修改。相同主版本号、次版本号、不同发布版本号的共享库完全兼容。

### SO-NAME和ldconfig
libfoo.so.1.2.3的SO-NAME就是libfoo.so.1也就是只保存主版本号。同时linux会在共享库的目录里面创建一个软连接名字就是共享库的SO-NAME，同时指向这个共享库。如创建软连接lib.so.1指向libfoo.so.1.2.3。这个软连接会指向同样主版本号的共享库中最新的（次版本号和发布版本号最大）那一个。因为同一个主版本号的共享库高版本的都会向后兼容低版本的，同时当文件A依赖于文件B是在.dynamic段中就会有DT_NEED的字段，这个字段如果指向B的文件名如libB.so.2.3.1，那么当共享库升级为libB.so.2.7.1时系统就必须保留全部的libB包括原始的。但是在有了SO-NAME之后，DT_NEED就可以只保留SO-NAME，因为系统里面的软连接会一直保存主版本相同的最新版本，而主版本相同的最新版本又兼容主版本相同的旧版本，这样系统就可以只保留主版本相同同时最新的的这一个版本，同时删除其他主版本相同的旧版本。ldconfig就是当安装或者更新一个共享库的时候需要运行，它会遍历所有的默认共享库目录，如/lib、/usr/lib等，然后更新软连接使其指向最新的共享库。如果安装了新的共享库，那么ldconfig就会创建新的共享库。

### 链接名
在编译器中使用共享库libXXX.so.1.2.3只需要指定-lXXX就可以了。

### 共享库系统路径
linux系统遵循FHS标准，这个标准规定了系统里面的系统文件如何存放，各个目录的组织结构和作用。共享库的位置一般为:
1./lib存放系统中最关键和最基础的共享库，如动态链接器、C语言运行时库和数学库等。这些库主要是/bin和/sbin下的程序需要使用到的库，同时也有系统启动时需要的库。
2./usr/lib主要存放非系统运行时所需要的关键性共享库，主要是开发时需要的库，一般不会被用户或者shell脚本调用到，还可能存放开发时需要的静态库目标文件等。
3./usr/local/lib主要是存放跟操作系统本身不是十分相关的库。如第三方程序，比如安装python解释器之后共享库可能被放到/usr/loacl/lib/python，而可执行文件会被放在/usr/local/bin下。GNU标准推荐的第三放程序共享库的默认安装位置就是/usr/local/lib。

### 共享库的查找
如果DT_NEED里面保存的时绝对路径，那么连接器就会从绝对路径去找。如果保存的时相对路径，那么就会从/lib、/usr/lib和由/etc/ld.so.conf配置文件指定的目录去寻找共享库。同时ldconfig可以在为这些查找路径创建缓存在Linux下是/etc/ld.so.cache，加快查找速度。

### 编译和链接的命令
两个源代码文件libfoo1.c和libfoo2.c，希望生成一个libfoo.so.1.0.1的共享库，这个共享库依赖于libbar1.so和libbar2.so可以使用如下命令行：
`gcc -shared -fPIC -Wl,-soname,libfoo.so.1 -o libfoo.so.1.0.1 libfoo1.c libfoo2.c -lbar1 -lbar2`
-shared指示生成共享库，-fPIC表示生成和位置无关的代码，-Wl表示传递给连接器的参数，这里传递-soname,libfoo.so.1指定SO-NAME。注意：如果不指定的话就不会生成SO-NAME，ldconfig更新软连接的时候，对于这个共享库也无效。

### 返回大内存的对象
一般小内存对象的返回是通过寄存器保存对象的值，之后可以把寄存器里面的值赋值给一个变量或者使用。大对象一般是在栈中留下一个临时内存tmp之后返回时如果需要赋值就再把tmp拷贝到需要赋值的里面。比如
```C
typedef struct big_thing
{
    /*

    */
}big_thing;

big_thing func()
{
    big_thing b;
    return b;
}

int main()
{
    big_thing a;
    a = func();
}
```
就会把返回值b保存到栈上的临时内存tmp中，随后把tmp拷贝给a，然后释放tmp。注意:不同的编译器可能有不同的实现。

### 堆内存
堆内存一般由程序库向操作系统申请，后面程序需要的时候由程序库分配给程序。因为系统调用开销较大，一般程序库会申请一大块内存，当程序需要堆空间的时候，由程序库分配。在程序库堆空间不够的时候才去向操作系统申请。运行库等于向操作系统“批发”了一块较大的堆内存，之后”零售“给程序，当全部"售完"或者程序有大量的内存需求时，再向操作系统“进货”。

### 向操作系统申请内存
可以使用brk()函数改变进程数据段的位置。mmap()匿名内存映射分配内存。

### 进程结束时
进程结束的时候操作系统会回收进程的资源，包括物理内存、网络链接、地址空间打开的文件等都会被操作系统回收。

### C语言的cdecl调用惯例
出栈方:函数调用者，参数传递:从右到左顺序压栈，名字修饰：下划线+函数名。另外还有其他的调用惯例。

### 变长函数
比如`int printf(const char *format,...)`，根据cdecl调用惯例，从右向左压栈，那么就可以参数format就在栈顶的位置，栈底除了其他需要保存的参数之外就是...里面的参数，就知道了参数所在的内存的位置。在使用`printf("x=%d,y=%f",x,y)`的时候，可以根据传入的字符串里面的%d和%f来判断x和y的大小，这样就可以找到传入的参数了。

### thread_local
thread_local（TLS）线程局部储存，在定义一个TLS变量之后，编译器会把这个变量放入.tls的段中，当系统启动一个新的线程的时候，就会在堆中开辟一块空间，之后把.tls段的内容复制进去（对于C++可能需要构造和后续的析构）。同时在线程中会存在一个表用于保存TLS的位置。

### linux基于int的经典系统调用
将系统调用号放在寄存器中，系统调用参数也放入寄存器中。int 0x80中断。中断后硬件自动进行内核栈和用户栈的切换，同时在内核栈中压入传参进来的寄存器的值。寻找中断处理程序，使用压入栈中的值作为参数进行系统调用。调用完成恢复用户栈和寄存器，返回用户态。





































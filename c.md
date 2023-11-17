### C语言里面的字面值
在使用一个数字字面值的时候默认这个数字会被当做int解析。比如2，但是当数字超出了int的表达范围之后，编译器会将其视为long int，如果超出long int的范围则编译器会将其视为unsigned long int，如果还不够大，编译器会将其视为long long或unsigned long long类型。一个数字后面加上L后缀表示按照long类型对待如3L，加上LL后缀表示按照long long类型对待如7LL。可以使用U表示无符号数如9ULL。还有科学计数法1.0e9,3.2e-5等默认为double类型。

### C语言int
C语言中int被认为是计算机处理整数类型时最高效的类型。

### 左值和右值
左值 object locator value
右值 value of an expression

### 指向不同大小数组的指针是不一样的
```c
int p1[3],p2[4];//p1和p2都是指向int的指针，是一样的
int pp1[3][4],pp2[3][7];//这样就是，不一样的了。
/*
不一样
因为pp1指向int[4]而pp2指向int[7]
在执行比如pp1+1或者pp2+1的时候指针移动的距离是不一样的
*/
```

### C程序的一些设计
1.在去做一个功能之前首先去检查先决条件是不是完成了，之后在去做这个功能。比如函数在传入参数的之后首先应该去判断参数是不是存在或者参数的正确性，然后再去实现功能。函数里面的子功能，子模块也是类似的。

2.可以使用多个的goto来去模仿C++的析构过程。比如：
```C
void func()
{
    int *i1=NULL,*i2=NULL,*i3=NULL;

    i1 = malloc(sizeof(int));
    if(!i1)
    {
        goto i1_end; 
    }

    i2 = malloc(sizeof(int));
    if(!i2)
    {
        goto i2_end; 
    }

    i3 = malloc(sizeof(int));
    if(!i3)
    {
        goto i3_end; 
    }

    //do something

i3_end:
    free(i2);
i2_end:
    free(i1);
i1_end:
    return;
}
```

### malloc和calloc
`void *malloc(size_t size)`和`void *calloc(size_t nmemb, size_t size)`
malloc()会申请size大小的内存，且内存不会被初始化这片内存。calloc()会申请一片数组为nmemb个size大小的内存数组。同时calloc()会探测出nmemb*size的溢出情况，但malloc()不会。

### ffmpeg中的 *(AVClass **)
```C
//ffmpeg中的一个代码
void func(void *avcl)
{
    AVClass *avc = avcl ? *(AVClass **)avcl : NULL;
}
//这里传入的avcl是一个 AVCodecContext的结构体指针
struct AVCodecContext
{
    const AVClass *av_class;
    /*
    ...
    */ 
}
AVClass *avc = avcl ? (AVCodecContext *)avcl->av_class : NULL;
AVCodecContext codec_context, *p_codec_context = &codec_context;
//AVClass *是AVCodecContext的第一个部分，那么有
AVClass *p_avclass = (AVClass *)(*p_codec_context);
//什么解引用可以得到(AVClass *)呢？
//AVClass **
//因为AVClass *是AVCodecContext的第一个部分，地址是相同的，也就是传进来的void *avcl其实就是&(AVClass *)。所以有了上面的代码。
```


### h.264
#### 1
1.在h264的裸流中是不含有timestamp的信息的，相较于封装格式（如mp4等）h264裸流缺失了信息。在实践中，ffmpeg等可以进行best effort来对于这些缺失的信息进行推断。

#### 2
2.在h.264封装中，Video Coding Layer(VCL)->Network Abstraction Layer(NAL)->Transport Layer(file or stream)。VCL主要侧重于视频的编解码。(NAL unit) NALu可以描述一个流将如何被分割为包。同时NALu还描述了包的重要性（是否含有I帧等）信息，网络设备可以通过重要性，判断一个包是否应该被丢弃。

#### 3
3.如何从流中提取NALu,标准并未规定NALu的组织方式，目前有两种主流的方式：Annex B和AVCC。

Annex B使用start code去区分一块NALu,start code可以是三字节的0x000001和4字节的0x00000001。在这种情况下流的组织方式类似于：
```
([0x000001][first NAL unit]) | ([0x000001][second NAL unit]) | ([0x000001][third NAL unit]) | ...
```
有一些串口可以方便的把数据分割为4字节，就很容易发现4字节的start code。在其他的情况下为了减少字节数量，就采用3字节的start code。同时根据香农的信息熵公式，0x000001这些字节含有的信息量是很少的，这就意味着start code在h264的码流中不容易存在（压缩算法要尽量使用少的编码包含更多的信息）。但是在码流中依然可能会存在与start code相同的字节。它们会被按照如下的方式去替换：
0x000000->0x00000300
0x000001->0x00000301
0x000002->0x00000302
0x000003->0x00000303
这些被替换之后的字节被称为emulation pervention bytes（竞争防止字节）。含有竞争防止字节（替换后）的单元称为SODB(String of Data Bits),不含竞争防止字节的单元（替换前）被称为RBSP(Raw Byte Sequence Payload)。

AVCC 使用在每个NALu之前加入一个nal_size的方式去区分流中的NALu，nal_size标识了后面的NALu的长度。这个长度(length)可能可以使用1,2或者4个字节去存储。会有一个extradata/sequence header/AVCDecoderConfigurationRecord去标识length占用的字节的大小。这种情况下的流的组织方式类似于：
```
([extradata]) | ([length] NALu) | ([length] NALu) | ...
```
AVCDecoderConfigurationRecord的结构
```CPP
aligned(8) class AVCDecoderConfigurationRecord { 
    unsigned int(8) configurationVersion = 1; 
    unsigned int(8) AVCProfileIndication; 
    unsigned int(8) profile_compatibility; 
    unsigned int(8) AVCLevelIndication; 
    bit(6) reserved = '111111'b; 
    unsigned int(2) lengthSizeMinusOne; 
    bit(3) reserved = '111'b; 
    unsigned int(5) numOfSequenceParameterSets; 
    for (i=0; i< numOfSequenceParameterSets; i++) { 
        unsigned int(16) sequenceParameterSetLength;          
        bit(8*sequenceParameterSetLength) sequenceParameterSetNALUnit; 
    } 
    unsigned int(8) numOfPictureParameterSets; 
    for (i=0; i< numOfPictureParameterSets; i++) { 
        unsigned int(16) pictureParameterSetLength; 
        bit(8*pictureParameterSetLength) pictureParameterSetNALUnit; 
    }
}
```

### digital video introduction

#### aspcet ratio
aspect ratio(长宽比)，主要有两种长宽的比例，Display Aspcet Ratio(DAR)(显示宽高比 比如视频的16:9等)，Pixel Aspect Ratio(PAR)(像素宽高比，一个像素的宽和高之比)。

#### bit rate
一秒里面视频的bit数量。bit rate = width * height * bits per pixel * frames per second。同时当比特率接近固定值的时候，被称为constant bit rate(CBR),当有大的变化幅度的时候，被称为variable bit rate（VBR）。比如当视频的一块部分为纯黑色的时候，可能可以减少比特率。同时在过去，有一种称为interlaced video的方式使用隔行扫描，同时隔行传送的方式进行视频的输出。

#### 亮度和色度
在眼睛中含有很多的视觉细胞，视觉细胞可以分为视杆细胞和视锥细胞。视杆细胞主要区分亮度，视锥细胞（三种分别区对红绿蓝敏感）主要区分色彩。我们的视杆细胞要比视锥细胞多，人们对于亮度的敏感度要高于对色度的敏感度。

#### YCbCr
Y(brightness)用于代表亮度，Cb(chorma blue),Cr(chrome red)代表两个颜色通道。根据ITU-R的建议，可以按照以下的方法去计算YCbCr。
```
Y = 0.299R + 0.587G + 0.114B
Cb = 0.564(B - Y)
Cr = 0.713(R - Y)
```
转换为RGB
```
R = Y + 1.402Cr
B = Y + 1.772Cb
G = Y - 0.344Cb - 0.714Cr
```

#### I帧，B帧和P帧
I frame(intra frame)self-contained frame. 

P(predicted) frame can be rendered using the previous frame.(using difference). 

B(bi-predictive) frame reference the last and feature frames.

#### 视频的压缩
可以使用的压缩手段有inter prediction和intra prediction。inter prediction主要是移动预测，使用将一个frame分割为很多block，被预测的frame中的block保存移动信息（这个block是从reference的frame的哪个block移动预测得到的）。intra prediction通过对于frame内部进行压缩提取信息使用局部的信息预测周围block的数据得到。在这些预测中往往保存residual来进一步压缩数据。

#### 视频的分割
视频可以被分割为很多的slice,macro和many sub-partitions。分割为较小的块的部分有利于进行移动的预测，分割成较大的块的部分有利于对背景进行表示。和视频一样，slice也可以被分为I-slice,B-slice,I-macroblock等。

### 视频编码的各种模式

在理解视频编码的各种模式之前，首先需要了解不同的编码场景。

#### 视频编码的场景
1.存档:把视频压缩之后保存到储存介质(硬盘，网盘等)。这个时候关心的是在质量足够好的情况同时，码率尽可能低。但是不关心最终的实际视频的大小。

2.流媒体/点播:使用网络储传输一个视频文件，这个时候需要关心的是视频的码率不要超过网络的带宽。同时可能需要在不同的带宽下面提供不同的码率。

3.直播流：和流媒体类似，但是需要尽快（实时）的进行编码。同时直播过程中无法预计后续的视频内容。

4.面向设备的编码：比如把视频保存到DVD或者蓝光的碟片上面，这时可能想使得最终的文件达到特定的大小。

#### 码率的控制模式

##### 固定QP (constant QP, CQP)
对于复杂和简单的画面都使用固定的量化参数，这样往往会导致根据场景复杂度的不同，比特率的波动很大。除非明确知道自己在做什么，否则不要使用这个模式。

```shell
ffmpeg -i <input> -c:v libx264 -qp 23 <output>
ffmpeg -i <input> -c:v libx265 -x265-params qp=23 <output>
```

好处：用于视频编码的研究

坏处：几乎其他所有的应用

##### 1-pass （只进行一次编码） ABR (average bitrate)
给编码器设置一个目标码率，编码器计算如何达到这个码率。在1-pass的情况下面，编码器无法得到后续帧的信息，这个时候，编码器只能猜测如何达到设置的码率，这样会导致bitrate的波动以及画质的剧烈变化。（不要使用这个模式）

```shell
ffmpeg -i <input> -c:v libx264 -b:v 1M <output>
ffmpeg -i <input> -c:v libx265 -b:v 1M <output>
ffmpeg -i <input> -c:v libvpx-vp9 -b:v 1M <output>
```

好处：快速编码

坏处：几乎其他所有的应用

##### 恒定码率 （constant bitrate CBR）
设置使得编码器在整个流中保持在恒定的码率，这样也许会使得简单的画面浪费部分码率。

```shell
ffmpeg -i <input> -c:v libx264 -x264-params "nal-hrd=cbr:force-cfr=1" -b:v 1M -minrate 1M -maxrate 1M -bufsize 2M <output>
```

好处：保持恒定的码率

坏处：文档存储，高效利用带宽的场景会浪费部分码率

##### 2-pass （进行两次编码） ABR (average bitrate)
如果允许编码器两遍（或更多）编码那么它就可以提前看到未来还没有进行编码的内容（在第一个pass）。编码器可以在第一遍编码是计算编码代价，然后在第二遍编码中更高效的利用比特。这种模式使得在特定码率下输出的质量最好。

```shell
ffmpeg -i <input> -c:v libx265 -b:v 1M -x265-params pass=1 -f null /dev/null
ffmpeg -i <input> -c:v libx265 -b:v 1M -x265-params pass=2 <output>.mp4
```

这种编码是对流进行编码的简单方法，但是有两点需要注意：最终视频的质量是不确定的，可能需要使用多次实验来确保设置的码率足够去编码复杂的内容。第二个需要注意的点是这样可能导致出现局部码率峰值。这可能导致发送的数据bitrate超出了接收端的能力。

好处：达到特定的码率；面向设备的编码

坏处：需要快速编码的情况(比如直播流)

##### 恒定质量 （constant quality, CQ）/ 恒定速率因数 （constant rate factor, CRF）
主要是可以在整个流中提供恒定的质量。只需要设置这个参数，剩下的就交给编码器去做。

```shell
ffmpeg -i <input> -c:v libx265 -crf 28 <output>
```

在x264和x265中，crf值的范围都是[0,51]。对于x264一个比较好的默认值是23。对于x265则是28。在x264下设置crf的值为18（在x265则为24）在视觉上应该是透明的（视觉上感受不到区别），其他的低于这些值的crf设置都有很大的可能浪费文件的尺寸。设置±6的crf往往会导致最终编码的文件的体积缩小为原来的一半/翻倍。

这个模式唯一的缺点是不能确定最终文件的大小以及码率的波动情况。

和2-pass的ABR相比，在同等的码率下，crf和2-psss ABR应该具有相近似的视频质量。不同点在于，2-pass ABR下，设置的是最终的大小，而crf只是去设置画质参数。

好处：归档，实现最佳的视频质量
坏处：流式传输，需要设置特定的bitrate或者文件大小的情况

##### 码率受限的编码 (Constrained Encoding,  Video Buffering Verifier VBV)

VBV提供了一个方式去限制码率在一个最大范围之内。VBV可以用在2-pass VBR（每个阶段都可以），CRF中。它可以直接被添加到以及设置好的码控模型里面。

```shell
ffmpeg -i <input> -c:v libx265 -crf 28 -x265-params vbv-maxrate=1000:vbv-bufsize=2000 <output>
```

但是对于很小的CRF值，vbv-bufsize可能并不能满足。这时可以去对CRF的值进行调节。

好处：带宽受限的流式传输，直播（和CRF1，1-pass一起使用），流媒体点播 (和target bitrate, 2-pass一起使用)

坏处: 归档

#### CRF的解析
在x264和x265中crf值的范围为[0,51]，更小的crf值可以带来更大的视频质量，但是更大的视频体积，更大的crf值则相反，会带来更差的视频质量但是更小的视频体积。同时CRF可以和VBV一起使用从而限制最大的码率以用于流式传输。设置±6的crf会导致最终编码的文件的体积缩小为原来的一半/翻倍。

CRF和CBR是相反的，前者实现的是恒定的视频质量，后者则实现的是恒定的视频码率/体积。典型的一种CRF的实现是对于相同类型的帧使用相同压缩量进行编码。也就是对于相同类型的帧丢弃等同量的信息。在技术上就是维持一个恒定的QP（CQP），QP值决定了对于每一个编码块应该丢弃的信息的量。但是这也往往会使得视频的比特率在整体视频流中具有很大的波动。

CRF在此更进一步，它使用不同的压缩率来压缩不同的帧，从而根据需要改变QP，以保持一定成都的感知质量。为了做到这一点，CRF考虑到了视频的运动。对于高运动的部分它增大QP，而对于低运动的部分CRF减小QP。这样CRF持续的在整个流中改变bitrate的消耗。通常情况下面，CRF的bit消耗一直低于CQP，同时还保持了视频的感知质量。而CQP则有可能导致bit的浪费。

通俗地说，这是因为视觉系统会被屏幕上的一切所“分散注意力”，没有足够的时间去看到更重的压缩。稍微技术一点来说，高速运动会“掩盖”压缩伪影（如块状效应）的存在。另一方面，当画面中没有太多运动时，人们会有更多时间去查看图像，并且不会有任何东西来分散注意力或掩盖任何伪影，因此希望画面被压缩得尽可能少。在低运动情况下，压缩伪影会变得更加显眼（在视觉上更明显），因此更容易分散注意力（被观察到）。

同时在感官上，CRF和CQP的压缩质量是相同的，重要的是CQP会在人们不容易注意到的细节上面浪费bit。

具有高运动以及复杂细节的帧难以被压缩，具有低运动以及简单细节帧则容易被压缩。这里的难以压缩和容易压缩是指在相同的bit消耗下容易压缩的帧可以获得比难以压缩的帧更好的质量。

CRF的一个问题是，不发预计在某个CRF设置之下的流的大小。相同的CRF设置去编码不同的流也往往会获得不同大小的结果。


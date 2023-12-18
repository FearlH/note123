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
I frame(intra frame)self-contained frame. P(predicted) frame can be rendered using the previous frame.(using difference). B(bi-predictive) frame reference the last and feature frames.

#### 视频的压缩
可以使用的压缩手段有inter prediction和intra prediction。inter prediction主要是移动预测，使用将一个frame分割为很多block，被预测的frame中的block保存移动信息（这个block是从reference的frame的哪个block移动预测得到的）。intra prediction通过对于frame内部进行压缩提取信息使用局部的信息预测周围block的数据得到。在这些预测中往往保存residual来进一步压缩数据。

#### 视频的分割
视频可以被分割为很多的slice,macro和many sub-partitions。分割为较小的块的部分有利于进行移动的预测，分割成较大的块的部分有利于对背景进行表示。和视频一样，slice也可以被分为I-slice,B-slice,I-macroblock等。
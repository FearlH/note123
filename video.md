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

AVCC
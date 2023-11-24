### h.264
1.在h264的裸流中是不含有timestamp的信息的，相较于封装格式（如mp4等）h264裸流缺失了信息。在实践中，ffmpeg等可以进行best effort来对于这些缺失的信息进行推断。
2.在h.264封装中，Video Coding Layer(VCL)->Network Abstraction Layer(NAL)->Transport Layer(file or stream)。
### ffmpeg命令
```shell
ffmpeg的命令主要分为5个部分
$ ffmpeg \
-y \ # global options
-c:a libfdk_aac \ # input options
-i bunny_1080p_60fps.mp4 \ # input url
-c:v libvpx-vp9 -c:a libvorbis \ # output options
bunny_1080p_60fps_vp9.webm # output url
```

### ffplay播放网络流
```shell
ffplay -protocol_whitelist "file,udp,rtp,tcp" -i tcp://172.24.117.211:12345
```

### ffmpeg推流
```shell
ffmpeg -re -i juren.flv -c:v copy -f mpegts udp://172.24.117.211:12345 
```
### shell 脚本里面执行命令

可以使用 `eval "$command"`去执行命令，eval会去解析字符串，之后执行字符串代表的命令。其中的一个例子：
```shell
#!/bin/bash

j=$1

read command

for((i=0;i<j;i++))
do
        eval "$command" &
done

sleep 45

killall ffmpeg

sleep 5

echo killall
```
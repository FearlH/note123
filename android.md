### 编译选项
WITH_DEXPREOPT=false 取消预先dex2oad，内存不够的系统容易编译不通过。

### AOSP下的build/envsetup.sh
`source ./build/envsetup.sh`会为当前用户shell配置环境变量，不同的用户shell如果需要使用的话都需要首先执行这个命令

### lspci
`lspci -nn`显示系统中的pci设备
01:00.0 xxxxxx [1d82:0401]
其中前面的数字01:00.0表示pci标记，后面的数字1d82表示vendor号，0401表示设备号

### adb forward 端口映射
`adb forward tcp:20001 tcp:20002`
把android设备上的20002端口映射到主机上的20001端口

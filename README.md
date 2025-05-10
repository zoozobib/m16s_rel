# m16s_rel
天猫魔盒m16s Wi-Fi 相关 驱动 armbian 

1、驱动目前支持到 5.10 5.4 下编译成功
2、6.1版本驱动未测试
3、chainloader 启动内核 uboot(native) - > uboot(new)
4、chainloader方式启动的最新版 armbian kernel 6.1+ 无法直接 armbian-update -k 5.10 ，此操作更新内核等相关文件成功，但启动会无法找到ROOFS报UUID错误，目前还不知道原因，等有机会检索
5、跨平台编译驱动理论上没问题，但内核文件难找，因此最后采用在生产环境上编译，j8 很快

origin	https://github.com/ophub/amlogic-s9xxx-armbian.git
sudo ./recompile -k 5.10.237 
将内核copy 到 板子上 解压进入目录
sudo armbian-update 等待更新完成，重启


【跨平台】
wget https://raw.githubusercontent.com/ophub/amlogic-s9xxx-armbian/refs/heads/main/compile-kernel/tools/config/config-5.10

make mrproper ARCH=arm64 CROSS_COMPILE=/home/zoozobib/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
make ARCH=arm64 CROSS_COMPILE=/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu- prepare -j8
make ARCH=arm64 CROSS_COMPILE=/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu- modules_prepare -j8
make EXTRA_CFLAGS="-w -I/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/aarch64-none-linux-gnu/libc/usr/include/linux/" CROSS_COMPILE=/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu- -f Makefile.x86

【生产】
make EXTRA_CFLAGS="-w" CROSS_COMPILE= -f Makefile.x86 -j8

调整Makefile.x86
1、target 调整 arm64
2、sdio
3、kernel src

【部署】
7668_firmware 复制到 /lib/firmware ; /usr/lib/firmware 
复制ko模块到 /usr/lib/firmware/drivers/wireless/mediatek 等相关路径
depend -a 
/etc/modules 增加模块

sudo modprobe cfg80211
sudo modprobe wlan_mt76x8
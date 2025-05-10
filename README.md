# m16s_rel
天猫魔盒m16s Wi-Fi 相关 驱动 armbian 

1、驱动目前支持到 5.10 5.4 下编译成功
2、6.1版本驱动未测试
3、chainloader 启动内核 uboot(native) - > uboot(new)
4、chainloader方式启动的最新版 armbian kernel 6.1+ 无法直接 armbian-update -k 5.10 ，此操作更新内核等相关文件成功，但启动会无法找到ROOFS报UUID错误，目前还不知道原因，等有机会检索
5、跨平台编译驱动理论上没问题，但内核文件难找，因此最后采用在生产环境上编译，j8 很快

make mrproper ARCH=arm64 CROSS_COMPILE=/home/zoozobib/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
make ARCH=arm64 CROSS_COMPILE=/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu- prepare -j8
make ARCH=arm64 CROSS_COMPILE=/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu- modules_prepare -j8
make EXTRA_CFLAGS="-w -I/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/aarch64-none-linux-gnu/libc/usr/include/linux/" CROSS_COMPILE=/home/xxx/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu- -f Makefile.x86


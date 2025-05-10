# 天猫魔盒M16S Armbian MT7668 WiFi驱动编译与部署指南 (m16s_rel)

本文档记录了为天猫魔盒M16S在Armbian系统上编译和部署MT7668 WiFi驱动的过程和注意事项。

## 驱动支持情况

1. 驱动目前在内核版本 **5.10** 和 **5.4** 下编译成功。

2. 内核 **6.1** 版本的驱动支持情况**未测试**。

## 启动方式与内核更新问题

* **启动链**: `uboot (native) -> uboot (new) -> chainloader 启动内核`

* **内核更新限制**: 通过chainloader方式启动的最新版Armbian (内核6.1+) **无法直接通过 `armbian-update -k 5.10` 降级内核**。此操作虽然会成功更新内核及相关文件，但在重启后会导致系统**无法找到ROOTFS并报告UUID错误**。具体原因尚不明确，有待后续研究。

## 驱动编译

### 编译环境准备

* **源码来源**: `https://github.com/ophub/amlogic-s9xxx-armbian.git`

* **切换到指定内核版本 (示例)**:

  ```bash
  sudo ./recompile -k 5.10.237
  ```

* 将编译好的内核源码复制到目标板子上，解压并进入目录。

* 在板子上运行 `sudo armbian-update`，等待更新完成，然后重启。

### 编译方式

#### 1. 跨平台编译 (Cross-Compilation)

理论上可行，但寻找匹配的内核头文件较为困难，因此最终推荐在目标设备（生产环境）上直接编译。

**步骤示例：**

1. **获取内核配置文件**:

   ```bash
   wget [https://raw.githubusercontent.com/ophub/amlogic-s9xxx-armbian/refs/heads/main/compile-kernel/tools/config/config-5.10](https://raw.githubusercontent.com/ophub/amlogic-s9xxx-armbian/refs/heads/main/compile-kernel/tools/config/config-5.10)
   ```

2. **清理源码树**:

   ```bash
   make mrproper ARCH=arm64 CROSS_COMPILE=/home/zoozobib/Downloads/m16s/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
   ```

3. **准备内核头文件**:

   ```bash
   make ARCH=arm64 CROSS_COMPILE=/path/to/your/toolchain/bin/aarch64-none-linux-gnu- prepare -j8
   make ARCH=arm64 CROSS_COMPILE=/path/to/your/toolchain/bin/aarch64-none-linux-gnu- modules_prepare -j8
   ```

   *(请将 `/path/to/your/toolchain/` 替换为你的实际工具链路径)*

4. **编译驱动模块**:

   ```bash
   make EXTRA_CFLAGS="-w -I/path/to/your/toolchain/aarch64-none-linux-gnu/libc/usr/include/linux/" \
        CROSS_COMPILE=/path/to/your/toolchain/bin/aarch64-none-linux-gnu- \
        -f Makefile.x86
   ```

   *(请将 `/path/to/your/toolchain/` 替换为你的实际工具链路径)*

#### 2. 在目标设备上编译 (生产环境编译 - 推荐)

这种方式因为环境一致性高，编译速度也很快（例如在J8核心设备上）。

**编译命令示例：**

```bash
make EXTRA_CFLAGS="-w" CROSS_COMPILE= -f Makefile.x86 -j8
```

*(`CROSS_COMPILE=` 为空表示使用本地编译器)*

### Makefile.x86 调整

在编译驱动前，需要对 `Makefile.x86` (或其他驱动使用的Makefile) 进行以下调整：

1. **目标架构 (Target)**: 调整为 `arm64`。

2. **接口类型 (Interface)**: 确保配置为 `sdio`。

3. **内核源码路径 (Kernel Source)**: 指向正确的内核源码树路径。

## 驱动部署

1. **固件复制**:

   * 将 `7668_firmware` (MT7668的固件文件) 复制到以下两个路径：

     * `/lib/firmware/`

     * `/usr/lib/firmware/` (有些系统可能使用此路径)

   * (通常固件文件会放在 `/lib/firmware/mediatek/` 目录下，请根据驱动实际需求确认)

2. **内核模块 (.ko 文件) 复制**:

   * 将编译生成的 `.ko` 模块文件复制到内核模块的标准路径下，例如：

     * `/lib/modules/$(uname -r)/kernel/drivers/wireless/mediatek/` (具体子路径可能因驱动而异)

     * 或者一个通用的路径如 `/usr/lib/modules/$(uname -r)/extra/`

3. **更新模块依赖**:

   ```bash
   sudo depmod -a
   ```

4. **开机自动加载模块 (可选)**:

   * 编辑 `/etc/modules` (或 `/etc/modules-load.d/your-wifi.conf`) 文件，增加需要加载的模块名。

   * 例如，添加 `wlan_mt76x8_sdio` (根据你实际编译出的模块名)。

## 加载模块测试

1. **加载依赖模块 (如果需要)**:

   ```bash
   sudo modprobe cfg80211
   ```

2. **加载WiFi驱动模块**:

   ```bash
   sudo modprobe wlan_mt76x8_sdio # 或者你编译出的实际模块名，例如 wlan_mt76x8
   
# FreeRTOS Milk-V Duo S 测试报告

## 测试环境

### 操作系统信息

- 构建系统：Ubuntu 24.04 LTS x86_64
- 系统版本：Duo-V1.1.4
- 下载链接：https://github.com/milkv-duo/duo-buildroot-sdk/releases
- 参考安装文档：https://github.com/milkv-duo/duo-buildroot-sdk
    - FreeRTOS: https://milkv.io/zh/docs/duo/getting-started/rtoscore

### 硬件信息

- Milk-V Duo S (512M, SG2000)
- USB 电源适配器一个
- USB-A to C 或 USB C to C 线缆一条，用于给开发板供电
- microSD 卡一张
- USB 读卡器一个
- USB to UART 调试器一个（如：CP2102, FT2232 等，注意不可使用 CH340/341 系列，会出现乱码）
- 杜邦线三根

## 安装步骤

### 构建 mailbox-test 二进制

拉取 duo-examples 仓到本地并构建。

```shell
sudo apt install -y wget git make
git clone https://github.com/milkv-duo/duo-examples --depth=1
cd duo-examples
source envsetup.sh
cd mailbox-test
make
```

### 使用 `ruyi` CLI 刷写镜像到 microSD 卡

安装 [`ruyi`](https://github.com/ruyisdk/ruyi) 包管理器，运行 `ruyi device provision` 并按提示操作。

#### 将构建出的二进制复制进 microSD 卡

```shell
sudo mount /dev/sdX2 /mnt
cp ~/duo-examples/mailbox-test/mailbox_test /mnt/root/
sudo umount /mnt
sudo eject /dev/sdX2
```

至此，存储卡准备完成。插入开发板，准备启动。

### 登录系统

通过串口登录系统。

## 预期结果

系统正常启动，通过板载串口登录后运行 `mailbox_test` 二进制，板载蓝色 LED 灯先亮后灭。

（待机状态为蓝色 LED 闪烁）

## 实际结果

系统启动正常，并能顺利通过板载串口登录。

`mailbox_test` 测试程序可以成功执行，LED 灯的行为符合预期。

### 启动信息

```log
[root@milkv-duo]~#
[root@milkv-duo]~# ls
mailbox_test
[root@milkv-duo]~# ./mailbox_test
[root@milkv-duo]~# find / -name *rtos*
find: /proc/355: No such file or directory
/sys/class/misc/cvi-rtos-cmdqu
/sys/class/cvi-rtos-cmdqu
/sys/devices/platform/1900000.rtos_cmdqu
/sys/devices/virtual/misc/cvi-rtos-cmdqu
/sys/bus/platform/devices/1900000.rtos_cmdqu
/sys/bus/platform/drivers/cvi-rtos-cmdqu
/sys/bus/platform/drivers/cvi-rtos-cmdqu/1900000.rtos_cmdqu
/sys/firmware/devicetree/base/rtos_cmdqu
/sys/firmware/devicetree/base/reserved-memory/rtos_dram@0x9fe00000
/dev/cvi-rtos-cmdqu
[root@milkv-duo]~#
```

屏幕录像：

[![asciicast](https://asciinema.org/a/3JlGE9DBVemEnRqIffzKhNKcT.svg)](https://asciinema.org/a/3JlGE9DBVemEnRqIffzKhNKcT)

## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

测试成功。

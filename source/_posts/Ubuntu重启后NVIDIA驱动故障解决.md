---
title: Ubuntu重启后NVIDIA驱动故障解决
date: 2025-3-28 15:07:12
tags: 
    - NVIDIA驱动
    - 服务器
    - 显卡
---

## 问题描述

服务器重启后，图形界面频繁黑屏/闪退，同时命令行输入`nvidia-smi`报错无法连接到驱动。

由于网络配置问题，重启后默认采用已失效的旧配置，导致联网失败。

## 解决过程

1. 解决图形化界面和联网问题

由于显卡驱动故障，电脑可能只能调用CPU加载图形界面，黑屏可能是操作过快导致的。因此，在开机/重启后一段时间内不要进行任何操作，以降低CPU负担。一段时间后再操作电脑，点击`Ctrl+Alt+F1`和`Ctrl+Alt+F2`切换图形界面/命令行终端，就有机会可以顺利进入图形界面操作。

进入图形界面后需要修改网络设置，将已失效的旧配置文件删除，默认启用新配置文件（在图形界面的设置(Settings)中）。

2. NVIDIA驱动修复

DeepSeek指出通常是系统内核与驱动版本不匹配导致的，给出了如下建议：
>  在Linux Ubuntu系统中，重启后出现`nvidia-smi`提示无法连接到NVIDIA驱动的问题，通常是由于系统内核更新导致驱动与内核版本不匹配引起的。以下是综合多篇搜索结果整理的具体解决方案：
> 
> ---
> 
> ### **方法一：通过DKMS重新生成驱动模块**
> 1. **安装DKMS工具**  
>    DKMS（Dynamic Kernel Module Support）能够自动为内核外的驱动生成适配模块，避免因内核更新导致驱动失效。  
>    ```bash
>    sudo apt install dkms
>    ```
> 
> 2. **查看当前NVIDIA驱动版本**  
>    ```bash
>    ls /usr/src | grep nvidia  # 例如输出可能为nvidia-470.74
>    ```
> 
> 3. **通过DKMS重新安装驱动模块**  
>    使用查到的驱动版本号（如470.74）执行以下命令：  
>    ```bash
>    sudo dkms install -m nvidia -v 470.74
>    sudo reboot
>    ```
>    - 完成后通过`dkms status`确认模块状态。
> 
> ---
> 
> ### **方法二：切换回旧内核版本**
> 如果内核自动更新后驱动失效，可手动切换回旧内核：  
> 1. **查看可用内核版本**  
>    ```bash
>    grep menuentry /boot/grub/grub.cfg  # 列出所有内核选项
>    uname -r  # 查看当前内核版本
>    ```
> 
> 2. **修改GRUB默认启动项**  
>    - 编辑GRUB配置文件：  
>      ```bash
>      sudo vim /etc/default/grub
>      ```
>    - 修改`GRUB_DEFAULT`的值，例如设置为旧内核的路径（如`"1>3"`表示选择第2个主菜单项下的第4个子项）：  
>      ```
>      GRUB_DEFAULT="1>3"
>      ```
>    - 更新GRUB并重启：  
>      ```bash
>      sudo update-grub
>      sudo reboot
>      ```
> 
> ---
> 
> ### **方法三：禁止内核自动更新**
> 为避免后续内核更新导致驱动失效，可锁定内核版本：  
> 1. **禁用自动更新**  
>    - 修改配置文件`/etc/apt/apt.conf.d/10periodic`和`/etc/apt/apt.conf.d/20auto-upgrades`，将所有数值设为`0`以关闭自动更新。
>    - 在“Software & Updates”中关闭自动更新选项。
> 
> 2. **锁定当前内核**  
>    ```bash
>    sudo apt-mark hold linux-image-<当前内核版本>  # 例如linux-image-5.4.0-60-generic
>    ```
> 
> ---
> 
> ### **方法四：重新安装NVIDIA驱动**
> 若上述方法无效，可尝试彻底重装驱动：  
> 1. **卸载旧驱动**  
>    ```bash
>    sudo apt purge nvidia*
>    sudo apt autoremove
>    ```
> 
> 2. **安装匹配的驱动版本**  
>    - 推荐使用官方驱动或`ubuntu-drivers`自动安装适配版本：  
>      ```bash
>      sudo ubuntu-drivers autoinstall  # 自动安装兼容驱动
>      ```
>    - 或手动下载驱动安装包，指定内核源码路径（需安装`linux-headers-$(uname -r)`）：  
>      ```bash
>      sudo ./NVIDIA-Linux-x86_64-<驱动版本>.run --kernel-source-path=/usr/src/linux-headers-$(uname -r)
>      ```
> 
> ---
> 
> ### **附加建议**
> - **检查依赖环境**  
>   若DKMS报错（如`gcc版本不兼容`），需切换GCC版本：  
>   ```bash
>   sudo update-alternatives --config gcc  # 选择兼容版本
>   ```
> - **启用持久化模式**  
>   解决`nvidia-smi`响应慢的问题：  
>   ```bash
>   sudo nvidia-persistenced --persistence-mode
>   ```
> 
> ---
> 
> ### **总结**
> - **优先使用DKMS**：适合驱动版本与内核兼容但未正确加载的情况。  
> - **切换内核或锁定版本**：适合长期维护稳定性的场景。  
> - **重装驱动**：作为兜底方案，但需注意CUDA等依赖的兼容性。  
> 
> 如果问题反复出现，建议结合禁用内核更新和DKMS工具，以彻底解决问题。

运行`dkms`修复失败，因此选择了重新安装驱动的方法：
``` shell
# 添加官方显卡驱动PPA
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update

# 查看推荐驱动版本
ubuntu-drivers devices

# 安装推荐版本（例如535版本）
sudo apt install nvidia-driver-535

# 重启生效
sudo reboot
```
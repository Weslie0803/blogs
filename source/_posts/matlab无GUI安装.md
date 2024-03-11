---
title: matlab无GUI安装
date: 2024-3-11 15:32:12
tags: matlab; 服务器; 安装;
---

## 问题描述

由于需要在服务器中安装matlab，但无法启动服务器的图形界面，尝试RDP连接等方式无果后，决定采用命令行安装。

## 安装过程

从网站上获取Matlab R2023b Linux安装包（R2023b_Linux.iso）

挂载到系统中：
``` bash
sudo mount -r ./R2023b_Linux.iso ./installer 
cd ./installer 
ls
```

挂载目录中即出现安装文件。其中`install`为安装脚本，`installer_input.txt`为安装配置文件。

设置访问权限：
``` bash
sudo chmod 777 ./installer_input.txt
```

用vim打开安装配置文件
``` bash
vim ./installer_input.txt
```

结合配置文件中的提示，修改配置参数，主要包括安装位置、安装文件密钥、需要安装的软件包内容等。

配置完成后运行安装脚本：
``` bash
sudo ./install -inputFile ./installer_input.txt
```

如没有在安装配置文件中另外设置，Linux系统默认的安装日志文件应位于`/tmp/mathworks_root.log`路径。可打开文件查看安装结果。
``` bash
vim /tmp/mathworks_root.log
```

完成安装后，环境变量可能没有被自动添加，可运行`echo $PATH`查看，或直接运行`matlab`命令查看。

手动添加环境变量：
- 临时添加：
``` bash
export PATH=$PATH:/usr/local/MATLAB/R2023b/bin
```

- 为当前用户永久添加：
``` bash
vim ~/.bashrc # 在文件最后一行加入export PATH=$PATH:/usr/local/MATLAB/R2023b/bin
source ~/.bashrc
```

## 注意事项
1. 安装文件有.iso文件和.zip文件，前者体积较大（约10GB），后者较小（约200MB）。先前使用.zip文件解压后安装失败，经mathworks官网论坛问答结果提示，改用.iso文件进行安装。

2. 如未使用sudo安装，日志文件的文件名应为`/tmp/mathworks_{username}.log`

3. 以上安装步骤未包含激活
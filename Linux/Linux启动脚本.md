# 嵌入式系统启动分析

## 设备升级

### 网页端升级

### 串口升级

### 通过某种demo升级

## BootLoader
在嵌入式系统中，控制启动流程的核心是引导加载程序（`Bootloader`）当设备通电或复位时，硬件首先执行存储在非易失性存储器（如ROM或Flash）中的引导加载程序。引导加载程序负责初始化硬件环境，加载和启动内核。

常见的`BootLoader`，比如`U-BOOT`，

`BootLoader`和`U-BOOT`之间的关系就好像是类和实例的关系。

## 内核（Kernal）

内核随后接管控制，执行初始化过程，包括加载和执行启动脚本（`initrun.sh`），启动脚本由`init`进程或者`systemd`服务管理器执行。

## 启动脚本(initrun.sh)

桌面版的启动脚本`initrun.sh`（或者其他名字）的存放和编写方式由具体的Linux发行版和系统配置。有可能放置在以下位置

    /etc/init.d/
    /etc/rc.local/                      
    /etc/systemd/system/                #对于使用Systemd的系统，启动脚本通道存放在系统的这个目录下
    /etc/init/

而在嵌入式系统，启动脚本可能被放置在一些自定义路径中，在定制化的启动脚本中，就会做如下配置：

- 设置环境变量
- 设置OOM阈值等
- 配置网络接口
- 定制启动各个进程app


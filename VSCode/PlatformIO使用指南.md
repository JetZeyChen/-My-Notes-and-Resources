# PlatformIO IDE使用指南

## PlatformIO VSCode 扩展安装

​	在VSCode扩展里面搜索PlatformIO，点击"install"即可。由于该源码为与Github服务器，所以国内本地的链接会比较慢，开VPN的情况下可能有一定的改善，等待安装完成即可。PlatformIO支持Windows以及Linux系统，此次我们只关注Windows系统下的使用方法。

## PlatformIO GUI界面介绍

### Home

​	主页面下可以点击Quick Access快速访问下的4个选项，分别是：

* New Project - 新建工程

  >点击新建工程会跳出一个对话框，对话框会有4个选项提供选择：
  >
  >1. 工程名称
  >2. 目标开发板/芯片
  >3. 开发平台/框架
  >4. 工程文件夹位置

* Import Arduino Project - 导入Arduino工程

* Open Project - 打开已有工程

* Project Examples - 工程示例

### Project

​	在工程界面下，你可以搜索已经安装的嵌入式平台的模板示例工程，或者查看已经建立的工程。

### Inspect

​	在检视界面下，可以查看某个工程已经相应的开发板环境下，对内存的占用以及代码检查功能。

### Libraries

​	库界面，可以搜索已经在PlatformIO平台注册登录的第三方库，根据相应的需求安装第三方库。

### Boards

​	在开发板界面可以查看PlatformIO支持的各种开发板的ID，平台以及框架，和MCU的型号、频率、ROM和RAM的大小。

### Platforms

​	在平台界面可以安装一些常见的嵌入式开发平台，或者查看已经安装的嵌入式平台。

### Devices

​	设备界面查看端口已经链接的设备状态。



## PlatformIO 工程文件树

​	PlatformIO工程的文件树结构如下图1-1所示：

![image-20250703120327366](D:\Desktop\Github_folder\-My-Notes-and-Resources\Images and  Shortcut Resources\image-20250703120327366.png)

​									图1-1 PlatformIO文件树结构图

在该工作区下存在一个工程，工程名称叫做“JokerMemory”。工程文件夹包含以下几个子目录

* .pio - 该文件夹是PlatformIO生产的文件夹，里面包含了工程相关的配置文件
* .vscode - 该文件夹是VSCode自动生产的文件夹，里面包含了编译器以及相关配置文件
* include - 该文件夹是自动生成给用户存放头文件的文件夹
* lib - 该文件夹是自动生成给用户存放依赖库的文件夹
* src - 该文件夹是自动生产给用户存放源文件的
* test - 该文件夹是自动生产给用户存放测试代码的
* .gitignore - 该文件是git版本管理相关文件，不需要改动
* paltformio.ini - 该文件是platformio工程的初始化配置文件，用户需要根据实际的开发需求变更内部环境参数标志。

### 工程build的关键

​	配置好paltformio.ini 文件十分关键，该文件的配置选项以及方法，在官方文档有详细的说明：https://docs.platformio.org/page/projectconf.html。

​	该文件在每一次变更都会同步更新到platformio相关的配置文件中，对当前的工程环境进行变更认证。只有配置好相关的.ini文件才能够确保正确的构建工程，进行相关的嵌入式开发工作。
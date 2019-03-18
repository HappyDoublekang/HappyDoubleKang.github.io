---
title: win10专业版安装git bash闪退问题终极解决方案
date: 2019-03-18 14:59:55
tags: [Git]
---

### 问题描述

Win10 64位专业版安装git 2.x之后出现 Git闪退,安装1.x出现bash: /dev/null: No such device or address fatal: open /dev/null or dup failed: No such file or directory 错误。

<!-- more -->

### 背景描述

由于换了新系统(Win10 64专业版)，需要重新安装Git,于是去官网下了Git的最新版本，安装完之后，发现不能用，一点开Git bash 就退出了，不知道怎么回事。我以前win10 家庭版也是官网下的最新版本，可以正常使用。于是，我初步断定是操作系统的原因，问了身边的同事，他们也都是win10,但是安装Git的时候没有出现类似的问题，很顺利的安装成功，但他们貌似都不是Win10专业版，都是什么家庭版，旗舰版。于是我去网上寻找答案，大家都知道网上的答案五花八门，很多是针对win7的，针对win10 的很少，且有的答案按照其说的做了仍然不能解决问题，下面我将分享我解决问题的 过程。

### 问题解决过程描述

* 网上有答案说是C:/Window/System32/drivers/null.sysnull.sys 这个系统文件损坏，于是我从同事那里拷贝一个过来，覆盖之，重启。没有解决问题

* 以管理员身份运行CMD，在CMD下输入 sfc /scannow 进行系统扫描修复。我的安装100%重启后问题依旧， 如果此过程中扫中途时候出现了如下的错误 

![图片上传失败...](https://raw.githubusercontent.com/HappyDoublekang/image/master/cmd.png)

* 解决步骤二中的错误 

第一步：在联网情况下，按按(Windows+X)+A，也就是在powershell命令提示符中输入

```c
DISM.exe /Online /Cleanup-image /Scanhealth    //按回车键。
DISM.exe /Online /Cleanup-image /Restorehealth //按回车键。
```

完成后请重启电脑。 

上面的貌似也没解决我的问题。在此分享下终极解决办法，继续查找原因：

Windows 上也有 /dev/null？？？？Google 一圈后发现确实有，是用一个系统服务模拟的：

在 windows/system32/cmd.exe 右键管理员方式运行：

```c
C:\Users\Administrator>sc query null           //按回车键。
```

![图片上传失败...](https://github.com/HappyDoublekang/image/blob/master/cmd1.png?raw=true)

手动启动该服务报错：

```c
C:\Users\Administrator>sc start null           //按回车键。
[SC] StartService 失败 577:

Windows 无法验证此文件的数字签名。某软件或硬件最近有所更改，可能安装了签名错误或损毁的文件，或者安装的文件可能是来路不明的恶意软件。
```

C:\Windows\System32\drivers\null.sys 从其他系统上拷贝一个过来覆盖，再启动 null 服务就正常了：

如何确定null.sys是否正常，很简单。实行如下命令：

```c
C:\Users\Administrator>sc start null

SERVICE_NAME: null
        TYPE               : 1  KERNEL_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 0
        FLAGS              :
```

如果你一下子找不到可用的 null.sys，可以试试我这个（for Windows10 64位）。

如果sc start null 启动成功。OK，问题解决。

再次右键git bash here ，没有闪退了。皆大欢喜。这就是用盗版系统的悲剧。自己给自己挖的坑。

最后附上我的null的地址

链接: https://pan.baidu.com/s/1ohcwtFmsPF-JHgNEmL4PIw 提取码: jy17





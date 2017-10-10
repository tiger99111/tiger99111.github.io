---
layout: post
title: "Linux ALSA声卡驱动之一:ALSA架构简介"
date: 2017-10-10
description: "Linux ALSA声卡驱动之一:ALSA架构简介"
tag: Linux ALSA声卡驱动 
---   

**声明：**本博内容均由http://blog.csdn.net/droidphone原创

## **一. 概述**
&emsp;&emsp;**ALSA**是Advanced Linux Sound Architecture 的缩写，目前已经成为了linux的主流音频体系结构，想了解更多的关于ALSA的这一开源项目的信息和知识，请查看以下网址：http://www.alsa-project.org/。

&emsp;&emsp;在内核设备驱动层，ALSA提供了alsa-driver，同时在应用层，ALSA为我们提供了alsa-lib，应用程序只要调用alsa-lib提供的API，即可以完成对底层音频硬件的控制。 
![alsa的软件体系结构](http://img.blog.csdn.net/20171010133231003?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVGlnZXI5OTExMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

&emsp;&emsp;由上图可以看出，用户空间的alsa-lib对应用程序提供统一的API接口，这样可以隐藏了驱动层的实现细节，简化了应用程序的实现难度。内核空间中，alsa-soc其实是对alsa-driver的进一步封装，他针对嵌入式设备提供了一些列增强的功能。
&emsp;&emsp;本系列博文仅对嵌入式系统中的alsa-driver和alsa-soc进行讨论。 
   
## **二. ALSA设备文件结构**
  我们从alsa在linux中的设备文件结构开始我们的alsa之旅. 看看我的电脑中的alsa驱动的设备文件结构: 

```
$ cd /dev/snd 
$ ls -l 
crw-rw----+ 1 root audio 116, 8 2011-02-23 21:38 controlC0 
crw-rw----+ 1 root audio 116, 4 2011-02-23 21:38 midiC0D0 
crw-rw----+ 1 root audio 116, 7 2011-02-23 21:39 pcmC0D0c 
crw-rw----+ 1 root audio 116, 6 2011-02-23 21:56 pcmC0D0p 
crw-rw----+ 1 root audio 116, 5 2011-02-23 21:38 pcmC0D1p 
crw-rw----+ 1 root audio 116, 3 2011-02-23 21:38 seq 
crw-rw----+ 1 root audio 116, 2 2011-02-23 21:38 timer 
$
```

&emsp;&emsp;我们可以看到以下设备文件:   
&emsp;&emsp;&emsp;&emsp;midiC0D0 --> 用于播放midi音频   
&emsp;&emsp;&emsp;&emsp;pcmC0D0c --〉 用于录音的pcm设备   
&emsp;&emsp;&emsp;&emsp;pcmC0D0p --〉 用于播放的pcm设备   
&emsp;&emsp;&emsp;&emsp;seq --〉 音序器   
&emsp;&emsp;&emsp;&emsp;timer --〉 定时器   
&emsp;&emsp;其中，C0D0代表的是声卡0中的设备0，pcmC0D0c最后一个c代表capture，pcmC0D0p最后一个p代表playback，这些都是alsa-driver中的命名规则。   
&emsp;&emsp;从上面的列表可以看出，我的声卡下挂了6个设备，根据声卡的实际能力，驱动实际上可以挂上更多种类的设备，在include/sound/core.h中，定义了以下设备类型： 

```
#define SNDRV_DEV_CONTROL ((__force snd_device_type_t) 1) 
#define SNDRV_DEV_LOWLEVEL_PRE ((__force snd_device_type_t) 2) 
#define SNDRV_DEV_LOWLEVEL_NORMAL ((__force snd_device_type_t) 0x1000)
#define SNDRV_DEV_PCM ((__force snd_device_type_t) 0x1001) 
#define SNDRV_DEV_RAWMIDI ((__force snd_device_type_t) 0x1002) 
#define SNDRV_DEV_TIMER ((__force snd_device_type_t) 0x1003) 
#define SNDRV_DEV_SEQUENCER ((__force snd_device_type_t) 0x1004)
#define SNDRV_DEV_HWDEP ((__force snd_device_type_t) 0x1005) 
#define SNDRV_DEV_INFO ((__force snd_device_type_t) 0x1006) 
#define SNDRV_DEV_BUS ((__force snd_device_type_t) 0x1007) 
#define SNDRV_DEV_CODEC ((__force snd_device_type_t) 0x1008) 
#define SNDRV_DEV_JACK ((__force snd_device_type_t) 0x1009) 
#define SNDRV_DEV_LOWLEVEL ((__force snd_device_type_t) 0x2000) 
```
　　通常，我们更关心的是pcm和control这两种设备。 
  
## **三. 驱动的代码文件结构**
```
sound/
├── core　　　//该目录包含了ALSA驱动的中间层，它是整个ALSA驱动的核心部分
│   ├── oss　　// 包含模拟旧的OSS架构的PCM和Mixer模块
│   └── seq　　//有关音序器相关的代码
│       └── oss
├── drivers　　//放置一些与CPU、BUS架构无关的公用代码
├── firewire
├── i2c　　　　//ALSA自己的I2C控制代码
│   └── other
├── isa　　　　//isa声卡的顶层目录，子目录包含各种isa声卡的代码
│   ├── ad1816a
│   ├── galaxy
│   ├── ......
│   └── wss
├── mips
├── oss
│   └── dmasound
├── parisc
├── pci　　　　//pci声卡的顶层目录，子目录包含各种pci声卡的代码
│   ├── ac97
│   ├── ......
│   └── ymfpci
├── pcmcia
│   ├── pdaudiocf
│   └── vx
├── ppc
├── sh
├── soc　　　　//针对system-on-chip体系的中间层代码
│   ├── codecs　　//针对soc体系的各种codec的代码，与平台无关
│   ├── fsl
│   ├── samsung
│   ├── ...
│   └── ux500
├── sparc
├── spi
├── synth
│   └── emux
└── usb
    ├── 6fire
	├── caiaq
	├── misc
	└── usx2y 
```

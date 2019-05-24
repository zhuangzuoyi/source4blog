---
title: 表情包世界读卡器
date: 2019-05-24 22:59:43
tags: [ESP32]
categories: ESP32 NFC 
---
# 1
&nbsp;&nbsp;有一类表情包很流行，（这里应该有配图，不过，怕侵权，没了解过这些表情包图片能不能用在个人文章中）这类表情包都会有这么一个格式的文字：滴，XXX卡，有一天，突然想到，可以做个类似的东西玩玩。              
&nbsp;&nbsp;硬件方面，语音模块首先想到的是[TTS](https://baike.baidu.com/item/TTS/3512737)模块，这类模块应该是很成熟，某宝上很多，使用起来也很简单，直接把想播放的语音的文字通过串口发送到这里模块就行了。             
&nbsp;&nbsp;后来想起来手上有一块ESP32Lyrat音频开发板，也许可以用，就不用再买TTS模块（舍不得钱），看了下乐鑫提供的[音频开发框架（adf）](https://github.com/espressif/esp-adf)中有播放mp3的例程，只要做出相应语言的mp3文件就行了，            
&nbsp;&nbsp;因为我用过阿里云，知道阿里云有提供tts服务，实验了下，可以用阿里云的tts服务来生成mp3文件。                 
&nbsp;&nbsp;读卡模块，首先想到的是手上有的RC522模块，手上的RC522模块是spi接口的，不过，esp32 lytea板子可用的IO太少了，最后选用了可以用串口通讯的PN532模块，
总体思路是程序中先保存几个卡片的UID，然后把PN532读取卡片的UID跟已近保存到程序中的UID比较，如果读取到的卡片的UID存在程序中的UID则返回一个参数，然后根据这个参数播放不同的mp3文件。原理上很简单。
# 2，硬件
* 读卡模块：PN532 NFC 模块
![image](http://halin.oss-cn-shanghai.aliyuncs.com/bqbsjduq/PN532.jpg)

* ESP32开发板：ESP32-LyraT 4.2
![image](http://halin.oss-cn-shanghai.aliyuncs.com/bqbsjduq/lyrat.jpg)

# 3、开发环境:windown7
1. 编译器
参考：[https://docs.espressif.com/projects/esp-idf/zh_CN/stable/get-started/windows-setup.html](https://docs.espressif.com/projects/esp-idf/zh_CN/stable/get-started/windows-setup.html)

    下载esp32_win32_msys2_environment_and_toolchain，解压即可，
2. SDK
下载音频开发框架（adf）:参考：[https://github.com/espressif/esp-adf](https://github.com/espressif/esp-adf)
> git clone --recursive https://github.com/espressif/esp-adf.git

# 4、使用adf中的工程
这里使用esp-adf/examples/player/pipeline_sdcard_mp3工程,打开编译工具，进入到该目录，输入命令**make menuconfig**进行配置，这里主要配置如下几项：

1、选择相应的板子的型号，我这里是ESP32-LyraT 4.2，配置入下：
![](http://halin.oss-cn-shanghai.aliyuncs.com/bqbsjduq/board.PNG)

2、然后设置串口、Flash参数，如下图：
![](http://halin.oss-cn-shanghai.aliyuncs.com/bqbsjduq/port.PNG)


配置完之后，使用命令**make flash**进行烧录，默认的工程是播放SD卡中的**test.mp3**文件，把有**test.mp3**文件的SD卡插入板子上的SD卡卡座，重新开机，就可以播放音乐了

# 5、使用阿里云TTS服务生成MP3文件
1、TTS在阿里云里面，是[智能语音交互](https://ai.aliyun.com/nls)中的[语音合成](https://ai.aliyun.com/nls/tts?spm=5176.12061031.1228726.4.44883cb4tPtjyl)的功能，里面的描述是：语音合成服务，通过先进的深度学习技术，将文本转换成自然流畅的语音。文本转换成语音就是TTS。要使用该服务，首先需要个阿里云账号，然后开通该服务并创建一个项目，完成后，在控制台中可以找到**AccessToken**跟项目**Appkey**。如下图，这两个参数在生成mp3文件时需要用到。

![](http://halin.oss-cn-shanghai.aliyuncs.com/bqbsjduq/aliyun_tts.png)

2、阿里云提供的语音合成的RESTful API 有两个（详情查看[文档](https://help.aliyun.com/document_detail/94737.html?spm=a2c4g.11186623.6.593.68b25275DRasFX)）,我使用了其中的get方法，该方法的url格式为：
> https://nls-gateway.cn-shanghai.aliyuncs.com/stream/v1/tts?appkey=${您的appkey}&token=${您的token}&text=${想要合成的文本}&format=mp3&sample_rate=16000

把上面的${您的appkey}、${您的token}、${想要合成的文本}替换成你的就可以了，我一开始使用用chrome一个http调试插件来生成文件的，后来发现了个更简单的方法，就是直接用浏览器打开该URL，然后可以播放该音频，如果满足效果的话，另存为即可，既可以验证也可以下载，一举两得。

# 6、PN532驱动
由于这里只需要读取卡片的UID，所以只需用到PN532两个命令（不包括唤醒命令）：
1.设置卡片为读卡器模式，并且是读14443的卡（因为我手上的是M1卡）
> 0x00,0xff,0x03,0xfd,0xd4,0x14,0x01,0x17,x00


2. 然后是寻卡指令，可以获得卡片的UID
> 0x00,0xff,0x04,0xfc,0xd4,0x4a,0x02,0x00,0xe0,0x00


# 7、
整个做下了也还是很简单的，自己写的代码主要是PN532的驱动，播放音频部分乐鑫已近做了只要拿来用就行，全部代码及音频文件都在[meme_reader](https://github.com/zhuangzuoyi/Embedded-coding/tree/master/esp32/meme_reader)
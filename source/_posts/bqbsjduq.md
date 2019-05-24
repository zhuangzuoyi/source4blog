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

# 2，硬件
* 读卡模块：PN532 NFC 模块

* ESP32开发板：ESP32-LyraT 4.2
![image](https://docs.espressif.com/projects/esp-adf/en/latest/_images/esp32-lyrat-v4.2-layout.jpg)

# 3、开发环境:windown7
1. 编译器
参考：[https://docs.espressif.com/projects/esp-idf/zh_CN/stable/get-started/windows-setup.html](https://docs.espressif.com/projects/esp-idf/zh_CN/stable/get-started/windows-setup.html)

    下载esp32_win32_msys2_environment_and_toolchain，解压即可，
2. SDK
下载音频开发框架（adf）:参考：[https://github.com/espressif/esp-adf](https://github.com/espressif/esp-adf)
> git clone --recursive https://github.com/espressif/esp-adf.git

# 4、使用adf中的工程
这里使用esp-adf/examples/player/pipeline_sdcard_mp3工程：
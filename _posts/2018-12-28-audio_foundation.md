---
layout: post
title:  "Audio foundation"
date:   2018-12-28 16:47:53
categories: audio
---
### 基础知识

#### 编译windows
 
1.生成VS项目文件 

set DEPOT_TOOLS_WIN_TOOLCHAIN=0 

set GYP_GENERATORS=msvs-ninja,ninja 

set GYP_MSVS_VERSION=2015 (这里是2013会出现问题，生成的文件缺失很多)

生成VS2013项目文件(推荐使用) 

gn gen out/Default –ide=vs2013 

生成VS2015项目文件 

gn gen out/Default –ide=vs2015 

2.编译调试 

以VS2013为例，用VS2013打开all.sln 

将webrtc项目下example下的peerconnection_client设为启动项



#### audio基本概念介绍

https://blog.csdn.net/pds574834424/article/details/78174097

#### audio开发，依赖的系统库(winsdk or directx sdk)

https://stackoverflow.com/questions/39748072/linking-native-webrtc-application-with-visual-studio

#### PCM

https://blog.csdn.net/wsyabcde/article/details/79359938

#### vad

https://en.wikipedia.org/wiki/Voice_activity_detection


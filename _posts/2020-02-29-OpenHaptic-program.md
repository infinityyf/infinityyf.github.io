---
layout:     post
title:      OpenHaptic 编程教程
subtitle:   力觉渲染教程
date:       2020-02-29
author:     infinityyf
header-img: img/nvidia.jpg
catalog: true
tags:
    - OpenHaptic
---
# HLAPI / HDAPI 
HLAPI 用于上层的力反馈渲染，使用和OpenGL相似。HDAPI则偏向底层编程，HLAPI建立在HDAPI基础之上。  
HDAPI必须通过HHD获取力反馈设备handle，在HLAPI中使用HHLRC _(haptic rendering context)_ 。使用`hdEnable()/hdDisable()` 打开或关闭HDAPI中的特性。  
使用`hdEndFrame()`将计算结果提交到力反馈设备上。
![](../img/OpenHaptic/OpenHaptics_Overview.PNG)  

# use HDAPI
使用HDAPI的基本步骤是：
1. 初始化设备。  
2. 创建scheduler callback _（计算位置和力）_  
3. 使设备输出力  
4. 运行scheduler  
5. clean up!

![](../img/OpenHaptic/HDAPI_program.PNG)
```c
HHD hHD = hdInitDevice(HD_DEFAULT_DEVICE); //部分设备需要手动矫正
hdEnable(HD_FORCE_OUTPUT);  //使其可以输出力
hdStartScheduler(); // 对于多设备每个设备都要启动

hdMakeCurrentDevice(hHD); //切换到当前设备

//使用hdBeginFrame()和hdEndFrame()定义设备状态绑定的scope

//定义call back
HDCallbackCode HDCALLBACK DeviceStateCallback(void *pUserData);
//在这个方法中获取设备的信息，例如位置和力

HDCallbackCode HDCALLBACK DeviceStateCallback(void *pUserData)
{
    DeviceDisplayState *pDisplayState =static_cast<DeviceDisplayState *>(pUserData);
    hdGetDoublev(HD_CURRENT_POSITION,
    pDisplayState->position);
    hdGetDoublev(HD_CURRENT_FORCE,
    pDisplayState->force);
    // execute this only once
    return HD_CALLBACK_DONE;
}


```
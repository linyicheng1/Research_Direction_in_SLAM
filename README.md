# 视觉SLAM中的研究点总结 (更新中)


本文为个人学习、阅读文献以及工程项目实践中对于SLAM算法目前存在的问题和一些可行的论文创新点的总结。对于其中的内容有不当之处欢迎在Issues中进行讨论。

> **目录**
> 1. 概述
> 2. 针对视觉SLAM系统各模块的研究
>     1. 信息采集——新式传感器/多传感器
>     2. 信息提取——特征提取
>     3. 信息关联——特征匹配
>     4. 信息处理（位姿）—— 基于关键帧结构的后端优化
>     5. 信息处理（地图）—— 地图构建
>     6. 端到端的多任务处理
> 3. 视觉SLAM相关的其他任务
>     1. 先构建视觉特征地图后定位的视觉定位算法

## News 

- 2023.5.15 更新了[手工直线特征提取](https://github.com/linyicheng1/Research_Direction_in_SLAM/blob/main/theme/%E7%9B%B4%E7%BA%BF%E7%89%B9%E5%BE%81%E6%8F%90%E5%8F%96.md)的综述

## 概述

视觉SLAM算法目前遇到的困境在于理论的成熟度与实际的落地应用情况的不匹配。在经典的ORB-SLAM2,ORB-SLAM3等解决方案面前似乎SLAM是一个已经被解决的问题。但是在众多的移动机器人以及各种应用如VR，AR中却迟迟没有落地，这是由于其在鲁棒性和精度等方面依旧纯在明显的短板。另一方面，近年来越来越多的研究者加入这一领域，却发现前辈们的工作理论复杂度高、工程实现难、算法改进更是无从谈起，因此跟随书籍、课程学习一段相关基础知识便陷入无从下手的困境。因此，本文抛砖引玉，提出一些SLAM中可以去深入挖掘的一些点，给研究者提供一个思路。本文会随着个人的学习、实践中的成长，逐步修改添加其中的一些问题。也欢迎各位加入共同完善这一工作。

对下文中提出的问题，会给出一个简单的描述，然后给出一个简单的创新点，最后凭借个人主观论断给出一个难度评估。部分问题会在一个单独的文档中给出详细的相关文献甚至领域发展脉络为进一步的研究提供依据，并将链接附在表格最后一列。

## 信息采集——新式传感器/多传感器

目标：**在各种极端环境下采集到尽量多的数据**

关键字：**互补**

信息采集上需要考虑目前常用的视觉传感器的不足之处，采用可以补充这一部分信息的传感器并融合或者单独使用在特定环境下对现有算法进行补充。

如：
1. 视觉 + IMU
    1. 单目视觉具有尺度不确定性，添加IMU中的加速度计补充尺度信息
    2. 低级视觉特征对视角不变性差，因此添加IMU中的陀螺仪信息,提高特征关联的效率
2. RGB相机 + 红外相机
    1. RGB 相机对光照变化敏感，在黑夜环境中无法使用，因此添加红外相机进行互补
3. 鱼眼相机/360°相机
    1. 传统相机视角小，快速运动下若有几帧数据没有及时处理视角转到另一个角度后必然导致跟踪失败。而360°相机或者尽量大广角的相机则可以更好的避免这一点。

### 单一传感器

#### 鱼眼镜头
鱼眼镜头的优势在于单一传感器即可获取超大视野，一般可以超过180°。但是缺点也很明显，即畸变严重、标定复杂。

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|鱼眼镜头标定|提出新相机模型替代针孔模型|考虑鱼眼镜头结构提出更快、表达能力强的相机模型|:star::star::star::star:|
|鱼眼镜头标定|现有模型下提出标定方案|鱼眼相机畸变严重，传统标定特征失效|:star::star::star:|
|鱼眼畸变下的特征提取|畸变导致图像变化，如直线特征等特征提取困难|基于鱼眼图像的特征提取算法|:star::star::star:|

#### 卷帘相机
SLAM算法中常用的是全局曝光相机，这种相机同时采集得到所有图像内所有点的像素信息，因此没有果冻效应。但是卷帘相机有如下的优势：
1. 动态范围大，卷帘相机的动态范围远远大于相同价位的全局曝光相机，在高动态光照场景下具有巨大的优势
2. 应用普遍，在手机、相机等大部分的消费电子设备中采用的均为卷帘相机。
3. 图像质量更高

缺点也是很明显的，即**果冻效应**

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|卷帘相机应用|提出果冻效应补偿方案|考虑相机果冻效应模型，对SLAM算法中的特征进行位姿补偿|:star::star::star:|
|卷帘相机应用|提出果冻效应补偿方案|考虑相机果冻效应模型，加入到优化项中同时优化|:star::star::star::star:|

#### 360相机
基于目前商用的360相机做SLAM算法研究。
|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|360相机中的特征|考虑特殊相机畸变导致特征提取性能下降|考虑特殊的相机模型，对图像特征进行提取|:star::star::star::star:|
|360相机中的多视图几何|提出经典问题在360相机中的求解方法|考虑相机模型，重新推导传统的多视图几何问题|:star::star::star::star:|

#### 相机参数的主动控制

> 相机在突然进入高光强环境或者突然进入很暗的场景中，会出现纯白、纯黑的图像。
> 相机在黑暗环境下会产生运动模糊，使得SLAM算法失效

因此传感器本身的参数设置对于图像质量的影响也是至关重要,通常包括**曝光时间**和**相机增益**两个参数。

曝光时间：黑暗环境下增加进光量，但会导致图像产生运动模糊。

相机增益：对图像信号进行硬件放大，会同时放大噪声导致图像质量下降。

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|相机参数的主动控制|提出控制方法获取曝光合适、无模糊的图像|考虑SLAM运动场景，预测控制相机参数获得更优图像|:star::star::star:|

#### 事件相机
事件相机是一个新型传感器，（TODO ）

### 多传感器融合

#### VIO （视觉惯性里程计）

VIO 是最为经典的多传感器组合，目前研究较为全面且完善。

#### 多相机的全景相机

多个相机朝着不同方向排布，共同获取得到全景视野的信息。
|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|离线外参标定问题|提出高效高精度的外参标定算法|外参标定算法流程|:star::star::star:|
|在线外参标定问题|将外参添加到BA优化中在线更新|考虑外参变化的多目SLAM算法|:star::star::star::star:|
|高效特征提取匹配|多相机带来前端处理数据加大|提出特征处理加速方法|:star::star::star::star:|

#### RGB相机 + 红外相机


### 其他

#### 主动光源

模拟人类视觉方案，在前置灯光条件下，对SLAM算法在夜间的性能进行改进与提高。

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|主动光源模型|光源模型标定|提出光场分布模型的标定算法|:star::star::star::star:|
|主动光源外参|光源位置与相机外参标定|提出光场分布与相机位置关系的标定算法|:star::star::star::star:|
|主动光源SLAM|考虑光场模型的SLAM算法|将光场分布先验应用到特征匹配中|:star::star::star::star:|
|主动光源SLAM|兼顾高效且低照度性能的特征匹配算法|新环境下也能很好工作的匹配算法|:star::star::star::star:|

## 信息提取——特征提取

目标：**在既定计算资源提取包含尽量多且精确的信息**

关键字：**效率**

特征匹配中的直接法可以认为直接把原始像素作为特征，这是最为底层的信息。而点特征、直线特征和语义特征等则是包含的语义信息量逐步增加。
其中低级特征的对比如表

|方法|优势|劣势|
|---- | ---- | ---- |
|低级特征|1. 精度高 2. 精度高|1. 对视角、光照等变化不鲁棒 2.关联难度大，非常依赖稳定的相机运动|
|高级特征|1. 视角、光照等变化更鲁棒 2. 在无先验运动信息情况下匹配稳定，能提高系统整体稳定性|1. 精度略低 2. 消耗大量计算资源|

### 直接法
直接法是基于最原始的图像像素信息的方法，目前较为成熟。

### 点特征

点特征与深度学习结合已经被广泛应用在SfM等任务中，但是在SLAM中的应用还未成为主流。这是由于基于学习的特征点在如下几个方面存在不足：

1. 计算量大，特征点提取的计算量大大增加，难以在移动端的CPU上实时运行。
2. 精度不高，特征点提取的精度不如传统的特征点提取算法。

由于SLAM系统运行在图像序列中，因此前后帧的信息、地图构建得到的深度信息等都可以作为先验信息。极大的降低了匹配的难度，因此现有方法基本属于性能过剩，实时性不足的状态。

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|高效学习特征提取|CPU下的实时特征|针对图像序列进行训练，考虑实时性，降低网络复杂度|:star::star::star:|
|精确学习特征提取|提高特征位置精度|用局部少量的信息确定特征点位置，并用周围较多的信息进行描述|:star::star::star:|
|大雾/水下环境的特征提取|提高特征对环境的鲁棒性|大雾环境下的特征点得分快速降低，现有方法不能自适应提取|:star::star::star:|

### 直线特征

直线特征提取的传统方法前几年被广泛的添加到各种点特征的SLAM中，并在无纹理、结构化环境中取得了更优的效果。

### 语义特征

语义特征是指在图像中具有语义信息的特征，如车辆、行人、建筑等。这些特征在SLAM中的应用可以帮助系统更好的理解环境，从而提高系统的鲁棒性，是目前的热点之一，也是未来SLAM发展的方向。

## 信息关联——特征匹配

目标：**在不同的匹配阶段，得到足够的特征关联信息**

关键字：**多阶段需求**

特征匹配在SLAM中分为多次进行，并且有的只针对部分关键帧执行。分为如下几个阶段：
1. 帧间匹配，连续两帧图像之间的匹配
2. 关键帧和当前帧匹配
3. 关键帧和关键帧之间的匹配
4. 关键帧和回环帧之间的匹配

|匹配类型|执行频率|实时性需求|性能需求|
|---- | ---- | ---- | ---- |
|帧间匹配|高|高|低|
|关键帧和当前帧|一般|一般|较高|
|关键帧和关键帧|较高|一般|一般|
|关键帧和回环帧|低|较低|较高|

### 稀疏光流法
光流法很好的利用了相邻帧之间的信息，能够在小运动序列中以很低的成本获得很好的效果。但是随着图像序列难度的加大，光流法的性能会急剧下降。因此如何提高光流法对光照、大视角变化的鲁棒性是一个值得研究的问题。

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|光流法改进|提高光流法的鲁棒性|修正灰度不变性假设，提出新的光流法，可以从学习和传统的方法考虑|:star::star::star::star:|

### 性能递增的特征匹配

对于前端连续两帧之间的匹配，关键帧和当前帧，关键帧之间以及回环帧与关键帧之间的匹配需求是不一样的。因此如何根据不同的匹配需求采用计算量尽可能低的匹配方法是一个值得研究的问题。

|研究问题|动机|创新点|难度|详细信息|
|---- | ---- | ---- | ---- | ---- |
|特征匹配|提高匹配效率|在SLAM系统的不同阶段，采用不同的匹配方法，保证整体的计算量最小|:star::star::star::star::star:|


## 信息处理（位姿）—— 基于关键帧结构的后端优化

TODO

## 信息处理（地图）—— 地图构建

### nerf 在地图构建中的应用



## 端到端的多任务处理

### 端到端的提取与匹配任务

### 端到端的VO算法


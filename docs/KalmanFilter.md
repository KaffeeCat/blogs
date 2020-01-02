## 卡尔曼滤波（Kalman Filter）目标状态跟踪算法
### 背景
机器视觉中，视频跟踪是一个非常有意思的课题，要让计算机在视频中，对某种特定物体进行追踪，如下图所示：
![](http://5b0988e595225.cdn.sohucs.com/q_70,c_zoom,w_640/images/20181230/f2d382534c9b428eaa0156f1f87df7d7.gif)
从该动图中，我们引出两个问题：<br>
1. 如何对兔子进行跟踪？ <br>
2. 卡尔曼滤波在跟踪中的作用？<br>

### 如何对兔子进行跟踪
视频是由连续帧图像组成的（你也可以认为视频是连环画，一般每秒钟播放30张）。在这个短短的视频中，我们使用**物体检测**技术，来检测出每帧图像中的兔子，得到每一时刻兔子的状态。假设我们只关心兔子的中心坐标<img src="https://latex.codecogs.com/svg.latex?\Large&space;（x,y）" />，那么跟踪算法就是将这一连续的<img src="https://latex.codecogs.com/svg.latex?\Large&space;（x,y）" />关联起来，形成轨迹。

### 卡尔曼滤波在跟踪中的作用
因为**物体检测**并步完美，存在一定的错误，可能会将狼检测成兔子，也有可能物体遮挡，找不到兔子了，那么这个时候，大名鼎鼎的卡尔曼滤波就登场了
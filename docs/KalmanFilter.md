## 卡尔曼滤波（Kalman Filter）目标状态跟踪算法
### 1. 算法背景
视频跟踪是一个非常有意思的方向，在视频中对某种物体进行追踪，如下图：<br>
![](http://5b0988e595225.cdn.sohucs.com/q_70,c_zoom,w_640/images/20181230/f2d382534c9b428eaa0156f1f87df7d7.gif)
<br>
跟踪算法持续的识别出兔子，实时绘制物体标注框。由此，我们引出两个问题：<br>
1. 如何对兔子进行跟踪？ <br>
2. 卡尔曼滤波在跟踪中的作用？<br>

### 2. 如何对兔子进行跟踪
视频是由连续帧图像组成的（可以认为视频是连环画，一般每秒钟播放30张）。在这个短短的视频中，使用**图像物体检测**技术，检测出每帧图像中的兔子坐标。假设我们只关心兔子的中心坐标<img src="https://latex.codecogs.com/svg.latex?\Large&space;（x,y）" />，那么跟踪算法就是将这一连续的<img src="https://latex.codecogs.com/svg.latex?\Large&space;（x,y）" />关联起来，形成轨迹。

### 3. 卡尔曼滤波在跟踪中的作用
由于**物体检测**并非完美，存在一定的错误，可能会将狼检测成兔子，也有可能物体遮挡，找不到兔子了，那么这个时候，大名鼎鼎的卡尔曼滤波就登场了。

#### 3.1 状态预测
假设追踪的兔子，其状态仅与位置、速度相关，其k时刻的最佳状态估计（期望）为：

<img src="https://latex.codecogs.com/svg.latex?\Large&space;
\widehat{x}_k=
\left[\begin{matrix} 
position \\ velocity 
\end{matrix}\right]
=\left[\begin{matrix} p_k \\ v_k 
\end{matrix}\right]" />

状态信息符合分高斯分布，其k时刻的协方差矩阵为<img src="https://latex.codecogs.com/svg.latex?\Large&space; P_k " />：

<img src="https://latex.codecogs.com/svg.latex?\Large&space;
P_k=
\left[\begin{matrix} 
\Sigma_{pp} & \Sigma_{pv} \\
\Sigma_{vp} & \Sigma_{vv} \\ 
\end{matrix}\right]" />

假设从k-1时刻，到k时刻，状态变化是线性的，通过上一时刻状态的期望和分布为
## 卡尔曼滤波目标状态跟踪算法
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

<img src="https://latex.codecogs.com/png.latex?
\widehat{x}_k=
\left[\begin{matrix} 
position \\ velocity 
\end{matrix}\right]
=\left[\begin{matrix} p_k \\ v_k 
\end{matrix}\right]" />

状态信息符合分高斯分布，其k时刻的协方差矩阵为<img src="https://latex.codecogs.com/svg.latex?\Large&space; P_k " />：

<img src="https://latex.codecogs.com/png.latex?
P_k=
\left[\begin{matrix} 
\Sigma_{pp} & \Sigma_{pv} \\
\Sigma_{vp} & \Sigma_{vv} \\ 
\end{matrix}\right]" />

假设从k-1时刻，到k时刻，状态变化是线性的，上一时刻状态的期望和分布，通过状态映射矩阵<img src="https://latex.codecogs.com/svg.latex?\Large&space; F_x" />进行预测，当前时刻的期望和分布分别是：

<img src="https://latex.codecogs.com/png.latex?
\widehat{x}_k=F_x\widehat{x}_{k-1}
" />

<img src="https://latex.codecogs.com/png.latex?
P_x=F_kP_{k-1}F_k^T
" />

#### 3.2 增加外部影响及不确定性（过程噪音）
一些外部因素会对系统产生影响，例如运动加速度，用<img src="https://latex.codecogs.com/svg.latex?\Large&space; B_k " />：来表达控制矩阵，<img src="https://latex.codecogs.com/svg.latex?\Large&space; \vec{u}_k " />来表达控制向量，纳入预测系统进行修正：

<img src="https://latex.codecogs.com/png.latex?
\widehat{x}_k=F_x\widehat{x}_{k-1}+B_k\vec{u}_k
" />

以矩阵形式表达运动状态估计系统为：

<img src="https://latex.codecogs.com/png.latex?
\left[\begin{matrix}
position_{k} \\
velocity_{k}
\end{matrix}\right]
=\left[\begin{matrix}
1 & \Delta t \\
0 & 1
\end{matrix}\right]
\left[\begin{matrix}
position_{k-1} \\
velocity_{k-1}
\end{matrix}\right]
+\left[\begin{matrix}
\frac{\Delta t^2}{2} \\ \Delta t
\end{matrix}\right]
a
" />

另外，在追踪过程中，也会遇到一些不确定因素，例如上坡、转弯等，预测状态可能会因此收到偏移，因此，需要在预测过程中，增加不确定因素，称之为**过程噪音**，亦符合高斯分布，协方差矩阵为<img src="https://latex.codecogs.com/svg.latex?\Large&space; Q_k" />，下一时刻的状态估计分布协方差矩阵，加入**过程噪音**后，表达为：

<img src="https://latex.codecogs.com/png.latex?
P_k=F_kP_{k-1}F_k^T+Q_k
" />

#### 3.3 预测和观测信息相互结合，得到当前时刻的最佳估计，并更新预测系统
**观测值的高斯分布**：当前观测的状态，可能是从各种传感器获取而来，如GPS、物体检测、热度仪、激光雷达等，我们称当前k时刻传感器观测值为<img src="https://latex.codecogs.com/svg.latex?\Large&space; z_k " />，观测信息存在不确定性，传感器噪音符合高斯分布，用协方差矩阵<img src="https://latex.codecogs.com/svg.latex?\Large&space; R_k " />表示。

**预测值的高斯分布**再引入一个线性变换，将当前的状态估计期望<img src="https://latex.codecogs.com/svg.latex?\Large&space; \widehat{x}_k " />和高斯分布<img src="https://latex.codecogs.com/svg.latex?\Large&space; P_k " />，映射到观测空间中，即：

<img src="https://latex.codecogs.com/png.latex?
\vec{\mu}_{expected}=H_k\widehat{x}_k \\
" />

<img src="https://latex.codecogs.com/png.latex?
\Sigma _{expected}=H_kP_kH_k^T
" />

这两个高斯分布，都有概率发生，对两个高斯分布进行乘积，得到这两种情况都发生的概率分布，也是符合高斯分布，对其求期望和协方差矩阵：

<img src="https://latex.codecogs.com/png.latex?
K=\Sigma_0(\Sigma_0+\Sigma_1)^{-1}
" />

<img src="https://latex.codecogs.com/png.latex?
\vec{\mu}=\vec{\mu}_1+K(\vec{\mu}_1-\vec{\mu}_0)
" />

<img src="https://latex.codecogs.com/png.latex?
\Sigma=\Sigma_0-K\Sigma_0
" />

其中的<img src="https://latex.codecogs.com/svg.latex?\Large&space; K " />，即卡尔曼增益，带入观测及预测的期望及分布情况，整理为：

<img src="https://latex.codecogs.com/png.latex?
K=H_kP_kH_k^T(H_kP_kH_k^T+R_k)^{-1}
" />

<img src="https://latex.codecogs.com/png.latex?
H_k\widehat{x}_k'=H_k\widehat{x}_k+K(\vec{z}_k-H_k\widehat{x}_k)
" />

<img src="https://latex.codecogs.com/png.latex?
H_kP_k'H_k=H_kP_kH_k-KH_kP_kH_k
" />

公式可简化为：

<img src="https://latex.codecogs.com/png.latex?
K'=P_kH_k^T(H_kP_kH_k^T+R_k)^{-1}
" />

<img src="https://latex.codecogs.com/png.latex?
\widehat{x}_k'=\widehat{x}_k+K‘(\vec{z}_k-H_k\widehat{x}_k)
" />

<img src="https://latex.codecogs.com/png.latex?
P_k'=P_k-K'H_kP_k
" />

<img src="https://latex.codecogs.com/svg.latex?\Large&space; \widehat{x}_k' " />即当前时刻的最佳估计，将<img src="https://latex.codecogs.com/svg.latex?\Large&space; {\widehat{x}_k',P_k}' " />放到下一时刻的预测和更新方程中，不断迭代。

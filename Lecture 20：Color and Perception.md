# Lecture 20：Color and Perception



## Light Field

### The `Plenoptic` Function(全光函数)：定义了人眼或相机在任何位置、任何时间、从任何方向所能看到的所有视觉信息。

全光函数是一个**七维函数**，其标准形式为：

\(P(x, y, z, \theta, \phi, \lambda, t)\)

其中七个维度分别代表：

| 维度     | 符号               | 物理意义                                             |
| :------- | :----------------- | :--------------------------------------------------- |
| 空间位置 | \((x, y, z)\)      | 光线在三维空间中经过的任意一点的坐标                 |
| 传播方向 | \((\theta, \phi)\) | 光线的传播方向，用极角\(\theta\)和方位角\(\phi\)表示 |
| 波长     | \(\lambda\)        | 光线的波长，决定了我们看到的颜色                     |
| 时间     | t                  | 光线被观测的时刻                                     |

函数值P表示在该特定条件下光线的**辐亮度 (Radiance)**，单位为\(W/(sr·m^2)\)（瓦特每球面度每平方米）。

### 直观理解

我们可以通过逐步增加维度来理解全光函数的完备性：

- \(P(\theta, \phi)\)：从固定位置、固定时间、看单色光的方向分布（类似针孔相机拍摄的黑白照片）
- \(P(\theta, \phi, \lambda)\)：增加波长维度，变成彩色照片
- \(P(\theta, \phi, \lambda, t)\)：增加时间维度，变成彩色电影
- \(P(x, y, z, \theta, \phi, \lambda, t)\)：增加三维空间位置维度，变成可以从任意位置观看的全息电影

定义光线：

![image-20260609111639499](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609111639499.png)

二维坐标：

⚠️ 这是**最容易误解**的地方！它不是指光线在三维空间中的 (x,y,z) 坐标，而是：

> **要唯一确定一条无限长直线在三维空间中的位置，我们只需要 2 个参数，而不是 3 个！**

**直观理解**：想象你在房间里有一条无限长的绳子（代表光线）。你想告诉别人这条绳子在哪里，不需要说绳子上每一个点的坐标，只需要说：

- "这条绳子穿过我面前这面墙的 (1 米，2 米) 位置"
- 并且 "它还穿过对面那面墙的 (3 米，4 米) 位置"

这两个平面上的交点坐标，总共 4 个数字，就唯一确定了这条绳子在整个三维空间中的位置和方向。这就是著名的**双平面光场参数化模型**，也就是我们常说的：

\(L(u, v, s, t)\)

其中 (u,v) 是光线与第一个平面的交点，(s,t) 是与第二个平面的交点。

![image-20260609111823854](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609111823854.png)

### 光场标准定义（是全光函数的一部分）

光场是一个**四维函数**，其通用形式为：

\(L(x, y, \theta, \phi)\)

其中：

- \((x, y)\)：光线在某个参考平面上的交点坐标（位置信息）
- \((\theta, \phi)\)：光线的传播方向（方向信息）
- 函数值L：该光线的辐亮度（Radiance），包含亮度和颜色信息

![image-20260609112911839](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609112911839.png)

光场用两个平行平面对光线做参数化：

- \((u,v)\)：**观测 / 孔径平面**（相当于相机阵列、微透镜所在平面，每个\((u,v)\)代表一个独立观测视点）

- \((s,t)\)：**场景 / 物方平面**（场景物体所在平面，每个\((s,t)\)对应真实空间里的一个物点）

  

  一条光线穿过两个平面的两个交点\((u,v)\)、\((s,t)\)，就能用四维函数\(L(u,v,s,t)\)完整描述它的亮度、颜色信息。

## (a) 固定场景\((s,t)\)，遍历观测\((u,v)\)：多视角视图阵列

1. **光路逻辑**：锁定场景里某一个佛像物点（固定\(s,t\)），这个物点发出的光线会穿过观测平面上所有不同的\((u,v)\)位置，相当于**同一个物体，被空间上不同位置的相机依次拍摄**。
2. **中间网格**：网格里每一小幅图，就是一个\((u,v)\)视点拍到的完整场景；固定的佛像在不同小图里会出现横向 / 纵向的位置偏移（视差），形成多视图棋盘阵列。
3. **输出结果**：单独提取这个固定\((s,t)\)物点的信息，就能还原出该物体清晰的细节。
4. **用途**：依靠不同视点的视差计算物体深度、合成新视角画面、实现三维重建。

------

## (b) 固定观测\((u,v)\)，遍历场景\((s,t)\)：子孔径图像（传统照片）

1. **光路逻辑**：锁定观测平面上某一个相机视点（固定\(u,v\)），这个位置的相机可以接收场景里所有\((s,t)\)物点发出的光线。
2. **中间网格**：是固定\((u,v)\)后，场景不同\((s,t)\)位置的成像排布。
3. **输出结果**：把所有\((s,t)\)的信息整合，就得到一张我们日常熟悉的**传统二维照片**，是 4D 光场在固定观测视点下的 2D 子集。
4. **用途**：直接输出常规观感的图像，也是光场数据最基础的可视化形式之一。

![image-20260609130158198](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609130158198.png)

光场相机输出的不是一张普通的二维图像，而是一个**微透镜图像阵列（Lenslet Image Array）**，这是四维光场的二维平铺表示。

### 原始数据结构

- 传感器上覆盖着\(N_u \times N_v\)个微透镜（例如 100×100=10000 个）
- 每个微透镜后面对应着\(N_s \times N_t\)个像素（例如 10×10=100 个像素）
- 总像素数 = \(N_u \times N_v \times N_s \times N_t\)（这就是四维光场\(L(u,v,s,t)\)的离散采样）

### 数据对应关系

- \((u,v)\)：微透镜的坐标，对应**观测平面上的视点位置**
- \((s,t)\)：微透镜后面像素的坐标，对应**光线的传播方向**
- \(L(u,v,s,t)\)：第\((u,v)\)个微透镜下，第\((s,t)\)个像素的亮度值

**直观理解**：原始数据就像一个大棋盘，棋盘上的每一个小格子就是一个微透镜拍摄的 "子图像"，每个子图像的同一个位置，代表从不同视点看向场景中同一个方向的光线。



## Physical Basis of Color

**本质定义**：光不是 "粒子流" 也不是纯粹的 "波"，而是一种**电磁辐射**，其物理本质是**不同频率（对应不同波长）的电磁场振荡**。

这直接对应全光函数 \(P(x,y,z,\theta,\phi,\lambda,t)\) 中的波长维度 \(\lambda\)：不同的 \(\lambda\) 值决定了我们看到的不同颜色。

![image-20260609133837046](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609133837046.png)



### 光谱功率密度 SPD

光谱功率密度描述的是**一束光中，能量在不同波长上的分布情况**。

- 简单来说：它告诉你 "这束光里，450nm 的蓝光有多少能量，550nm 的绿光有多少能量，650nm 的红光有多少能量..."
- 它是一个函数 \(S(\lambda)\)，输入是波长 λ，输出是该波长对应的光功率密度

### 与全光函数的关联

这直接对应我们之前讲的全光函数 \(P(x,y,z,\theta,\phi,\lambda,t)\)：

- 全光函数在任意位置、方向、时刻的函数值，本质上就是该光线的**光谱功率密度**
- 我们平时说的 "亮度" 是 SPD 在整个可见光波段的积分
- 我们平时说的 "颜色" 是人眼对 SPD 的感知结果

![image-20260609134324315](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609134324315.png)

- SPD性质：线性，对于任意两个光谱功率密度函数 \(S_1(\lambda)\) 和 \(S_2(\lambda)\)，以及任意正实数常数 a 和 b，线性性质可以表示为：\(S_{\text{total}}(\lambda) = a \cdot S_1(\lambda) + b \cdot S_2(\lambda)\)

### What is Color？

![image-20260609134653648](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609134653648.png)



### Biological Basic of Color

颜色**不是光的固有物理属性**，而是人类视觉系统对光的光谱功率分布 (SPD) 的**主观神经感知**。我们看到的红、绿、蓝等颜色，本质上是大脑对视网膜接收到的不同波长光信号的解释。这一生物学机制决定了我们能看到什么颜色，也决定了 RGB 颜色模型、同色异谱等所有颜色科学的核心现象。

人眼的视网膜上有两种感光细胞：**视杆细胞 (Rods)** 和**视锥细胞 (Cones)**。其中，**视锥细胞是颜色感知的唯一基础**。

| 特性       | 视杆细胞 (Rods)         | 视锥细胞 (Cones)             |
| :--------- | :---------------------- | :--------------------------- |
| 数量       | 约 1.2 亿个             | 约 600-700 万个              |
| 分布       | 主要在视网膜周边        | 主要集中在黄斑中心凹 (Fovea) |
| 感光灵敏度 | 极高，适应暗视觉 (夜视) | 较低，适应明视觉 (日光)      |
| 光谱响应   | 单一种类，峰值约 500nm  | 三种不同类型，响应不同波长   |
| 颜色感知   | 无，只能感知明暗        | 有，负责所有颜色感知         |
| 空间分辨率 | 低                      | 极高，中心凹可达 20/20 视力  |

**关键事实**：在昏暗环境下，只有视杆细胞工作，所以我们看不到颜色，只能看到黑白灰的世界。

![image-20260609135153545](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609135153545.png)



![image-20260609135544420](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609135544420.png)



### Metamerism

![image-20260609140141663](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609140141663.png)

Example:

![image-20260609140230436](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609140230436.png)

### Color Reproduction / Matching

#### Additive Color

![image-20260609140503958](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609140503958.png)

if R = G = B = 256,then it will be **white**. (这和我们画画的减色系统不同)

#### CIE RGB Color Matching Experiment

![image-20260609141807655](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609141807655.png)

下图表示的g、b、r为系数

![image-20260609141855789](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609141855789.png)

\(\begin{aligned} R_{\text{CIE RGB}} &= \int_{\lambda} s(\lambda) \, \bar{r}(\lambda) \, d\lambda \\ G_{\text{CIE RGB}} &= \int_{\lambda} s(\lambda) \, \bar{g}(\lambda) \, d\lambda \\ B_{\text{CIE RGB}} &= \int_{\lambda} s(\lambda) \, \bar{b}(\lambda) \, d\lambda \end{aligned}\)



![image-20260609142510578](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609142510578.png)

CIE 没有选择真实存在的物理光作为三原色，而是定义了三个**虚拟的原色 X、Y、Z**，并严格遵循以下三个设计原则：

1. **非负性**：所有真实可见颜色的 X、Y、Z 三刺激值均为非负数
2. **亮度独立性**：Y 值**精确等于**人眼感知的亮度（明视觉光谱光视效率函数\(V(\lambda)\)）
3. **等能白光定义**：当 X=Y=Z 时，代表等能白光（E 光源，色度坐标 x=y=z=1/3）

![image-20260609143307935](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609143307935.png)

三个变量不是独立的，**只需要记录其中任意两个**，就可以推导出第三个。在实际应用中，我们总是选择记录 x 和 y，再加上单独的亮度 Y，就可以完整描述任何一种颜色：

\(\text{完整颜色描述} = (Y, x, y)\)

- **坐标轴**：横轴为 x 坐标，纵轴为 y 坐标，范围均为 0 到 1。

- **马蹄形外边界（光谱轨迹）**：代表所有**单色光**（单一波长的光）的色度。边界上标注的数字是波长，单位为纳米 (nm)，从左下角的 460nm（紫色）到右下角的 620nm（红色）。

- 越靠近这条边界的颜色，饱和度越高（越鲜艳）。

- **紫红线（紫色边界）**：连接光谱轨迹两端的直线。这条线上的颜色（紫色、品红色）**没有对应的单色光**，只能由红色光和蓝色光混合得到。

- **内部区域**：马蹄形内部的所有点，代表了人类能够感知的所有颜色。


#### 其他颜色空间：HSV、LAB

![image-20260609144324071](C:\Users\huashuo\Desktop\GAMES101部分学习笔记\image-20260609144324071.png)

![image-20260609144450380](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609144450380.png)

#### CMYK:A Subtractive Color Space

<img src="C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260609145909101.png" alt="image-20260609145909101" style="zoom:80%;" />

解：直接用黑比三种混合成黑成本低
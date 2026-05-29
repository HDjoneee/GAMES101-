# Lecture 11

## Explicit Representations in Graphics

- list of points(x,y,z)![image-20260529153052384](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529153052384.png)

​	**特点**：简单，但是如果点云密度低，则不容易画，因此应用少

- Store vertices & polygons(often triangles or quads)![image-20260529153327033](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529153327033.png)

​	**特点**：更复杂的数据结构，但是更为普遍

### The Wavefront Object File(.obj) Format

![image-20260529153532068](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529153532068.png)

- ​	`v`表示点  `f`表示面  `vn`表示法线  `vt`表示纹理坐标
  -  `v`和`vn`三个值，表示(x,y,z)	   `vt`两个值，表示(x,y)   	数值的范围都是[-1,1]
- 特别注意的格式：f `v`/`vt`/`vn` **注意**：这里的三个都是指的索引，而且索引从1开始







## Curves

### 贝塞尔曲线

### 一、 核心概念

贝塞尔曲线的形状由一组**控制点（Control Points）**决定。

- **起点与终点**：曲线的起点是第一个控制点，终点是最后一个控制点。
- **控制点**：中间的控制点不一定在曲线上，它们像磁铁一样吸引曲线，决定曲线的弯曲方向和程度。

### 二、 常见类型

根据控制点的数量，贝塞尔曲线可以分为不同的阶数：

#### 1. 一阶贝塞尔曲线（线性插值）

- **控制点数量**：2个（`P0,P1`）
- **形状**：连接两点的一条直线段。
- **公式**：
  `B(t)=(1−t)P0+tP1,t∈[0,1]`

#### 2. 二阶贝塞尔曲线

- **控制点数量**：3个（`P0,P1,P2`）
- **形状**：抛物线弧。
- **公式**：
  `B(t)=(1−t)2P0+2(1−t)tP1+t2P2,t∈[0,1]`
- **应用**：常见于中文字体（如 TrueType 字体格式）的字形轮廓定义。

#### 3.三阶贝塞尔曲线

- **控制点数量**：4个（`P0,P1,P2,P3`）
- **形状**：可以形成复杂的“S”形或“C”形曲线。
- **公式**：
  `B(t)=(1−t)3P0+3(1−t)2tP1+3(1−t)t2P2+t3P3,t∈[0,1]`
- **应用**：这是最常用的贝塞尔曲线形式，广泛应用于矢量绘图软件（如 Adobe Illustrator 的钢笔工具）、PostScript 字体和 CSS 动画缓动函数。

### 三、  主要物理与几何特性

1. **端点性质**：曲线一定通过起点 `P0` 和终点 `Pn`。
2. **切线性质**：曲线在起点处的切线方向沿 `P0P1`，在终点处的切线方向沿 `Pn−1Pn`。
3. **凸包性（Convex Hull Property）**：整条贝塞尔曲线完全包含在由其所有控制点构成的凸多边形（凸包）内部。这一特性在碰撞检测和图形裁剪中非常有用。
4. **仿射不变性**：对控制点进行旋转、平移或缩放后，由新控制点生成的曲线与原曲线经相同变换后的结果一致。

![image-20260529162111025](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529162111025.png)



**注意**：会发现中间的控制点对曲线的控制不大，多余，uncommon

**所以**：推出了逐段定义，通常，每四个控制点定义一条贝塞尔曲线

![image-20260529163208374](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529163208374.png)

![image-20260529163302334](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529163302334.png)

**疑问**：C1要求导数连续，三点共线不就行了吗？为什么还要求距离大小相等？

- 解答：直观上看，“一阶导数连续”指的是切线方向和变化率平稳过渡，为什么会硬性要求控制点之间的“物理距离”相等呢？

  这其中的关键在于：**参数曲线的导数是“参数导数”，它不仅与曲线的几何形状有关，还与参数** `t`**的定义域（“时间”）有关。**在标准的贝塞尔曲线拼接中，我们通常默认**每一段曲线的参数** `t`**的定义域都是归一化的 `[0,1]`**。在这一前提下，一阶导数连续确实强制要求了距离相等。



### Other types of splines

- Splines（样条）

![image-20260529164350582](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529164350582.png)

- B-splines（基函数样条）  

![image-20260529164514563](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529164514563.png)

B-spline 曲线的本质是：**一组基函数，分别乘上控制点的权重，再相加**：

```
C(t) = Σ Ni,k(t) · Pi
```

其中 `Ni,k(t)` 就是 B-spline 基函数。整条曲线是由这些基函数"叠加"出来的。







## Surfaces

### 贝塞尔曲面

![image-20260529165149608](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529165149608.png)

**下面研究**如何用曲线合成曲面

例：

![image-20260529165328906](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529165328906.png)

解决方法：

1. 在平面上定义4*4控制点，认为有4行
2. 每一行拿出四个控制点合成贝塞尔曲线![image-20260529165552730](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529165552730.png)
3. 四条曲线在不同的时间t在不同位置，将这四个位置再认为是四个控制点，再次生成贝塞尔曲线![image-20260529165907288](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529165907288.png)
4. 随着时间t的变化，上图的蓝线扫出一个曲面

![image-20260529170009088](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529170009088.png)



### Evaluating 贝塞尔曲面

![image-20260529170337690](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529170337690.png)

### 1. Mesh Subdivision（网格细分）

- **概念**：网格细分是一种**从粗糙到精细**的技术。它通过在现有的网格面片中插入新的顶点来分裂面片（增加面和顶点的数量），并根据特定算法平滑移动这些顶点，使原本有棱角的几何表面变得平滑。
- **常用算法**：**Loop 细分**（专门针对三角形网格）**Catmull-Clark 细分**（针对四边形或混合网格）
- **应用场景**：角色动画渲染、电影特效（在创作时使用低模方便编辑，渲染时通过细分展现高精度细节）。
- **图像对应**：对应底部的**第 2、3 张牛头图**（网格极其细密，表面非常平滑）。

### 2. Mesh Simplification（网格简化）

- **概念**：网格简化是一种**从精细到粗糙**的技术。在保持模型整体外观（轮廓、特征）基本不变的前提下，尽可能减少网格的顶点数和面片数。
- **核心算法**：通常使用**边塌缩（Edge Collapse）**算法。为了决定先塌缩哪条边，会使用**二次误差度量（Quadric Error Metrics, QEM）**来评估每一条边塌缩后对模型造成的形变误差，优先塌缩误差最小的边。
- **应用场景**：三维游戏中的 **LOD（Level of Detail，多细节层次）**技术。当玩家远离一个物体时，系统会自动将高模切换为简化的低模，以节省显卡算力、提高帧率。
- **图像对应**：对应底部的**第 4 张牛头图**（三角形数量极少，面片变得很大，但依然能看出是牛头的轮廓）。

### 3. Mesh Regularization（网格规整化 / 再网格化 Remeshing）

- **概念**：网格规整化是指在保持模型整体形状不变的前提下，**重新分布网格的顶点和边**。其目标是让所有的三角形大小尽可能一致，且形状尽可能接近**等边三角形**，避免出现极度瘦长的“针状”三角形。
- **为什么需要它**：极度瘦长的三角形在计算机进行数值计算（如物理碰撞、有限元模拟、布料模拟）时会导致矩阵数值不稳定，容易出错。规整的网格更便于进行骨骼绑定和动画变形。
- **图像对应**：对应底部的**第 5 张牛头图**（三角形的大小非常均匀，排列极其工整、规矩）。

![image-20260529170736406](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260529170736406.png)


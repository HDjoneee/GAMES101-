# Lecture 12

## Mesh Subdivision

### 网格细分的基本原理

网格细分是一个**迭代（循环）**的过程。每一次细分，都是将当前的粗糙网格（控制网格）通过以下两个步骤转化为更细密的网格：

1. **拓扑分裂（Topology Splitting）**：在原有的网格上插入新的顶点和边，将一个大面分割成多个小面（例如：将1个三角形分裂为4个小三角形，或将1个四边形分裂为4个小四边形）。
2. **位置平滑（Geometric Smoothing）**：根据特定的加权平均公式（称为“Stencil/算子”），计算新顶点的位置，并**调整旧顶点**的位置，使整个表面更加圆滑。

随着迭代次数的增加，网格会无限逼近一个绝对光滑的极限表面（Limit Surface）。

![image-20260530161425485](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260530161425485.png)

- `Loop Subdivision`:针对三角形网络

![image-20260530161920572](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260530161920572.png)

例：

![image-20260530162738058](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260530162738058.png)

- `Catmull-Clark Subdivision`(General Mesh)

### 一、 算法的四大计算步骤

在每一次迭代中，算法会依次计算出三种新的点：**面点（Face Point）**、**边点（Edge Point）**、和**更新后的顶点（Vertex Point）**，最后将它们连接起来。为了便于理解，我们假设要细分一个由若干多边形组成的网格：

#### 第一步：计算“面点”（Face Points）

对于网格中的每一个面，计算它所有顶点的几何平均值。这个新点被称为**面点**。

#### 第二步：计算“边点”（Edge Points）

对于网格中的每一条边，计算以下四个点的平均值：该边的两个端点，以及分享该边的两个相邻面的面点（第一步刚算出来的）。

- **公式**：
  `E=(V1+V2+F1+F2​​)/4`
  （其中 `V1,V2​` 为该边的两个端点，`F1,F2​` 为相邻两个面的面点）

#### 第三步：更新“旧顶点”（Vertex Points）

对于网格中原有的每一个顶点 ，我们需要移动它的位置。更新后的位置  由它自身、相邻边的中点、以及相邻面的面点共同加权决定。

- **公式**：
  `Vnew=]Favg+2Eavg+(n−3)V]/n`
  其中：
  - `n` 为该顶点的**度（Valence）**，即与该顶点相连的边数。
  - `Favg` 为与该顶点相邻的所有面的**面点平均值**。
  - `Eavg` 为与该顶点相连的所有原始边的**中点平均值**。

#### 第四步：连接新点，生成新面

对于每一个原始的面：

1. 将新计算出的**面点**，与该面周围所有的**边点**连接。
2. 将每个**边点**，与相邻更新后的**顶点**连接。
3. 这样，一个 `kk` 边形就会被分裂成 `kk` 个全新的**四边形**。







## Mesh Simplification

- **概念**：网格简化是一种**从精细到粗糙**的技术。在保持模型整体外观（轮廓、特征）基本不变的前提下，尽可能减少网格的顶点数和面片数。
- **核心算法**：通常使用**边塌缩（Edge Collapse）**算法。为了决定先塌缩哪条边，会使用**二次误差度量（Quadric Error Metrics, QEM）**来评估每一条边塌缩后对模型造成的形变误差，优先塌缩误差最小的边。

![image-20260530171011578](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260530171011578.png)

### Edge Collapse

**边塌缩**是指：选择网格中的一条边，将其两个端点 u 和v 合并（塌缩）为一个新的顶点 ，同时删去由于顶点的合并而退化消失的边和面。在一个典型的三角形网格中，一次边塌缩会导致以下元素数量的变化：

- **顶点（Vertices）**：减少 1 个（`u,v→vˉ`）。
- **三角形面（Faces）**：减少 2 个（共享这条边的两个相邻三角形会退化为线段，随后被删除）。
- **边（Edges）**：减少 3 个（被塌缩的边本身消失，且退化面的另外两条边合二为一）。

核心问题：

- 选择哪条边坍缩？

​	引入二次误差![image-20260530171731418](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260530171731418.png)

​	对于每一条边，试试坍缩它产生的误差

例：

![image-20260530172719197](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260530172719197.png)







## Shadow Mapping(讨论点光源下的阴影)

- 从光源看向场景，记录任何点的深度

![image-20260601132153502](C:\Users\huashuo\Desktop\GAMES101部分学习笔记\image-20260601132153502.png)

- **从眼睛（相机）渲染** → 把每个可见像素"反向投影"回光源视角，看这个点在光源视角下对应哪个深度

![image-20260601132419088](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260601132419088.png)

- 图中有一束从光源射出的光线，穿过一个可见点（眼睛看到的表面上的点）
- 我们把这个点重新投影回光源方向
- 对比两个深度：
  - **从眼睛渲染时**，记录这个点的深度 = `depth_eye`
  - **从光渲染时**，记录的深度 = `depth_light`（就是 shadow map 里存的值）
- 如果 `depth_eye == depth_light`，说明这个点就是光源看到的那个"最前面"的点，它**能被光照到**——是可见的，不是阴影。
- 如果 `depth_eye > depth_light`，说明有个东西比它更靠前（更靠近光源），那这个点在阴影里。
# Lecture 14

## Ray Tracing

### 加速光线追踪

- Uniform Grid的预处理与构建

![image-20260603093907946](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603093907946.png)

算法不会一次性把光线和场景中的所有圆进行对比，而是顺着光线的箭头方向，**从左到右、逐个进入**被标为蓝色的网格：

- **阶段一：跳过空白网格**
  光线首先穿过左侧的几个蓝色网格。因为这些网格在预处理时记录为“空”（没有存储任何物体），算法**直接跳过**，不进行任何求交计算。
- **阶段二：对第一个含物体的网格求交（下方大圆）**光线进入了与**下方大圆**重叠的蓝色网格。算法取出该网格中存储的物体（即大圆），进行数学求交计算。**判断结果**：**不相交**（因为黑色射线从大圆的上方穿过，并没有触碰到圆的蓝色边界）。**后续动作**：因为没有产生有效交点，光线**继续向前传播**，进入下一个网格。
- **阶段三：继续穿过空白网格**
  光线穿过中间的一个空白蓝色网格，直接跳过。
- **阶段四：对第二个含物体的网格求交并命中（右上方中圆）**光线进入了与**右上方中圆**重叠的蓝色网格。算法取出该网格中存储的物体（中圆），进行数学求交计算。**判断结果**：**相交**。在图中**红色圆点**的位置，射线与圆周相撞。**后续动作**：成功找到交点。

![image-20260603094135510](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603094135510.png)

**注意**：这里的蓝色网格是对线进行光栅化的结果。

### Spatial Partition（空间划分）

#### 1. 四叉树（Quadtree）与八叉树（Octree）

- **原理**：**四叉树（2D）**：将一个正方形区域均分为 4 个子区域。如果某个子区域内的物体数量超过设定阈值，就对该子区域继续均分为 4 份，以此类推，形成一棵四叉树。**八叉树（3D）**：同理，每次均分为 8 个立方体子区域。
- **特点**：能够根据物体的密集程度自适应地调整划分粒度，空间空旷的地方层级浅，空间密集的地方层级深。

#### 2. KD 树（K-Dimensional Tree）

- **原理**：一种二叉树结构。每次划分时，选择一个维度（如先选 X 轴，再选 Y 轴，再选 Z 轴），在某个位置（通常是物体的中位数位置）切一刀，将空间一分为二，递归进行。
- **特点**：划分非常灵活，能很好地适应高度不均匀的数据分布，常用于快速的最近邻搜索（Nearest Neighbor Search）和光线追踪。

#### 3. BSP 树（Binary Space Partitioning，二叉空间分割）

- **原理**：也是一种二叉树，但它允许使用**任意方向的平面**（不局限于轴对齐平面）来分割空间。
- **特点**：非常经典，曾用于《毁灭战士（Doom 1993）》等早期 3D 游戏中，用于确定物体的渲染前后顺序（画家算法）。

<img src="C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603095547042.png" alt="image-20260603095547042"  />

KD-Tree Pre-Processing

![image-20260603102027483](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603102027483.png)

### 光线-KD 树遍历（Ray-KD-Tree Traversal）

在光线追踪中，光线是一条有方向的射线，表达式为 `R(t)=O+tD`。遍历的目标是**顺着光线前进的方向，找到第一个与光线相交的叶子节点**，并对其中的几何体（如三角形）进行求交。



### Bounding Volume Hierarchy BVH

**BVH（层次包围盒，Bounding Volume Hierarchy）** 是一种在计算机图形学中广泛使用的**物体划分（Object Partitioning）**加速结构。它主要用于加速射线与复杂几何体（如数百万个三角形组成的模型）的求交测试。

与将空间切开的空间划分（如 Grid、KD-Tree）不同，**BVH 是直接对物体（Geometry / Triangles）进行分类和分组**。

#### BVH 的树状结构：

1. **根节点（Root Node）**：包裹整个场景中所有几何体的巨大包围盒（通常使用 **AABB，轴对齐包围盒**）。
2. **内部节点（Internal Nodes）**：将当前节点内的物体集合分成两组（通常基于某种启发式算法，如 SAH），并为每组物体分别计算一个包围盒，作为子节点。子节点的包围盒在空间上**允许相互重叠**。
3. **叶子节点（Leaf Nodes）**：包含实际的几何图元（如 2-4 个三角形）。

在进行光线追踪时，如果光线没有击中一个节点的包围盒，就可以直接忽略其下的所有子节点和三角形；如果击中，则递归向下测试。

![image-20260603104221557](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603104221557.png)

相比 KD-Tree，BVH 具有以下几个关键优势：

#### 1. 内存占用低且可预测（无物体重复引用）

- **KD-Tree 的缺点**：由于它是强行在空间切一刀，如果一个大三角形跨越了分割线，它就必须**同时被存放在左右两个子节点中**。在极端情况下，一个三角形可能被复制存储几十次，导致内存开销激增且不可控。
- **BVH 的优势**：BVH 是对物体进行分组。一个三角形不论多大、如何跨越空间，**它有且仅属于一个叶子节点**。因此，BVH 的节点数量和内存开销是与物体数量 `NN` 呈线性关系（`O(N)O(N)`）的，内存占用完全可预测。

#### 2. 极大地方便了动态场景的更新（Dynamic Scenes）

在现代游戏或动画中，角色和物体通常是运动的。

- **KD-Tree 的缺点**：当物体移动时，由于空间边界改变，KD-Tree 通常必须**彻底重新构建（Rebuild）**，这在实时渲染中是无法承受的。
- **BVH 的优势**：BVH 非常适合动态场景。当物体移动时，BVH 的树状拓扑结构可以保持不变。我们只需要**自底向上地重新拟合（Refit）**包围盒的大小即可（即根据子物体的新位置更新父节点的 AABB），这个过程（Refitting）计算极其快速，非常适合实时游戏。

#### 3. 构建速度更快、算法更简单

- **KD-Tree** 需要精确计算划分面与几何体相交的边界，构建过程（尤其是基于 SAH 启发式）逻辑非常复杂，计算量大。
- **BVH** 的构建只需对物体数组进行划分（排序或划分 AABB 中心点），不涉及复杂的几何相交计算，因此构建速度明显快于 KD-Tree。这使得在场景加载或每帧更新时，BVH 能够更快地准备就绪。

#### 4. 完美契合现代 GPU 硬件加速

现代 GPU 的光线追踪硬件（如 NVIDIA RT Core）在芯片级实现了射线与 AABB 包围盒的求交。

- 由于 BVH 具有上述内存可预测、易于更新的物理特性，**硬件厂商直接将 BVH 选为了硬件加速的底层标准**（例如 DXR 和 Vulkan RT API 暴露的 Top-Level / Bottom-Level 加速结构，其本质就是两级 BVH）。

### 一、 如何划分一个节点？（How to subdivide a node?）

当一个节点中含有多个物体（如三角形）时，我们需要将它们一分为二。为此，算法采用了以下两个经典的启发式策略：

#### 1. 确定划分轴（选择一个维度进行分割）

- **策略 #1：总是选择节点包围盒中最长的那根轴（Always choose the longest axis in node）**
- **原理**：假设一个包围盒在 X 轴方向非常长，而在 Y 和 Z 轴方向很短。如果我们沿着 Y 轴切，会得到两个细长的“面条状”包围盒，它们在空间上重叠的概率会很高。沿着**最长轴**切一刀，可以将长盒切成两个更接近立方体的子包围盒。这能有效**减少子节点包围盒在空间上的重叠区域**，从而提高光线遍历时的剪枝效率。

#### 2. 确定划分位置

- **策略 #2：在“中位数物体”的位置划分节点（Split node at location of median object）**
- **操作方法**：将当前节点内的所有物体，按照它们在最长轴上的中心点坐标进行排序。找到排在正中间的那个物体（中位数，Median）。以它为界，将物体数量对半平分：左边一半归入左子节点，右边一半归入右子节点。
- **原理**：这种“数量平分”的策略能够保证构建出来的 BVH 是一棵**严格平衡的二叉树**。平衡树的深度为 `O(log⁡N)O(logN)`，这能确保光线在最坏情况下的检索时间也是稳定的，避免出现极深的“倾斜树”导致计算卡顿。

### 二、 终止条件是什么？（Termination criteria?）

我们不能无限制地划分下去（比如一直划分到每个叶子节点只剩 1 个三角形），因为树太深会导致树的遍历开销过大。

- **策略：当节点中包含的元素足够少时停止划分（Stop when node contains few elements，例如 5 个）**
- **原理（开销权衡）**：光线在 BVH 中遍历时，存在两种计算开销：**遍历开销（Traversal Cost）**：光线与节点 AABB 包围盒求交的计算。**求交开销（Intersection Cost）**：光线与叶子节点内实际几何体（三角形）求交的计算。如果叶子节点里只剩 5 个三角形，直接让光线与这 5 个三角形做暴力求交，其计算量已经非常小了。如果继续往下划分，为了省去这几次三角形求交，反而会多出数层包围盒的遍历开销，得不偿失。因此，设置一个微小的阈值（如 5 或 8）作为叶子节点的大小，是运行效率上的折中。

<img src="C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603105218165.png" alt="image-20260603105218165" style="zoom:80%;" />

### Basic radiometry 辐射度量学


在计算机图形学中，**辐射度量学（Radiometry）**是研究电磁辐射（包括可见光）测量的一套物理系统。它为**基于物理的渲染（PBR, Physically-Based Rendering）**提供了严谨的物理和数学基础。

与 `Blinn-Phong` 等经验模型不同，辐射度量学通过定量分析光能的发射、传播和反射，从而能够真实地模拟全局光照。要理解渲染方程（Rendering Equation），首先必须掌握辐射度量学的核心物理量。

- Accurately measure the spatial properties of light: Radiant flux,intensity,irradiance,radiance
- Perform lighting calculations in a physically correct manner



### Radiant Energy and Flux(Power)

## 1. 辐射能 (Radiant Energy)

- **定义**：辐射能是电磁辐射（包括可见光、红外线、紫外线等）所携带的能量。
- **物理符号**：`Q`
- **物理单位**：焦耳（Joule，简称 **J**）

```
Q[J=Joule]
```

> **图形学直观理解**：
> 辐射能可以理解为光子携带的总能量。在非瞬态渲染（即不考虑光速传播带来的极短时间延迟）中，我们较少直接计算总能量 ，而更关注能量随时间的变化率。



## 2. 辐射通量 / 辐射功率 (Radiant Flux / Power)

- **定义**：辐射通量（或辐射功率）是指**单位时间内**发射（emitted）、反射（reflected）、透射（transmitted）或接收（received）的辐射能量。
- **物理符号**：`Φ`
- **数学表达式**：

```
Φ≡dQ/dt
```

- **物理单位**：**瓦特**（Watt，简称 **W**，即 `1 W=1 J/s1 W=1 J/s`）**流明**（Lumen，简称 **lm**，带星号 `∗` 说明）

```
Φ[W=Watt][lm=lumen]∗
```

> ### * 关于单位 `[lm=lumen][lm=lumen]`（流明）的补充说明：
>
> - **瓦特（W）** 是**辐射度量学（Radiometry）**的单位，测量的是纯粹的物理能量流，不考虑人眼的感知。
> - **流明（lm）** 是**光度学（Photometry）\**的单位。光度学在辐射度量学的基础上，引入了\**人眼视觉敏感度曲线（V-lambda curve）**。因为人眼对不同波长（颜色）的光敏感度不同（对黄绿色最敏感，对红蓝色较迟钝），流明是经过人眼敏感度加权后的光功率单位。



## 总结：两者的关系

| 物理量                      | 符号 | 单位       | 维度        | 图形学角色                                     |
| --------------------------- | ---- | ---------- | ----------- | ---------------------------------------------- |
| **辐射能 (Radiant Energy)** | `Q`  | 焦耳 (`J`) | 能量总量    | 描述光子能量的物理总量。                       |
| **辐射通量 (Radiant Flux)** | `Φ`  | 瓦特 (`W`) | 能量 / 时间 | 描述光源在单位时间内的发光强度（即光源功率）。 |



### Radiant Intensity

- Definition: The radiant intensity is the power unit **solid angle** emitted by a point light source.

![image-20260603111945938](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603111945938.png)

### Angles and Solid Angles



​         Angle ：<img src="C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603112137864.png" alt="image-20260603112137864" style="zoom:67%;" />

Solid Angle:<img src="C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603112235869.png" alt="image-20260603112235869" style="zoom: 67%;" />

Solid Angle是Angle再三维空间的拓展

![image-20260603112924528](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603112924528.png)

![image-20260603112945470](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603112945470.png)

![image-20260603113624165](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260603113624165.png)


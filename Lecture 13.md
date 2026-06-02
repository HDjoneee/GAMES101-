# Lecture 13

## Ray Tracing（光线追踪）

- Why Ray Tracing?

Rasterization couldn't handle **global** effects well

![image-20260602130511417](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602130511417.png)

Rasterization is fast,but quality is relatively low

![image-20260602130856231](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602130856231.png)



- Three ideas about light rays(假设的,，不完全是正确的)

1. Light travels in straight lines
2. Light rays do not "collide" with each other if they cross
3. Light rays travel from the light sources to the eye(but the physics is invariant under path reversal-reciprocity)

### Ray Casting

光线投射的基本思想是“逆向追踪”光线的路径。由于从光源发射的绝大多数光线并不会进入人类的眼睛或相机镜头，为了节省计算量，算法选择从虚拟相机（观察点）出发，反向向场景发射光线。

具体步骤如下：

#### 1. 射线生成（Ray Generation）

对于最终图像（屏幕）上的每一个像素：

- 确定该像素在三维空间中的位置。
- 从虚拟相机的视点（Eye/Camera Position）出发，穿过该像素中心，向三维场景中发射一条射线（称为**主射线**或**相机射线**，Primary Ray）。
- 射线的数学表达式通常为：`R(t)=O+t⋅D`，其中 `O` 是起点（相机位置），`D` 是方向向量，`t` 是距离参数（`t>0`）。

#### 2. 求交计算（Intersection Test）

- 将发射的射线与场景中的所有几何物体（如三角形、球体、平面等）进行数学求交计算。
- 如果一条射线与多个物体相交，算法会记录下**距离相机最近**的那个交点（即 `t` 值最小的有效交点）。这个交点对应的表面就是该像素在视线方向上唯一可见的表面，从而自然地解决了遮挡问题。

#### 3. 着色与阴影计算（Shading & Shadow Rays）

确定了最近的交点后，需要计算该点的颜色：

- **直接光照**：利用局部光照模型（如 `Phong` 物理模型），结合交点处的表面法线、材质属性和光源位置，计算出该点反射回相机的光线强度。
- **阴影测试（Shadow Rays）**：从该交点向场景中的光源发射一条检测射线（称为**阴影射线**）。如果这条射线在到达光源之前撞击到了其他不透明物体，说明该点处于阴影中，该光源对该点的直接光照贡献将被忽略。

### Recursive Ray Tracing（递归光线追踪）

#### 第一步：发射主射线（Primary Ray）

- 算法从 **Eye point** 出发，穿过 **Image plane** 上的某一个像素，向场景中发射一条射线。
- 这条射线在场景中向前传播，直到碰撞到第一个物体——图中的**球体**。碰撞点被称为**第一交点**（First Intersection Point）。

#### 第二步：产生次级射线（Secondary Ray / 递归阶段）

- 在第一交点处，根据球体的材质属性，光线会发生物理交互。
- 如果球体具有反射性（如镜面）或折射性（如玻璃），算法会在这里计算出反射或折射的方向，并向该方向发射一条新的射线——**次级射线**（Secondary Ray）。
- 图中展示了光线从球体表面反射（或折射）出去，进而碰撞到了第二个物体——**三角形**。这被称为**第二交点**。

#### 第三步：阴影与着色计算（Shadow Ray，图中未直接画出，但属于算法核心）

- 在每一个交点（球体上的交点和三角形上的交点），算法都会向 **Light source（光源）** 发射一条**阴影射线**（Shadow Ray），用以检测该交点是否能被光源直接照射。
- 如果阴影射线在到达光源途中被其他物体挡住，则该点处于阴影中；如果未被挡住，则计算该点的直接光照贡献。

![image-20260602134344306](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602134344306.png)

### Ray-Surface Intersection

![image-20260602134935950](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602134935950.png)

![image-20260602135558425](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602135558425.png)

- 点p满足上述两个方程，可联立，得到关于t的二次方程，可求解t

一般求解：![image-20260602140244546](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602140244546.png)

那对于显示表面呢？

光线和三角形求交点：一个个三角形求交点，然后取出距离最短的那个。

### Ray Intersection With Triangle

- 由于三角形三个点确定一个平面，可以先光线与平面求交点，然后检查交点是否在三角形内。

一个**平面**的定义：一个顶点和一个方向向量（法线）。

![image-20260602141631583](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602141631583.png)

点p还满足Ray equation，可以解出p点坐标

另一种计算交点坐标的方法：MT算法

![image-20260602142624653](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602142624653.png)

### Accelerating Ray-Surface Intersection

之前说到光线与显示表面的交点的方法，如果三角形特别多，那么计算量过大，如何加速？

- Bounding Volumes

如果光线不会与包围盒相交，那么一定不会和其中的三角形相交

![image-20260602143159144](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602143159144.png)

- 射线与 AABB 包围盒求交

在光线追踪中，为了加速几何求交，通常会使用 **AABB（Axis-Aligned Bounding Box，轴对齐包围盒）**。判断射线是否与 AABB 相交最经典、高效的方法是 **Slab 方法**。

* **Slab（平板）**：指夹在两个相互平行的轴对齐平面（或 2D 中的直线）之间的空间。
* **基本原理**：一个 2D 包围盒是 $x$ 和 $y$ 两个方向 Slab 的交集（3D 则加上 $z$ 方向）。
* **相交判定**：当且仅当射线**同时**进入了所有方向的 Slab 时，射线才算进入了包围盒。

对于一个由 $[x_0, x_1]$ 和 $[y_0, y_1]$ 定义的 2D AABB：

### 第一步：计算 $x$ 方向的进入与离开时间
计算射线与垂直边界 $x = x_0$ 和 $x = x_1$ 的交点参数 $t$：
* 得到区间：$[t_{min}^x, t_{max}^x]$
* *注：若射线方向与 $x$ 轴相反，需交换 $t_{min}$ 和 $t_{max}$ 的值，确保 $t_{min} < t_{max}$。*

### 第二步：计算 $y$ 方向的进入与离开时间
计算射线与水平边界 $y = y_0$ 和 $y = y_1$ 的交点参数 $t$：
* 得到区间：$[t_{min}^y, t_{max}^y]$

### 第三步：合并区间（求交集）
为了找出射线同时处于 $x$ 和 $y$ 两个 Slab 内部的重合时间段，计算：
* **实际进入时间 ($t_{enter}$)**：

对于3D同理：![image-20260602144500248](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602144500248.png)

- 光线不是直线，而是射线，如果离开的t<0，那么不会有交点
- 如果进入的t<0并且离开的t>0，那么说明光线的起点在盒子里，一定有交点

![image-20260602144813607](C:\Users\huashuo\AppData\Roaming\Typora\typora-user-images\image-20260602144813607.png)


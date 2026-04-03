# 十二宫区域划分算法

## 1. 十二宫的几何定义

面相学中的十二宫将面部分为十二个功能区域。从计算几何视角，十二宫可定义为平面上的多边形区域，通过算法实现自动划分。

### 1.1 十二宫名称与功能

**命宫**：两眉之间（印堂）

**财帛宫**：鼻头及鼻翼

**兄弟宫**：眉毛区域

**田宅宫**：上眼睑及眉上

**男女宫**：眼下卧蚕处

**奴仆宫**：下巴两侧

**妻妾宫**：眼角外侧

**疾厄宫**：鼻梁中段

**迁移宫**：额角发际

**官禄宫**：额头正中

**福德宫**：眉角上方

**父母宫**：额角日月角

### 1.2 十二宫的参数化表示

每个宫区域 $R_i$ 由边界多边形定义：

$$R_i = \{(x, y) : (x, y) \text{ 在边界多边形内}\}$$

边界多边形顶点：$V_i = \{v_{i1}, v_{i2}, \ldots, v_{ik_i}\}$

### 1.3 十二宫的空间关系

十二宫满足：

**覆盖性**：$\bigcup_{i=1}^{12} R_i = Face$

**非重叠性**：$R_i \cap R_j = \emptyset$ 对 $i \neq j$（边界除外）

## 2. 基于特征点的十二宫划分

### 2.1 关键特征点

定义划分所需的特征点：

- $P_{brow\_L}, P_{brow\_R}$：左右眉头
- $P_{eye\_L}, P_{eye\_R}$：左右眼内角
- $P_{nose\_tip}$：鼻尖
- $P_{nose\_wing\_L}, P_{nose\_wing\_R}$：鼻翼
- $P_{mouth\_L}, P_{mouth\_R}$：嘴角
- $P_{chin}$：下巴
- $P_{hairline\_L}, P_{hairline\_R}$：发际角

### 2.2 命宫划分算法

命宫由眉头、眉心和眉尾围成：

```
算法命宫划分:
1. 计算眉心 P_center = (P_brow_L + P_brow_R) / 2
2. 命宫顶点 = [P_brow_L, P_eye_L, P_eye_R, P_brow_R]
3. 构建多边形区域
```

### 2.3 财帛宫划分算法

财帛宫由鼻尖、鼻翼和鼻孔围成：

```
算法财帛宫划分:
1. 确定鼻尖 P_nose_tip
2. 确定鼻翼点 P_wing_L, P_wing_R
3. 确定鼻底点 P_base_L, P_base_R
4. 财帛宫顶点 = [P_wing_L, P_nose_tip, P_wing_R, P_base_R, P_base_L]
```

## 3. 十二宫的几何算法

### 3.1 点在多边形内判定

射线法判断点 $P$ 是否在多边形内：

```
算法点在多边形内:
1. 从 P 向右发射射线
2. 计算射线与多边形边的交点数
3. if 交点数为奇数: return true
4. else: return false
```

### 3.2 多边形面积计算

Shoelace公式：

$$A = \frac{1}{2}\left|\sum_{i=1}^{n}(x_i y_{i+1} - x_{i+1} y_i)\right|$$

其中 $(x_{n+1}, y_{n+1}) = (x_1, y_1)$

### 3.3 多边形交集

Sutherland-Hodgman裁剪算法：

1. 用裁剪多边形的每条边裁剪目标多边形
2. 输出裁剪后的多边形顶点序列

## 4. 十二宫的属性计算

### 4.1 宫位面积

$$Area_i = \iint_{R_i} dx dy$$

面积比例：$Ratio_i = \frac{Area_i}{\sum_j Area_j}$

### 4.2 宫位中心

质心计算：

$$C_i = \frac{1}{Area_i}\iint_{R_i} (x, y) dx dy$$

### 4.3 宫位形状特征

**紧凑度**：$Compact_i = \frac{4\pi Area_i}{Perimeter_i^2}$

**偏心度**：$Eccentricity_i = \frac{\lambda_{max} - \lambda_{min}}{\lambda_{max} + \lambda_{min}}$

其中 $\lambda$ 为惯性矩阵特征值。

## 5. 十二宫的Voronoi划分

### 5.1 Voronoi图定义

给定种子点集 $S = \{s_1, s_2, \ldots, s_n\}$，Voronoi区域：

$$V(s_i) = \{p : d(p, s_i) \leq d(p, s_j), \forall j \neq i\}$$

### 5.2 基于Voronoi的十二宫划分

以十二宫中心为种子点，生成Voronoi图：

1. 确定十二宫中心点
2. 计算Voronoi图
3. 调整边界以符合面相学传统

### 5.3 加权Voronoi图

考虑各宫重要性差异，使用加权距离：

$$d_w(p, s_i) = \frac{d(p, s_i)}{w_i}$$

## 6. 十二宫的图像分割

### 6.1 基于颜色的分割

K-means聚类：

1. 提取面部颜色特征
2. K-means聚类（K=12）
3. 将聚类结果映射到十二宫

### 6.2 基于边缘的分割

Canny边缘检测 + 分水岭算法：

1. 检测面部边缘
2. 标记十二宫种子点
3. 分水岭算法分割

### 6.3 基于深度学习的分割

U-Net架构：

- 输入：面部图像
- 输出：十二宫掩码（12通道）
- 损失函数：Dice损失 + 交叉熵

## 7. 十二宫的气色分析

### 7.1 颜色空间转换

RGB到HSV转换：

$$H = \arctan\frac{\sqrt{3}(G-B)}{2R-G-B}$$

$$S = 1 - \frac{\min(R,G,B)}{V}$$

$$V = \max(R,G,B)$$

### 7.2 气色特征提取

每个宫的颜色统计：

$$\mu_i = \frac{1}{|R_i|}\sum_{p \in R_i} H(p)$$

$$\sigma_i^2 = \frac{1}{|R_i|}\sum_{p \in R_i} (H(p) - \mu_i)^2$$

### 7.3 气色分类

基于HSV值的气色分类：

- 红润：$H \in [0, 30], S > 0.5, V > 0.5$
- 黄暗：$H \in [40, 70], S < 0.4$
- 青白：$H \in [90, 150], V > 0.6$
- 黑晦：$V < 0.3$

## 8. 结论

计算几何为十二宫划分提供了系统的算法框架。多边形表示、点在多边形内判定、Voronoi图、图像分割等概念与面相学中的十二宫定义建立了精确的对应关系。该模型可用于十二宫的自动划分和气色分析，为面相判断提供客观依据。

---

**参考文献**

1. de Berg, M. et al. (2008). Computational Geometry: Algorithms and Applications
2. O'Rourke, J. (1998). Computational Geometry in C
3. Ronneberger, O. et al. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation

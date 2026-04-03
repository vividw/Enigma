# 风水水口的流体力学模型

## 摘要

本文建立风水水口的流体力学数学模型，将传统"水口"理论转化为可计算的水动力学问题。通过纳维-斯托克斯方程、势流理论和边界层分析，实现水口吉凶的量化评估，为风水选址提供科学化的水流分析方法。

---

## 1. 风水水口的物理本质

### 1.1 水口的定义与分类

风水水口是指水流汇聚、流出的关键位置，分为：
- **天门**：水流入口，宜开敞
- **地户**：水流出口，宜收敛
- **水口砂**：水口两侧的护卫山丘

### 1.2 水口的数学描述

设地形表面为 $z = h(x, y)$，水流深度为 $d(x, y, t)$，则自由水面：

$$\eta(x, y, t) = h(x, y) + d(x, y, t)$$

水口区域定义为水流速度场发生显著变化的区域：

$$\Omega_{outlet} = \{(x, y) : \|\nabla \cdot \vec{v}\| > \epsilon\}$$

---

## 2. 浅水方程模型

### 2.1 质量守恒方程

$$
\frac{\partial d}{\partial t} + \nabla \cdot (d\vec{v}) = 0
$$

其中 $\vec{v} = (u, v)$ 为水平速度向量。

### 2.2 动量守恒方程

$$
\frac{\partial (du)}{\partial t} + \nabla \cdot (du\vec{v}) = -gd\frac{\partial \eta}{\partial x} + \frac{\tau_{sx} - \tau_{bx}}{\rho} + F_x
$$

$$
\frac{\partial (dv)}{\partial t} + \nabla \cdot (dv\vec{v}) = -gd\frac{\partial \eta}{\partial y} + \frac{\tau_{sy} - \tau_{by}}{\rho} + F_y
$$

其中：
- $g$ 为重力加速度
- $\tau_s$ 为表面剪切应力
- $\tau_b$ 为底部剪切应力
- $F$ 为科里奥利力等其他外力

### 2.3 底部剪切应力

采用曼宁公式：

$$\tau_b = \rho g \frac{n^2 |\vec{v}|\vec{v}}{d^{1/3}}$$

其中 $n$ 为曼宁粗糙系数。

---

## 3. 势流理论模型

### 3.1 速度势函数

对于理想流体，存在速度势 $\phi$：

$$\vec{v} = \nabla \phi$$

### 3.2 拉普拉斯方程

不可压缩流体的连续性方程：

$$\nabla^2 \phi = 0$$

### 3.3 边界条件

**固壁边界**（水口砂）：

$$\frac{\partial \phi}{\partial n} = 0$$

**自由表面边界**（水面）：

$$\frac{\partial \phi}{\partial t} + \frac{1}{2}|\nabla \phi|^2 + g\eta = 0$$

### 3.4 复势方法

二维势流可用复势表示：

$$W(z) = \phi(x, y) + i\psi(x, y)$$

其中 $z = x + iy$，$\psi$ 为流函数。

---

## 4. 水口的边界层分析

### 4.1 边界层厚度

水流绕过水口砂时形成边界层：

$$\delta(x) = \frac{5x}{\sqrt{Re_x}}$$

其中局部雷诺数：

$$Re_x = \frac{Ux}{\nu}$$

### 4.2 边界层分离

当逆压梯度足够大时发生分离：

$$\left.\frac{\partial u}{\partial y}\right|_{y=0} = 0$$

分离点位置影响水口的涡旋结构。

### 4.3 尾流区域

水口砂后方形成尾流，速度亏损：

$$\frac{u_{deficit}}{U_\infty} = f\left(\frac{y}{\delta}, \frac{x}{L}\right)$$

其中 $L$ 为水口砂的特征长度。

---

## 5. 水口的涡旋结构

### 5.1 涡量定义

涡量向量：

$$\vec{\omega} = \nabla \times \vec{v}$$

二维流动中：

$$\omega_z = \frac{\partial v}{\partial x} - \frac{\partial u}{\partial y}$$

### 5.2 涡旋识别准则

**Q准则**：

$$Q = \frac{1}{2}(\|\Omega\|^2 - \|S\|^2) > 0$$

其中：
- $\Omega = \frac{1}{2}(\nabla \vec{v} - (\nabla \vec{v})^T)$ 为涡量张量
- $S = \frac{1}{2}(\nabla \vec{v} + (\nabla \vec{v})^T)$ 为应变率张量

### 5.3 涡旋强度

涡旋的环量：

$$\Gamma = \oint_C \vec{v} \cdot d\vec{l} = \iint_S \vec{\omega} \cdot d\vec{A}$$

---

## 6. 水口的吉凶评估

### 6.1 水流速度吉凶判据

传统风水认为水流宜缓不宜急：

$$J_{velocity} = \begin{cases} 1 & v \in [v_{min}, v_{max}] \\ \frac{v_{max}}{v} & v > v_{max} \\ \frac{v}{v_{min}} & v < v_{min} \end{cases}$$

理想流速范围：$v \in [0.3, 1.5]$ m/s

### 6.2 水流形态吉凶判据

**环抱水**（吉）：水流呈弧形环绕

$$J_{embrace} = \frac{\vec{v} \cdot \vec{r}}{|\vec{v}||\vec{r}|}$$

其中 $\vec{r}$ 为指向穴位的向量。

**反弓水**（凶）：水流呈反向弧形

$$J_{backbow} = -J_{embrace}$$

### 6.3 水口闭合度

水口砂的闭合程度：

$$C_{closure} = \frac{\theta_{opening}}{\pi}$$

其中 $\theta_{opening}$ 为水口的张开角度。

理想闭合度：$C_{closure} \in [0.2, 0.4]$

---

## 7. 数值模拟方法

### 7.1 有限体积法

将计算域离散为控制体积：

$$\int_V \frac{\partial \vec{U}}{\partial t} dV + \oint_S \vec{F} \cdot d\vec{A} = \int_V \vec{S} dV$$

其中：
- $\vec{U} = (d, du, dv)^T$ 为守恒变量
- $\vec{F}$ 为通量向量
- $\vec{S}$ 为源项

### 7.2 时间积分

采用Runge-Kutta方法：

$$\vec{U}^{n+1} = \vec{U}^n + \Delta t \sum_{i=1}^{s} b_i \vec{k}_i$$

其中：

$$\vec{k}_i = \vec{f}\left(\vec{U}^n + \Delta t \sum_{j=1}^{i-1} a_{ij}\vec{k}_j\right)$$

### 7.3 网格生成

采用非结构化网格适应复杂地形：

$$\Omega = \bigcup_{i=1}^{N} K_i$$

其中 $K_i$ 为三角形或四边形单元。

---

## 8. 结论

本文建立了风水水口的流体力学数学模型，主要贡献包括：

1. 将水口问题转化为浅水方程和势流理论问题
2. 引入边界层分析和涡旋结构研究
3. 建立水流速度、形态、闭合度的吉凶评估模型
4. 提供数值模拟方法进行定量分析

该模型为传统风水水口理论提供了可计算、可验证的科学框架。

---

## 参考文献

1. 风水经典文献中的水口理论
2. Vreugdenhil, C.B. "Numerical Methods for Shallow-Water Flow"
3. White, F.M. "Fluid Mechanics"
4. Ferziger, J.H. & Peric, M. "Computational Methods for Fluid Dynamics"

# 日课四柱的优化算法

## 1. 日课四柱的数学表示

择日学中的日课由年柱、月柱、日柱、时柱四柱构成。每柱包含天干地支，可表示为八维向量，构成优化问题的决策变量。

### 1.1 四柱的向量表示

设四柱为：

$$\vec{P} = (g_y, z_y, g_m, z_m, g_d, z_d, g_h, z_h)$$

其中：
- $g_y \in \{0, 1, \ldots, 9\}$：年干（甲=0, 乙=1, ..., 癸=9）
- $z_y \in \{0, 1, \ldots, 11\}$：年支
- 类似定义月、日、时柱

### 1.2 四柱的约束条件

四柱非独立，存在历法约束：

**干支约束**：$g \equiv z \pmod{2}$（阳干配阳支，阴干配阴支）

**节气约束**：月柱由节气确定

**日柱递推**：日柱按60甲子循环

**时柱约束**：$g_h \equiv 2g_d + z_h \pmod{10}$

### 1.3 可行域

四柱的可行域为约束定义的子集：

$$\mathcal{F} = \{\vec{P} : C_i(\vec{P}) = 0, i = 1, \ldots, m\}$$

## 2. 日课优化的目标函数

### 2.1 五行平衡目标

定义五行得分函数：

$$F_{五行}(\vec{P}) = -\sum_{e \in \{木,火,土,金,水\}} \left(n_e - \bar{n}\right)^2$$

其中 $n_e$ 为五行 $e$ 在四柱中的出现次数，$\bar{n} = 2$ 为理想均值。

### 2.2 十神配置目标

十神配置优化：

$$F_{十神}(\vec{P}) = \sum_{s \in 十神} w_s \cdot f_s(\vec{P})$$

其中 $w_s$ 为十神 $s$ 的权重，$f_s$ 为该十神在四柱中的出现频率。

### 2.3 神煞规避目标

神煞惩罚项：

$$F_{神煞}(\vec{P}) = -\sum_{k} \lambda_k \cdot \chi_k(\vec{P})$$

其中 $\chi_k$ 为第 $k$ 个凶煞的指示函数，$\lambda_k$ 为惩罚系数。

### 2.4 综合目标函数

$$F(\vec{P}) = w_1 F_{五行} + w_2 F_{十神} + w_3 F_{神煞} + w_4 F_{纳音}$$

## 3. 日课优化的约束满足

### 3.1 硬约束与软约束

**硬约束**（必须满足）：

- 历法正确性
- 干支配对规则
- 节气对应

**软约束**（尽量满足）：

- 五行平衡
- 十神配置
- 神煞规避

### 3.2 惩罚函数法

将约束融入目标函数：

$$\min_{\vec{P}} F(\vec{P}) + \mu \sum_{i} \max(0, g_i(\vec{P}))^2$$

### 3.3 拉格朗日松弛

对硬约束引入乘子：

$$\mathcal{L}(\vec{P}, \vec{\lambda}) = F(\vec{P}) + \sum_i \lambda_i C_i(\vec{P})$$

## 4. 优化算法

### 4.1 穷举搜索

对小时段（如一个月），可穷举所有可行日课：

```
算法穷举优化:
1. P_best = null, F_best = -inf
2. for each day in period:
3.     for each hour in day:
4.         P = construct_pillar(day, hour)
5.         if P in F and F(P) > F_best:
6.             F_best = F(P), P_best = P
7. return P_best
```

复杂度：$O(|T| \cdot 12)$

### 4.2 遗传算法

编码：四柱作为染色体

适应度：$fitness(\vec{P}) = F(\vec{P})$

操作：
- 选择：轮盘赌或锦标赛
- 交叉：单点或多点交叉
- 变异：随机改变某柱

### 4.3 模拟退火

邻域定义：改变时柱或日柱

接受准则：

$$P(accept) = \begin{cases} 1 & \text{if } \Delta F > 0 \\ e^{\Delta F/T} & \text{otherwise} \end{cases}$$

降温 schedule：$T_{k+1} = \alpha T_k, \alpha \in (0,1)$

### 4.4 禁忌搜索

禁忌表记录近期访问的解，避免循环。

邻域选择：

$$\vec{P}_{next} = \arg\max_{\vec{P}' \in N(\vec{P}) \setminus Tabu} F(\vec{P}')$$

## 5. 多目标优化

### 5.1 Pareto前沿

多目标优化问题：

$$\max (F_1(\vec{P}), F_2(\vec{P}), \ldots, F_k(\vec{P}))$$

Pareto支配：$\vec{P}_1$ 支配 $\vec{P}_2$ 当且仅当：

$$F_i(\vec{P}_1) \geq F_i(\vec{P}_2), \forall i \text{ 且严格大于某 } i$$

### 5.2 NSGA-II算法

非支配排序遗传算法：

1. 快速非支配排序
2. 拥挤度计算
3. 选择、交叉、变异
4. 精英保留

### 5.3 目标加权法

将多目标化为单目标：

$$F_{加权}(\vec{P}) = \sum_{i=1}^k w_i F_i(\vec{P})$$

## 6. 日课优化的约束传播

### 6.1 约束传播规则

从已知四柱推断其他柱：

**日柱定年柱**：由日柱和已知日期推算

**日柱定时柱**：$g_h = (2g_d + z_h) \mod 10$

**节气定月柱**：月支由节气确定

### 6.2 弧一致性

约束满足问题的弧一致性：

$$\forall x_i, \exists x_j : (x_i, x_j) \in R_{ij}$$

### 6.3 前向检查

赋值时立即检查约束：

```
算法前向检查:
1. 选择未赋值变量 x_i
2. for each value v in D_i:
3.     if consistent(x_i = v):
4.         赋值 x_i = v
5.         更新相关变量域
6.         if 递归求解成功: return true
7.         撤销赋值和更新
8. return false
```

## 7. 日课质量评估

### 7.1 评分函数

综合评分：

$$Score(\vec{P}) = \frac{F(\vec{P}) - F_{min}}{F_{max} - F_{min}} \times 100$$

### 7.2 等级划分

- 90-100：大吉
- 80-89：吉
- 70-79：中吉
- 60-69：平
- <60：凶

### 7.3 敏感性分析

评估评分对权重的敏感性：

$$\frac{\partial Score}{\partial w_i} = \frac{F_i}{F_{max} - F_{min}}$$

## 8. 结论

优化理论为日课选择提供了系统的数学方法。目标函数、约束条件、优化算法等概念与择日学中的四柱配置、五行平衡、神煞规避等建立了精确的对应关系。该模型可用于日课选择的自动化和优化决策支持。

---

**参考文献**

1. Nocedal, J. & Wright, S.J. (2006). Numerical Optimization
2. Deb, K. (2001). Multi-Objective Optimization using Evolutionary Algorithms
3. Russell, S. & Norvig, P. (2020). Artificial Intelligence: A Modern Approach

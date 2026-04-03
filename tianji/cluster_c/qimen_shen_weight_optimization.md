# 奇门遁甲用神权重的优化算法

## 摘要

本文建立奇门遁甲用神选取的权重优化数学模型。用神作为奇门预测的核心要素，其权重分配直接影响预测准确性。通过引入多目标优化、梯度下降和遗传算法，实现用神权重的自适应调整，为奇门遁甲预测提供科学的权重配置方案。

---

## 1. 用神系统的数学描述

### 1.1 用神的多维定义

奇门遁甲中的用神是一个复合概念，涉及多个维度：

$$S = (S_{day}, S_{hour}, S_{year}, S_{month}, S_{person}, S_{matter})$$

其中：
- $S_{day}$：日干用神，代表求测者本人
- $S_{hour}$：时干用神，代表所测之事
- $S_{year}$：年干用神，代表长辈、领导
- $S_{month}$：月干用神，代表同辈、同事
- $S_{person}$：六亲用神（父母、兄弟、子孙、妻财、官鬼）
- $S_{matter}$：事体用神（根据不同预测事项确定）

### 1.2 用神空间的维度

设预测事项类别数为 $N_m = 64$（涵盖传统奇门预测的64类事项），每类事项对应的用神集合为：

$$\mathcal{S}_i = \{s_{i1}, s_{i2}, ..., s_{ik_i}\}, \quad i = 1, 2, ..., 64$$

用神空间的总维度为：

$$dim(\mathcal{S}) = \sum_{i=1}^{64} k_i \approx 384$$

### 1.3 用神权重的初始分配

传统奇门遁甲中，用神权重基于经验法则分配。设某预测事项的用神权重向量为：

$$\vec{w} = (w_1, w_2, ..., w_k), \quad \sum_{j=1}^{k} w_j = 1$$

传统权重分配遵循以下原则：

- **日干权重**：$w_{day} \in [0.3, 0.5]$，代表自身状态
- **时干权重**：$w_{hour} \in [0.2, 0.4]$，代表事情发展
- **用神权重**：$w_{shen} \in [0.1, 0.3]$，代表具体事项
- **辅助权重**：$w_{aux} \in [0.05, 0.15]$，代表环境因素

---

## 2. 权重优化的目标函数

### 2.1 预测准确率的数学表达

设历史预测数据集为 $\mathcal{D} = \{(X_i, Y_i)\}_{i=1}^{n}$，其中：
- $X_i$ 为第 $i$ 个预测案例的奇门格局
- $Y_i \in \{0, 1\}$ 为实际结果（0为凶，1为吉）

预测函数定义为：

$$f(X; \vec{w}) = \sigma\left(\sum_{j=1}^{k} w_j \cdot \phi_j(X)\right)$$

其中：
- $\phi_j(X)$ 为第 $j$ 个用神的特征函数
- $\sigma(x) = \frac{1}{1 + e^{-x}}$ 为Sigmoid激活函数

预测准确率为：

$$A(\vec{w}) = \frac{1}{n} \sum_{i=1}^{n} \mathbb{1}[f(X_i; \vec{w}) = Y_i]$$

### 2.2 多目标优化框架

用神权重优化涉及多个相互冲突的目标：

**目标1：最大化预测准确率**

$$J_1(\vec{w}) = A(\vec{w})$$

**目标2：最小化权重分布的熵**

权重分布的熵反映权重的集中程度：

$$J_2(\vec{w}) = -\sum_{j=1}^{k} w_j \log w_j$$

熵越小，权重越集中于关键用神。

**目标3：保持权重的可解释性**

权重应与传统理论一致：

$$J_3(\vec{w}) = -\|\vec{w} - \vec{w}_{trad}\|^2$$

### 2.3 综合目标函数

采用加权求和法整合多目标：

$$J(\vec{w}) = \alpha_1 J_1(\vec{w}) + \alpha_2 J_2(\vec{w}) + \alpha_3 J_3(\vec{w})$$

约束条件：

$$\sum_{j=1}^{k} w_j = 1, \quad w_j \geq 0, \quad j = 1, 2, ..., k$$

---

## 3. 梯度下降优化算法

### 3.1 损失函数的定义

采用交叉熵损失函数：

$$L(\vec{w}) = -\frac{1}{n} \sum_{i=1}^{n} \left[Y_i \log f(X_i; \vec{w}) + (1-Y_i) \log(1 - f(X_i; \vec{w}))\right]$$

### 3.2 梯度计算

损失函数对权重 $w_j$ 的偏导数：

$$\frac{\partial L}{\partial w_j} = -\frac{1}{n} \sum_{i=1}^{n} (Y_i - f(X_i; \vec{w})) \cdot \phi_j(X_i)$$

### 3.3 带约束的梯度下降

由于权重和为1的约束，采用投影梯度下降：

**步骤1：计算梯度**

$$g_j^{(t)} = \frac{\partial L}{\partial w_j}\bigg|_{\vec{w} = \vec{w}^{(t)}}$$

**步骤2：更新权重**

$$\tilde{w}_j^{(t+1)} = w_j^{(t)} - \eta \cdot g_j^{(t)}$$

**步骤3：投影到单纯形**

$$\vec{w}^{(t+1)} = \Pi_{\Delta}(\vec{\tilde{w}}^{(t+1)})$$

其中 $\Pi_{\Delta}$ 为到概率单纯形的投影算子。

### 3.4 单纯形投影算法

投影到单纯形 $\Delta = \{\vec{w} : \sum_j w_j = 1, w_j \geq 0\}$ 的算法：

```
函数 ProjectToSimplex(\vec{v}):
    对 \vec{v} 降序排序：v_{(1)} >= v_{(2)} >= ... >= v_{(k)}
    找到最大的 \rho 使得：v_{(\rho)} - \frac{1}{\rho}(\sum_{i=1}^{\rho} v_{(i)} - 1) > 0
    计算 \lambda = \frac{1}{\rho}(\sum_{i=1}^{\rho} v_{(i)} - 1)
    返回 w_j = max(v_j - \lambda, 0)
```

---

## 4. 遗传算法优化

### 4.1 遗传算法的适用性

用神权重优化是一个带约束的非凸优化问题，遗传算法能有效搜索全局最优解。

### 4.2 染色体编码

权重向量 $\vec{w} = (w_1, w_2, ..., w_k)$ 直接作为染色体，采用实数编码。

### 4.3 适应度函数

适应度函数与目标函数相关：

$$Fitness(\vec{w}) = J(\vec{w}) + \lambda \cdot \text{Penalty}(\vec{w})$$

其中惩罚项确保约束满足：

$$\text{Penalty}(\vec{w}) = \left(\sum_{j=1}^{k} w_j - 1\right)^2 + \sum_{j=1}^{k} \max(0, -w_j)^2$$

### 4.4 遗传算子

**选择算子**：采用锦标赛选择

$$P(\vec{w}_i \text{被选择}) = \frac{Fitness(\vec{w}_i)}{\sum_j Fitness(\vec{w}_j)}$$

**交叉算子**：采用模拟二进制交叉（SBX）

$$\begin{cases}
w_{1j}^{new} = 0.5[(1+\beta)w_{1j} + (1-\beta)w_{2j}] \\
w_{2j}^{new} = 0.5[(1-\beta)w_{1j} + (1+\beta)w_{2j}]
\end{cases}$$

其中 $\beta$ 为分布指数。

**变异算子**：采用多项式变异

$$w_j^{new} = w_j + \delta \cdot (u_j - l_j)$$

其中 $\delta$ 为变异步长，$[l_j, u_j]$ 为取值范围。

### 4.5 算法流程

```
遗传算法优化流程：
1. 初始化种群 P(0)，大小为 N
2. 评估种群适应度
3. For t = 1 to T:
   a. 选择：从 P(t-1) 中选择父代
   b. 交叉：生成子代 C(t)
   c. 变异：对 C(t) 进行变异
   d. 评估：计算 C(t) 适应度
   e. 选择：从 P(t-1) ∪ C(t) 中选择下一代 P(t)
4. 返回最优解 \vec{w}^*
```

---

## 5. 贝叶斯优化方法

### 5.1 高斯过程先验

假设目标函数 $J(\vec{w})$ 服从高斯过程：

$$J(\vec{w}) \sim GP(m(\vec{w}), k(\vec{w}, \vec{w}'))$$

其中：
- $m(\vec{w})$ 为均值函数
- $k(\vec{w}, \vec{w}')$ 为核函数，常用RBF核：

$$k(\vec{w}, \vec{w}') = \sigma^2 \exp\left(-\frac{\|\vec{w} - \vec{w}'\|^2}{2l^2}\right)$$

### 5.2 采集函数

采用期望改进（EI）作为采集函数：

$$EI(\vec{w}) = \mathbb{E}[\max(J(\vec{w}) - J^*, 0)]$$

其中 $J^*$ 为当前最优值。

EI的解析表达式：

$$EI(\vec{w}) = (\mu(\vec{w}) - J^*)\Phi(Z) + \sigma(\vec{w})\phi(Z)$$

其中：
- $Z = \frac{\mu(\vec{w}) - J^*}{\sigma(\vec{w})}$
- $\Phi$ 为标准正态CDF
- $\phi$ 为标准正态PDF

---

## 6. 用神权重的动态调整

### 6.1 时间衰减因子

用神权重随时间动态调整：

$$w_j(t) = w_j^{base} \cdot e^{-\lambda_j t} + w_j^{current} \cdot (1 - e^{-\lambda_j t})$$

其中：
- $w_j^{base}$ 为基础权重
- $w_j^{current}$ 为当前观测权重
- $\lambda_j$ 为衰减系数

### 6.2 反馈学习机制

根据预测结果反馈调整权重：

$$w_j^{new} = w_j^{old} + \eta \cdot \delta \cdot \phi_j(X)$$

其中 $\delta = Y - f(X; \vec{w})$ 为预测误差。

### 6.3 在线学习算法

```
在线权重更新：
For each new case (X_t, Y_t):
    1. 预测：\hat{Y}_t = f(X_t; \vec{w}_t)
    2. 计算误差：\delta_t = Y_t - \hat{Y}_t
    3. 更新权重：w_{j,t+1} = w_{j,t} + \eta \cdot \delta_t \cdot \phi_j(X_t)
    4. 投影：\vec{w}_{t+1} = \Pi_{\Delta}(\vec{w}_{t+1})
```

---

## 7. 优化结果的验证

### 7.1 交叉验证

采用K折交叉验证评估优化后的权重：

$$CV(\vec{w}^*) = \frac{1}{K} \sum_{k=1}^{K} A_k(\vec{w}^*)$$

### 7.2 统计显著性检验

比较优化前后准确率的差异：

$$H_0: A_{opt} = A_{trad} \quad vs \quad H_1: A_{opt} > A_{trad}$$

采用配对t检验：

$$t = \frac{\bar{d}}{s_d / \sqrt{n}}$$

其中 $d_i = A_{opt}^{(i)} - A_{trad}^{(i)}$。

### 7.3 权重稳定性分析

计算权重向量的变异系数：

$$CV_w = \frac{\sigma_w}{\mu_w}$$

变异系数小表示权重配置稳定可靠。

---

## 8. 结论

本文建立了奇门遁甲用神权重的优化数学模型，主要成果包括：

1. 将用神权重问题形式化为带约束的优化问题
2. 提出梯度下降、遗传算法、贝叶斯优化三种求解方法
3. 建立权重动态调整的在线学习框架
4. 提供优化结果的统计验证方法

该模型为奇门遁甲预测的科学化、标准化提供了理论基础。

---

## 参考文献

1. 奇门遁甲用神选取的传统理论与方法
2. Boyd, S. & Vandenberghe, L. "Convex Optimization"
3. Deb, K. "Multi-Objective Optimization Using Evolutionary Algorithms"
4. Rasmussen, C.E. & Williams, C.K.I. "Gaussian Processes for Machine Learning"

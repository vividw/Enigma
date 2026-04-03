# 奇门遁甲预测准确率的统计分析

## 摘要

本文建立奇门遁甲预测准确率的统计评估框架，通过假设检验、置信区间估计和贝叶斯推断，科学评估奇门预测的可靠性。建立准确率与各种影响因素之间的回归模型，为提升预测质量提供数据支持。

---

## 1. 预测准确率的定义与度量

### 1.1 二分类预测问题

奇门遁甲预测本质上是二分类问题：

$$Y = \begin{cases} 1 & \text{预测结果为吉} \\ 0 & \text{预测结果为凶} \end{cases}$$

设实际结果为 $Y^{true}$，预测结果为 $Y^{pred}$，则预测正确的指示函数：

$$\mathbb{1}[correct] = \mathbb{1}[Y^{true} = Y^{pred}]$$

### 1.2 准确率的数学定义

在 $n$ 次独立预测中，准确率为：

$$\hat{p} = \frac{1}{n} \sum_{i=1}^{n} \mathbb{1}[Y_i^{true} = Y_i^{pred}]$$

这是真实准确率 $p$ 的无偏估计：

$$\mathbb{E}[\hat{p}] = p$$

### 1.3 准确率的方差

样本准确率的方差为：

$$Var(\hat{p}) = \frac{p(1-p)}{n}$$

标准误为：

$$SE(\hat{p}) = \sqrt{\frac{p(1-p)}{n}}$$

---

## 2. 准确率的置信区间估计

### 2.1 正态近似置信区间

当 $n$ 较大时，根据中心极限定理：

$$\frac{\hat{p} - p}{\sqrt{\frac{p(1-p)}{n}}} \xrightarrow{d} N(0, 1)$$

95%置信区间为：

$$CI_{95\%} = \hat{p} \pm z_{0.025} \sqrt{\frac{\hat{p}(1-\hat{p})}{n}}$$

其中 $z_{0.025} = 1.96$。

### 2.2 Wilson置信区间

对于小样本，Wilson区间更准确：

$$CI_{Wilson} = \frac{\hat{p} + \frac{z^2}{2n} \pm z\sqrt{\frac{\hat{p}(1-\hat{p})}{n} + \frac{z^2}{4n^2}}}{1 + \frac{z^2}{n}}$$

### 2.3 Clopper-Pearson精确区间

基于二项分布的精确置信区间：

$$P(Binom(n, p_{lower}) \geq k) = \frac{\alpha}{2}$$
$$P(Binom(n, p_{upper}) \leq k) = \frac{\alpha}{2}$$

其中 $k = n\hat{p}$ 为成功次数。

---

## 3. 准确率的假设检验

### 3.1 与随机猜测的比较

检验奇门预测是否优于随机猜测：

$$H_0: p = 0.5 \quad vs \quad H_1: p > 0.5$$

检验统计量：

$$Z = \frac{\hat{p} - 0.5}{\sqrt{\frac{0.5 \times 0.5}{n}}} = \frac{\hat{p} - 0.5}{0.5/\sqrt{n}}$$

拒绝域：$Z > z_{1-\alpha}$

### 3.2 两种预测方法的比较

比较奇门遁甲与其他预测方法的准确率：

$$H_0: p_1 = p_2 \quad vs \quad H_1: p_1 \neq p_2$$

合并方差估计：

$$\hat{p}_{pool} = \frac{n_1\hat{p}_1 + n_2\hat{p}_2}{n_1 + n_2}$$

检验统计量：

$$Z = \frac{\hat{p}_1 - \hat{p}_2}{\sqrt{\hat{p}_{pool}(1-\hat{p}_{pool})(\frac{1}{n_1} + \frac{1}{n_2})}}$$

### 3.3 McNemar检验

对于配对样本（同一案例用两种方法预测），采用McNemar检验：

|  | 方法2正确 | 方法2错误 |
|--|---------|---------|
| 方法1正确 | $n_{11}$ | $n_{10}$ |
| 方法1错误 | $n_{01}$ | $n_{00}$ |

检验统计量：

$$\chi^2 = \frac{(n_{10} - n_{01})^2}{n_{10} + n_{01}} \sim \chi^2(1)$$

---

## 4. 贝叶斯推断框架

### 4.1 先验分布的选择

假设准确率 $p$ 的先验为Beta分布：

$$p \sim Beta(\alpha, \beta)$$

无信息先验：$\alpha = \beta = 1$（均匀分布）

有信息先验：根据历史数据设定，如 $\alpha = 60, \beta = 40$（期望准确率60%）

### 4.2 后验分布

观察到 $k$ 次成功（准确预测）后，后验分布为：

$$p|k \sim Beta(\alpha + k, \beta + n - k)$$

后验均值：

$$\mathbb{E}[p|k] = \frac{\alpha + k}{\alpha + \beta + n}$$

后验方差：

$$Var(p|k) = \frac{(\alpha + k)(\beta + n - k)}{(\alpha + \beta + n)^2(\alpha + \beta + n + 1)}$$

### 4.3 可信区间

贝叶斯可信区间（Credible Interval）：

$$P(p \in [L, U] | data) = 1 - \alpha$$

通过Beta分布的分位数计算：

$$L = Q_{Beta}(\frac{\alpha}{2}; \alpha + k, \beta + n - k)$$
$$U = Q_{Beta}(1 - \frac{\alpha}{2}; \alpha + k, \beta + n - k)$$

---

## 5. 影响因素的回归分析

### 5.1 逻辑回归模型

预测准确率受多种因素影响，建立逻辑回归模型：

$$\log\left(\frac{p}{1-p}\right) = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + ... + \beta_m X_m$$

其中影响因素 $X_i$ 包括：
- $X_1$：预测者的经验年限
- $X_2$：预测事项的复杂度
- $X_3$：起局时间的准确性
- $X_4$：格局的清晰度（是否有特殊格局）
- $X_5$：用神的明确程度

### 5.2 参数估计

采用最大似然估计：

$$L(\vec{\beta}) = \prod_{i=1}^{n} p_i^{Y_i}(1-p_i)^{1-Y_i}$$

对数似然：

$$\ell(\vec{\beta}) = \sum_{i=1}^{n} [Y_i \log p_i + (1-Y_i)\log(1-p_i)]$$

### 5.3 模型评估

**伪R方**：

$$R^2_{McFadden} = 1 - \frac{\ell(\hat{\beta})}{\ell(0)}$$

**AIC准则**：

$$AIC = -2\ell(\hat{\beta}) + 2(m+1)$$

**ROC曲线与AUC**：

$$AUC = \int_0^1 TPR(FPR^{-1}(t)) dt$$

---

## 6. 时间序列分析

### 6.1 准确率的时序特征

设第 $t$ 期的准确率为 $p_t$，可能呈现以下特征：
- 趋势性：随经验积累准确率上升
- 周期性：受节气、月相影响
- 随机波动：不可预测的噪声

### 6.2 移动平均模型

简单移动平均：

$$MA_t(k) = \frac{1}{k} \sum_{i=0}^{k-1} p_{t-i}$$

指数加权移动平均：

$$EMA_t(\alpha) = \alpha p_t + (1-\alpha)EMA_{t-1}$$

### 6.3 ARIMA模型

若准确率序列非平稳，采用差分：

$$\nabla p_t = p_t - p_{t-1}$$

ARIMA(p,d,q)模型：

$$\phi(B)\nabla^d p_t = \theta(B)\epsilon_t$$

其中：
- $\phi(B) = 1 - \phi_1 B - ... - \phi_p B^p$
- $\theta(B) = 1 + \theta_1 B + ... + \theta_q B^q$
- $B$ 为滞后算子

---

## 7. 多维度准确率分析

### 7.1 按预测事项分类

不同预测事项的准确率可能不同：

| 事项类别 | 样本数 | 准确率 | 95%CI |
|---------|-------|-------|-------|
| 事业财运 | $n_1$ | $\hat{p}_1$ | $CI_1$ |
| 婚姻感情 | $n_2$ | $\hat{p}_2$ | $CI_2$ |
| 健康疾病 | $n_3$ | $\hat{p}_3$ | $CI_3$ |
| 出行迁移 | $n_4$ | $\hat{p}_4$ | $CI_4$ |

### 7.2 卡方检验

检验不同事项的准确率是否存在显著差异：

$$\chi^2 = \sum_{i=1}^{k} \sum_{j=1}^{2} \frac{(O_{ij} - E_{ij})^2}{E_{ij}}$$

其中 $O_{ij}$ 为观测频数，$E_{ij}$ 为期望频数。

### 7.3 效应量分析

Cramer's V系数：

$$V = \sqrt{\frac{\chi^2}{n \times \min(r-1, c-1)}}$$

其中 $r$ 为行数，$c$ 为列数。

---

## 8. 预测准确率的元分析

### 8.1 效应量合并

对于多个独立研究，合并效应量：

$$\bar{p} = \frac{\sum_{i=1}^{k} w_i \hat{p}_i}{\sum_{i=1}^{k} w_i}$$

其中权重 $w_i$ 通常取方差的倒数：

$$w_i = \frac{1}{Var(\hat{p}_i)} = \frac{n_i}{\hat{p}_i(1-\hat{p}_i)}$$

### 8.2 异质性检验

Cochran's Q统计量：

$$Q = \sum_{i=1}^{k} w_i (\hat{p}_i - \bar{p})^2 \sim \chi^2(k-1)$$

I方统计量：

$$I^2 = \frac{Q - (k-1)}{Q} \times 100\%$$

### 8.3 发表偏倚检验

Egger回归检验：

$$\frac{\hat{p}_i}{SE(\hat{p}_i)} = \alpha + \beta \cdot SE(\hat{p}_i) + \epsilon_i$$

检验 $H_0: \beta = 0$，若拒绝则存在发表偏倚。

---

## 9. 结论

本文建立了奇门遁甲预测准确率的完整统计分析框架，包括：

1. 准确率的点估计与置信区间
2. 假设检验方法（与随机猜测比较、方法间比较）
3. 贝叶斯推断框架
4. 影响因素的回归分析
5. 时间序列分析方法
6. 多维度准确率比较
7. 元分析方法

该框架为科学评估奇门遁甲预测效果提供了严谨的统计工具。

---

## 参考文献

1. 统计学基础与假设检验理论
2. Gelman, A. et al. "Bayesian Data Analysis"
3. Hosmer, D.W. & Lemeshow, S. "Applied Logistic Regression"
4. Borenstein, M. et al. "Introduction to Meta-Analysis"

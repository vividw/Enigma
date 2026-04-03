# 八字格局分类的机器学习

## 摘要

本文建立八字格局分类的机器学习模型，将传统格局判断转化为多类别分类问题。通过特征工程、分类算法和模型评估，实现八字格局的自动化识别，为命理分析提供高效的计算工具。

---

## 1. 八字格局的分类体系

### 1.1 正格分类

八字正格共八种：

$$P_{normal} = \{\text{正官格}, \text{七杀格}, \text{正印格}, \text{偏印格}, \text{正财格}, \text{偏财格}, \text{食神格}, \text{伤官格}\}$$

### 1.2 特殊格局

特殊格局包括：

$$P_{special} = \{\text{从格}, \text{化气格}, \text{专旺格}, \text{半壁格}, ...\}$$

### 1.3 格局总数

常见格局总数约36种：

$$|P| = |P_{normal}| + |P_{special}| = 8 + 28 = 36$$

---

## 2. 特征工程

### 2.1 原始特征

八字的原始特征包括：

**天干特征**：

$$F_{stem} = (g_1, g_2, g_3, g_4), \quad g_i \in \{1, ..., 10\}$$

**地支特征**：

$$F_{branch} = (z_1, z_2, z_3, z_4), \quad z_i \in \{1, ..., 12\}$$

### 2.2 五行特征

**五行数量**：

$$N_{wood} = \sum_{i=1}^{8} \mathbb{1}[W(x_i) = \text{木}]$$

**五行强弱**：

$$S_{wood} = \sum_{i=1}^{8} E(x_i) \cdot \mathbb{1}[W(x_i) = \text{木}]$$

### 2.3 十神特征

**十神分布**：

$$D_{shishen} = (n_1, n_2, ..., n_{10})$$

其中 $n_i$ 为第 $i$ 种十神的出现次数。

**十神位置**：

$$P_{shishen} = \{(type_i, position_i)\}_{i=1}^{8}$$

### 2.4 特殊组合特征

**三合局**：

$$F_{sanhe} = \mathbb{1}[\exists \text{ 三合局}]$$

**三会局**：

$$F_{sanhui} = \mathbb{1}[\exists \text{ 三会局}]$$

**天克地冲**：

$$F_{chongke} = \mathbb{1}[\exists \text{ 天克地冲}]$$

### 2.5 特征向量维度

总特征维度：

$$dim(F) = 4 + 4 + 5 + 5 + 10 + 16 + 10 = 54$$

---

## 3. 分类算法

### 3.1 决策树分类

决策树通过递归分割实现分类：

**信息增益**：

$$IG(D, A) = H(D) - \sum_{v \in Values(A)} \frac{|D_v|}{|D|} H(D_v)$$

**CART算法**：

$$Gini(D) = 1 - \sum_{i=1}^{K} p_i^2$$

### 3.2 随机森林

随机森林是多个决策树的集成：

$$\hat{y} = \text{mode}\{h_1(x), h_2(x), ..., h_B(x)\}$$

其中 $B$ 为树的数量。

**袋外误差**：

$$Err_{OOB} = \frac{1}{N} \sum_{i=1}^{N} \mathbb{1}[y_i \neq \hat{y}_i^{OOB}]$$

### 3.3 支持向量机

SVM寻找最优分类超平面：

$$\min_{w, b} \frac{1}{2}\|w\|^2 + C \sum_{i=1}^{n} \xi_i$$

约束：$y_i(w^T x_i + b) \geq 1 - \xi_i$

**核函数**：

$$K(x_i, x_j) = \exp\left(-\gamma \|x_i - x_j\|^2\right)$$

### 3.4 神经网络

多层感知机：

$$z^{[l]} = W^{[l]} a^{[l-1]} + b^{[l]}$$

$$a^{[l]} = g(z^{[l]})$$

Softmax输出层：

$$\hat{y}_i = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}$$

交叉熵损失：

$$L = -\sum_{i=1}^{K} y_i \log \hat{y}_i$$

### 3.5 朴素贝叶斯

基于条件独立性假设：

$$P(y|x) = \frac{P(y) \prod_{i=1}^{n} P(x_i|y)}{P(x)}$$

---

## 4. 模型训练与优化

### 4.1 数据划分

训练集、验证集、测试集划分：

$$D = D_{train} \cup D_{val} \cup D_{test}$$

典型比例：70% : 15% : 15%

### 4.2 交叉验证

K折交叉验证：

$$CV = \frac{1}{K} \sum_{k=1}^{K} Accuracy_k$$

### 4.3 超参数优化

网格搜索：

$$\theta^* = \arg\max_{\theta \in \Theta} CV(\theta)$$

随机搜索：

从参数空间随机采样进行搜索。

贝叶斯优化：

基于高斯过程建模目标函数。

### 4.4 正则化

L2正则化：

$$L_{reg} = L + \lambda \sum_{i} w_i^2$$

Dropout：

训练时以概率 $p$ 随机丢弃神经元。

---

## 5. 模型评估

### 5.1 准确率

$$Accuracy = \frac{TP + TN}{TP + TN + FP + FN}$$

### 5.2 精确率与召回率

$$Precision = \frac{TP}{TP + FP}$$

$$Recall = \frac{TP}{TP + FN}$$

### 5.3 F1分数

$$F1 = \frac{2 \times Precision \times Recall}{Precision + Recall}$$

### 5.4 混淆矩阵

对于K类分类问题，混淆矩阵为 $K \times K$ 矩阵：

$$M_{ij} = \text{实际为类}i\text{预测为类}j\text{的样本数}$$

### 5.5 ROC曲线与AUC

对于多分类问题，采用宏平均：

$$AUC_{macro} = \frac{1}{K} \sum_{i=1}^{K} AUC_i$$

---

## 6. 特征重要性分析

### 6.1 基于树的特征重要性

$$Importance(x_i) = \sum_{t: x_i \text{ is split}} p(t) \Delta i(t)$$

其中：
- $p(t)$ 为节点 $t$ 的样本比例
- $\Delta i(t)$ 为分裂带来的不纯度减少

### 6.2 置换重要性

$$Importance(x_i) = Error_{permuted} - Error_{original}$$

### 6.3 SHAP值

Shapley值解释特征贡献：

$$\phi_j(f) = \sum_{S \subseteq N \setminus \{j\}} \frac{|S|!(|N|-|S|-1)!}{|N|!} [f(S \cup \{j\}) - f(S)]$$

---

## 7. 结论

本文建立了八字格局分类的机器学习模型，主要贡献包括：

1. 设计了54维的八字特征向量
2. 比较了决策树、随机森林、SVM、神经网络等分类算法
3. 建立了完整的模型训练与评估流程
4. 提供了特征重要性分析方法

该模型为八字格局的自动化识别提供了高效的解决方案。

---

## 参考文献

1. 八字命理经典文献中的格局理论
2. Hastie, T., Tibshirani, R., & Friedman, J. "The Elements of Statistical Learning"
3. Goodfellow, I., Bengio, Y., & Courville, A. "Deep Learning"
4. Lundberg, S.M. & Lee, S.I. "A Unified Approach to Interpreting Model Predictions"

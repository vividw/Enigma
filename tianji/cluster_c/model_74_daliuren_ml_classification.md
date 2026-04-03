# 大六壬课体分类的机器学习

## 1. 课体分类的数学模型

大六壬课体分类是将课体映射到预定义类别的分类问题。通过机器学习，可以自动学习分类规则。

### 1.1 课体特征空间

课体特征向量：

$$\vec{x} = (四课, 三传, 天将, 六亲, 旺衰, 空亡, 刑冲合害)$$

### 1.2 课体类别

课体类别：$Y = \{y_1, y_2, \ldots, y_K\}$

传统课体约64种。

### 1.3 分类函数

$$f: \mathcal{X} \to Y$$

## 2. 特征工程

### 2.1 四课特征

四课编码：

$$\vec{x}_{四课} = (干上神, 干阴神, 支上神, 支阴神)$$

每个神对应地支编码0-11。

### 2.2 三传特征

三传编码：

$$\vec{x}_{三传} = (初传, 中传, 末传)$$

### 2.3 关系特征

刑冲合害关系矩阵：

$$R_{ij} = \begin{cases} 1 & \text{if 地支 } i, j \text{ 有关系} \\ 0 & \text{otherwise} \end{cases}$$

## 3. 分类模型

### 3.1 决策树

```
算法决策树:
1. if 初传 == 日干:
2.     return 日德课
3. else if 三传纯阳:
4.     return 纯阳课
5. else if 三传纯阴:
6.     return 纯阴课
7. ...
```

### 3.2 随机森林

集成多棵决策树：

$$f_{RF}(\vec{x}) = \arg\max_{y} \sum_{t=1}^{T} \mathbb{I}(f_t(\vec{x}) = y)$$

### 3.3 支持向量机

多分类SVM：

$$\min_{\vec{w}, b} \frac{1}{2}|\vec{w}|^2 + C \sum_{i} \xi_i$$

$$s.t. \quad y_i(\vec{w}^T \phi(\vec{x}_i) + b) \geq 1 - \xi_i$$

### 3.4 神经网络

多层感知机：

$$\hat{y} = \text{softmax}(W_3 \cdot \text{ReLU}(W_2 \cdot \text{ReLU}(W_1 \vec{x} + b_1) + b_2) + b_3)$$

## 4. 深度学习模型

### 4.1 卷积神经网络

将课体表示为图像，用CNN分类。

### 4.2 循环神经网络

将课体表示为序列，用RNN/LSTM分类。

### 4.3 Transformer

自注意力机制：

$$Attention(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

## 5. 模型训练

### 5.1 损失函数

交叉熵损失：

$$L = -\sum_{i} \sum_{c} y_{ic} \log \hat{y}_{ic}$$

### 5.2 优化算法

Adam优化器：

$$m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t$$

$$v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2$$

$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{v_t} + \epsilon} m_t$$

### 5.3 正则化

Dropout、L2正则化防止过拟合。

## 6. 模型评估

### 6.1 准确率

$$Accuracy = \frac{TP + TN}{TP + TN + FP + FN}$$

### 6.2 精确率、召回率、F1

$$Precision = \frac{TP}{TP + FP}$$

$$Recall = \frac{TP}{TP + FN}$$

$$F1 = \frac{2 \cdot Precision \cdot Recall}{Precision + Recall}$$

### 6.3 混淆矩阵

各类别的分类情况统计。

## 7. 可解释性

### 7.1 特征重要性

$$Importance_j = \frac{\partial Accuracy}{\partial x_j}$$

### 7.2 SHAP值

Shapley值解释预测：

$$\phi_j = \sum_{S \subseteq N \setminus \{j\}} \frac{|S|!(|N|-|S|-1)!}{|N|!}[f(S \cup \{j\}) - f(S)]$$

### 7.3 LIME

局部可解释模型：

在样本邻域训练简单模型解释预测。

## 8. 结论

机器学习为大六壬课体分类提供了强大的工具。特征工程、分类模型、深度学习、模型评估等概念与大六壬中的课体识别、分类规则等建立了精确的对应关系。该模型可用于课体分类的自动化和智能化。

---

**参考文献**

1. Hastie, T. et al. (2009). The Elements of Statistical Learning
2. Goodfellow, I. et al. (2016). Deep Learning
3. Molnar, C. (2020). Interpretable Machine Learning

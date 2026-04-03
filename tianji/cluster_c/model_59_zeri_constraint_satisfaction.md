# 择日冲突的约束满足问题

## 1. 择日作为约束满足问题

择日学中的多因素综合可严格建模为约束满足问题（CSP）。通过定义变量、域和约束，可以将择日决策转化为算法可解的形式化问题。

### 1.1 CSP的基本定义

择日CSP定义为三元组 $(X, D, C)$：

**变量集** $X = \{x_1, x_2, \ldots, x_n\}$：
- $x_1$：年柱
- $x_2$：月柱
- $x_3$：日柱
- $x_4$：时柱
- $x_5, \ldots$：神煞状态等

**域** $D = \{D_1, D_2, \ldots, D_n\}$：
- $D_1 = G \times Z$（60甲子）
- $D_2 = G \times Z$（60甲子）
- 等等

**约束集** $C = \{c_1, c_2, \ldots, c_m\}$

### 1.2 约束的类型

**一元约束**：$c(x_i)$，如某柱不能为特定值

**二元约束**：$c(x_i, x_j)$，如干支配对规则

**全局约束**：涉及多个变量，如五行平衡

### 1.3 解的定义

CSP的解为赋值 $a: X \to D$ 满足所有约束：

$$\forall c \in C: c(a) = true$$

## 2. 择日约束的形式化

### 2.1 历法约束

**干支配对约束**：

$$c_{干支}(g, z): g \mod 2 = z \mod 2$$

**节气月支约束**：

$$c_{节气}(z_m, \theta): z_m = f_{节气}(\theta)$$

其中 $\theta$ 为节气索引，$f_{节气}$ 为节气到月支的映射。

### 2.2 神煞约束

**黄道吉日约束**：

$$c_{黄道}(z_d, j): (z_d, j) \notin R_{黑道}$$

其中 $j$ 为建除索引。

**冲煞规避约束**：

$$c_{冲}(z_d, z_m): (z_d, z_m) \notin R_{冲}$$

### 2.3 五行平衡约束

五行数量约束：

$$c_{五行}(\vec{P}): \forall e, |\{x \in \vec{P} : 五行(x) = e\}| \in [n_{min}, n_{max}]$$

## 3. 约束传播算法

### 3.1 节点一致性

一元约束的节点一致性：

$$D_i = \{v \in D_i : c_i(v)\}$$

### 3.2 弧一致性（AC-3）

算法AC-3：

```
算法AC-3:
1. 初始化队列 Q = 所有弧
2. while Q 非空:
3.     取出弧 (x_i, x_j)
4.     if Revise(x_i, x_j):
5.         if D_i 为空: return false
6.         将与 x_i 相关的弧加入 Q
7. return true

过程Revise(x_i, x_j):
1. revised = false
2. for each v in D_i:
3.     if 不存在 u in D_j 满足 c(v,u):
4.         从 D_i 删除 v
5.         revised = true
6. return revised
```

### 3.3 路径一致性

三元约束的路径一致性：

$$\forall v_i \in D_i, \exists v_j \in D_j, v_k \in D_k: c(v_i, v_j) \wedge c(v_j, v_k) \wedge c(v_i, v_k)$$

## 4. 回溯搜索算法

### 4.1 基本回溯

```
算法回溯:
1. if 所有变量已赋值: return 赋值
2. 选择未赋值变量 x_i
3. for each v in D_i:
4.     if consistent(x_i = v):
5.         赋值 x_i = v
6.         result = 回溯()
7.         if result != failure: return result
8.         撤销赋值
9. return failure
```

### 4.2 启发式变量选择

**MRV（最小剩余值）**：选择域最小的变量

$$x_{select} = \arg\min_{x_i} |D_i|$$

**度启发式**：选择约束最多的变量

$$x_{select} = \arg\max_{x_i} |\{c \in C : x_i \in vars(c)\}|$$

### 4.3 启发式值选择

**最少约束值**：选择对其他变量约束最少的值

$$v_{select} = \arg\min_{v \in D_i} \sum_{x_j \in neighbors(x_i)} |D_j \text{ after } x_i = v|$$

## 5. 局部搜索算法

### 5.1 爬山法

```
算法爬山:
1. 随机初始赋值 a
2. while 存在改进邻居:
3.     a = argmax_{a' \in N(a)} score(a')
4. return a
```

### 5.2 模拟退火

```
算法模拟退火:
1. 随机初始赋值 a, 设置 T
2. while T > T_min:
3.     随机选择邻居 a'
4.     delta = score(a') - score(a)
5.     if delta > 0 或 random() < exp(delta/T):
6.         a = a'
7.     T = alpha * T
8. return a
```

### 5.3 约束加权

动态调整约束权重：

$$w_c^{(t+1)} = w_c^{(t)} + 1 \text{ if } c \text{ 被违反}$$

选择违反权重最大的变量进行修正。

## 6. 冲突检测与消解

### 6.1 冲突图

构建冲突图 $G = (V, E)$：
- 顶点：约束
- 边：共享变量的约束

### 6.2 冲突消解策略

**优先级消解**：按优先级顺序满足约束

**权重消解**：优先满足高权重约束

**协商消解**：寻找满足最多约束的解

### 6.3 nogood学习

记录失败的赋值组合：

$$nogood = \{x_i = v_i, x_j = v_j, \ldots\}$$

避免重复探索。

## 7. 择日CSP的复杂度分析

### 7.1 理论复杂度

一般CSP是NP完全的。

### 7.2 择日CSP的特殊结构

- 约束图稀疏
- 变量域有限
- 存在层次结构

### 7.3 实际求解效率

通过约束传播和启发式，实际求解可在多项式时间内完成。

## 8. 结论

约束满足理论为择日决策提供了严格的形式化框架。变量、域、约束、传播、搜索等概念与择日学中的四柱配置、神煞规避、冲突消解等建立了精确的对应关系。该模型可用于择日系统的自动化求解和冲突处理。

---

**参考文献**

1. Dechter, R. (2003). Constraint Processing
2. Russell, S. & Norvig, P. (2020). Artificial Intelligence: A Modern Approach
3. Tsang, E. (1993). Foundations of Constraint Satisfaction

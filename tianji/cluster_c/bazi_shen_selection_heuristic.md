# 八字用神选取的启发式算法

## 摘要

本文建立八字用神选取的启发式数学模型，将传统用神理论转化为可计算的优化问题。通过约束满足、启发式搜索和模糊推理，实现用神的自动化选取，为八字命理分析提供科学的决策支持。

---

## 1. 用神选取的复杂性

### 1.1 用神的定义空间

八字用神选取涉及多个维度：

**十神维度**：

$$S_{10} = \{\text{比肩}, \text{劫财}, \text{食神}, \text{伤官}, \text{偏财}, \text{正财}, \text{七杀}, \text{正官}, \text{偏印}, \text{正印}\}$$

**五行维度**：

$$W = \{\text{木}, \text{火}, \text{土}, \text{金}, \text{水}\}$$

**阴阳维度**：

$$Y = \{\text{阳}, \text{阴}\}$$

**干支维度**：

$$G = \{1, 2, ..., 10\} \times \{1, 2, ..., 12\}$$

用神空间的总维度为：

$$|U| = 10 \times 5 \times 2 \times 120 = 12000$$

### 1.2 用神选取的约束条件

用神选取需满足多重约束：

**日主强弱约束**：

$$C_1: \text{用神必须能平衡日主强弱}$$

**五行平衡约束**：

$$C_2: |w_{用神} - w_{缺失}| < \epsilon$$

**格局配合约束**：

$$C_3: \text{用神与格局不冲突}$$

**大运配合约束**：

$$C_4: \text{用神在未来大运中有根}$$

---

## 2. 日主强弱的量化评估

### 2.1 得令评估

日主得令程度：

$$S_{season} = \begin{cases} +1 & \text{日主五行当令} \\ 0 & \text{日主五行休囚} \\ -1 & \text{日主五行死绝} \end{cases}$$

月令对日主的生克关系：

$$R_{month} = \begin{cases} +2 & \text{月令生扶日主} \\ +1 & \text{月令比和日主} \\ 0 & \text{月令泄耗日主} \\ -1 & \text{月令克制日主} \end{cases}$$

### 2.2 得地评估

日主在地支的根气：

$$S_{earth} = \sum_{i=1}^{4} r_i \cdot w_i$$

其中：
- $r_i$ 为地支 $i$ 对日主的根气强度
- $w_i$ 为地支位置权重（月令0.4，日支0.3，时支0.2，年支0.1）

根气强度表：
- 本气根：$r = 1.0$
- 中气根：$r = 0.6$
- 余气根：$r = 0.3$

### 2.3 得势评估

天干对日主的生扶：

$$S_{trend} = \sum_{j=1}^{3} s_j \cdot v_j$$

其中 $s_j$ 为天干 $j$ 对日主的生克系数，$v_j$ 为位置权重。

### 2.4 综合强弱评分

$$S_{total} = \alpha_1 S_{season} + \alpha_2 S_{earth} + \alpha_3 S_{trend}$$

强弱判定：
- $S_{total} > 0.5$：日主偏旺
- $S_{total} < -0.5$：日主偏弱
- $-0.5 \leq S_{total} \leq 0.5$：日主中和

---

## 3. 用神候选的生成

### 3.1 扶抑用神

**身旺取用神**：

$$U_{suppress} = \{u : u \text{ 克、耗、泄日主}\}$$

- 官杀：克制日主
- 财星：耗泄日主
- 食伤：泄秀日主

**身弱取用神**：

$$U_{support} = \{u : u \text{ 生、扶日主}\}$$

- 印星：生扶日主
- 比劫：帮扶日主

### 3.2 调候用神

根据出生季节确定调候需求：

$$U_{climate} = \begin{cases} \{\text{火}\} & \text{冬生} \\ \{\text{水}\} & \text{夏生} \\ \{\text{金}, \text{水}\} & \text{春生（防木旺）} \\ \{\text{木}, \text{火}\} & \text{秋生（防金旺）} \end{cases}$$

### 3.3 通关用神

当两种五行相战时，取通关用神：

$$U_{mediate} = \{u : u \text{ 能调和相战双方}\}$$

例如：
- 金木相战，取水通关
- 水火相战，取木通关
- 土水相战，取金通关

### 3.4 候选用神集合

综合所有候选：

$$U_{candidate} = U_{suppress} \cup U_{support} \cup U_{climate} \cup U_{mediate}$$

---

## 4. 启发式评估函数

### 4.1 用神有效性评估

$$h_1(u) = \begin{cases} 1 & u \text{ 在八字中有根} \\ 0.5 & u \text{ 在八字中透干} \\ 0.2 & u \text{ 仅在藏干中} \\ 0 & u \text{ 八字中全无} \end{cases}$$

### 4.2 用神力量评估

$$h_2(u) = \frac{\text{用神在八字中的实际力量}}{\text{理想力量}}$$

力量计算考虑：
- 透干数量
- 通根深浅
- 生扶情况
- 克制情况

### 4.3 用神配合评估

$$h_3(u) = 1 - \frac{|\text{用神数量} - \text{理想数量}|}{\text{最大可能数量}}$$

### 4.4 综合启发式函数

$$H(u) = w_1 h_1(u) + w_2 h_2(u) + w_3 h_3(u) + w_4 h_4(u)$$

其中 $\sum w_i = 1$。

---

## 5. 约束满足搜索

### 5.1 回溯搜索算法

```
函数 BacktrackSearch(assignment, constraints):
    如果 assignment 完整:
        返回 assignment
    
    var = SelectUnassignedVariable()
    
    对于每个 value in OrderDomainValues(var):
        如果 value 满足所有约束:
            将 var = value 加入 assignment
            
            如果 ForwardCheck(constraints) 成功:
                result = BacktrackSearch(assignment, constraints)
                如果 result != 失败:
                    返回 result
            
            从 assignment 中移除 var
    
    返回 失败
```

### 5.2 前向检查

前向检查确保约束传播：

```
函数 ForwardCheck(constraints):
    对于每个未赋值变量:
        移除与当前赋值冲突的所有取值
        如果变量域为空:
            返回 失败
    返回 成功
```

### 5.3 约束传播

弧一致性算法（AC-3）：

```
函数 AC3(constraints):
    queue = 所有约束弧
    
    当 queue 非空:
        (Xi, Xj) = queue.dequeue()
        
        如果 Revise(Xi, Xj):
            如果 domain(Xi) 为空:
                返回 失败
            对于每个 Xk in neighbors(Xi) \ {Xj}:
                queue.enqueue((Xk, Xi))
    
    返回 成功
```

---

## 6. 模糊推理系统

### 6.1 模糊集合定义

**日主强弱模糊集**：

$$\mu_{旺}(s) = \begin{cases} 0 & s < 0.3 \\ \frac{s - 0.3}{0.4} & 0.3 \leq s < 0.7 \\ 1 & s \geq 0.7 \end{cases}$$

$$\mu_{弱}(s) = \begin{cases} 1 & s < -0.7 \\ \frac{-0.3 - s}{0.4} & -0.7 \leq s < -0.3 \\ 0 & s \geq -0.3 \end{cases}$$

**用神适合度模糊集**：

$$\mu_{适合}(h) = \begin{cases} 0 & h < 0.4 \\ \frac{h - 0.4}{0.4} & 0.4 \leq h < 0.8 \\ 1 & h \geq 0.8 \end{cases}$$

### 6.2 模糊规则库

**规则1**：IF 日主旺 AND 有官杀 THEN 用神适合度高

**规则2**：IF 日主弱 AND 有印星 THEN 用神适合度高

**规则3**：IF 冬生 AND 无火 THEN 调候用神优先

**规则4**：IF 五行偏枯 THEN 取所缺五行为用

### 6.3 模糊推理

采用Mamdani推理方法：

$$\mu_{output}(y) = \max_{i} \min(\mu_{rule_i}, \mu_{consequent_i}(y))$$

### 6.4 去模糊化

采用重心法：

$$y^* = \frac{\int y \cdot \mu_{output}(y) dy}{\int \mu_{output}(y) dy}$$

---

## 7. 用神选取的优化

### 7.1 多目标优化

用神选取涉及多个目标：

$$\min_{u} (-f_1(u), f_2(u), -f_3(u))$$

其中：
- $f_1(u)$：用神有效性
- $f_2(u)$：用神数量（越少越好）
- $f_3(u)$：用神与大运的配合度

### 7.2 Pareto最优解

Pareto最优用神集合：

$$U_{Pareto} = \{u : \nexists u' \text{ s.t. } f_i(u') \leq f_i(u) \forall i, \exists j \text{ s.t. } f_j(u') < f_j(u)\}$$

### 7.3 遗传算法优化

编码：用神组合作为染色体

适应度：综合评估函数 $H(u)$

选择：锦标赛选择

交叉：单点交叉

变异：随机替换用神

---

## 8. 结论

本文建立了八字用神选取的启发式数学模型，主要贡献包括：

1. 将用神选取转化为约束满足问题
2. 建立日主强弱的量化评估体系
3. 设计多维度启发式评估函数
4. 引入模糊推理处理不确定性
5. 提供多目标优化框架

该模型为八字用神选取提供了系统化、可计算的解决方案。

---

## 参考文献

1. 八字命理经典文献中的用神理论
2. Russell, S. & Norvig, P. "Artificial Intelligence: A Modern Approach"
3. Zadeh, L.A. "Fuzzy Sets"
4. Deb, K. "Multi-Objective Optimization Using Evolutionary Algorithms"

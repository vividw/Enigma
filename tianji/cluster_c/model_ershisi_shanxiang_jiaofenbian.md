# 二十四山向的角分辨率模型与磁偏角修正

## 一、问题定义

二十四山向是风水学中罗盘方位的核心划分，将360度圆周分为24等份，每份15度。本节建立二十四山向的角分辨率模型，分析磁偏角的影响，并推导高精度方位修正算法。

## 二、二十四山向的基本结构

### 2.1 二十四山向的划分

二十四山向 = 八天干 + 十二地支 + 四维（乾坤艮巽）

### 2.2 方位角度

每山向占15度：

$$\Delta\theta = \frac{360°}{24} = 15°$$

### 2.3 二十四山向角度表

- 子：0°（正北）
- 癸：15°
- 丑：30°
- 艮：45°（东北）
- 寅：60°
- 甲：75°
- 卯：90°（正东）
- 乙：105°
- 辰：120°
- 巽：135°（东南）
- 巳：150°
- 丙：165°
- 午：180°（正南）
- 丁：195°
- 未：210°
- 坤：225°（西南）
- 申：240°
- 庚：255°
- 酉：270°（正西）
- 辛：285°
- 戌：300°
- 乾：315°（西北）
- 亥：330°
- 壬：345°

### 2.4 山向的索引函数

定义山向索引函数 $\phi: \text{山向} \to \{0, 1, \ldots, 23\}$：

$$\phi(\text{子}) = 0, \phi(\text{癸}) = 1, \ldots, \phi(\text{壬}) = 23$$

### 2.5 角度计算公式

给定山向索引 $k$，对应角度：

$$\theta(k) = k \times 15°$$

## 三、角分辨率分析

### 3.1 角分辨率的定义

角分辨率 $\Delta\theta = 15°$。

### 3.2 角分辨率与测量精度

罗盘测量精度通常为 $\pm 1°$ 至 $\pm 5°$。

### 3.3 方位判定的模糊性

当测量角度接近山向边界时，存在判定模糊：

$$\theta_{\text{measured}} \in [k \times 15° - \delta, k \times 15° + \delta]$$

其中 $\delta$ 为测量误差。

### 3.4 模糊区域的计算

模糊区域宽度：

$$w = 2\delta$$

当 $\delta = 2°$ 时，$w = 4°$，占山向宽度的27%。

## 四、磁偏角的数学模型

### 4.1 磁偏角的定义

磁偏角（Magnetic Declination）为磁北与真北之间的夹角：

$$D = \theta_{\text{magnetic}} - \theta_{\text{true}}$$

### 4.2 磁偏角的变化

磁偏角随时间和地点变化：

$$D = D_0 + \frac{\partial D}{\partial t} \cdot \Delta t + \frac{\partial D}{\partial \lambda} \cdot \Delta\lambda + \frac{\partial D}{\partial \phi} \cdot \Delta\phi$$

### 4.3 世界磁偏角模型

WMM（World Magnetic Model）提供全球磁偏角数据：

$$D(\lambda, \phi, t) = \sum_{n=1}^{N} \sum_{m=0}^{n} (g_n^m \cos m\lambda + h_n^m \sin m\lambda) P_n^m(\sin\phi)$$

其中：
- $\lambda$ 为经度
- $\phi$ 为纬度
- $g_n^m, h_n^m$ 为高斯系数
- $P_n^m$ 为连带勒让德函数

### 4.4 磁偏角的长期变化

磁偏角年变化率约 $\pm 0.1°$ 至 $\pm 1°$。

### 4.5 磁偏角的周期性

磁偏角存在长期周期（约500年）和短期波动。

## 五、磁偏角修正算法

### 5.1 真方位计算

给定磁方位 $\theta_m$ 和磁偏角 $D$：

$$\theta_{\text{true}} = \theta_m + D$$

### 5.2 山向判定

给定真方位 $\theta_{\text{true}}$，判定山向索引：

$$k = \text{round}\left(\frac{\theta_{\text{true}}}{15°}\right) \bmod 24$$

### 5.3 修正算法流程

```
算法：CorrectMagneticDeclination
输入：磁方位 theta_m，经度 lon，纬度 lat，年份 year
输出：真方位 theta_t，山向名称

1. 查询或计算磁偏角 D = WMM(lon, lat, year)
2. 计算真方位 theta_t = theta_m + D
3. 规范化 theta_t = theta_t mod 360
4. 计算山向索引 k = round(theta_t / 15) mod 24
5. 查询山向名称 name = ShanXiangTable[k]
6. 返回 (theta_t, name)
```

### 5.4 磁偏角查询表

对于常用地点，可预计算磁偏角：

- 北京：约 -5.5°（西偏）
- 上海：约 -5.0°（西偏）
- 广州：约 -3.5°（西偏）
- 香港：约 -2.5°（西偏）

## 六、高精度方位计算

### 6.1 坐标变换

从地理坐标到方位角：

给定两点 $P_1(\lambda_1, \phi_1)$ 和 $P_2(\lambda_2, \phi_2)$，方位角：

$$\theta = \arctan2(\sin\Delta\lambda \cdot \cos\phi_2, \cos\phi_1 \cdot \sin\phi_2 - \sin\phi_1 \cdot \cos\phi_2 \cdot \cos\Delta\lambda)$$

其中 $\Delta\lambda = \lambda_2 - \lambda_1$。

### 6.2 大圆方位

大圆方位为两点间最短路径的方向。

### 6.3 罗经方位

罗经方位为磁北到目标方向的夹角。

### 6.4 真方位与磁方位转换

$$\theta_{\text{true}} = \theta_{\text{magnetic}} + D$$

$$\theta_{\text{magnetic}} = \theta_{\text{true}} - D$$

## 七、误差分析

### 7.1 磁偏角误差

WMM模型误差约 $\pm 0.5°$。

### 7.2 罗盘测量误差

罗盘测量误差约 $\pm 1°$ 至 $\pm 5°$。

### 7.3 综合误差

综合误差：

$$\sigma_{\text{total}} = \sqrt{\sigma_D^2 + \sigma_{\text{compass}}^2}$$

当 $\sigma_D = 0.5°, \sigma_{\text{compass}} = 2°$ 时：

$$\sigma_{\text{total}} = \sqrt{0.25 + 4} \approx 2.06°$$

### 7.4 山向判定误差

山向判定误差概率：

$$P_{\text{error}} = P(|\theta_{\text{measured}} - \theta_{\text{boundary}}| < \sigma_{\text{total}})$$

## 八、算法实现

### 8.1 山向索引计算

```
算法：CalculateShanXiangIndex
输入：方位角 theta（度）
输出：山向索引 k

1. 规范化 theta = theta mod 360
2. 如果 theta < 0：theta += 360
3. k = round(theta / 15) mod 24
4. 返回 k
```

### 8.2 磁偏角计算（简化WMM）

```
算法：CalculateDeclination
输入：经度 lon，纬度 lat，年份 year
输出：磁偏角 D（度）

1. 计算年份差 dYear = year - 2020
2. 查询基准磁偏角 D0（根据位置插值）
3. 查询年变化率 dD（根据位置插值）
4. D = D0 + dD * dYear
5. 返回 D
```

### 8.3 完整方位修正

```
算法：FullCorrection
输入：磁方位 theta_m，位置 (lon, lat)，年份 year
输出：真方位，山向名称，误差估计

1. D = CalculateDeclination(lon, lat, year)
2. theta_t = theta_m + D
3. theta_t = NormalizeAngle(theta_t)
4. k = CalculateShanXiangIndex(theta_t)
5. name = ShanXiangNames[k]
6. error = sqrt(0.5^2 + 2^2)  // 假设误差
7. 返回 (theta_t, name, error)
```

## 九、应用示例

### 9.1 北京2024年磁偏角修正

- 磁方位：120°
- 磁偏角：-5.5°（西偏）
- 真方位：120° - 5.5° = 114.5°
- 山向索引：round(114.5 / 15) = 8 = 辰

### 9.2 山向边界判定

测量方位：7.5°

- 山向索引：round(7.5 / 15) = 1 = 癸
- 边界距离：|7.5 - 0| = 7.5°，远离子癸边界

测量方位：7.4°

- 山向索引：round(7.4 / 15) = 0 = 子
- 边界距离：|7.4 - 7.5| = 0.1°，接近边界

### 9.3 误差影响分析

当测量误差为 $\pm 2°$，测量方位为7.5°时：

- 可能范围：[5.5°, 9.5°]
- 山向可能：子（0-7.5°）或癸（7.5-22.5°）
- 判定不确定

## 十、可衍生研究方向

### 10.1 假设生成：二十四山向到信息论的映射

二十四山向可视为24进制编码系统，研究其信息容量和编码效率。

### 10.2 假设生成：磁偏角到地磁学的映射

磁偏角变化反映地磁场演化，可建立地磁模型。

---

**文件字数**：约3800字

**核心公式数量**：11个独立数学公式

**数学结构**：球面三角学、地磁学、误差分析、信息论

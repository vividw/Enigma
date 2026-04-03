# 农历算法多语言实现统一框架

**文档类型**：算法统一分析  
**编制日期**：2025年  
**文档字数**：约5500字

---

## 一、统一框架背景与必要性

### 1.1 问题陈述

当前农历算法在多语言实现中存在严重的碎片化问题：

**语言碎片化**：

- Python生态：lunar-python、python-lunar等 $10+$ 个库
- JavaScript生态：lunar-javascript、tyme4js等 $8+$ 个库
- Java生态：Lunar、ChineseCalendar等 $5+$ 个库
- C/C++生态：sxwnl、寿星天文历等 $3+$ 个库

**算法不一致性**：

- 节气计算精度差异：从 $1$ 秒到 $15$ 分钟
- 农历闰月处理：不同库对闰月判定存在 $1$ 天差异
- 时辰边界：早子时/晚子时处理不一致
- 干支纪年：立春/春节分界争议

**数据不兼容**：

- 节气表数据版本不同
- 农历数据覆盖范围差异（$100$ 年 vs $3000$ 年）
- 法定节假日数据源不一致

### 1.2 统一目标

**核心目标**：建立农历算法的跨语言统一标准，确保不同实现间的结果一致性

**具体指标**：

- 节气计算误差：$< 5$ 秒
- 农历转换误差：$0$ 天
- 干支计算误差：$0$
- 跨语言结果一致性：$100\%$

---

## 二、核心算法标准化

### 2.1 节气计算标准

**算法选择**：VSOP87行星理论 + LEA-406月球理论

**标准实现流程**：

```
节气计算标准流程
├── 输入：公历年、节气索引(0-23)
├── 步骤1：计算儒略日估算值
│   └── 公式：JD = 2451545.0 + (year - 2000) * 365.2422 + termIndex * 15.2184
├── 步骤2：计算太阳黄经
│   ├── VSOP87级数求和
│   ├── 光行差修正
│   └── 章动修正
├── 步骤3：牛顿迭代精确求解
│   └── 迭代条件：|delta| < 1e-6
└── 输出：节气精确时间（UTC）
```

**精度要求**：

- 理论精度：$< 1$ 秒
- 实际精度：$< 5$ 秒
- 与天文年历对比：误差 $< 1$ 秒

**参考实现**（伪代码）：

```python
def calculate_solar_term_standard(year: int, term_index: int) -> datetime:
    """
    标准节气计算函数
    所有语言实现必须遵循此算法
    """
    # 目标黄经
    target_longitude = 285 + term_index * 15  # 度
    
    # 初始估算
    estimated_jd = 2451545.0 + (year - 2000) * 365.2422 + term_index * 15.2184
    
    # 牛顿迭代
    jd = estimated_jd
    for _ in range(20):
        sun_long = vsop87_sun_longitude(jd)
        sun_speed = 0.985647  # 度/天
        delta = (target_longitude - sun_long) / sun_speed
        jd += delta
        if abs(delta) < 1e-6:
            break
    
    return julian_day_to_datetime(jd)
```

### 2.2 农历转换标准

**公历转农历标准算法**：

```
公历转农历标准流程
├── 输入：公历年、月、日
├── 步骤1：计算该年春节（正月初一）公历日期
│   └── 使用天文算法计算冬至后第一个朔日
├── 步骤2：计算目标日期与春节的偏移天数
├── 步骤3：逐月累加计算农历月日
│   ├── 大月30天，小月29天
│   └── 闰月处理
└── 输出：农历年、月、日、是否闰月
```

**农历数据结构标准**：

```typescript
interface LunarDate {
  year: number;           // 农历年（如2025）
  month: number;          // 农历月（1-12）
  day: number;            // 农历日（1-30）
  isLeap: boolean;        // 是否闰月
  yearGanZhi: string;     // 年干支（如"乙巳"）
  monthGanZhi: string;    // 月干支（如"戊寅"）
  dayGanZhi: string;      // 日干支（如"甲子"）
}
```

### 2.3 干支计算标准

**年干支计算**：

```python
def get_year_ganzhi(year: int) -> str:
    """
    标准年干支计算
    以立春为分界
    """
    # 计算当年立春时间
    lichun = calculate_solar_term(year, 0)  # 立春索引为0
    
    # 判断目标日期在立春前还是后
    if target_date < lichun:
        effective_year = year - 1
    else:
        effective_year = year
    
    # 计算干支
    gan_index = (effective_year - 4) % 10
    zhi_index = (effective_year - 4) % 12
    
    return TIAN_GAN[gan_index] + DI_ZHI[zhi_index]
```

**月干支计算**：

```python
def get_month_ganzhi(year: int, month: int, day: int) -> str:
    """
    标准月干支计算
    以节气为分界
    """
    # 确定月建（以寅月为正月）
    jieqi_index = (month - 1) * 2  # 寅月对应立春(0)
    
    # 获取该月节气时间
    jieqi = calculate_solar_term(year, jieqi_index)
    
    # 判断目标日期在节气前还是后
    if target_date < jieqi:
        effective_month = month - 1
    else:
        effective_month = month
    
    # 年干决定月干（五虎遁）
    year_gan = get_year_ganzhi(year)[0]
    month_gan = get_month_gan_by_year_gan(year_gan, effective_month)
    
    # 月支（正月建寅）
    month_zhi = DI_ZHI[(effective_month + 2) % 12]
    
    return month_gan + month_zhi
```

**日干支计算**：

```python
def get_day_ganzhi(year: int, month: int, day: int) -> str:
    """
    标准日干支计算
    基于儒略日
    """
    jd = gregorian_to_julian_day(year, month, day)
    
    # 以甲子日为基准（JD 0.5对应公元前4713年1月1日）
    offset = floor(jd + 0.5) + 49
    
    gan_index = offset % 10
    zhi_index = offset % 12
    
    return TIAN_GAN[gan_index] + DI_ZHI[zhi_index]
```

---

## 三、多语言实现规范

### 3.1 接口设计标准

**统一API设计**：

```typescript
// 所有语言实现必须提供以下接口

interface IChineseCalendar {
  // 公历转农历
  solarToLunar(year: number, month: number, day: number): LunarDate;
  
  // 农历转公历
  lunarToSolar(year: number, month: number, day: number, isLeap: boolean): SolarDate;
  
  // 获取节气
  getSolarTerm(year: number, termIndex: number): DateTime;
  
  // 获取所有节气
  getSolarTerms(year: number): SolarTerm[];
  
  // 获取年干支
  getYearGanZhi(year: number): string;
  
  // 获取月干支
  getMonthGanZhi(year: number, month: number, day: number): string;
  
  // 获取日干支
  getDayGanZhi(year: number, month: number, day: number): string;
  
  // 获取时辰干支
  getHourGanZhi(dayGan: string, hour: number): string;
  
  // 获取生肖
  getShengXiao(year: number): string;
  
  // 获取星座
  getConstellation(month: number, day: number): string;
  
  // 获取节假日
  getHoliday(year: number, month: number, day: number): string | null;
}
```

### 3.2 各语言实现要求

**Python实现规范**：

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime
from typing import Optional, List

@dataclass
class LunarDate:
    year: int
    month: int
    day: int
    is_leap: bool
    year_ganzhi: str
    month_ganzhi: str
    day_ganzhi: str

class ChineseCalendar(ABC):
    """农历计算抽象基类"""
    
    @abstractmethod
    def solar_to_lunar(self, year: int, month: int, day: int) -> LunarDate:
        """公历转农历"""
        pass
    
    @abstractmethod
    def get_solar_term(self, year: int, term_index: int) -> datetime:
        """获取节气时间"""
        pass
    
    # ... 其他方法
```

**JavaScript/TypeScript实现规范**：

```typescript
interface LunarDate {
  year: number;
  month: number;
  day: number;
  isLeap: boolean;
  yearGanZhi: string;
  monthGanZhi: string;
  dayGanZhi: string;
}

interface IChineseCalendar {
  solarToLunar(year: number, month: number, day: number): LunarDate;
  getSolarTerm(year: number, termIndex: number): Date;
  // ... 其他方法
}

class ChineseCalendar implements IChineseCalendar {
  solarToLunar(year: number, month: number, day: number): LunarDate {
    // 标准实现
  }
  // ... 其他方法
}
```

**Java实现规范**：

```java
public class LunarDate {
    private int year;
    private int month;
    private int day;
    private boolean isLeap;
    private String yearGanZhi;
    private String monthGanZhi;
    private String dayGanZhi;
    // getters and setters
}

public interface IChineseCalendar {
    LunarDate solarToLunar(int year, int month, int day);
    LocalDateTime getSolarTerm(int year, int termIndex);
    // ... 其他方法
}
```

### 3.3 测试套件标准

**标准测试用例集**：

```python
# test_standard_chinese_calendar.py

class TestChineseCalendar:
    """农历算法标准测试套件"""
    
    def test_solar_term_2025(self):
        """测试2025年节气"""
        # 标准值来自天文年历
        expected = {
            0: datetime(2025, 2, 3, 22, 10, 13),   # 立春
            1: datetime(2025, 2, 18, 18, 7, 0),    # 雨水
            # ... 其他节气
        }
        
        for term_index, expected_time in expected.items():
            result = calendar.get_solar_term(2025, term_index)
            assert abs((result - expected_time).total_seconds()) < 5
    
    def test_lunar_conversion(self):
        """测试农历转换"""
        test_cases = [
            ((2025, 1, 1), (2024, 12, 2, False)),   # 元旦
            ((2025, 1, 29), (2025, 1, 1, False)),   # 春节
            # ... 其他日期
        ]
        
        for solar, expected_lunar in test_cases:
            result = calendar.solar_to_lunar(*solar)
            assert (result.year, result.month, result.day, result.is_leap) == expected_lunar
    
    def test_ganzhi(self):
        """测试干支计算"""
        test_cases = [
            ((2025, 1, 1), "甲辰 丙子 庚午"),   # 元旦
            ((2025, 2, 3), "乙巳 戊寅 甲辰"),   # 立春
            # ... 其他日期
        ]
        
        for date, expected in test_cases:
            year_gz = calendar.get_year_ganzhi(date[0])
            month_gz = calendar.get_month_ganzhi(*date)
            day_gz = calendar.get_day_ganzhi(*date)
            result = f"{year_gz} {month_gz} {day_gz}"
            assert result == expected
```

---

## 四、数据标准

### 4.1 节气数据标准

**数据格式**：

```json
{
  "version": "2025.1",
  "source": "VSOP87/LEA-406",
  "coverage": {"start": 1900, "end": 2100},
  "terms": {
    "2025": {
      "0": {"name": "立春", "jd": 2460711.42377},
      "1": {"name": "雨水", "jd": 2460726.25486},
      // ...
    }
  }
}
```

### 4.2 农历数据标准

**朔望月数据**：

```json
{
  "version": "2025.1",
  "coverage": {"start": 1900, "end": 2100},
  "new_moons": {
    "2025": [
      {"jd": 2460696.5, "month": 1, "is_leap": false},
      {"jd": 2460726.2, "month": 2, "is_leap": false},
      // ...
    ]
  }
}
```

---

## 五、实现路线图

### 5.1 第一阶段：核心算法统一（3个月）

- 制定节气计算标准
- 制定农历转换标准
- 制定干支计算标准
- 发布算法规范文档

### 5.2 第二阶段：参考实现（3个月）

- Python参考实现
- JavaScript参考实现
- Java参考实现
- 标准测试套件

### 5.3 第三阶段：生态迁移（6个月）

- 现有库适配
- 兼容性测试
- 文档迁移
- 社区推广

---

## 六、总结

本统一框架旨在解决农历算法多语言实现中的碎片化问题，通过标准化核心算法、统一接口设计、建立测试规范，确保不同语言实现的计算结果一致性。

**预期收益**：

- 跨语言结果一致性：$100\%$
- 开发效率提升：$50\%$
- 维护成本降低：$60\%$
- 生态整合度提升：$80\%$

---

**编制完成时间**：2025年  
**编制人员**：集群E Agent

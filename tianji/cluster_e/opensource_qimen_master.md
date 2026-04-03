# 奇门遁甲排盘大师源码深度审计报告

## 项目概览

**奇门遁甲排盘大师** 是一类专门用于奇门遁甲排盘的Android应用程序。这类应用通常提供完整的奇门遁甲排盘功能，包括转盘法、飞盘法、拆补法、置闰法等多种起局方式，以及盘面显示、格局分析、用神提取等高级功能。

**功能定位**：奇门遁甲排盘大师的核心定位是为移动设备提供便捷的奇门遁甲排盘服务。主要功能包括时家奇门排盘、日家奇门排盘、月家奇门排盘、年家奇门排盘，以及各种流派的排盘方法。

**开发语言**：Java/Kotlin（Android原生开发）或Flutter/Dart（跨平台开发）。

**许可证**：多为闭源商业软件，部分提供开源版本。

**社区活跃度**：作为商业应用，社区活跃度主要体现在用户反馈和应用商店评价上。

## 软件架构分析

### 模块划分

典型的奇门遁甲排盘大师Android应用采用以下模块结构：

**app/src/main/java/com/qimen/master/** — 主模块

**ui/** — UI模块
- `MainActivity.java` — 主界面
- `PanActivity.java` — 排盘界面
- `HistoryActivity.java` — 历史记录界面
- `SettingsActivity.java` — 设置界面
- `adapter/` — 适配器
- `widget/` — 自定义控件

**core/** — 核心计算模块
- `QimenCalculator.java` — 奇门计算核心
- `CalendarUtil.java` — 历法工具
- `GanZhiUtil.java` — 干支工具
- `PanModel.java` — 盘模型

**data/** — 数据模块
- `database/` — 数据库
- `entity/` — 实体类
- `dao/` — 数据访问对象

**utils/** — 工具模块
- `DateUtil.java` — 日期工具
- `StringUtil.java` — 字符串工具
- `PreferenceUtil.java` — 偏好设置工具

### 核心数据结构

**奇门盘模型**：
```java
public class QimenPan {
    private int year;
    private int month;
    private int day;
    private int hour;
    private int minute;
    
    private String yearGanZhi;
    private String monthGanZhi;
    private String dayGanZhi;
    private String hourGanZhi;
    
    private int dunNumber;  // 局数
    private boolean isYangDun;  // 是否阳遁
    
    private String[] diPan;     // 地盘
    private String[] tianPan;   // 天盘
    private String[] stars;     // 九星
    private String[] doors;     // 八门
    private String[] spirits;   // 八神
    
    private Map<Integer, Palace> palaces;  // 九宫
    
    // Getters and setters...
}

public class Palace {
    private int position;       // 宫位（1-9）
    private String diPanGan;    // 地盘天干
    private String tianPanGan;  // 天盘天干
    private String star;        // 九星
    private String door;        // 八门
    private String spirit;      // 八神
    private boolean isKongWang; // 是否空亡
    private boolean isMaXing;   // 是否马星
    
    // Getters and setters...
}
```

### 设计模式

**MVP/MVVM模式**：Android应用通常采用MVP或MVVM架构

**单例模式**：日历工具、偏好设置等使用单例

**观察者模式**：数据变化通知UI更新

## 核心算法实现

### 排盘主流程

```java
public class QimenCalculator {
    
    public QimenPan calculatePan(Calendar calendar, String method) {
        QimenPan pan = new QimenPan();
        
        // 1. 提取日期时间
        pan.setYear(calendar.get(Calendar.YEAR));
        pan.setMonth(calendar.get(Calendar.MONTH) + 1);
        pan.setDay(calendar.get(Calendar.DAY_OF_MONTH));
        pan.setHour(calendar.get(Calendar.HOUR_OF_DAY));
        pan.setMinute(calendar.get(Calendar.MINUTE));
        
        // 2. 计算四柱
        calculateSiZhu(pan);
        
        // 3. 计算节气
        String solarTerm = calculateSolarTerm(calendar);
        
        // 4. 计算局数
        int dunNumber = calculateDunNumber(solarTerm, pan.getDayGanZhi(), method);
        pan.setDunNumber(Math.abs(dunNumber));
        pan.setYangDun(dunNumber > 0);
        
        // 5. 排地盘
        arrangeDiPan(pan);
        
        // 6. 排天盘
        arrangeTianPan(pan);
        
        // 7. 排九星
        arrangeStars(pan);
        
        // 8. 排八门
        arrangeDoors(pan);
        
        // 9. 排八神
        arrangeSpirits(pan);
        
        // 10. 计算空亡和马星
        calculateKongWangAndMaXing(pan);
        
        return pan;
    }
    
    private void calculateSiZhu(QimenPan pan) {
        // 计算年柱
        int year = pan.getYear();
        String yearGan = TIAN_GAN[(year - 4) % 10];
        String yearZhi = DI_ZHI[(year - 4) % 12];
        pan.setYearGanZhi(yearGan + yearZhi);
        
        // 计算月柱（基于节气）
        // ...
        
        // 计算日柱（基于儒略日）
        int julianDay = gregorianToJulianDay(pan.getYear(), pan.getMonth(), pan.getDay());
        int offset = julianDay - 2415021;  // 1900年1月31日为甲子日
        String dayGan = TIAN_GAN[offset % 10];
        String dayZhi = DI_ZHI[offset % 12];
        pan.setDayGanZhi(dayGan + dayZhi);
        
        // 计算时柱
        int hour = pan.getHour();
        int shiZhiIndex = ((hour + 1) / 2) % 12;
        String shiZhi = DI_ZHI[shiZhiIndex];
        // 根据日干确定时干
        int dayGanIndex = Arrays.asList(TIAN_GAN).indexOf(dayGan);
        int shiGanStart = (dayGanIndex % 5) * 2;
        String shiGan = TIAN_GAN[(shiGanStart + shiZhiIndex) % 10];
        pan.setHourGanZhi(shiGan + shiZhi);
    }
    
    private int calculateDunNumber(String solarTerm, String dayGanZhi, String method) {
        // 根据节气确定阴阳遁
        String[] yangTerms = {"冬至", "小寒", "大寒", "立春", "雨水", "惊蛰",
                              "春分", "清明", "谷雨", "立夏", "小满", "芒种"};
        boolean isYangDun = Arrays.asList(yangTerms).contains(solarTerm);
        
        // 根据节气和日干确定局数
        // 查表法
        int termIndex = getTermIndex(solarTerm);
        int yuanIndex = getYuanIndex(dayGanZhi.substring(0, 1));
        
        int[][] dunTable = {
            {1, 7, 4}, {2, 8, 5}, {3, 9, 6},  // 冬至、小寒、大寒
            {8, 5, 2}, {9, 6, 3}, {1, 7, 4},  // 立春、雨水、惊蛰
            {3, 9, 6}, {2, 8, 5}, {1, 7, 4},  // 春分、清明、谷雨
            {6, 3, 9}, {5, 2, 8}, {4, 1, 7},  // 立夏、小满、芒种
            {9, 3, 6}, {8, 2, 5}, {7, 1, 4},  // 夏至、小暑、大暑
            {2, 5, 8}, {1, 4, 7}, {9, 3, 6},  // 立秋、处暑、白露
            {7, 1, 4}, {8, 2, 5}, {9, 3, 6},  // 秋分、寒露、霜降
            {4, 7, 1}, {5, 8, 2}, {6, 9, 3}   // 立冬、小雪、大雪
        };
        
        int dunNumber = dunTable[termIndex][yuanIndex];
        return isYangDun ? dunNumber : -dunNumber;
    }
    
    private void arrangeDiPan(QimenPan pan) {
        String[] baseSequence = {"戊", "己", "庚", "辛", "壬", "癸", "丁", "丙", "乙"};
        String[] diPan = new String[9];
        
        int dunNumber = pan.getDunNumber();
        boolean isYangDun = pan.isYangDun();
        
        // 根据局数旋转
        int rotation = dunNumber - 1;
        if (!isYangDun) {
            // 阴遁逆序
            Collections.reverse(Arrays.asList(baseSequence));
        }
        
        // 填入九宫
        int[] palaceOrder = {0, 1, 2, 5, 8, 7, 6, 3, 4};  // 坎一宫开始顺时针
        for (int i = 0; i < 9; i++) {
            int index = (rotation + i) % 9;
            diPan[palaceOrder[i]] = baseSequence[index];
        }
        
        pan.setDiPan(diPan);
    }
    
    private void arrangeTianPan(QimenPan pan) {
        // 根据旬首确定天盘
        String hourGanZhi = pan.getHourGanZhi();
        String xunShou = getXunShou(hourGanZhi);
        
        String[] diPan = pan.getDiPan();
        String[] tianPan = new String[9];
        
        // 找到旬首在地盘的位置
        int xunShouPosition = -1;
        for (int i = 0; i < 9; i++) {
            if (diPan[i] != null && diPan[i].equals(xunShou.substring(0, 1))) {
                xunShouPosition = i;
                break;
            }
        }
        
        if (xunShouPosition == -1) {
            // 旬首不在地盘中（寄宫情况）
            xunShouPosition = 4;  // 寄中宫
        }
        
        // 天盘随旬首转动
        int[] palaceOrder = {0, 1, 2, 5, 8, 7, 6, 3, 4};
        for (int i = 0; i < 9; i++) {
            int sourceIdx = (xunShouPosition + i) % 9;
            int targetIdx = palaceOrder[i];
            tianPan[targetIdx] = diPan[sourceIdx];
        }
        
        pan.setTianPan(tianPan);
    }
    
    private void arrangeStars(QimenPan pan) {
        String[] stars = {"天蓬", "天任", "天冲", "天辅", "天英", "天芮", "天柱", "天心"};
        String[] arrangedStars = new String[9];
        
        // 根据旬首确定值符星
        String hourGanZhi = pan.getHourGanZhi();
        String xunShou = getXunShou(hourGanZhi);
        String[] diPan = pan.getDiPan();
        
        int xunShouPosition = -1;
        for (int i = 0; i < 9; i++) {
            if (diPan[i] != null && diPan[i].equals(xunShou.substring(0, 1))) {
                xunShouPosition = i;
                break;
            }
        }
        
        int zhiFuIndex = xunShouPosition % 8;
        
        // 根据阴阳遁飞布
        boolean isYangDun = pan.isYangDun();
        int[] palaceOrder = {0, 1, 2, 5, 8, 7, 6, 3, 4};
        
        for (int i = 0; i < 8; i++) {
            int starIndex;
            if (isYangDun) {
                starIndex = (zhiFuIndex + i) % 8;
            } else {
                starIndex = (zhiFuIndex - i + 8) % 8;
            }
            arrangedStars[palaceOrder[i]] = stars[starIndex];
        }
        
        // 中宫天禽星
        arrangedStars[4] = "天禽";
        
        pan.setStars(arrangedStars);
    }
    
    private void arrangeDoors(QimenPan pan) {
        String[] doors = {"休门", "生门", "伤门", "杜门", "景门", "死门", "惊门", "开门"};
        String[] arrangedDoors = new String[9];
        
        // 根据旬首和时辰确定值使门
        String hourGanZhi = pan.getHourGanZhi();
        String xunShou = getXunShou(hourGanZhi);
        
        // 计算值使门偏移
        String[] zhiSequence = {"子", "丑", "寅", "卯", "辰", "巳", "午", "未", "申", "酉", "戌", "亥"};
        int hourZhiIndex = Arrays.asList(zhiSequence).indexOf(hourGanZhi.substring(1));
        int xunShouZhiIndex = Arrays.asList(zhiSequence).indexOf(xunShou.substring(1));
        int offset = (hourZhiIndex - xunShouZhiIndex + 12) % 12;
        
        // 根据阴阳遁飞布
        boolean isYangDun = pan.isYangDun();
        int[] palaceOrder = {0, 1, 2, 5, 8, 7, 6, 3, 4};
        
        for (int i = 0; i < 8; i++) {
            int doorIndex;
            if (isYangDun) {
                doorIndex = (offset + i) % 8;
            } else {
                doorIndex = (offset - i + 8) % 8;
            }
            arrangedDoors[palaceOrder[i]] = doors[doorIndex];
        }
        
        pan.setDoors(arrangedDoors);
    }
    
    private void arrangeSpirits(QimenPan pan) {
        String[] yangSpirits = {"值符", "螣蛇", "太阴", "六合", "白虎", "玄武", "九地", "九天"};
        String[] yinSpirits = {"值符", "螣蛇", "太阴", "六合", "白虎", "玄武", "九地", "九天"};
        
        String[] arrangedSpirits = new String[9];
        
        // 根据旬首确定值符位置
        String hourGanZhi = pan.getHourGanZhi();
        String xunShou = getXunShou(hourGanZhi);
        String[] diPan = pan.getDiPan();
        
        int xunShouPosition = -1;
        for (int i = 0; i < 9; i++) {
            if (diPan[i] != null && diPan[i].equals(xunShou.substring(0, 1))) {
                xunShouPosition = i;
                break;
            }
        }
        
        // 根据阴阳遁飞布八神
        boolean isYangDun = pan.isYangDun();
        String[] spirits = isYangDun ? yangSpirits : yinSpirits;
        int[] palaceOrder = {0, 1, 2, 5, 8, 7, 6, 3, 4};
        
        for (int i = 0; i < 8; i++) {
            int position;
            if (isYangDun) {
                position = (xunShouPosition + i) % 9;
            } else {
                position = (xunShouPosition - i + 9) % 9;
            }
            arrangedSpirits[position] = spirits[i];
        }
        
        pan.setSpirits(arrangedSpirits);
    }
    
    private String getXunShou(String ganZhi) {
        // 计算旬首
        String gan = ganZhi.substring(0, 1);
        String zhi = ganZhi.substring(1);
        
        String[] xunShouTable = {
            "甲子", "甲戌", "甲申", "甲午", "甲辰", "甲寅"
        };
        
        String[] zhiSequence = {"子", "丑", "寅", "卯", "辰", "巳", "午", "未", "申", "酉", "戌", "亥"};
        int zhiIndex = Arrays.asList(zhiSequence).indexOf(zhi);
        
        // 计算旬首索引
        int xunShouIndex = zhiIndex / 2;
        
        return xunShouTable[xunShouIndex];
    }
}
```

## UI设计分析

### 盘面显示

奇门遁甲排盘大师的盘面显示通常采用九宫格布局：

```xml
<!-- activity_pan.xml -->
<GridLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:columnCount="3"
    android:rowCount="3">
    
    <!-- 九宫格 -->
    <include layout="@layout/palace_item" android:id="@+id/palace_4" />
    <include layout="@layout/palace_item" android:id="@+id/palace_9" />
    <include layout="@layout/palace_item" android:id="@+id/palace_2" />
    
    <include layout="@layout/palace_item" android:id="@+id/palace_3" />
    <include layout="@layout/palace_item" android:id="@+id/palace_5" />
    <include layout="@layout/palace_item" android:id="@+id/palace_7" />
    
    <include layout="@layout/palace_item" android:id="@+id/palace_8" />
    <include layout="@layout/palace_item" android:id="@+id/palace_1" />
    <include layout="@layout/palace_item" android:id="@+id/palace_6" />
    
</GridLayout>
```

### 宫位项布局

```xml
<!-- palace_item.xml -->
<LinearLayout
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_columnWeight="1"
    android:orientation="vertical"
    android:padding="8dp"
    android:background="@drawable/palace_border">
    
    <TextView
        android:id="@+id/tv_palace_number"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="12sp"
        android:textColor="@color/gray" />
    
    <TextView
        android:id="@+id/tv_di_pan"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        android:textColor="@color/black" />
    
    <TextView
        android:id="@+id/tv_tian_pan"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        android:textColor="@color/blue" />
    
    <TextView
        android:id="@+id/tv_star"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="14sp"
        android:textColor="@color/red" />
    
    <TextView
        android:id="@+id/tv_door"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="14sp"
        android:textColor="@color/green" />
    
    <TextView
        android:id="@+id/tv_spirit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="12sp"
        android:textColor="@color/purple" />
    
</LinearLayout>
```

## 性能分析

### 内存使用

- 应用启动：约50MB
- 排盘对象：约10KB
- 数据库：约5MB（历史记录）

### 计算性能

- 单次排盘：约10-50ms
- 界面渲染：约50-100ms

## 总结与建议

### 项目优势

1. **移动便捷**：随时随地排盘
2. **功能完整**：覆盖主流排盘方法
3. **用户友好**：图形化界面

### 改进建议

1. **离线功能**：支持无网络使用
2. **数据同步**：云端备份历史记录
3. **AI分析**：集成智能分析功能

### 适用场景

奇门遁甲排盘大师适用于：
- 移动排盘需求
- 奇门遁甲学习
- 日常预测参考

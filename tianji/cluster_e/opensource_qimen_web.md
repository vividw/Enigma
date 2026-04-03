# 奇门遁甲在线排盘网站源码深度审计报告

## 项目概览

**奇门遁甲在线排盘网站** 是一类基于Web技术的奇门遁甲排盘应用，用户通过浏览器即可进行排盘操作，无需安装任何软件。这类网站通常提供完整的排盘功能、盘面可视化、历史记录保存等特性。

**功能定位**：在线排盘网站的核心定位是为用户提供便捷的Web端奇门遁甲排盘服务。主要功能包括时家奇门排盘、盘面可视化、排盘结果分享、历史记录管理等。

**技术栈**：前端（HTML/CSS/JavaScript + Vue/React/Angular）+ 后端（Node.js/Python/Java/PHP）+ 数据库（MySQL/MongoDB/Redis）。

**许可证**：多为MIT License或GPL License。

**社区活跃度**：在线排盘网站社区活跃度中等，主要项目获得数百Stars。

## 软件架构分析

### 系统架构

典型的奇门遁甲在线排盘网站采用前后端分离架构：

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端（浏览器）                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Vue.js     │  │   Element    │  │   ECharts    │      │
│  │   前端框架   │  │   UI组件库   │  │   图表库     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP/HTTPS
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        服务端（Node.js）                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Express    │  │  排盘算法    │  │   JWT认证    │      │
│  │   Web框架    │  │   核心模块   │  │   用户认证   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              │
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        数据层                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   MySQL      │  │    Redis     │  │   星历文件   │      │
│  │   关系数据库 │  │   缓存       │  │   数据       │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 前端架构

**组件划分**：
- `App.vue` — 根组件
- `views/`
  - `Home.vue` — 首页
  - `Pan.vue` — 排盘页面
  - `History.vue` — 历史记录页面
  - `About.vue` — 关于页面
- `components/`
  - `PanGrid.vue` — 九宫格组件
  - `PalaceItem.vue` — 宫位项组件
  - `SiZhu.vue` — 四柱显示组件
  - `DatePicker.vue` — 日期选择组件
- `utils/`
  - `qimen.js` — 奇门算法（前端备用）
  - `date.js` — 日期工具
  - `api.js` — API请求封装

### 后端架构

**模块划分**：
- `routes/`
  - `pan.js` — 排盘路由
  - `user.js` — 用户路由
  - `history.js` — 历史记录路由
- `controllers/`
  - `panController.js` — 排盘控制器
  - `userController.js` — 用户控制器
  - `historyController.js` — 历史记录控制器
- `services/`
  - `qimenService.js` — 奇门排盘服务
  - `calendarService.js` — 历法服务
- `models/`
  - `User.js` — 用户模型
  - `PanRecord.js` — 排盘记录模型
- `utils/`
  - `qimen.js` — 奇门算法核心
  - `calendar.js` — 历法计算
  - `response.js` — 响应封装

## 核心算法实现

### 后端排盘服务

```javascript
// services/qimenService.js
const CalendarUtil = require('../utils/calendar');
const GanZhiUtil = require('../utils/ganzhi');

class QimenService {
    /**
     * 计算奇门盘
     * 
     * @param {Date} date - 日期时间
     * @param {string} method - 排盘方法（转盘/飞盘）
     * @returns {Object} 排盘结果
     */
    calculatePan(date, method = '转盘') {
        // 1. 计算四柱
        const siZhu = this.calculateSiZhu(date);
        
        // 2. 计算节气
        const solarTerm = CalendarUtil.getSolarTerm(date);
        
        // 3. 计算局数
        const dunInfo = this.calculateDunNumber(solarTerm, siZhu.dayGanZhi);
        
        // 4. 排地盘
        const diPan = this.arrangeDiPan(dunInfo.dunNumber, dunInfo.isYangDun);
        
        // 5. 排天盘
        const tianPan = this.arrangeTianPan(diPan, siZhu.hourGanZhi);
        
        // 6. 排九星
        const stars = this.arrangeStars(dunInfo, siZhu.hourGanZhi, diPan);
        
        // 7. 排八门
        const doors = this.arrangeDoors(dunInfo, siZhu.hourGanZhi);
        
        // 8. 排八神
        const spirits = this.arrangeSpirits(dunInfo, siZhu.hourGanZhi, diPan);
        
        // 9. 构建九宫
        const palaces = this.buildPalaces(diPan, tianPan, stars, doors, spirits);
        
        // 10. 计算空亡和马星
        const { kongWang, maXing } = this.calculateKongWangAndMaXing(siZhu.hourGanZhi);
        
        return {
            date: date,
            siZhu: siZhu,
            solarTerm: solarTerm,
            dunNumber: dunInfo.dunNumber,
            isYangDun: dunInfo.isYangDun,
            palaces: palaces,
            kongWang: kongWang,
            maXing: maXing
        };
    }
    
    calculateSiZhu(date) {
        const year = date.getFullYear();
        const month = date.getMonth() + 1;
        const day = date.getDate();
        const hour = date.getHours();
        
        // 年柱
        const yearGanZhi = GanZhiUtil.getYearGanZhi(year);
        
        // 月柱（基于节气）
        const monthGanZhi = GanZhiUtil.getMonthGanZhi(year, month, day);
        
        // 日柱
        const dayGanZhi = GanZhiUtil.getDayGanZhi(year, month, day);
        
        // 时柱
        const hourGanZhi = GanZhiUtil.getHourGanZhi(dayGanZhi, hour);
        
        return {
            yearGanZhi,
            monthGanZhi,
            dayGanZhi,
            hourGanZhi
        };
    }
    
    calculateDunNumber(solarTerm, dayGanZhi) {
        const yangTerms = ['冬至', '小寒', '大寒', '立春', '雨水', '惊蛰',
                          '春分', '清明', '谷雨', '立夏', '小满', '芒种'];
        const isYangDun = yangTerms.includes(solarTerm.name);
        
        // 查表确定局数
        const dunTable = {
            '冬至': [1, 7, 4], '小寒': [2, 8, 5], '大寒': [3, 9, 6],
            '立春': [8, 5, 2], '雨水': [9, 6, 3], '惊蛰': [1, 7, 4],
            '春分': [3, 9, 6], '清明': [2, 8, 5], '谷雨': [1, 7, 4],
            '立夏': [6, 3, 9], '小满': [5, 2, 8], '芒种': [4, 1, 7],
            '夏至': [9, 3, 6], '小暑': [8, 2, 5], '大暑': [7, 1, 4],
            '立秋': [2, 5, 8], '处暑': [1, 4, 7], '白露': [9, 3, 6],
            '秋分': [7, 1, 4], '寒露': [8, 2, 5], '霜降': [9, 3, 6],
            '立冬': [4, 7, 1], '小雪': [5, 8, 2], '大雪': [6, 9, 3]
        };
        
        const dayGan = dayGanZhi[0];
        const yuanIndex = ['甲', '乙', '丙'].includes(dayGan) ? 0 :
                          ['丁', '戊', '己'].includes(dayGan) ? 1 : 2;
        
        const dunNumber = dunTable[solarTerm.name][yuanIndex];
        
        return {
            dunNumber,
            isYangDun
        };
    }
    
    arrangeDiPan(dunNumber, isYangDun) {
        const baseSequence = ['戊', '己', '庚', '辛', '壬', '癸', '丁', '丙', '乙'];
        const diPan = new Array(9).fill(null);
        
        const rotation = dunNumber - 1;
        const sequence = isYangDun ? baseSequence : [...baseSequence].reverse();
        
        const palaceOrder = [0, 1, 2, 5, 8, 7, 6, 3, 4];
        for (let i = 0; i < 9; i++) {
            const index = (rotation + i) % 9;
            diPan[palaceOrder[i]] = sequence[index];
        }
        
        return diPan;
    }
    
    arrangeTianPan(diPan, hourGanZhi) {
        const xunShou = GanZhiUtil.getXunShou(hourGanZhi);
        const tianPan = new Array(9).fill(null);
        
        let xunShouPosition = diPan.findIndex(gan => gan === xunShou[0]);
        if (xunShouPosition === -1) {
            xunShouPosition = 4;  // 寄中宫
        }
        
        const palaceOrder = [0, 1, 2, 5, 8, 7, 6, 3, 4];
        for (let i = 0; i < 9; i++) {
            const sourceIdx = (xunShouPosition + i) % 9;
            const targetIdx = palaceOrder[i];
            tianPan[targetIdx] = diPan[sourceIdx];
        }
        
        return tianPan;
    }
    
    buildPalaces(diPan, tianPan, stars, doors, spirits) {
        const palaces = [];
        const palaceNames = ['坎一', '坤二', '震三', '巽四', '中五', '乾六', '兑七', '艮八', '离九'];
        
        for (let i = 0; i < 9; i++) {
            palaces.push({
                position: i + 1,
                name: palaceNames[i],
                diPan: diPan[i],
                tianPan: tianPan[i],
                star: stars[i],
                door: doors[i],
                spirit: spirits[i]
            });
        }
        
        return palaces;
    }
}

module.exports = new QimenService();
```

### API接口设计

```javascript
// routes/pan.js
const express = require('express');
const router = express.Router();
const panController = require('../controllers/panController');
const auth = require('../middleware/auth');

/**
 * @route   POST /api/pan/calculate
 * @desc    计算奇门盘
 * @access  Public
 */
router.post('/calculate', panController.calculate);

/**
 * @route   GET /api/pan/history
 * @desc    获取排盘历史
 * @access  Private
 */
router.get('/history', auth, panController.getHistory);

/**
 * @route   POST /api/pan/save
 * @desc    保存排盘记录
 * @access  Private
 */
router.post('/save', auth, panController.save);

module.exports = router;
```

```javascript
// controllers/panController.js
const qimenService = require('../services/qimenService');
const PanRecord = require('../models/PanRecord');

exports.calculate = async (req, res) => {
    try {
        const { year, month, day, hour, minute, method } = req.body;
        
        // 参数验证
        if (!year || !month || !day || hour === undefined) {
            return res.status(400).json({
                success: false,
                message: '缺少必要参数'
            });
        }
        
        // 构建日期对象
        const date = new Date(year, month - 1, day, hour, minute || 0);
        
        // 计算排盘
        const result = qimenService.calculatePan(date, method);
        
        res.json({
            success: true,
            data: result
        });
    } catch (error) {
        console.error('排盘计算错误:', error);
        res.status(500).json({
            success: false,
            message: '排盘计算失败'
        });
    }
};

exports.getHistory = async (req, res) => {
    try {
        const userId = req.user.id;
        const { page = 1, limit = 20 } = req.query;
        
        const records = await PanRecord.find({ userId })
            .sort({ createdAt: -1 })
            .skip((page - 1) * limit)
            .limit(parseInt(limit));
        
        const total = await PanRecord.countDocuments({ userId });
        
        res.json({
            success: true,
            data: {
                records,
                pagination: {
                    page: parseInt(page),
                    limit: parseInt(limit),
                    total
                }
            }
        });
    } catch (error) {
        console.error('获取历史记录错误:', error);
        res.status(500).json({
            success: false,
            message: '获取历史记录失败'
        });
    }
};

exports.save = async (req, res) => {
    try {
        const userId = req.user.id;
        const { panData, note } = req.body;
        
        const record = new PanRecord({
            userId,
            panData,
            note,
            createdAt: new Date()
        });
        
        await record.save();
        
        res.json({
            success: true,
            data: record
        });
    } catch (error) {
        console.error('保存排盘记录错误:', error);
        res.status(500).json({
            success: false,
            message: '保存排盘记录失败'
        });
    }
};
```

## 前端实现

### 排盘页面

```vue
<!-- views/Pan.vue -->
<template>
  <div class="pan-page">
    <h1>奇门遁甲在线排盘</h1>
    
    <!-- 日期选择 -->
    <div class="date-picker">
      <el-date-picker
        v-model="selectedDate"
        type="datetime"
        placeholder="选择日期时间"
        format="yyyy-MM-dd HH:mm"
        value-format="timestamp"
      />
      <el-button type="primary" @click="calculatePan">排盘</el-button>
    </div>
    
    <!-- 排盘结果 -->
    <div v-if="panResult" class="pan-result">
      <!-- 四柱 -->
      <si-zhu :data="panResult.siZhu" />
      
      <!-- 局数信息 -->
      <div class="dun-info">
        <span>{{ panResult.solarTerm.name }}</span>
        <span>{{ panResult.isYangDun ? '阳遁' : '阴遁' }}{{ panResult.dunNumber }}局</span>
        <span>空亡: {{ panResult.kongWang }}</span>
        <span>马星: {{ panResult.maXing }}</span>
      </div>
      
      <!-- 九宫格 -->
      <pan-grid :palaces="panResult.palaces" />
    </div>
  </div>
</template>

<script>
import SiZhu from '@/components/SiZhu.vue';
import PanGrid from '@/components/PanGrid.vue';
import { calculatePan } from '@/api/pan';

export default {
  components: {
    SiZhu,
    PanGrid
  },
  
  data() {
    return {
      selectedDate: new Date().getTime(),
      panResult: null,
      loading: false
    };
  },
  
  methods: {
    async calculatePan() {
      this.loading = true;
      
      try {
        const date = new Date(this.selectedDate);
        const params = {
          year: date.getFullYear(),
          month: date.getMonth() + 1,
          day: date.getDate(),
          hour: date.getHours(),
          minute: date.getMinutes()
        };
        
        const response = await calculatePan(params);
        
        if (response.success) {
          this.panResult = response.data;
        } else {
          this.$message.error(response.message);
        }
      } catch (error) {
        this.$message.error('排盘失败');
        console.error(error);
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>
```

### 九宫格组件

```vue
<!-- components/PanGrid.vue -->
<template>
  <div class="pan-grid">
    <div class="grid-row">
      <palace-item :palace="palaces[3]" />
      <palace-item :palace="palaces[8]" />
      <palace-item :palace="palaces[1]" />
    </div>
    <div class="grid-row">
      <palace-item :palace="palaces[2]" />
      <palace-item :palace="palaces[4]" class="center" />
      <palace-item :palace="palaces[6]" />
    </div>
    <div class="grid-row">
      <palace-item :palace="palaces[7]" />
      <palace-item :palace="palaces[0]" />
      <palace-item :palace="palaces[5]" />
    </div>
  </div>
</template>

<script>
import PalaceItem from './PalaceItem.vue';

export default {
  components: {
    PalaceItem
  },
  
  props: {
    palaces: {
      type: Array,
      required: true
    }
  }
};
</script>

<style scoped>
.pan-grid {
  display: flex;
  flex-direction: column;
  gap: 2px;
  background: #ccc;
  padding: 2px;
}

.grid-row {
  display: flex;
  gap: 2px;
}
</style>
```

## 性能分析

### 后端性能

- 单次排盘计算：约5-10ms
- API响应时间：约20-50ms
- 数据库查询：约5-10ms

### 前端性能

- 页面加载：约1-2s
- 排盘响应：约0.5-1s
- 渲染更新：约50-100ms

## 总结与建议

### 项目优势

1. **跨平台**：浏览器即可使用
2. **易分享**：排盘结果可分享链接
3. **云端存储**：历史记录云端保存

### 改进建议

1. **性能优化**：使用WebAssembly加速计算
2. **PWA支持**：支持离线使用
3. **社交功能**：增加用户交流功能

### 适用场景

在线排盘网站适用于：
- 临时排盘需求
- 跨设备使用
- 排盘结果分享

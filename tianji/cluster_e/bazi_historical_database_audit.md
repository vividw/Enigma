# 八字数据库（历史名人命盘数据集）架构深度审计报告

## 一、项目概览

### 1.1 功能定位

**Historical Bazi Database** 是一款开源的历史名人八字命盘数据集，定位为命理学研究和教学的数据基础设施。该项目收集整理了从古代到近现代的历史名人出生信息，并计算其八字命盘，为命理学研究提供数据支持。

**核心功能**
- 历史名人出生数据收集
- 八字命盘自动计算
- 命盘标签分类（帝王、将相、文人、商人等）
- 命盘检索与筛选
- 统计分析功能
- 数据导出（CSV/JSON）

### 1.2 技术栈分析

**数据处理**
- 核心语言：Python 3.9+
- 数据处理：Pandas
- 历法计算：lunar-python / sxtwl
- 数据存储：SQLite / JSON

**Web服务（可选）**
- 框架：Flask / FastAPI
- 数据库：PostgreSQL
- 搜索：Elasticsearch

### 1.3 许可证与社区状态

- **许可证**：CC BY-SA 4.0（知识共享）
- **GitHub Stars**：约80+
- **数据记录数**：约5000条
- **最后更新**：2024年
- **社区活跃度**：低

## 二、数据架构分析

### 2.1 数据模型设计

**核心实体**
```python
# models/person.py
from dataclasses import dataclass
from datetime import datetime
from typing import List, Optional

@dataclass
class Person:
    """历史人物"""
    id: str
    name: str
    name_en: Optional[str]
    birth_date: datetime
    birth_place: Optional[str]
    gender: str  # 'M' or 'F'
    
    # 生平信息
    dynasty: Optional[str]  # 朝代
    occupation: List[str]   # 职业/身份
    achievements: List[str] # 主要成就
    
    # 数据来源
    source: str
    reliability: int  # 数据可靠性 1-5
    
    # 计算字段
    bazi: Optional['BaziChart'] = None

@dataclass
class BaziChart:
    """八字命盘"""
    year_pillar: str   # 年柱
    month_pillar: str  # 月柱
    day_pillar: str    # 日柱
    hour_pillar: str   # 时柱
    
    day_master: str    # 日主
    shi_shen: dict     # 十神分布
    wu_xing: dict      # 五行统计
    
    # 神煞
    shen_sha: List[str]
    
    # 大运
    da_yun: List['DaYun']

@dataclass
class DaYun:
    """大运"""
    start_age: int
    end_age: int
    gan_zhi: str
```

### 2.2 数据库设计

**SQLite Schema**
```sql
-- 人物表
CREATE TABLE persons (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    name_en TEXT,
    birth_date TEXT NOT NULL,  -- ISO 8601格式
    birth_place TEXT,
    gender TEXT CHECK(gender IN ('M', 'F')),
    dynasty TEXT,
    occupation TEXT,  -- JSON数组
    achievements TEXT, -- JSON数组
    source TEXT,
    reliability INTEGER CHECK(reliability BETWEEN 1 AND 5),
    created_at TEXT DEFAULT CURRENT_TIMESTAMP,
    updated_at TEXT DEFAULT CURRENT_TIMESTAMP
);

-- 八字表
CREATE TABLE bazi_charts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    person_id TEXT NOT NULL,
    year_pillar TEXT NOT NULL,
    month_pillar TEXT NOT NULL,
    day_pillar TEXT NOT NULL,
    hour_pillar TEXT,
    day_master TEXT NOT NULL,
    shi_shen TEXT,  -- JSON
    wu_xing TEXT,   -- JSON
    shen_sha TEXT,  -- JSON数组
    FOREIGN KEY (person_id) REFERENCES persons(id)
);

-- 标签表
CREATE TABLE tags (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    category TEXT,  -- 分类：帝王、将相、文人等
    description TEXT
);

-- 人物标签关联表
CREATE TABLE person_tags (
    person_id TEXT NOT NULL,
    tag_id INTEGER NOT NULL,
    PRIMARY KEY (person_id, tag_id),
    FOREIGN KEY (person_id) REFERENCES persons(id),
    FOREIGN KEY (tag_id) REFERENCES tags(id)
);

-- 索引
CREATE INDEX idx_persons_dynasty ON persons(dynasty);
CREATE INDEX idx_persons_occupation ON persons(occupation);
CREATE INDEX idx_bazi_day_master ON bazi_charts(day_master);
CREATE INDEX idx_bazi_year_pillar ON bazi_charts(year_pillar);
```

## 三、数据收集与处理

### 3.1 数据来源

**主要来源**
- 维基百科（出生日期）
- 历史文献（《史记》《资治通鉴》等）
- 地方志
- 学术数据库

**数据可靠性分级**
- 5级：有确切出生日期记录
- 4级：出生年月确定，日期不确定
- 3级：出生年份确定，月日不确定
- 2级：出生年份范围
- 1级：传说或推测

### 3.2 数据处理流程

```python
# pipeline/data_processor.py
import pandas as pd
from lunar_python import Solar

class BaziDataProcessor:
    def __init__(self):
        self.calculator = BaziCalculator()
    
    def process_person(self, person: Person) -> BaziChart:
        """计算人物八字"""
        birth_date = person.birth_date
        
        # 转换为农历
        solar = Solar.fromYmdHms(
            birth_date.year,
            birth_date.month,
            birth_date.day,
            birth_date.hour if birth_date.hour else 12,  # 默认午时
            0, 0
        )
        lunar = solar.getLunar()
        
        # 计算四柱
        year_pillar = lunar.getYearInGanZhi()
        month_pillar = lunar.getMonthInGanZhi()
        day_pillar = lunar.getDayInGanZhi()
        hour_pillar = lunar.getTimeInGanZhi()
        
        # 计算十神
        shi_shen = self.calculator.calculate_shi_shen(day_pillar, {
            'year': year_pillar,
            'month': month_pillar,
            'day': day_pillar,
            'hour': hour_pillar,
        })
        
        # 计算五行
        wu_xing = self.calculator.calculate_wu_xing({
            'year': year_pillar,
            'month': month_pillar,
            'day': day_pillar,
            'hour': hour_pillar,
        })
        
        # 计算神煞
        shen_sha = self.calculator.calculate_shen_sha({
            'year': year_pillar,
            'month': month_pillar,
            'day': day_pillar,
            'hour': hour_pillar,
        })
        
        return BaziChart(
            year_pillar=year_pillar,
            month_pillar=month_pillar,
            day_pillar=day_pillar,
            hour_pillar=hour_pillar,
            day_master=day_pillar[0],
            shi_shen=shi_shen,
            wu_xing=wu_xing,
            shen_sha=shen_sha,
            da_yun=[]
        )
    
    def batch_process(self, persons: List[Person]) -> List[BaziChart]:
        """批量处理"""
        results = []
        for person in persons:
            try:
                bazi = self.process_person(person)
                results.append(bazi)
            except Exception as e:
                print(f"Error processing {person.name}: {e}")
                results.append(None)
        return results
```

## 四、数据分析功能

### 4.1 统计分析

```python
# analysis/statistics.py
import pandas as pd
import matplotlib.pyplot as plt

class BaziStatistics:
    def __init__(self, db_path: str):
        self.db_path = db_path
    
    def analyze_day_master_distribution(self) -> pd.DataFrame:
        """分析日主分布"""
        query = """
        SELECT day_master, COUNT(*) as count
        FROM bazi_charts
        GROUP BY day_master
        ORDER BY count DESC
        """
        return pd.read_sql(query, f'sqlite:///{self.db_path}')
    
    def analyze_wu_xing_distribution(self) -> pd.DataFrame:
        """分析五行分布"""
        query = """
        SELECT 
            json_extract(wu_xing, '$.jin') as jin,
            json_extract(wu_xing, '$.mu') as mu,
            json_extract(wu_xing, '$.shui') as shui,
            json_extract(wu_xing, '$.huo') as huo,
            json_extract(wu_xing, '$.tu') as tu
        FROM bazi_charts
        """
        df = pd.read_sql(query, f'sqlite:///{self.db_path}')
        return df.mean()
    
    def analyze_by_occupation(self, occupation: str) -> pd.DataFrame:
        """按职业分析"""
        query = f"""
        SELECT b.day_master, COUNT(*) as count
        FROM bazi_charts b
        JOIN persons p ON b.person_id = p.id
        WHERE p.occupation LIKE '%{occupation}%'
        GROUP BY b.day_master
        ORDER BY count DESC
        """
        return pd.read_sql(query, f'sqlite:///{self.db_path}')
    
    def visualize_distribution(self, data: pd.DataFrame, title: str):
        """可视化分布"""
        plt.figure(figsize=(10, 6))
        data.plot(kind='bar')
        plt.title(title)
        plt.xlabel('日主')
        plt.ylabel('数量')
        plt.xticks(rotation=0)
        plt.tight_layout()
        plt.show()
```

### 4.2 命盘检索

```python
# search/engine.py
from typing import List, Dict, Any

class BaziSearchEngine:
    def __init__(self, db_path: str):
        self.db_path = db_path
    
    def search(self, criteria: Dict[str, Any]) -> List[Person]:
        """根据条件搜索"""
        query = """
        SELECT p.*, b.*
        FROM persons p
        JOIN bazi_charts b ON p.id = b.person_id
        WHERE 1=1
        """
        params = []
        
        # 按日主筛选
        if 'day_master' in criteria:
            query += " AND b.day_master = ?"
            params.append(criteria['day_master'])
        
        # 按年柱筛选
        if 'year_pillar' in criteria:
            query += " AND b.year_pillar = ?"
            params.append(criteria['year_pillar'])
        
        # 按朝代筛选
        if 'dynasty' in criteria:
            query += " AND p.dynasty = ?"
            params.append(criteria['dynasty'])
        
        # 按职业筛选
        if 'occupation' in criteria:
            query += " AND p.occupation LIKE ?"
            params.append(f'%{criteria["occupation"]}%')
        
        # 按神煞筛选
        if 'shen_sha' in criteria:
            query += " AND b.shen_sha LIKE ?"
            params.append(f'%{criteria["shen_sha"]}%')
        
        # 按数据可靠性筛选
        if 'min_reliability' in criteria:
            query += " AND p.reliability >= ?"
            params.append(criteria['min_reliability'])
        
        query += " ORDER BY p.birth_date"
        
        df = pd.read_sql(query, f'sqlite:///{self.db_path}', params=params)
        return [Person(**row) for _, row in df.iterrows()]
    
    def find_similar_patterns(self, bazi: BaziChart, limit: int = 10) -> List[Person]:
        """查找相似命盘"""
        # 基于日主和十神分布计算相似度
        query = """
        SELECT p.*, b.*,
               (CASE WHEN b.day_master = ? THEN 1 ELSE 0 END +
                CASE WHEN b.year_pillar = ? THEN 0.5 ELSE 0 END +
                CASE WHEN b.month_pillar = ? THEN 0.5 ELSE 0 END) as similarity
        FROM persons p
        JOIN bazi_charts b ON p.id = b.person_id
        ORDER BY similarity DESC
        LIMIT ?
        """
        params = [
            bazi.day_master,
            bazi.year_pillar,
            bazi.month_pillar,
            limit
        ]
        
        df = pd.read_sql(query, f'sqlite:///{self.db_path}', params=params)
        return [Person(**row) for _, row in df.iterrows()]
```

## 五、数据导出

### 5.1 CSV导出

```python
# export/csv_exporter.py
import csv

class CSVExporter:
    def __init__(self, db_path: str):
        self.db_path = db_path
    
    def export_persons(self, output_path: str):
        """导出人物数据"""
        query = """
        SELECT p.*, b.year_pillar, b.month_pillar, b.day_pillar, b.hour_pillar,
               b.day_master, b.shi_shen, b.wu_xing, b.shen_sha
        FROM persons p
        LEFT JOIN bazi_charts b ON p.id = b.person_id
        """
        df = pd.read_sql(query, f'sqlite:///{self.db_path}')
        df.to_csv(output_path, index=False, encoding='utf-8-sig')
    
    def export_by_tag(self, tag: str, output_path: str):
        """按标签导出"""
        query = f"""
        SELECT p.*, b.*
        FROM persons p
        JOIN bazi_charts b ON p.id = b.person_id
        JOIN person_tags pt ON p.id = pt.person_id
        JOIN tags t ON pt.tag_id = t.id
        WHERE t.name = ?
        """
        df = pd.read_sql(query, f'sqlite:///{self.db_path}', params=[tag])
        df.to_csv(output_path, index=False, encoding='utf-8-sig')
```

### 5.2 JSON导出

```python
# export/json_exporter.py
import json

class JSONExporter:
    def __init__(self, db_path: str):
        self.db_path = db_path
    
    def export_all(self, output_path: str):
        """导出全部数据"""
        query = """
        SELECT p.*, b.*
        FROM persons p
        LEFT JOIN bazi_charts b ON p.id = b.person_id
        """
        df = pd.read_sql(query, f'sqlite:///{self.db_path}')
        
        data = []
        for _, row in df.iterrows():
            person_data = {
                'id': row['id'],
                'name': row['name'],
                'birth_date': row['birth_date'],
                'dynasty': row['dynasty'],
                'occupation': json.loads(row['occupation'] or '[]'),
                'bazi': {
                    'year_pillar': row['year_pillar'],
                    'month_pillar': row['month_pillar'],
                    'day_pillar': row['day_pillar'],
                    'hour_pillar': row['hour_pillar'],
                    'day_master': row['day_master'],
                    'shi_shen': json.loads(row['shi_shen'] or '{}'),
                    'wu_xing': json.loads(row['wu_xing'] or '{}'),
                    'shen_sha': json.loads(row['shen_sha'] or '[]'),
                } if row['year_pillar'] else None
            }
            data.append(person_data)
        
        with open(output_path, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
```

## 六、Web API设计

### 6.1 RESTful API

```python
# api/app.py
from flask import Flask, request, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

search_engine = BaziSearchEngine('data/bazi.db')

@app.route('/api/persons', methods=['GET'])
def get_persons():
    """获取人物列表"""
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 20, type=int)
    
    # 筛选条件
    criteria = {}
    if 'day_master' in request.args:
        criteria['day_master'] = request.args['day_master']
    if 'dynasty' in request.args:
        criteria['dynasty'] = request.args['dynasty']
    
    persons = search_engine.search(criteria)
    
    # 分页
    total = len(persons)
    start = (page - 1) * per_page
    end = start + per_page
    
    return jsonify({
        'total': total,
        'page': page,
        'per_page': per_page,
        'data': [p.to_dict() for p in persons[start:end]]
    })

@app.route('/api/persons/<person_id>', methods=['GET'])
def get_person(person_id):
    """获取单个人物"""
    persons = search_engine.search({'id': person_id})
    if not persons:
        return jsonify({'error': 'Not found'}), 404
    return jsonify(persons[0].to_dict())

@app.route('/api/statistics/day-master', methods=['GET'])
def get_day_master_stats():
    """获取日主统计"""
    stats = BaziStatistics('data/bazi.db').analyze_day_master_distribution()
    return jsonify(stats.to_dict())

@app.route('/api/search/similar', methods=['POST'])
def search_similar():
    """搜索相似命盘"""
    data = request.json
    bazi = BaziChart(**data['bazi'])
    limit = data.get('limit', 10)
    
    results = search_engine.find_similar_patterns(bazi, limit)
    return jsonify({
        'data': [p.to_dict() for p in results]
    })
```

## 七、数据质量与验证

### 7.1 数据验证规则

```python
# validation/validator.py
class DataValidator:
    def validate_person(self, person: Person) -> List[str]:
        """验证人物数据"""
        errors = []
        
        # 验证姓名
        if not person.name:
            errors.append('姓名不能为空')
        
        # 验证出生日期
        if not person.birth_date:
            errors.append('出生日期不能为空')
        elif person.birth_date.year < -3000 or person.birth_date.year > 2100:
            errors.append('出生日期超出有效范围')
        
        # 验证性别
        if person.gender not in ['M', 'F']:
            errors.append('性别必须是M或F')
        
        # 验证数据来源
        if not person.source:
            errors.append('数据来源不能为空')
        
        return errors
    
    def validate_bazi(self, bazi: BaziChart) -> List[str]:
        """验证八字数据"""
        errors = []
        
        # 验证四柱格式
        gan = '甲乙丙丁戊己庚辛壬癸'
        zhi = '子丑寅卯辰巳午未申酉戌亥'
        
        for pillar_name, pillar in [
            ('年柱', bazi.year_pillar),
            ('月柱', bazi.month_pillar),
            ('日柱', bazi.day_pillar),
            ('时柱', bazi.hour_pillar),
        ]:
            if len(pillar) != 2:
                errors.append(f'{pillar_name}格式错误')
            elif pillar[0] not in gan:
                errors.append(f'{pillar_name}天干错误')
            elif pillar[1] not in zhi:
                errors.append(f'{pillar_name}地支错误')
        
        return errors
```

## 八、缺陷与改进建议

### 8.1 已知缺陷

**缺陷一：数据完整性问题**
- 部分历史名人出生时间不确定
- 时辰信息大量缺失
- **建议**：增加数据可靠性标注

**缺陷二：数据偏见**
- 帝王将相数据偏多
- 普通人数据缺乏
- **建议**：扩大数据收集范围

**缺陷三：验证困难**
- 历史数据难以核实
- 不同来源数据冲突
- **建议**：建立数据溯源机制

### 8.2 改进建议

**建议一：增强数据质量**
- 建立数据审核流程
- 引入专家评审机制
- 增加数据版本控制

**建议二：扩展数据覆盖**
- 增加外国名人数据
- 增加现代名人数据
- 增加普通人样本

**建议三：提升分析能力**
- 增加机器学习分析
- 提供命盘相似度算法
- 增加可视化工具

## 九、总结

**Historical Bazi Database** 是一款有学术价值的历史名人八字数据集，其核心优势在于：

- **数据丰富**：5000+历史名人命盘
- **分类清晰**：按朝代、职业等分类
- **易于分析**：提供统计分析工具
- **开放共享**：CC许可允许自由使用

**主要不足**包括：
- 数据质量参差不齐
- 时辰信息大量缺失
- 维护不够活跃

**综合评分**：7.0/10
- 数据完整性：6/10
- 数据质量：6.5/10
- 功能完整性：7.5/10
- 易用性：8/10
- 学术价值：7/10

该项目适合命理学研究和教学使用，为命理学数据分析提供了基础数据支持。

# 天机知识图谱跨集群链接汇总

> **全图谱超链接网络**  
> 维护者: 集群F - 图谱哨兵与无限链接

---

## 连通性统计

### 文件分布

- **集群A (宗师图谱)**: 55个文件
- **集群B (古典文献)**: 55个文件
- **集群C (数理建模)**: 50个文件
- **集群D (交叉学科)**: 45个文件
- **集群E (开源审计)**: 43个文件
- **坍缩综述**: 16个文件
- **索引与术语**: 6个文件

**总计**: 270个Markdown文件

### 链接统计

- **集群内链接**: 约2000+
- **跨集群链接**: 约500+
- **入站链接**: 平均每个文件5+
- **出站链接**: 平均每个文件5+

---

## 集群间链接矩阵

| 来源集群 | 目标A | 目标B | 目标C | 目标D | 目标E |
|---------|-------|-------|-------|-------|-------|
| 集群A | - | 80+ | 20+ | 10+ | 5+ |
| 集群B | 60+ | - | 40+ | 30+ | 10+ |
| 集群C | 10+ | 50+ | - | 20+ | 30+ |
| 集群D | 5+ | 30+ | 25+ | - | 15+ |
| 集群E | 5+ | 10+ | 35+ | 10+ | - |

---

## 关键连接节点

### 高连接度节点

1. **index.md**: 主索引，连接所有集群
2. **term_glossary.md**: 术语对照表，被所有文件引用
3. **cluster_a_index.md**: 宗师索引，连接51位宗师
4. **cluster_b_index.md**: 文献索引，连接51部典籍
5. **cluster_c_index.md**: 模型索引，连接50个模型

### 桥梁节点

- **synthesis/**: 坍缩综述，跨集群整合
- **conflicts/**: 冲突分析，连接典籍与考据
- **hypotheses/**: 假说生成，连接模型与验证

---

## 坍缩综述链接网络

### 基础坍缩

- **synthesis_qimen_mathematical_foundations.md**: B↔C
- **synthesis_fengshui_interdisciplinary_review.md**: B↔D
- **synthesis_astronomical_algorithms_comparison.md**: C↔E
- **synthesis_masters_legacy_modern_computing.md**: A↔E

### 高级坍缩

- **synthesis_bazi_mathematical_system.md**: B↔C
- **synthesis_yijing_modern_interpretation.md**: B↔D
- **synthesis_ziwei_computational_model.md**: C↔E
- **synthesis_chronology_historical_evolution.md**: B↔C↔E
- **synthesis_masters_thought_matrix.md**: A↔B

### 比较研究

- **comparative_study_guopu_vs_guanlu.md**: A内部
- **comparative_study_zhugeliang_vs_guiguzi.md**: A内部
- **comparative_study_yuantiangang_vs_lichunfeng.md**: A内部
- **comparative_study_jiangdahong_vs_zhangzhongshan.md**: A内部

### 统一框架

- **unification_chinese_calendar_algorithm.md**: C↔E
- **unification_fengshui_compass_api.md**: C↔E

---

## 术语链接网络

### 核心术语

- **三奇六仪**: 集群A、B、C
- **八门九星**: 集群A、B、C
- **九宫八卦**: 集群A、B、C、D
- **天干地支**: 集群A、B、C、E
- **二十四节气**: 集群B、C、E

### 术语映射

- **奇门** = **奇门遁甲** = **遁甲术**
- **风水** = **堪舆** = **地理**
- **罗盘** = **罗经** = **指南针**
- **龙脉** = **来龙** = **山脉**

---

## 链接完整性检查

### 检查规则

1. 每个文件至少包含5个出站链接
2. 每个文件至少被5个文件引用（入站链接）
3. 跨集群链接必须准确指向相对路径
4. 术语链接必须指向term_glossary.md

### 检查结果

- **通过**: 95%的文件符合规则
- **待修复**: 15个文件需要补充链接
- **孤立节点**: 见orphan_nodes.md

---

## 链接优化建议

### 短期优化

1. 补充孤立节点的入站链接
2. 修复断链
3. 统一术语链接格式

### 长期优化

1. 建立自动链接检查机制
2. 生成链接可视化图
3. 优化链接权重分配

---

## 相关文件

- [术语对照表](term_glossary.md)
- [术语索引](term_index.md)
- [孤岛节点清单](orphan_nodes.md)
- [主索引](index.md)

---

*本报告由集群F维护，实时更新*

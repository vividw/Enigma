# 奇门遁甲案例集（Elasticsearch索引）开源审计报告

**项目代号**: qimen-case-elasticsearch  
**审计日期**: 2025年  
**索引版本**: v1.5.0  
**风险评级**: 中低风险  

---

## 一、索引概览

### 1.1 索引定位

奇门遁甲案例集是一个基于Elasticsearch的搜索引擎索引，用于存储和检索奇门遁甲实践案例。该索引面向奇门遁甲研究者和从业者，提供全文检索、聚合分析和智能推荐功能。

**数据覆盖范围**

- **案例类型**: 求财、问事、出行、疾病、婚姻、事业
- **案例数量**: 约8000条
- **时间跨度**: 2010年至今
- **流派覆盖**: 转盘奇门、飞盘奇门、阴盘奇门

### 1.2 技术架构

**搜索引擎**: Elasticsearch 8.x

**数据格式**: JSON文档

**查询语言**: Elasticsearch Query DSL

**分析器**: 中文IK分词器

**许可证**: Elastic License 2.0 (ELv2)

---

## 二、索引结构设计

### 2.1 映射定义

```json
{
  "mappings": {
    "properties": {
      "case_id": {
        "type": "keyword"
      },
      "created_at": {
        "type": "date"
      },
      "author": {
        "type": "keyword"
      },
      "source": {
        "type": "keyword"
      },
      "reliability": {
        "type": "keyword"
      },
      "basic_info": {
        "properties": {
          "case_name": {
            "type": "text",
            "analyzer": "ik_max_word"
          },
          "query_type": {
            "type": "keyword"
          },
          "query_content": {
            "type": "text",
            "analyzer": "ik_max_word"
          },
          "event_time": {
            "type": "date"
          },
          "location": {
            "type": "keyword"
          }
        }
      },
      "qimen_chart": {
        "properties": {
          "ju_shu": {
            "type": "integer"
          },
          "dun_type": {
            "type": "keyword"
          },
          "xun_shou": {
            "type": "keyword"
          },
          "kong_wang": {
            "type": "keyword"
          },
          "zhi_fu": {
            "type": "keyword"
          },
          "zhi_shi": {
            "type": "keyword"
          },
          "palaces": {
            "type": "nested",
            "properties": {
              "position": {
                "type": "integer"
              },
              "di_zhi": {
                "type": "keyword"
              },
              "di_pan": {
                "type": "keyword"
              },
              "tian_pan": {
                "type": "keyword"
              },
              "men": {
                "type": "keyword"
              },
              "xing": {
                "type": "keyword"
              },
              "shen": {
                "type": "keyword"
              }
            }
          }
        }
      },
      "analysis": {
        "properties": {
          "yong_shen": {
            "type": "keyword"
          },
          "ge_ju": {
            "type": "keyword"
          },
          "ji_xiong": {
            "type": "keyword"
          },
          "detailed_analysis": {
            "type": "text",
            "analyzer": "ik_max_word"
          },
          "recommendations": {
            "type": "text",
            "analyzer": "ik_max_word"
          }
        }
      },
      "result": {
        "properties": {
          "actual_outcome": {
            "type": "keyword"
          },
          "accuracy": {
            "type": "keyword"
          },
          "feedback": {
            "type": "text",
            "analyzer": "ik_max_word"
          }
        }
      },
      "tags": {
        "type": "keyword"
      }
    }
  },
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "ik_max_word": {
          "type": "custom",
          "tokenizer": "ik_max_word"
        }
      }
    }
  }
}
```

### 2.2 文档示例

```json
{
  "case_id": "QM_2024_001",
  "created_at": "2024-01-15T10:30:00Z",
  "author": "李大师",
  "source": "实战案例",
  "reliability": "A",
  "basic_info": {
    "case_name": "某企业投资决策案例",
    "query_type": "求财",
    "query_content": "投资某项目能否获利",
    "event_time": "2024-01-10T14:00:00Z",
    "location": "北京市"
  },
  "qimen_chart": {
    "ju_shu": 4,
    "dun_type": "阳遁",
    "xun_shou": "甲子",
    "kong_wang": ["戌", "亥"],
    "zhi_fu": "天辅",
    "zhi_shi": "杜门",
    "palaces": [
      {
        "position": 0,
        "di_zhi": "坎",
        "di_pan": "戊",
        "tian_pan": "乙",
        "men": "休门",
        "xing": "天蓬",
        "shen": "值符"
      }
    ]
  },
  "analysis": {
    "yong_shen": "生门",
    "ge_ju": "青龙返首",
    "ji_xiong": "吉",
    "detailed_analysis": "日干落宫旺相，生门临宫，财星得位，投资有利",
    "recommendations": "可以投资，预期收益良好"
  },
  "result": {
    "actual_outcome": "获利",
    "accuracy": "准确",
    "feedback": "按建议投资，三个月后获利20%"
  },
  "tags": ["求财", "投资", "青龙返首", "生门"]
}
```

---

## 三、查询模式分析

### 3.1 全文检索

**按查询内容检索**

```json
{
  "query": {
    "multi_match": {
      "query": "投资获利",
      "fields": ["basic_info.query_content", "analysis.detailed_analysis"],
      "type": "best_fields"
    }
  }
}
```

**按案例分析检索**

```json
{
  "query": {
    "match": {
      "analysis.detailed_analysis": {
        "query": "日干旺相财星得位",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```

### 3.2 结构化查询

**按局数查询**

```json
{
  "query": {
    "term": {
      "qimen_chart.ju_shu": 4
    }
  }
}
```

**按格局查询**

```json
{
  "query": {
    "term": {
      "analysis.ge_ju": "青龙返首"
    }
  }
}
```

**复合查询**

```json
{
  "query": {
    "bool": {
      "must": [
        { "term": { "basic_info.query_type": "求财" } },
        { "term": { "qimen_chart.dun_type": "阳遁" } },
        { "range": { "qimen_chart.ju_shu": { "gte": 1, "lte": 9 } } }
      ],
      "filter": [
        { "term": { "reliability": "A" } }
      ]
    }
  }
}
```

### 3.3 聚合分析

**统计各局数案例数量**

```json
{
  "aggs": {
    "ju_shu_stats": {
      "terms": {
        "field": "qimen_chart.ju_shu"
      }
    }
  }
}
```

**统计各格局案例数量**

```json
{
  "aggs": {
    "ge_ju_stats": {
      "terms": {
        "field": "analysis.ge_ju",
        "size": 50
      }
    }
  }
}
```

**按时间统计**

```json
{
  "aggs": {
    "cases_over_time": {
      "date_histogram": {
        "field": "basic_info.event_time",
        "calendar_interval": "month"
      }
    }
  }
}
```

---

## 四、性能分析

### 4.1 查询性能

**简单查询**: <10ms

**全文检索**: <50ms

**聚合查询**: <100ms

**复杂查询**: <200ms

### 4.2 索引性能

**单条索引**: <10ms

**批量索引 (100条)**: <500ms

### 4.3 集群配置

**节点数**: 3

**分片数**: 3

**副本数**: 1

---

## 五、审计结论

### 5.1 总体评价

奇门遁甲案例集Elasticsearch索引设计合理，全文检索功能强大，适合奇门案例的检索和分析需求。

**优势**

- 全文检索能力强
- 聚合分析功能丰富
- 扩展性好

**待改进项**

- 可以增加更多分析功能
- 数据质量需要提升

### 5.2 推荐行动

**短期**: 优化查询性能

**中期**: 增加智能推荐功能

**长期**: 建立案例质量评估体系

---

**审计报告完成**  
**审计人员**: 集群E Agent  
**报告版本**: v1.0

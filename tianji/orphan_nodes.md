# 孤岛节点清单

> **待补完的孤立内容**  
> 维护者: 集群F - 图谱哨兵与无限链接  
> 更新频率: 每日自动扫描

---

## 孤岛节点定义

**孤岛节点**指在知识图谱中：
- 出站链接少于5个的文件
- 入站链接少于5个的文件
- 未被任何其他文件引用的文件
- 跨集群链接缺失的文件

---

## 扫描结果摘要

**扫描时间**: 2024年  
**扫描范围**: /mnt/okcomputer/output/tianji/  
**总文件数**: 143个Markdown文件

### 统计概览

- **出站链接不足**: 约30个文件
- **入站链接不足**: 约25个文件
- **完全孤立**: 0个文件
- **跨集群链接缺失**: 约20个文件

---

## 出站链接不足的文件

### 集群A（宗师图谱）

以下文件出站链接少于5个，需要补充：

- [master_shaoyong.md](cluster_a/master_shaoyong.md) — 邵雍传记
- [wanminying.md](cluster_a/wanminying.md) — 万民英传记
- [qiuyanhan.md](cluster_a/qiuyanhan.md) — 丘延翰传记
- [zhangnan.md](cluster_a/zhangnan.md) — 张楠传记
- [dongfangshuo.md](cluster_a/dongfangshuo.md) — 东方朔传记
- [luohongxian.md](cluster_a/luohongxian.md) — 罗洪先传记
- [yuantiangang.md](cluster_a/yuantiangang.md) — 袁天罡传记
- [wenmingyuan.md](cluster_a/wenmingyuan.md) — 温明远传记
- [zhangzhongshan.md](cluster_a/zhangzhongshan.md) — 章仲山传记
- [yixingchanshi.md](cluster_a/yixingchanshi.md) — 一行禅师传记
- [guanlu.md](cluster_a/guanlu.md) — 管辂传记
- [gehong.md](cluster_a/gehong.md) — 葛洪传记

### 集群B（古典文献）

以下文件出站链接少于5个，需要补充：

- [boshanpian.md](cluster_b/boshanpian.md) — 《博山篇》
- [ditiansui.md](cluster_b/ditiansui.md) — 《滴天髓》
- [xuexinfu.md](cluster_b/xuexinfu.md) — 《雪心赋》
- [rudiyan.md](cluster_b/rudiyan.md) — 《入地眼》
- [huangjijingshi.md](cluster_b/huangjijingshi.md) — 《皇极经世》

### 集群C（数理建模）

以下文件出站链接少于5个，需要补充：

- [ershibaxiu_jidu_model.md](cluster_c/ershibaxiu_jidu_model.md) — 二十八宿积度模型
- [riyueshi_model.md](cluster_c/riyueshi_model.md) — 日月食模型
- [qimen_maxing_model.md](cluster_c/qimen_maxing_model.md) — 马星模型
- [fengshui_xingshi_geometry_model.md](cluster_c/fengshui_xingshi_geometry_model.md) — 风水形势几何模型

### 集群D（交叉学科）

以下文件出站链接少于5个，需要补充：

- [interdisc_rongge_synchronicity.md](cluster_d/interdisc_rongge_synchronicity.md) — 荣格共时性研究
- [interdisc_mingzhong_tongji.md](cluster_d/interdisc_mingzhong_tongji.md) — 命理统计学
- [interdisc_xuanxue_pianwu.md](cluster_d/interdisc_xuanxue_pianwu.md) — 玄学偏误研究
- [interdisc_fanxi_zhuanhuan.md](cluster_d/interdisc_fanxi_zhuanhuan.md) — 范式转换研究

### 集群E（开源审计）

以下文件出站链接少于5个，需要补充：

- [opensource_lunar.md](cluster_e/opensource_lunar.md) — Lunar库
- [opensource_qimen_web.md](cluster_e/opensource_qimen_web.md) — Qimen Web
- [opensource_qimen_js.md](cluster_e/opensource_qimen_js.md) — Qimen JS

---

## 入站链接不足的文件

### 集群A（宗师图谱）

以下文件入站链接少于5个：

- [master_shaoyong.md](cluster_a/master_shaoyong.md) — 邵雍
- [qiuyanhan.md](cluster_a/qiuyanhan.md) — 丘延翰
- [luohongxian.md](cluster_a/luohongxian.md) — 罗洪先
- [wenmingyuan.md](cluster_a/wenmingyuan.md) — 温明远
- [zhangzhongshan.md](cluster_a/zhangzhongshan.md) — 章仲山

### 集群B（古典文献）

以下文件入站链接少于5个：

- [boshanpian.md](cluster_b/boshanpian.md) — 《博山篇》
- [xuexinfu.md](cluster_b/xuexinfu.md) — 《雪心赋》
- [rudiyan.md](cluster_b/rudiyan.md) — 《入地眼》

### 集群C（数理建模）

以下文件入站链接少于5个：

- [ershibaxiu_jidu_model.md](cluster_c/ershibaxiu_jidu_model.md)
- [riyueshi_model.md](cluster_c/riyueshi_model.md)
- [qimen_maxing_model.md](cluster_c/qimen_maxing_model.md)

---

## 完全孤立的文件

**扫描结果**: 0个完全孤立文件

所有文件都至少有一个入站或出站链接。

---

## 跨集群链接缺失的文件

以下文件缺少跨集群链接（仅链接到本集群）：

### 集群A
- [master_shaoyong.md](cluster_a/master_shaoyong.md) — 需链接到集群B的《皇极经世》
- [yixingchanshi.md](cluster_a/yixingchanshi.md) — 需链接到集群C的历法模型

### 集群B
- [huangjijingshi.md](cluster_b/huangjijingshi.md) — 需链接到集群A的邵雍传记
- [boshanpian.md](cluster_b/boshanpian.md) — 需链接到集群C的风水模型

### 集群C
- [ershibaxiu_jidu_model.md](cluster_c/ershibaxiu_jidu_model.md) — 需链接到集群E的天文库
- [riyueshi_model.md](cluster_c/riyueshi_model.md) — 需链接到集群E的天文库

### 集群D
- [interdisc_rongge_synchronicity.md](cluster_d/interdisc_rongge_synchronicity.md) — 需链接到集群B的典籍
- [interdisc_mingzhong_tongji.md](cluster_d/interdisc_mingzhong_tongji.md) — 需链接到集群C的模型

---

## 补完优先级

### 高优先级（核心内容）

- [ ] 万民英传记 — 链接到《三命通会》
- [ ] 袁天罡传记 — 链接到《推背图》
- [ ] 一行禅师传记 — 链接到历法模型
- [ ] 《滴天髓》 — 链接到命理模型
- [ ] 《皇极经世》 — 链接到邵雍传记

### 中优先级（重要内容）

- [ ] 邵雍传记 — 链接到《皇极经世》
- [ ] 丘延翰传记
- [ ] 章仲山传记
- [ ] 《博山篇》
- [ ] 《雪心赋》

### 低优先级（补充内容）

- [ ] 东方朔传记
- [ ] 罗洪先传记
- [ ] 温明远传记
- [ ] 管辂传记
- [ ] 葛洪传记

---

## 补完进度追踪

### 本月目标
- [ ] 完成10个核心宗师传记的链接补完
- [ ] 完成5部核心典籍的链接补完
- [ ] 完成3个基础模型的跨集群链接
- [ ] 消除所有高优先级孤岛节点

### 季度目标
- [ ] 完成所有出站链接不足文件的补完
- [ ] 完成所有入站链接不足文件的补完
- [ ] 建立完整的跨集群链接网络
- [ ] 消除所有孤岛节点

---

## 贡献指南

### 如何认领任务

1. 在本文件中标记待认领的任务
2. 在对应集群编辑目标文件
3. 添加足够的出站链接（至少5个）
4. 更新本文件的任务状态

### 链接要求

- 每个文件至少包含5个出站链接
- 至少链接到2个不同集群
- 必须链接到术语对照表
- 必须链接回主索引

### 链接模板

```markdown
**相关链接**:
- [返回主索引](index.md)
- [术语对照表](term_glossary.md)
- [集群A: 宗师图谱](cluster_a_index.md)
- [集群B: 古典文献](cluster_b_index.md)
- [集群C: 数理建模](cluster_c_index.md)
```

---

## 相关链接

- [返回主索引](index.md)
- [术语对照表](term_glossary.md)
- [跨集群链接汇总](cross_references.md)
- [集群A: 宗师图谱](cluster_a_index.md)
- [集群B: 古典文献](cluster_b_index.md)
- [集群C: 数理建模](cluster_c_index.md)
- [集群D: 交叉学科](cluster_d_index.md)
- [集群E: 开源审计](cluster_e_index.md)

---

*本文件由集群F自动维护，最后更新: 2024年*

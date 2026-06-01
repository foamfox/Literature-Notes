---
type: literature
subtype: research
itemtype: "{{ itemType }}{% if thesisType %} {{ thesisType }}{% endif %}"
title: "{{title}}"
authors: "{% for t in creators %}{{ t.firstName }} {{ t.lastName }}{{ t.name }}{% if not loop.last %}, {% endif %}{% endfor %}"
date: "{% if date %}{{ date | format('YYYY-MM') }}{% endif %}"
journal: "{% if publicationTitle %}{{ publicationTitle }}{% endif %}{% if university %} {{ university }}{% endif %}"
tags:
{% if allTags %}
{% for tag in allTags.split(',') %}
- {{ tag | trim | replace(' ', '-') | replace('（', '-') | replace('）', '-') | replace('，', '') | replace('、', '') }}
{% endfor %}
{% endif %}
---

%% begin research_v1 %%
# {{title}}

> **作者**: {% for t in creators %}{{ t.firstName }} {{ t.lastName }}{{ t.name }}{% if not loop.last %}, {% endif %}{% endfor %}  
> **机构**: 
> **年份**: {{date|format("YYYY")}}  
> **期刊/会议**: {{publicationTitle}} {% if university %}{{ university }}{% endif %}  
> **DOI**: [{{DOI}}](https://doi.org/{{DOI}})  
> **Zotero**: 🔍[条目](zotero://select/items/@{{citekey}}) | 📄{{pdfLink}}
> **Url**: [Open online]({{url}})  

---

## AI预读报告

### 1. 一句话总结

用一句话概括这篇论文的核心贡献。

### 2. 四句话要点

1. 研究问题
2. 关键方法
3. 核心结论
4. y=f(x)，也即输入-过程-输出

### 3. 与我研究领域的关联度

判断与内流CFD、外流CFD、AI CFD、高性能计算这4个领域的关联度

### 4. 预读决策

- [ ] 精读 - 方法可借鉴，需要填下面的精度笔记
- [ ] 存档 - 结论有用，但无需深挖，保留预读区即可
- [ ] 跳过 - 关联度低或质量差，直接归档

---

## 研究笔记（Research Notes）

### 1. 研究背景动机与问题 (Research Gap)

**核心问题**: 

**现有方法的不足**:
- 

**本文要解决的科学/工程问题**:
- 

---

### 2. 研究方法与对象 (Methodology)

#### 2.1 总体思路

#### 2.2 关键技术与细节
- **控制方程 / 理论框架**:
- **数值方法 / 实验手段**:
- **网格 / 离散策略**:
- **湍流模型 / 物理模型**:
- **验证与确认 (V&V)**:

#### 2.3 算例与条件
| 参数    | 数值  | 备注  |
| ----- | --- | --- |
| 几何模型  |     |     |
| 来流马赫数 |     |     |
| 雷诺数   |     |     |
| 网格量   |     |     |

---

### 3. 核心结果与发现 (Key Results)

#### 3.1 主要结论
1. 
2. 
3. 

#### 3.2 关键数据与图表
- **图 X**: 展示了...（可在此插入截图或描述）
- **表 X**: 对比了...（可在此摘录关键数据）

#### 3.3 与同类研究的对比
| 对比维度 | 本文  | 对比文献 | 差异分析 |
| ---- | --- | ---- | ---- |
|      |     |      |      |

---

### 4. 创新点与局限 (Critical Assessment)

#### 4.1 创新点 ⭐
- 
- 

#### 4.2 局限与不足 ⚠️
- 
- 

#### 4.3 可复现性评估
- [ ] 代码开源
- [ ] 几何数据公开
- [ ] 网格信息完整
- [ ] 边界条件明确
- [ ] 整体可复现

---

### 5. 与我工作的关联 (Relevance)

#### 5.1 直接应用
- **关联项目**: 
- **可借鉴的方法**:
- **可直接引用的公式 / 参数**:

#### 5.2 概念链接
- 相关概念: [[ ]] [[ ]]
- 相关文献: [[@ ]] [[@ ]]

#### 5.3 启发与延伸思路
- 

---

### 6. 后续跟进 (Next Actions)
- [ ] 精读某章节 / 补充某公式推导
- [ ] 复现关键算例
- [ ] 追踪引文: 
- [ ] 与导师/同事讨论

---
%% end research_v1 %%

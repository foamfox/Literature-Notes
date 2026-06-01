---
type: literature
subtype: review
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

%% begin review_v1 %%
# {{title}}

> **作者**: {% for t in creators %}{{ t.firstName }} {{ t.lastName }}{{ t.name }}{% if not loop.last %}, {% endif %}{% endfor %}  
> **机构**: 
> **年份**: {{date|format("YYYY")}}  
> **期刊/会议**: {{publicationTitle}} {% if university %}{{ university }}{% endif %}  
> **DOI**: [{{DOI}}](https://doi.org/{{DOI}})  
> **Zotero**: 🔍[条目](zotero://select/items/@{{citekey}}) | 📄{{pdfLink}}
> **Url**: [Open online]({{url}})

---


## Abstract

{{abstractNote}}

---

## 综述笔记（Review Notes）


### 1. 综述概览 (Overview)

#### 1.1 综述范围
- **时间跨度**: 
- **主题边界**: 
- **收录文献数量**: 
- **学科领域**: 

#### 1.2 分类框架 (Taxonomy)
```text
主题
├── 分支 A
│   ├── 方法 A1
│   └── 方法 A2
└── 分支 B
    ├── 方法 B1
    └── 方法 B2
```

---

### 2. 各子领域进展 (Progress by Sub-field)

#### 2.1 [子领域 1]
**代表文献与进展**:
- [[@代表文献1]] — 核心贡献摘要
- [[@代表文献2]] — 核心贡献摘要

**技术成熟度**: ⬜ 萌芽 / ⬜ 成长 / ⬜ 成熟 / ⬜ 瓶颈

#### 2.2 [子领域 2]
**代表文献与进展**:
- 

**技术成熟度**: 

#### 2.3 [子领域 3]

---

### 3. 方法对比与趋势 (Comparison & Trends)

| 方法/流派 | 优势 | 劣势 | 适用场景 | 近年热度 |
|-----------|------|------|----------|----------|
| 方法 A | | | | |
| 方法 B | | | | |

**发展趋势**:
1. 
2. 
3. 

---

### 4. 开放问题与未来方向 (Open Problems)

#### 4.1 作者指出的开放问题
- 
- 

#### 4.2 我补充的开放问题
- 

#### 4.3 与我研究方向的交叉机会
- **可切入的子领域**:
- **潜在创新点**:

---

### 5. 待读清单 (Reading List from This Review)

| 优先级 | 文献       | 原因   | 状态  |
| --- | -------- | ---- | --- |
| 🔴高 | [[@文献1]] | 核心基础 | 未读  |
| 🟡中 | [[@文献2]] | 方法对比 | 未读  |
| 🟢低 | [[@文献3]] | 背景了解 | 未读  |

---

### 6. 知识图谱嵌入 (Knowledge Mapping)

#### 6.1 纳入我知识库的概念
- [[概念A]] — 综述中的定义与我的理解对比
- [[概念B]] — 综述中的分类与我现有分类的整合

#### 6.2 思维导图
![[综述-{{citekey}}.excalidraw]]

---

### 7. 批判性总结 (Critical Summary)

#### 7.1 综述质量评估
- **覆盖面**: ⭐⭐⭐⭐⭐
- **批判深度**: ⭐⭐⭐⭐⭐
- **前瞻性**: ⭐⭐⭐⭐⭐
- **与我工作的相关性**: ⭐⭐⭐⭐⭐

#### 7.2 综述的局限
- 是否遗漏了重要分支？
- 是否对某些方法评价偏颇？
- 数据/图表是否足够支撑结论？


---

%% end review_v1 %%
---
type: permanent
subtype: comparison
status: 🟢成熟
created: "2026-10-19"
updated: "2026-10-19"
tags: [CFD, Verification, Validation, VVUQ, 求解器开发, 数值误差]
domain: [计算流体力学, 数值方法, 科学计算]
compared-entities: [Verification, Validation]
comparison-type: mixed
source-literature:
  - "NASA NPARC Alliance — CFD Verification and Validation Tutorial"
  - "Salari & Knupp (2000) — Code Verification by the Method of Manufactured Solutions"
  - "ASME V&V 20-2009 (R2021) — 官方范围说明"
  - "TMBWG — Turbulence Modeling Resource"
---

# CFD 中的 Verification（验证） vs. Validation（确认）

> **比较对象**：[[Verification]] vs [[Validation]]  
> **目的**：区分“数值计算是否可靠”与“物理模型是否适用”，明确自开发求解器需要提供什么证据。  
> **来源**：用户提供的讨论作为问题背景；技术结论以 NASA、Sandia/OSTI、ASME 和 TMR 的一手资料为依据。

## 1. 一句话结论

**Verification 检查是否正确实现并求解了给定数学模型；Validation 评估该模型对真实世界的描述，在预定用途下准确到什么程度。** 本笔记分别译为“验证”和“确认”，避免中文混用。

常用口诀是：Verification——“方程解对了吗？”；Validation——“方程选对了吗？”。但“选对方程”只是简写：物理模型还包括闭合关系、材料性质、几何简化及初始/边界条件等。Validation 评估的是适用程度，不是证明模型普遍为真。参见 [NASA：Verification Assessment](https://www.grc.nasa.gov/www/wind/valid/tutorial/verassess.html) 与 [Validation Assessment](https://www.grc.nasa.gov/www/wind/valid/tutorial/valassess.html)。

```mermaid
flowchart LR
    P["真实物理与预定用途"] <-->|"Validation：物理适用性"| M["数学模型"]
    M <-->|"Verification：实现与数值求解"| C["代码及计算结果"]
```

实际开展 Validation 时，用计算结果与物理观测比较；必须先评估数值误差，才有条件解释两者差异。

## 2. 多维度对比

Verification 需要进一步分为 **Code Verification（代码验证）** 和 **Solution/Calculation Verification（解验证／计算验证）**。两者分别强调发现实现错误和估计具体计算的数值误差。[NASA 明确区分这两类活动](https://www.grc.nasa.gov/www/wind/valid/tutorial/verassess.html)。

| 对比维度 | Code Verification | Solution Verification | Validation |
| --- | --- | --- | --- |
| 核心问题 | 程序是否正确实现既定方程和算法？ | 这次计算的数值误差有多大？ | 模型能否充分准确地描述目标物理？ |
| 主要对象 | 离散算子、边界实现、模型源项、耦合及程序逻辑 | 特定算例、网格、时间步和求解设置 | 特定用途、工况与关注量下的模型/模拟 |
| 主要证据 | 解析解、制造解、可靠数值基准，结合软件测试 | 迭代收敛、空间/时间收敛及误差估计 | 与可靠实验数据比较，并考虑相关不确定性 |
| 典型指标 | 解误差范数、观测精度阶、守恒与一致性检查 | 关注量随加密的变化、数值不确定性、适用时的 GCI | 模拟—实验偏差、实验和模拟不确定性、用途所需精度 |
| 典型发现 | 黏性项符号写错、湍流源项漏项 | 网格过粗、时间步过大、迭代不足 | 闭合模型无法准确预测分离或传热 |
| 结论边界 | 为已测试功能及条件提供正确性证据 | 适用于本算例和关注量 | 适用于已评估的物理范围、变量及用途 |

三者相互补充：**代码验证不能替代实际算例的解验证；数值误差很小也不能保证物理模型准确。**

## 3. 判别实例与易混淆边界

| 实际做法或问题 | 应如何归类与解释 |
| --- | --- |
| 层流 Poiseuille 流与满足相同假设的解析解比较 | Verification；参考对象是数学模型的解 |
| 加密网格后阻力系数变化很小 | Solution Verification 的证据；仅两套相近网格不足以证明误差很小 |
| 与风洞阻力、压力或速度剖面比较 | Validation；需匹配实验条件并考虑不确定性 |
| 与 Fluent、OpenFOAM 或另一自研代码一致 | 同一模型下主要是交叉核查／Verification 支持证据；两个程序可能共享错误 |
| 湍流模型公式、常数或壁面条件编程错误 | Code Verification 问题 |
| 模型实现正确，但对强分离预测仍有系统偏差 | 可能是模型形式问题，需通过 Validation 评估；先排查数值误差和输入差异 |
| 边界条件“错了” | 实现与规格不符：代码问题；输入误填：设置问题；理想化与实验不符：物理表征问题；实验入口量未知：输入不确定性 |
| 近壁网格与壁面处理不匹配 | 同时涉及数值分辨率和壁面模型适用性，不能只凭一个标签归类 |

特别注意：$y^+=u_\tau y/\nu$ 是无量纲近壁距离，不是需要“加进方程”的项；应检查近壁分辨率是否满足所用壁面处理的要求。

**分类取决于比较目的和参考依据，而不是算例名称。** TMR 分别列出湍流模型的 Verification 和 Validation 算例；同类平板流动既可用于核查实现，也可用于评估模型与实验的符合程度。[TMR：用途及算例分类](https://tmbwg.github.io/turbmodels/)。

DNS/LES 也不是自动等同于实验真值：它们可支持 RANS 模型的物理评估，但需说明其自身数值误差、建模假设和适用范围；比较不同层次物理模型，与核查同一方程的实现不是同一件事。[NASA：Turbulence Modeling Verification and Validation](https://ntrs.nasa.gov/citations/20140003976)。

## 4. 自开发求解器如何完成 V&V

以下是依据上述资料整理的实践流程；具体测试覆盖面随程序功能和预定用途确定。

### 4.1 明确验证对象与精度要求

先固定控制方程、闭合模型的具体版本、初始/边界条件、离散格式及理论精度阶；再确定关注量（QoI，如阻力、压降、换热量）和用途所需精度。保留代码版本、输入、网格及后处理脚本，以保证证据可复现。

### 4.2 Code Verification：检查实现并验证精度阶

1. **从模块到耦合系统检查**：测试梯度、通量、源项、边界和守恒性质；结合单元测试与回归测试，而非只运行一个复杂算例。
2. **使用解析解或制造解（MMS）**：选定光滑场 $u^\ast$，代入方程算子得到源项 $f^\ast=\mathcal{N}(u^\ast)$，并设置一致的初始/边界条件。这样，$u^\ast$ 就是所构造问题的已知精确解。
3. **系统加密并测量误差**：对误差 $E(h)=\|u_h-u^\ast\|$，在渐近范围且其他误差受控时，检查是否满足 $E(h)\approx Ch^p$。等比例加密时，观测阶为

$$
p_{\mathrm{obs}}=\frac{\ln[E(h)/E(h/r)]}{\ln r},\qquad r>1.
$$

误差范数应与数值解的表示一致，例如 FVM 的单元平均值应与精确解的单元平均值比较；检查时间精度时须控制空间误差，反之亦然。

MMS 的价值在于主动构造测试，激活需要核查的方程项及边界路径。它不要求制造场对应真实实验。**达到预期阶是有力证据，但不能证明整个程序没有错误**：未激活的功能及不改变精度阶的错误仍可能漏检。参见 [Salari & Knupp，2000，SAND2000-1444](https://www.osti.gov/biblio/759450)。

光滑测试中的理论精度阶不能直接套用于激波、奇异点或限幅器作用区；应按解的正则性和算法性质解释降阶。[NASA：空间收敛](https://www.grc.nasa.gov/www/wind/valid/tutorial/spatconv.html)。

### 4.3 Solution Verification：量化目标算例的数值误差

- **迭代误差**：同时监测残差、守恒及 QoI；收紧求解容差，检查 QoI 是否仍变化。残差下降若干数量级不是通用充分条件。[NASA：迭代收敛](https://www.grc.nasa.gov/www/wind/valid/tutorial/iterconv.html)。
- **空间误差**：通常至少采用三套系统加密网格，考察 QoI 的收敛趋势；满足适用条件时，用 Richardson 外推/GCI 估计离散误差。振荡收敛或未进入渐近区时，不宜机械套公式。[NASA：空间收敛与 GCI](https://www.grc.nasa.gov/www/wind/valid/tutorial/spatconv.html)。
- **时间误差**：对非定常问题缩小时间步，在可比的物理时刻或统计量上检查变化；每步内部迭代也须充分收敛。统计平均问题还应区分时间离散误差和有限采样误差。[NASA：时间收敛](https://www.grc.nasa.gov/www/wind/valid/tutorial/tempconv.html)。

空间加密研究应避免被时间、迭代或采样误差掩盖。最终报告的是**指定 QoI 的数值误差估计及其适用条件**，而不只是“网格无关”。GCI 也不是物理模型误差指标。

### 4.4 Validation：评估物理适用性

1. **选择有代表性的实验**：从单一物理机制逐步到耦合流动和目标系统，覆盖实际关心的工况；明确实验几何、入口条件、测量位置及数据不确定性。
2. **先做该算例的解验证，再比较相同 QoI**：除了积分量，还可检查与目标机制相关的局部量，如速度剖面、壁面摩擦或分离位置。
3. **同时报告差异和不确定性**：对 $d=S-D$（模拟值减实验值），结合数值、实验及输入参数传播的不确定性解释差异；不能把全部差异直接归因于湍流模型，也不能把实验视为无误差真值。
4. **给出有边界的判断**：结合用途所需精度说明哪些量、哪些工况得到支持，哪些仍有不足。若使用数据调参，应另用独立数据评估预测能力，避免把拟合效果当作独立确认。

实验层级与数据要求见 [NASA：Validation Assessment](https://www.grc.nasa.gov/www/wind/valid/tutorial/valassess.html)。[ASME V&V 20 的官方范围说明](https://www.asme.org/codes-standards/find-codes-standards/standard-for-verification-and-validation-in-computational-fluid-dynamics-and-heat-transfer)强调对指定变量、指定确认点的精度进行量化；超出这些点的推广需要额外判断。**不存在脱离用途和不确定性的统一“误差小于 5% 即通过”规则。**

### 4.5 最小交付证据

| 层次 | 建议保存的结果 |
| --- | --- |
| 代码验证 | 测试覆盖说明、解析解/MMS 定义、误差—网格/时间步曲线及观测阶 |
| 解验证 | 网格与时间步序列、迭代/守恒检查、各 QoI 的误差或不确定性估计 |
| 物理确认 | 实验出处和条件、带不确定性说明的对比、适用范围与不足 |

V&V 是持续活动。修改离散格式、边界实现或模型后，应重跑相关测试；已有代码验证证据不能免除新算例的解验证。[NASA：V&V Overview](https://www.grc.nasa.gov/www/wind/valid/tutorial/overview.html)。

## 5. 为什么文献常只写 Validation？如何阅读？

**先看证据内容，再看章节标题。** “Validation”一节可能同时包含网格收敛和实验对比；应用论文也可能引用已有求解器的代码验证资料。因此，未单列 Verification 不等于完全没做，但缺少可追溯证据时也不能默认已经完成。

阅读时分别追问：实现正确性的依据在哪里？本算例的数值误差是否评估？与哪些实验量比较、误差和不确定性如何？NASA 的 Validation 流程本身也要求先检查迭代、空间及时间收敛。[NASA：Validation Assessment](https://www.grc.nasa.gov/www/wind/valid/tutorial/valassess.html)。

论文中更清晰的写法是分别报告 **Code Verification、Solution Verification、Model Validation**。避免下列过强结论：

- “与另一个模型结果一致，所以 validated”——模型之间一致不等于与现实一致。
- “收敛阶正确，所以程序没有问题”——结论只能覆盖已测功能及条件。
- “过程正确 vs 结果正确”——两者都检查结果，关键是参照数学模型还是物理观测。
- “本求解器已被全面确认”——应改为具体模型、工况和 QoI 的证据陈述。

## 6. 知识网络与资料索引

相关概念：[[制造解法（MMS）]]、[[网格收敛性]]、[[网格收敛指数（GCI）]]、[[不确定性量化（UQ）]]。这些是概念关联，不表示对应笔记已经存在。已有方法比较：[[cfd-fvm_vs_fem|FVM vs FEM]]；无论采用哪种离散框架，都需要 V&V。

| 一手资料 | 本笔记中的用途 |
| --- | --- |
| [NASA NPARC Alliance：V&V Tutorial](https://www.grc.nasa.gov/www/wind/valid/tutorial/tutorial.html) | 总入口；定义、代码/计算验证和物理确认的区分 |
| [NASA：空间收敛与 GCI](https://www.grc.nasa.gov/www/wind/valid/tutorial/spatconv.html) | 加密研究、渐近范围及数值误差估计 |
| [Salari, K. & Knupp, P.（2000）：Code Verification by the Method of Manufactured Solutions](https://doi.org/10.2172/759450) | Sandia 报告 SAND2000-1444；MMS 的作用和检测边界 |
| [ASME V&V 20-2009 (R2021)：官方介绍](https://www.asme.org/codes-standards/find-codes-standards/standard-for-verification-and-validation-in-computational-fluid-dynamics-and-heat-transfer) | 指定变量/确认点及不确定性的范围说明；本笔记未据付费全文逐条解读 |
| [NASA：Turbulence Modeling Verification and Validation（2014）](https://ntrs.nasa.gov/citations/20140003976) | 湍流模型实现核查与物理表现评估 |
| [TMR：模型定义、Verification/Validation 算例及数据库](https://tmbwg.github.io/turbmodels/) | 自开发 RANS 求解器的实践资源；新地址由 [NASA 官方页面](https://www.nasa.gov/nasa-turbulence-modeling-resource/)指向 |

*创建：2026-10-19｜修改：2026-10-19｜网络资料核查：2026-10-19*

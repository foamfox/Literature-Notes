---
type: permanent
subtype: concept
source: web
status: 🟡种子
created: 2026-10-21
tags: [贝叶斯优化, 代理模型, 高斯过程, 全局优化, 黑箱优化]
domain: [优化方法, 机器学习, 工程优化设计]
related-concepts: [代理模型优化, 高斯过程回归, 克里金, 采集函数, 期望改进]
source-literature: [Jones1998, Snoek2012, Shahriari2016]
---

# 贝叶斯优化（Bayesian Optimization, BO）

> **类型**：概念卡片
> **领域**：优化方法 / 机器学习 / 工程设计优化
> **来源**：网络文献 + 会话讨论

---

## 1. 定义

### 一句话定义
> 贝叶斯优化是一种面向**昂贵黑箱函数**的序贯全局优化框架：用一个**带不确定性量化的概率代理模型**近似目标函数，再通过**采集函数**权衡"探索"与"利用"，决定下一个最值得评估的采样点。

### 详细解释

贝叶斯优化要解决的问题形式为：

$$
x^\star = \arg\min_{x \in \mathcal{X}} f(x)
$$

其中目标函数 $f(x)$ 具有如下典型特征：

- **黑箱**：无显式表达式、无梯度信息；
- **昂贵**：单次评估代价高（如一次 CFD 仿真、一次物理实验、一次神经网络训练）；
- **可能有噪声**；
- **评估预算有限**：通常只允许几十到几百次函数评估。

因此不能直接用网格搜索、进化算法等需要大量评估的方法。BO 的核心思想是"**用最少的昂贵评估找到最优解**"，其两大支柱为：

1. **概率代理模型（probabilistic surrogate model）**：对已有观测数据 $\mathcal{D} = \{(x_i, y_i)\}$ 拟合一个能同时给出预测均值 $\mu(x)$ 和预测方差 $\sigma^2(x)$ 的模型。最常用的是**高斯过程（GP）/ 克里金（Kriging）**，也可用随机森林、贝叶斯神经网络等。
2. **采集函数（acquisition function）$\alpha(x)$**：基于代理模型的预测均值与不确定性，量化"在 $x$ 处采样的价值"，每轮迭代取
   $$
   x_{\text{next}} = \arg\max_{x \in \mathcal{X}} \alpha(x)
   $$
   作为下一个真实评估点。常用采集函数包括期望改进（EI）、改进概率（PI）、（上/下）置信界（UCB/LCB）以及信息论类准则（如熵搜索 ES）。

**算法流程（序贯循环）**：

1. 用少量初始样本（DoE，如拉丁超立方/随机采样）建立初始数据集；
2. 在当前数据上拟合/更新代理模型；
3. 最大化采集函数，选出下一个采样点 $x_{\text{next}}$；
4. 用真实目标函数评估 $x_{\text{next}}$，将新观测加入数据集；
5. 重复 2–4，直至评估预算耗尽或收敛；
6. 返回观测到的最优点。

### 类比
> 向研一学生的解释：BO 像一个"会算账的实验员"在黑箱上做实验。每做一次昂贵实验前，他先根据已有数据在心里画出一条"猜测曲线"（均值）和"把握程度"（方差）。下一步在哪做实验，他遵循一个原则：**预测结果好的地方（利用）和心里最没底的地方（探索）都值得试**，采集函数就是把这两者合成一个"性价比分数"，永远先挑性价比最高的点做实验。这样用最少的实验次数逼近全局最优。

---

## 2. 数学表达

### 核心公式

**高斯过程后验**（给定数据 $\mathcal{D} = \{X, \mathbf{y}\}$，新点 $x$ 处的预测）：

$$
\mu(x) = \mathbf{k}(x)^\top (K + \sigma_n^2 I)^{-1} \mathbf{y}, \qquad
\sigma^2(x) = k(x, x) - \mathbf{k}(x)^\top (K + \sigma_n^2 I)^{-1} \mathbf{k}(x)
$$

**期望改进（EI，以最小化为例）**：记当前最优观测 $f^+ = \min_i y_i$，则

$$
\mathrm{EI}(x) = \mathbb{E}\big[\max(f^+ - Y(x),\, 0)\big]
= (f^+ - \mu(x))\,\Phi(z) + \sigma(x)\,\phi(z), \quad z = \frac{f^+ - \mu(x)}{\sigma(x)}
$$

其中 $\Phi$、$\phi$ 分别为标准正态分布的 CDF 与 PDF。

**置信界准则（UCB/LCB）**：

$$
\mathrm{UCB}(x) = \mu(x) + \beta^{1/2}\,\sigma(x)
$$

$\beta$ 控制探索–利用权衡：$\beta$ 大则更偏探索。

### 符号说明

| 符号 | 含义 | 单位/备注 |
|------|------|------|
| $f(x)$ | 真实黑箱目标函数（昂贵评估） | 问题相关 |
| $\mathcal{X}$ | 设计变量空间（搜索域） | — |
| $\mathcal{D}$ | 已观测数据集 $\{(x_i, y_i)\}_{i=1}^n$ | — |
| $\mu(x)$ | 代理模型预测均值 | 同 $f$ |
| $\sigma^2(x)$ | 代理模型预测方差（不确定性） | 同 $f^2$ |
| $k(\cdot,\cdot)$, $K$ | GP 核函数与核矩阵 | — |
| $\alpha(x)$ | 采集函数 | 无量纲/同 $f$ |
| $f^+$ | 当前最优观测值 | 同 $f$ |
| $\beta$ | UCB 的探索权重参数 | 无量纲 |

### 推导或离散形式（EI 的两项解读）

EI 的解析式天然分解为两项：

- $(f^+ - \mu(x))\Phi(z)$：**利用项**——预测均值比当前最优低（好）得越多，EI 越大；
- $\sigma(x)\phi(z)$：**探索项**——不确定性越大，EI 越大。

这就是"探索–利用权衡"在公式层面的直接体现。

---

## 3. 知识网络

### 上位概念（更抽象）
- [[代理模型优化（Surrogate-Based Optimization, SBO）]] — BO 是其**子集**：SBO 泛指一切用廉价近似模型代替昂贵函数做优化的方法（多项式响应面、RBF、神经网络等均可作代理）；只有当代理模型**概率化地量化不确定性**、并据此通过**采集函数序贯决策采样点**时，才称为贝叶斯优化。
- [[序贯模型优化（Sequential Model-Based Optimization, SMBO）]]
- [[全局优化]] / [[黑箱优化]] / [[昂贵函数优化]]

### 下位概念（更具体 / 变体）
- [[高效全局优化（EGO, Jones et al., 1998）]] — 工程领域的经典 BO 实现：Kriging + EI，本质上就是 BO，只是早于"贝叶斯优化"这一名称流行之前提出。
- [[多目标贝叶斯优化（MOBO，如 ParEGO、EHVI）]]
- [[多保真度贝叶斯优化（Multi-fidelity BO）]] — 融合廉价低保真与昂贵高保真评估。
- [[约束贝叶斯优化]] / [[批量（并行）BO，如 qEI]] / [[高维 BO]]

### 相关/平行概念
- [[高斯过程回归 / 克里金]] — 联系：BO 最常用的代理模型，其突出优势是天然提供预测方差（不确定性）。
- [[网格搜索 / 随机搜索]] — 区别：不利用已评估信息指导后续采样；低维有效但样本效率远低于 BO。随机搜索常被用作 BO 的基线。
- [[进化算法（GA/DE/CMA-ES）]] — 区别：无需建模但需大量函数评估，适合廉价函数；BO 适合昂贵函数。
- [[多臂老虎机 / UCB]] — 联系：采集函数的思想与老虎机问题的探索–利用权衡同源。
- [[主动学习]] — 联系：同为序贯采样决策；区别在于主动学习以减少全局不确定性为目标，BO 以寻优为目标。

### 文献出处

| 文献 | 观点/用法 |
| ----- | ----- |
| [[Jones, Schonlau & Welch (1998), *Efficient Global Optimization of Expensive Black-Box Functions*, J. Global Optimization 13:455–492]] | EGO 算法的奠基文献：Kriging + 最大 EI 准则，BO 在工程优化中的源头。 |
| [[Močkus (1978), *Bayesian Approach to Global Optimization*]] | 贝叶斯全局优化的早期理论源头。 |
| [[Brochu, Cora & de Freitas (2010), *A Tutorial on Bayesian Optimization of Expensive Cost Functions*]] | 经典入门教程，arXiv:1012.2599。 |
| [[Snoek, Larochelle & Adams (2012), *Practical Bayesian Optimization of Machine Learning Algorithms*, NeurIPS]] | 将 BO 引入机器学习超参数调优（Spearmint），推动 BO 在 ML 社区流行。 |
| [[Shahriari et al. (2016), *Taking the Human Out of the Loop: A Review of Bayesian Optimization*, Proc. IEEE]] | 权威综述，系统梳理代理模型、采集函数与应用。 |
| [[Rasmussen & Williams (2006), *Gaussian Processes for Machine Learning*, MIT Press]] | GP 理论与后验公式的标准参考。 |
| [[Garnett (2023), *Bayesian Optimization*, Cambridge University Press]] | 近年系统性专著。 |

### 典型应用场景
- 机器学习**超参数调优**（Spearmint、Hyperopt/SMAC、Optuna、BoTorch/Ax）；
- 工程设计中基于昂贵仿真（CFD、FEA）的**气动/结构优化**；
- 材料发现、化学/生物**实验设计**；
- A/B 测试与推荐系统参数寻优。

### 常见误区
- ❌ "BO = 用代理模型做优化" —— 不准确。纯响应面法也是代理模型优化，但不量化不确定性、不做采集函数决策，就不是 BO。
- ❌ "BO 一定优于随机搜索" —— 仅当函数昂贵、预算有限、维度不太高（经典 GP-BO 一般 < 20 维）时优势显著；廉价函数或高维场景未必划算。
- ❌ "GP 是唯一选择" —— 高维、离散/混合变量场景常用随机森林（SMAC）、TPE、贝叶斯神经网络替代。

---

*创建: 2026-10-21*

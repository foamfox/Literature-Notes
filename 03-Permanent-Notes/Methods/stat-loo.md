---
type: permanent
subtype: method
source: web/ai
status: 🟢成熟
created: "2026-10-19"
modified: "2026-10-19"
aliases: [LOO, LOOCV, LOO-CV, 留一法, 留一交叉验证]
tags: [cross-validation, predictive-performance, model-comparison, Bayesian-statistics]
domain: [统计学, 机器学习, 贝叶斯推断]
related-methods: [K折交叉验证, 嵌套交叉验证, PSIS, WAIC, 留一组交叉验证]
related-equations: []
related-concepts: [泛化误差, 后验预测分布, ELPD, 数据泄漏, 偏差与方差]
source-literature: [Vehtari-Gelman-Gabry-2017, Gelfand-Dey-1994, scikit-learn-documentation, loo-documentation]
projects: []
---

# LOO：留一交叉验证（Leave-One-Out Cross-Validation）

> **类型**：预测性能评估与模型比较方法  
> **一句话定义**：每次留出一个观测，用其余数据重新拟合并预测它；遍历全部观测后，汇总这些样本外预测的表现。  
> **来源**：两篇博客的用户提供截图；学术论文与官方文档核对补充。文末附可追溯来源。

## 1. 核心思想与适用范围

LOO 是 **$K=n$ 的 K 折交叉验证**：$n$ 个观测对应 $n$ 折，每折训练集含 $n-1$ 个观测，验证集含 1 个。每个观测恰好被留出一次。它评估的是“未参与本次拟合的数据能被预测得怎样”，区别于训练集上的拟合优度。[交叉验证官方说明](https://scikit-learn.org/stable/modules/cross_validation.html#leave-one-out-loo)

需要区分三个层次：

| 层次 | 回答的问题 | 例子 |
|---|---|---|
| 数据划分 | 留出哪些数据？ | LOO、K 折、留一组 |
| 评价指标 | 什么算预测得好？ | MAE、均方误差、对数预测密度 |
| 计算方法 | 怎样得到留出预测？ | 每折重新拟合、解析公式、PSIS 近似 |

**适用条件**：关注同一总体中新观测的预测；观测单位与实际预测目标一致；逐折拟合的成本可接受。样本少、希望每折尽量利用训练数据时可考虑 LOO，但“小样本”并不保证评估可靠。

**划分应服从预测任务**：普通逐观测 LOO 常用于独立同分布数据；同一个体的重复测量若用于预测新个体，应整组留出；预测未来则应按时间顺序验证。对于时空相关或层次数据，须明确条件信息和留出单位，不能机械地逐行留出。[分组与时序验证](https://scikit-learn.org/stable/modules/cross_validation.html#cross-validation-iterators-for-grouped-data)

## 2. 一般 LOO：从留出预测到误差

设监督学习数据为 $D=\{(x_i,y_i)\}_{i=1}^{n}$，其中 $x_i$ 为特征、$y_i$ 为目标；$D_{-i}$ 为移除第 $i$ 个观测后的训练集。令

$$
\hat f_{-i}=\mathcal A(D_{-i}),
\qquad \hat y_i^{(-i)}=\hat f_{-i}(x_i),
$$

其中 $\mathcal A$ 包括预处理和模型拟合的完整流程。对逐观测损失 $L$，LOO 风险估计为

$$
\widehat R_{\mathrm{LOO}}
=\frac{1}{n}\sum_{i=1}^{n}L\!\left(y_i,\hat y_i^{(-i)}\right).
$$

- **回归**：$L=|y-\hat y|$ 得到 MAE；$L=(y-\hat y)^2$ 得到 MSE；RMSE 为汇总 MSE 后再开平方。
- **分类**：可汇总错误率，或使用预测概率计算对数损失。
- **方向**：损失越小越好。LOO 主要评估在 $n-1$ 个观测上训练的流程对新观测的预测表现，不能视为全数据最终模型误差的精确值。

**例子**：66 个独立样本做年龄预测，每折用 65 个训练，得到 66 个留出预测，再计算 $\mathrm{MAE}_{\mathrm{LOO}}=\frac{1}{66}\sum_i|y_i-\hat y_i^{(-i)}|$。这里是流程示例，不对应某项研究的实测误差。

## 3. 贝叶斯 LOO：预测分布、ELPD 与模型比较

### 3.1 留一后验预测分布

贝叶斯模型 $M$ 用留一训练集更新参数分布，再对参数不确定性积分：

$$
p(y_i\mid x_i,D_{-i},M)
=\int p(y_i\mid x_i,\theta,M)\,
p(\theta\mid D_{-i},M)\,\mathrm d\theta.
$$

它衡量真实观测 $y_i$ 在留一预测分布下得到的概率或密度，包含预测不确定性；仅把后验均值参数代入似然，通常不等于该积分。无协变量时，可简写为 $p(y_i\mid y_{-i},M)$。[Vehtari 等，2017，式 (4)–(5)](https://arxiv.org/abs/1507.04544)

### 3.2 对数预测评分

$$
\ell_i^M=\log p(y_i\mid x_i,D_{-i},M),
\qquad
\widehat{\mathrm{elpd}}_{\mathrm{LOO}}^M
=\sum_{i=1}^{n}\ell_i^M.
$$

ELPD 指 **expected log pointwise predictive density**（期望逐点对数预测密度）；上式是用现有数据计算的 LOO 估计，而非直接知道真实数据分布下的期望。文献与软件通常省略估计符号，记为 `elpd_loo`。[loo 术语表](https://mc-stan.org/loo/reference/loo-glossary.html)

| 指标 | 定义或含义 | 优劣方向 |
|---|---|---|
| `elpd_loo` | 所有留出观测的对数预测评分之和 | 越大越好 |
| 平均对数评分 | $\widehat{\mathrm{elpd}}_{\mathrm{LOO}}/n$ | 越大越好 |
| `looic` | $-2\widehat{\mathrm{elpd}}_{\mathrm{LOO}}$，转为偏差尺度 | 越小越好 |

`looic` 与 `elpd_loo` 给出的排序相同。[loo 输出定义](https://mc-stan.org/loo/reference/loo.html) 连续变量的密度可以大于 1，因此其对数评分可以为正；不能仅按正负判断好坏。模型间比较须使用相同的观测、预测目标、留出单位与计量尺度；若变换响应变量，应统一尺度并正确处理密度变换的 Jacobian。[loo 术语表](https://mc-stan.org/loo/reference/loo-glossary.html)

### 3.3 比较模型时同时报告不确定性

对同一批观测上的两个模型，先计算逐点配对差异，再求和：

$$
d_i=\ell_i^{M_1}-\ell_i^{M_2},\qquad
\Delta\widehat{\mathrm{elpd}}=\sum_i d_i,
\qquad
\widehat{\mathrm{SE}}(\Delta)\approx\sqrt{n\,s_d^2},
$$

其中 $s_d^2$ 为 $d_1,\ldots,d_n$ 的样本方差。$\Delta>0$ 表示模型 1 的预测评分更高，但须结合差值的标准误和实际意义判断；差异与不确定性相当时，不宜强行宣布优胜。标准误应由**配对差异**计算，不能用两个模型各自标准误相减替代。该近似在小样本、异常观测或模型预测极相近时可能不可靠，也不代表各折相互独立。[loo 模型比较说明](https://mc-stan.org/loo/reference/loo_compare.html)

### 3.4 与博客中伪贝叶斯因子的关系

知乎博客使用的伪贝叶斯因子（pseudo-Bayes factor，PSBF）为

$$
\mathrm{PSBF}_{12}
=\frac{\prod_i p(y_i\mid x_i,D_{-i},M_1)}
{\prod_i p(y_i\mid x_i,D_{-i},M_2)}
=\exp\!\left(\Delta\widehat{\mathrm{elpd}}\right).
$$

因此 $\mathrm{PSBF}_{12}>1$ 与 $\Delta\widehat{\mathrm{elpd}}>0$ 是同一个排序结论。但 PSBF 使用留一预测密度的乘积，真正的贝叶斯因子则使用边际似然之比 $p(D\mid M_1)/p(D\mid M_2)$；PSBF 不能直接当作模型的后验赔率。实际报告优先给出 $\Delta\widehat{\mathrm{elpd}}$ 及其不确定性，单纯指数化会放大数值差异而不反映不确定性。[预测分布组合论文](https://sites.stat.columbia.edu/gelman/research/unpublished/combine_predictive.pdf)、[Vehtari 等的讨论](https://sites.stat.columbia.edu/gelman/research/published/wagenmakers_loo_response.pdf)

## 4. 实施步骤与关键设置

```text
输入：数据 D、完整建模流程 A、预先确定的评价指标
1. 根据预测任务确定留出单位，检查组结构和时间依赖。
2. 对每个观测 i：
   a. 构造训练集 D_-i；仅用它拟合预处理步骤。
   b. 训练模型；若需调参，仅在 D_-i 内开展内层验证。
   c. 对留出的观测生成点预测或预测分布，保存逐点评分。
3. 汇总误差或 elpd；比较模型时使用同一批观测的配对差异。
4. 检查误差来源、异常观测，以及贝叶斯计算的收敛和近似诊断。
5. 选定建模流程后，用全部可用训练数据重新拟合最终模型。
```

**最重要的设置**是留出单位、评分指标和调参层级；普通 LOO 的折数固定为 $n$。LOO 的划分没有随机性，但随机优化、抽样或模型初始化仍可能带来随机性，应记录种子及计算设置。

- **防止数据泄漏**：标准化、缺失值填补、PCA 和特征筛选等需要从数据学习的步骤，应在每折训练集内拟合；可用 `Pipeline` 封装。[scikit-learn 常见陷阱](https://scikit-learn.org/stable/common_pitfalls.html#data-leakage)
- **控制选择偏差**：用同一份 LOO 分数反复调参后，再报告其中最优值，通常会过于乐观。需要评估调参后的完整流程时，可采用内层选参、外层评估的嵌套交叉验证，或另设独立测试集。[嵌套交叉验证示例](https://scikit-learn.org/stable/auto_examples/model_selection/plot_nested_cross_validation_iris.html)
- **匹配指标定义**：单个验证样本不能计算有意义的折内 $R^2$ 或 ROC-AUC。MAE、MSE 等逐点可加损失可直接汇总；若将全部留出预测合并计算其他指标，应说明其定义，不能等同于折内分数均值。

## 5. 计算成本与 PSIS-LOO

| 计算方式 | 主要成本 | 适用与限制 |
|---|---|---|
| 逐折重新拟合 | 约 $n\,T_{\mathrm{fit}}(n-1)$，另加预测成本 | 通用；各折可并行，贝叶斯每折仍有数值或抽样误差 |
| 解析或专用更新 | 依具体模型而定 | 某些线性模型等可避免完整重拟合，须核对适用条件 |
| PSIS-LOO | 一次完整后验拟合，加逐点重要性抽样计算 | 贝叶斯近似；需诊断，部分观测可能仍须重新拟合 |

PSIS 是 **Pareto smoothed importance sampling**（帕累托平滑重要性抽样）。在适当的逐点似然分解下，它重用完整数据的后验抽样，经重要性加权近似留一后验预测；**不是直接用完整后验预测已参与拟合的数据**。常用输入是后验抽样数 $S$ × 观测数 $n$ 的逐点 `log_lik` 矩阵，存储量约为 $O(Sn)$。[loo 计算接口](https://mc-stan.org/loo/reference/loo.html)

**必须检查 Pareto $\hat k$**：它诊断重要性抽样近似是否可靠。当前 `loo` 文档采用的可靠性参考阈值为

$$
k_{\mathrm{crit}}=\min\!\left(1-\frac{1}{\log_{10}S},\;0.7\right),
$$

其中 $S$ 是后验抽样数，不是观测数 $n$；还须关注 MCMC 有效样本量与收敛。超阈值时，应考虑对问题观测直接重拟合、矩匹配或改用 K 折验证。高 $\hat k$ 表示近似有问题，不等于已证明该观测是错误数据；精确重拟合的 LOO 本身也不因 PSIS 失效而失效。[PSIS 诊断与补救](https://mc-stan.org/loo/reference/pareto-k-diagnostic.html)

## 6. 优缺点、替代方法与易错点

**优势**：每折充分利用训练样本；给定数据后的划分确定；可定位逐点预测失败；概率评分可比较预测分布，PSIS 可降低部分贝叶斯模型的计算成本。

**局限**：通用实现需要多次拟合；各折训练集高度重叠，误差估计的方差未必较小；异常点、样本代表性不足及分布变化仍会影响结论。[交叉验证官方说明](https://scikit-learn.org/stable/modules/cross_validation.html#leave-one-out-loo)

| 情况 | 可考虑的方法 | 判断依据 |
|---|---|---|
| 普通监督学习、拟合成本较高 | 5 折或 10 折交叉验证 | 以较少拟合次数评估预测表现 |
| 贝叶斯模型，需评估预测分布 | PSIS-LOO；诊断失败时重拟合或 K 折 | 兼顾成本与近似可靠性 |
| 预测未见过的个体、实验或工况组 | 留一组或分组 K 折 | 验证集整组不参与训练 |
| 预测未来 | 时间顺序验证、滚动验证 | 避免使用预测时不可获得的信息 |

**不要误读**：

1. **划分确定不等于估计方差小**；LOO 并非必然比 K 折稳定。
2. **交叉验证评估过拟合风险，不会自动消除过拟合**；仍需合理建模与正则化。
3. **LOO 得分更高只说明相应预测任务上的表现更好**，不能直接证明模型机制真实或因果关系成立。
4. 原知乎截图将剩余 $n-1$ 个数据称为“测试集”，应更正为**训练集**；CSDN 截图对“结果更稳定”的表述应加上上述限定条件。

## 7. 知识关联与来源

- 上位框架：[[交叉验证]]；相近方法：[[K折交叉验证]]、[[嵌套交叉验证]]、[[留一组交叉验证]]。
- 核心概念：[[泛化误差]]、[[后验预测分布]]、[[数据泄漏]]、[[偏差与方差]]。
- 与 [[Jackknife]] 都有“删一”操作，但常见目标不同：LOO 评估预测表现，Jackknife 常用于估计统计量的偏差和方差。

### 入门来源（按用户提供截图阅读）

- [CSDN：留一法 Leave-One-Out, LOO（交叉验证技术的一种）](https://blog.csdn.net/weixin_45708839/article/details/146332131)：一般流程与 66 样本示例。
- [知乎：模型比较中的留一法交叉验证](https://zhuanlan.zhihu.com/p/156657195)：贝叶斯预测密度、ELPD 与 PSBF。

### 核心论文与官方资料

1. Vehtari, A., Gelman, A., & Gabry, J. (2017). **Practical Bayesian model evaluation using leave-one-out cross-validation and WAIC**. *Statistics and Computing*, 27, 1413–1432. [DOI](https://doi.org/10.1007/s11222-016-9696-4) · [开放预印本](https://arxiv.org/abs/1507.04544)。贝叶斯 LOO 与 PSIS 的主要方法依据。
2. Gelfand, A. E., & Dey, D. K. (1994). **Bayesian Model Choice: Asymptotics and Exact Calculations**. *JRSS Series B (Methodological)*, 56(3), 501–514. [DOI](https://doi.org/10.1111/j.2517-6161.1994.tb01996.x)。博客引用的模型比较背景文献；本文核对其书目信息，PSBF 的解释同时参照上述预测分布组合论文与方法作者讨论。
3. scikit-learn 官方文档：[交叉验证](https://scikit-learn.org/stable/modules/cross_validation.html)、[数据泄漏](https://scikit-learn.org/stable/common_pitfalls.html)、[嵌套交叉验证](https://scikit-learn.org/stable/auto_examples/model_selection/plot_nested_cross_validation_iris.html)。
4. Stan / `loo` 官方文档：[术语与指标](https://mc-stan.org/loo/reference/loo-glossary.html)、[模型比较](https://mc-stan.org/loo/reference/loo_compare.html)、[PSIS 诊断](https://mc-stan.org/loo/reference/pareto-k-diagnostic.html)。

*创建：2026-10-19｜修改：2026-10-19｜网页资料核对：2026-10-19*

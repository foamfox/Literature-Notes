---
type: permanent
subtype: concept
source: NACA Report 460 / NASA CR-176883 / discussion
status: 🟢成熟
created: 2026-10-15
tags: [NACA, airfoil, geometry, parameterization, aerodynamics]
domain: [空气动力学, 翼型几何, CFD]
related-concepts: [中弧线, 厚度分布, 弦线, 几何中轴, 翼型参数化]
source-literature:
  - Jacobs, Ward & Pinkerton, NACA Report 460 (1933)
  - Korkan, Camba & Morris, NASA CR-176883 (1986)
---

# NACA 四位数翼型

> **类型**：概念卡片  
> **领域**：空气动力学 / 翼型几何 / 参数化建模  
> **核心对象**：NACA 4-digit airfoil family

---

## 1. 一句话定义

> NACA 四位数翼型是一类将**中弧线（弯度）**与**厚度分布**分开定义，再组合生成上下表面的低维解析参数化翼型族。

---

## 2. 编号含义

以 **NACA 2412** 为例：

| 数字 | 参数 | 含义 |
|---|---|---|
| 2 | $m=0.02$ | 最大弯度为弦长的 2% |
| 4 | $p=0.4$ | 最大弯度位于前缘后 40% 弦长处 |
| 12 | $t=0.12$ | 最大厚度为弦长的 12% |

其中弦长 $c$ 为前缘点与后缘点之间的直线距离；连接二者的直线称为**弦线（chord line）**。

---

## 3. 几何构造思想

NACA 四位数翼型可概括为：

> **人为构造两段抛物线定“弯” + 参考成熟翼型几何厚度分布定“厚”。**

### 3.1 中弧线 $y_c$：解析构造

中弧线（mean camber line）不是从气动实验数据回归得到，而是人为规定为两段二次曲线，并通过几何条件确定系数。

要求：

- 前缘：$y_c(0)=0$
- 后缘：$y_c(1)=0$
- 最大弯度：$y_c(p)=m$
- 最大弯度处：$\left.\dfrac{dy_c}{dx}\right|_{x=p}=0$

因此（令 $x$ 已按弦长归一化，即 $x/c\to x$）：

$$
y_c(x)=
\begin{cases}
\dfrac{m}{p^2}(2px-x^2), & 0\le x<p \\
\dfrac{m}{(1-p)^2}\left[(1-2p)+2px-x^2\right], & p\le x\le1
\end{cases}
$$

其斜率为

$$
\frac{dy_c}{dx}=
\begin{cases}
\dfrac{2m}{p^2}(p-x), & x<p \\
\dfrac{2m}{(1-p)^2}(p-x), & x\ge p
\end{cases}
$$

故在最大弯度位置 $x=p$：

$$
\frac{dy_c}{dx}=0
$$

即中弧线切线水平。

### 3.2 厚度分布 $y_t$：经验几何归纳

经典 NACA 四位数的半厚度分布写为

$$
y_t=5t\left(
0.2969\sqrt{x}
-0.1260x
-0.3516x^2
+0.2843x^3
-0.1015x^4
\right)
$$

其中 $y_t$ 是**半厚度分布**，不是表面绝对 $y$ 坐标。

该基本厚度形状并非由升阻力数据“训练”得到。NACA 早期工作注意到 **Clark Y、Göttingen 398** 等当时已知且性能较好的翼型，在去除弯度并归一化最大厚度后，其厚度分布较为相近；据此构造了统一的基本厚度分布，并以解析式表达。

因此更准确地说：

> $y_t$ 是**参考已有成熟翼型几何厚度分布进行经验归纳，并结合几何约束确定系数**得到的，而不是现代机器学习意义上的代理模型。

经典四位数系列中，最大厚度位置约为

$$
x/c\approx0.30
$$

且该位置不是四位编号中的独立参数。

---

## 4. 从 $y_c$、$y_t$ 到上下表面

中弧线局部倾角：

$$
\theta=\arctan\left(\frac{dy_c}{dx}\right)
$$

沿中弧线法向向两侧各移动 $y_t$，得到上下表面：

$$
\begin{aligned}
x_u&=x-y_t\sin\theta \\
y_u&=y_c+y_t\cos\theta \\
x_l&=x+y_t\sin\theta \\
y_l&=y_c-y_t\cos\theta
\end{aligned}
$$

因此可概括为：

$$
\boxed{\text{翼型几何}=\text{中弧线}+\text{法向厚度分布}}
$$

其中：

- $(m,p)$ 控制“弯多少、在哪里最弯”；
- $t$ 控制“整体多厚”；
- 固定的 $y_t$ 基本函数控制“厚度沿弦向如何分布”。

---

## 5. 最大弯度点 ≠ 最大厚度点

以 NACA 2412 为例：

- 最大弯度位置：$x/c=0.4$
- 最大厚度位置：$x/c\approx0.30$

在 $x=p$ 处：

$$
\frac{dy_c}{dx}=0,\qquad \theta=0
$$

因此上下表面点恰位于中弧线点的正上、正下方：

$$
y_u=y_c+y_t,\qquad y_l=y_c-y_t
$$

但这**不意味着上下表面斜率为 0**。原因是此处一般仍有

$$
\frac{dy_t}{dx}\ne0
$$

所以：

> 最大弯度点只保证**中弧线水平**，不保证**翼型表面水平**。

---

## 6. 中弧线与几何中轴不是一回事

### 中弧线（mean camber line）

- 是 NACA 几何构造中人为定义的“骨架线”；
- 用于描述翼型弯度；
- 上下表面由沿其法向加减 $y_t$ 生成。

### 几何中轴（medial axis）

- 是几何边界决定的中轴；
- 可理解为一系列最大内切圆圆心的轨迹；
- 真正内切圆要求半径方向与表面法向一致，并与边界相切。

因此一般有：

$$
\boxed{\text{mean camber line}\neq\text{medial axis}}
$$

特别地，在最大弯度处，以中弧线点为圆心、$y_t$ 为半径的圆虽然通过对应的上下表面点，但通常**并不与上下表面相切**。

---

## 7. NACA 四位数的本质

NACA 四位数不是“用一批优秀翼型训练出的气动代理模型”。

它更准确地属于：

> **经验启发 + 解析构造的低维几何参数化模型。**

其映射是

$$
(m,p,t)\longrightarrow \text{airfoil geometry}
$$

而不是

$$
(m,p,t)\longrightarrow C_L,C_D,C_M
$$

后者才属于气动性能代理模型（surrogate model）的范畴。

经典编号只是在该参数化空间中的具体点；未被历史实验直接测试过的参数组合仍可生成几何，但**几何可生成不等于气动性能优良**。

---

## 8. 与 CFD / SBO 的关系

NACA 参数化能把复杂的翼型边界降维为少数设计变量：

$$
\mathbf{x}=(m,p,t)
$$

在 CFD / SBO 中可形成：

$$
(m,p,t)
\rightarrow
\text{NACA 几何}
\rightarrow
\text{CFD}
\rightarrow
(C_L,C_D,C_M)
$$

再由 CFD 样本训练真正的代理模型：

$$
(m,p,t)
\xrightarrow{\text{surrogate}}
\widehat{f}(m,p,t)
$$

因此 NACA 模型与 SBO 代理模型属于两个不同层次：

- **NACA**：设计变量 → 几何；
- **Surrogate**：设计变量 → 性能预测。

---

## 9. 易混概念速查

| 概念 | 定义 | 关键点 |
|---|---|---|
| 弦线 | 前缘—后缘连线 | 几何基准线 |
| 中弧线 $y_c$ | 描述翼型弯度的参数化骨架线 | NACA 中人为构造 |
| 半厚度 $y_t$ | 沿中弧线法向到上下表面的距离参数 | 不是绝对 $y$ 坐标 |
| 最大弯度 $m$ | 中弧线距弦线的最大偏离 | 四位数第 1 位给出 |
| 最大弯度位置 $p$ | 最大弯度的弦向位置 | 四位数第 2 位给出 |
| 最大厚度 $t$ | 最大总厚度 / 弦长 | 四位数后 2 位给出 |
| 几何中轴 | 最大内切圆圆心轨迹 | 一般不等于中弧线 |

---

## 10. 知识网络

### 上位概念
- [[翼型几何参数化]]
- [[空气动力学]]

### 下位概念
- [[NACA 2412]]
- [[NACA 0012]]
- [[NACA modified 4-digit series]]

### 相关/平行概念
- [[CST 参数化]] — 更灵活的现代翼型参数化方法
- [[PARSEC 参数化]] — 以具有几何意义的参数描述翼型
- [[B-spline / FFD]] — 更高自由度的形状参数化
- [[Surrogate-Based Optimization]] — NACA 可作为其几何参数化层，但本身不是性能代理模型

---

## 11. 文献出处

| 文献 | 主要用途 |
|---|---|
| Jacobs, E. N., Ward, K. E., Pinkerton, R. M., **NACA Report 460**, 1933 | NACA 相关翼型族的构造思想、厚度与中弧线变化及风洞试验 |
| Korkan, K. D., Camba, J. III, Morris, P. M., **NASA CR-176883**, 1986 | NACA 四位数翼型数据与几何定义的后续整理；明确提及 Clark Y、Göttingen 398 对基本厚度分布的影响 |

---

*创建：2026-10-15*

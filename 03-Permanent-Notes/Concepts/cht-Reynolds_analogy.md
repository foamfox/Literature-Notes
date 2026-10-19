---
type: permanent
subtype: concept
source: conversation
status: 🟢成熟
created:
  '2026-09-15':
tags: [传热学, 传质学, 流体力学, 传递现象]
domain: [传递现象, 边界层理论]
related-concepts: [边界层, N-S方程, Prandtl数, Schmidt数, Nusselt数, Sherwood数, Stanton数]
source-literature: []
---

# Reynolds 类比（Reynolds analogy）

> **类型**：概念卡片
> **领域**: 传递现象 / 传热传质 / 边界层理论
> **来源**: 本次会话整理

---

## 1. 定义

### 一句话定义

> Reynolds 类比利用动量、热量和质量输运方程在特定条件下的数学相似性，将壁面摩擦、换热和传质联系起来。

### 详细解释

Reynolds 类比**不是普遍物理定律**，也不是说动量、热量和质量是相同的物理过程。

三者分别遵循：

- 动量守恒 + Newton 黏性定律；
- 能量守恒 + Fourier 导热定律；
- 组分守恒 + Fick 扩散定律。

在边界层近似下，其输运方程分别具有“对流 = 扩散”的结构：

$$
u\frac{\partial u}{\partial x}
+v\frac{\partial u}{\partial y}
=
-\frac{1}{\rho}\frac{\partial p}{\partial x}
+\nu\frac{\partial^2u}{\partial y^2}
$$

$$
u\frac{\partial T}{\partial x}
+v\frac{\partial T}{\partial y}
=
\alpha\frac{\partial^2T}{\partial y^2}
$$

$$
u\frac{\partial C_A}{\partial x}
+v\frac{\partial C_A}{\partial y}
=
D_{AB}\frac{\partial^2C_A}{\partial y^2}
$$

当满足零压梯度、常物性、相应边界条件以及

$$
Pr=\frac{\nu}{\alpha}=1,
\qquad
Sc=\frac{\nu}{D_{AB}}=1
$$

即

$$
\nu=\alpha=D_{AB}
$$

时，三个问题可化为相同的无量纲数学问题，从而建立动量、热量和质量传递之间的类比。

**核心不是“物理机制相同”，而是“控制方程及边界条件在特定条件下具有相同数学结构”。**

### 类比

可以把三种输运看成同一个“对流—扩散”框架中的三个版本：

$$
\phi=
\begin{cases}
u & \text{动量}\\
T & \text{热量}\\
C_A & \text{质量}
\end{cases},
\qquad
\Gamma=
\begin{cases}
\nu & \text{动量}\\
\alpha & \text{热量}\\
D_{AB} & \text{质量}
\end{cases}
$$

若扩散率相同，则在相同的数学条件下，三个场的无量纲解可以对应起来。

---

## 2. 数学表达

### 核心公式

Reynolds 类比的典型表达式：

$$
\boxed{
\frac{C_f}{2}
=
St
=
\frac{Nu}{Re}
}
$$

同时考虑质量传递：

$$
\boxed{
\frac{C_f}{2}
=
St_m
=
\frac{Sh}{Re}
}
\qquad
(Pr=Sc=1)
$$

因此：

$$
\boxed{
Nu=Sh=\frac{C_fRe}{2}
}
$$

其中 \(C_f\) 为平均摩擦阻力系数时，\(Nu\)、\(Sh\)、\(Re\) 应采用相同的特征长度定义。

### 符号说明

| 符号 | 含义 | 单位 |
|------|------|------|
| \(u,v\) | 流向、法向速度 | m/s |
| \(T\) | 温度 | K |
| \(C_A\) | 组分 \(A\) 浓度 | 依定义而定 |
| \(\nu=\mu/\rho\) | 动量扩散率 | m²/s |
| \(\alpha=k/(\rho c_p)\) | 热扩散率 | m²/s |
| \(D_{AB}\) | 质量扩散系数 | m²/s |
| \(Pr=\nu/\alpha\) | Prandtl 数 | 1 |
| \(Sc=\nu/D_{AB}\) | Schmidt 数 | 1 |
| \(Re=UL/\nu\) | Reynolds 数 | 1 |
| \(Nu=hL/k\) | Nusselt 数 | 1 |
| \(Sh=h_mL/D_{AB}\) | Sherwood 数 | 1 |
| \(St=h/(\rho Uc_p)=Nu/(RePr)\) | Stanton 数 | 1 |
| \(St_m=Sh/(ReSc)\) | 质量传递 Stanton 数 | 1 |
| \(C_f=2\tau_w/(\rho U^2)\) | 摩擦阻力系数 | 1 |

### 推导或离散形式

三类壁面输运通量分别由本构关系给出：

$$
\tau_w
=
\mu
\left.\frac{\partial u}{\partial y}\right|_w
$$

$$
q_w''
=
-k
\left.\frac{\partial T}{\partial y}\right|_w
$$

$$
J_{A,w}
=
-D_{AB}
\left.\frac{\partial C_A}{\partial y}\right|_w
$$

其中 Newton、Fourier、Fick 定律分别描述**通量如何由相应梯度产生**；动量、能量和组分守恒方程则负责描述这些守恒量的输运与积累。

在 \(Pr=Sc=1\)、零压梯度及对应无量纲边界条件下：

$$
u^*=T^*=C_A^*
$$

从而：

$$
\left.\frac{\partial u^*}{\partial y^*}\right|_w
=
\left.\frac{\partial T^*}{\partial y^*}\right|_w
=
\left.\frac{\partial C_A^*}{\partial y^*}\right|_w
$$

进而得到上述 \(C_f\)、\(Nu\)、\(Sh\) 的关系。

---

## 3. 知识网络

### 上位概念（更抽象）

- [[传递现象]]
- [[边界层理论]]

### 下位概念（更具体）

- [[Prandtl 类比]]
- [[von Kármán 类比]]
- [[Chilton-Colburn 类比]]

### 相关/平行概念

- [[Newton 黏性定律]] — 动量通量与速度梯度的本构关系
- [[Fourier 导热定律]] — 热通量与温度梯度的本构关系
- [[Fick 扩散定律]] — 质量扩散通量与浓度梯度的本构关系
- [[Prandtl 数]] — 动量扩散率与热扩散率之比
- [[Schmidt 数]] — 动量扩散率与质量扩散率之比

### 文献出处

| 文献 | 观点/用法 |
| ----- | ----- |
| 本次会话 | 从守恒方程、本构关系和边界层近似出发梳理 Reynolds 类比 |

---

*创建: 2026-09-15*

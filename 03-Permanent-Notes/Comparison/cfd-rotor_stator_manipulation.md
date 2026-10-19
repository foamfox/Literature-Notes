---
type: permanent
subtype: comparison
status: 🟢成熟
created: "2026-10-14"
tags: [CFD, turbomachinery, rotor-stator, mixing-plane, frozen-rotor, sliding-mesh]
domain: [流体力学, CFD, 叶轮机械]
compared-entities: [Mixing Plane, Frozen Rotor, Sliding Mesh]
comparison-type: method
source-literature: []
---

# 混合平面、冻结转子与滑移网格方法对比

> **类型**: 比较笔记  
> **比较对象**: [[Mixing Plane]] vs [[Frozen Rotor]] vs [[Sliding Mesh]]  
> **比较维度**: 转静耦合方法 / 参考系 / 周向信息保留 / 周期性 / 计算成本

---

## 1. 比较概览

### 为什么要比较这三种方法？

三者都用于旋转机械中转子—静子之间的信息传递，但物理假设与数值代价不同。核心区别在于：

- 是否求解真实相对运动；
- 是否保留转静界面的周向非均匀性；
- 是否进行周向平均；
- 是否允许各叶排独立采用单通道周期模型。

### 一句话结论

> **Mixing Plane**：把周向非均匀性平均掉，求稳态平均性能。  
> **Frozen Rotor**：保留周向非均匀性，但冻结在某一相对相位上。  
> **Sliding Mesh**：让转静相对位置真实随时间变化，求解非定常过程。

可直观理解为：

- Mixing Plane：**平均图**
- Frozen Rotor：**一张照片**
- Sliding Mesh：**完整视频**

---

## 2. 多维度对比

| 对比维度 | Mixing Plane | Frozen Rotor | Sliding Mesh |
|---|---|---|---|
| 求解性质 | 稳态 | 稳态 | 非定常 |
| 转子区域 | 通常采用旋转参考系 | 通常采用旋转参考系 | 网格真实旋转 |
| 静子区域 | 通常采用惯性/静止参考系 | 通常采用惯性/静止参考系 | 静止网格 |
| 转静相对位置 | 不保留具体相位 | 固定在某一相位 | 随时间连续变化 |
| 周向非均匀性 | 被平均 | 保留 | 完整保留 |
| 尾迹传递 | 主要保留平均效应 | 保留固定位置尾迹 | 尾迹真实扫过下游叶排 |
| 叶频非定常 | 不解析 | 不解析 | 可解析 |
| Clocking effect | 不适合 | 可做稳态相位敏感性分析 | 可真实解析 |
| 单通道适应性 | 很好 | 有叶片数/周期匹配限制 | 更复杂 |
| 不同叶片数 | 很容易处理 | 需公共周期或 pitch-change 等处理 | 需满足时空周期或扩大计算域 |
| 计算成本 | 最低 | 低—中 | 最高 |
| 典型用途 | 压比、效率、流量、特性线、多级设计 | 局部转静干涉、尾迹位置、相位效应 | 非定常载荷、叶频脉动、噪声、真实转静干涉 |

---

## 3. 数值本质

### 3.1 Mixing Plane

Mixing Plane 的核心是：

\[
Q(r,\theta) \rightarrow \overline{Q}(r)
\]

即保留径向/叶高方向分布，对周向方向进行平均。

典型形式可写为：

\[
\overline{\phi}(r)
=
\frac{\int \phi(r,\theta)\,w\,d\theta}
{\int w\,d\theta}
\]

其中权重 \(w\) 可依据具体求解器采用：

- 面积加权；
- 质量流量加权；
- 守恒 mixed-out 平均。

因此 Mixing Plane 不是简单“网格值取算术平均”，而是构造一个满足一定守恒要求的周向平均剖面。

#### band 的含义

转静接口通常可理解为沿叶高方向划分若干环带：

\[
[r_i,r_{i+1}]
\]

对每个 band 内覆盖的接口面元进行周向积分/平均，得到：

\[
\overline{Q}(r_i)
\]

因此：

> 同一个 band 内，所有覆盖到该环带的接口面元都会对平均结果产生贡献。

这里的匹配依据是：

- 半径 \(r\)；
- 归一化叶高 span；
- 几何位置；

而不是网格编号。

---

### 3.2 Frozen Rotor

Frozen Rotor 保留周向分布：

\[
Q(r,\theta)
\]

但人为固定转子与静子的相对相位：

\[
\theta_{rel}=\theta_0
\]

因此求得的是：

\[
Q(\mathbf{x};\theta_0)
\]

而不是完整的：

\[
Q(\mathbf{x},t)
\]

这意味着结果可能依赖所选的 clocking position：

\[
Q(\theta_0) \neq Q(\theta_1)
\]

所以 Frozen Rotor 不是对真实非定常流动的时间平均，而是对某一相对位置的稳态近似。

它适合保留：

- 尾迹；
- 周向压力畸变；
- 势场干涉；
- 局部转静相互作用；

但不能描述这些结构随时间扫过另一叶排的过程。

---

### 3.3 Sliding Mesh

Sliding Mesh 中转子区域真实随时间旋转：

\[
\theta(t)=\theta_0+\Omega t
\]

转静接口为动态滑移接口，两侧面之间的对应关系随时间更新。

其本质包含：

1. 真实网格刚体旋转；
2. 动态非共形接口；
3. 非定常通量交换；
4. 时间推进求解。

因此可以得到：

\[
Q(\mathbf{x},t)
\]

并进一步通过时间平均得到：

\[
\overline{Q}
=
\frac{1}{T}
\int_0^T Q(t)\,dt
\]

这是三种方法中物理上最完整、计算成本也最高的方法。

---

## 4. 与参考系方法的关系

三者不属于完全相同的概念层级。

### Mixing Plane / Frozen Rotor

通常与 MRF / Multiple Reference Frame 搭配：

- 转子域：旋转参考系；
- 静子域：惯性或静止参考系；
- 网格本身不做真实时间运动；
- 差别主要在接口数据如何传递。

因此常见组合为：

\[
\text{MRF} + \text{Mixing Plane}
\]

或：

\[
\text{MRF} + \text{Frozen Rotor}
\]

### Sliding Mesh

通常采用真实 moving mesh：

- 转子网格做刚体旋转；
- 静子网格固定；
- 转静接口动态滑动；
- 必须做非定常时间推进。

因此它不只是“接口数据处理”，还涉及真实网格运动。

---

## 5. 共形与非共形接口下的 Mixing Plane

### 共形接口

如果转静接口两侧网格匹配，则可概念性理解为：

1. 对转子侧某一 band 内的接口量做周向平均；
2. 得到 \(\overline{Q}(r)\)；
3. 将该平均剖面作为静子侧对应径向位置的接口状态。

同一半径/叶高位置上：

\[
\frac{\partial \overline{Q}}{\partial \theta}=0
\]

因此静子不会接收到转子尾迹的具体周向位置。

### 非共形接口

如果两侧节点、面数量不同，则先建立共同的平均剖面：

\[
Q_R(r,\theta)
\rightarrow
\overline{Q}_R(r)
\]

然后依据静子侧的半径/span 坐标，将该剖面插值或投影到静子接口：

\[
\overline{Q}_R(r)
\rightarrow
Q_S
\]

因此 Mixing Plane 的接口匹配本质上依赖几何位置，而不是网格编号。

---

## 6. 单通道计算

### 6.1 Mixing Plane

若某叶排有 \(N\) 片叶片，则单通道节距：

\[
\Delta\theta=\frac{360^\circ}{N}
\]

转子和静子可分别采用自己的单通道：

\[
\Delta\theta_R=\frac{360^\circ}{N_R}
\]

\[
\Delta\theta_S=\frac{360^\circ}{N_S}
\]

即使：

\[
N_R \neq N_S
\]

也可以直接耦合。

原因是 Mixing Plane 已将转子出口周向分布：

\[
Q_R(r,\theta)
\]

压缩为：

\[
\overline{Q}_R(r)
\]

因此转静两侧的 pitch 被解耦。

这也是 Mixing Plane 特别适合多级压气机稳态计算的重要原因。

---

### 6.2 Frozen Rotor

Frozen Rotor 要保留 \(\theta\) 方向信息，因此不同叶片数会带来周期匹配问题。

如果：

\[
N_R=N_S
\]

则单通道最自然。

若：

\[
N_R \neq N_S
\]

则通常需要：

- 采用公共周期扇区；
- 或使用求解器提供的 pitch-change / profile scaling 类近似处理。

例如：

\[
N_R=30,\quad N_S=40
\]

最小公共周期由：

\[
\gcd(30,40)=10
\]

决定，因此最小共同扇区为：

\[
\frac{360^\circ}{10}=36^\circ
\]

其中包含：

- 3 个转子叶道；
- 4 个静子叶道。

---

### 6.3 Sliding Mesh

Sliding Mesh 若转静叶片数不同，还要进一步考虑：

- 几何周期；
- 时间周期；
- 叶频关系；
- 相位重复条件。

为了完全避免近似，最直接但最昂贵的方式是 full-annulus 非定常计算。

---

## 7. 精度与适用性

不能简单认为：

\[
\text{Frozen Rotor} > \text{Mixing Plane}
\]

或：

\[
\text{Sliding Mesh} > \text{Frozen Rotor}
\]

而应根据目标量选择。

| 目标 | 推荐方法 |
|---|---|
| 压比、效率、流量等平均总体性能 | Mixing Plane |
| 多级压气机特性线 | Mixing Plane |
| 大量稳态工况扫描 | Mixing Plane |
| 初步观察尾迹进入下游叶排 | Frozen Rotor |
| Clocking effect | Frozen Rotor / Sliding Mesh |
| 局部转静干涉 | Frozen Rotor |
| 叶频压力脉动 | Sliding Mesh |
| 非定常气动力/载荷 | Sliding Mesh |
| 噪声、BPF 相关问题 | Sliding Mesh |
| 高可信时间平均流场 | Sliding Mesh 后时间平均 |

---

## 8. 方法选择的本质

### Mixing Plane

优先保留：

\[
\text{平均性能}
\]

主动舍弃：

\[
\text{周向相位信息}
\]

优势是：

- 稳定；
- 便宜；
- 不依赖 clocking；
- 不同叶片数处理方便；
- 很适合单通道和多级计算。

---

### Frozen Rotor

优先保留：

\[
\text{周向空间非均匀性}
\]

但舍弃：

\[
\text{真实时间变化}
\]

优势是：

- 比 Mixing Plane 保留更多局部流场信息；
- 成本远低于真实非定常。

局限是：

- 结果依赖冻结相位；
- 某个相位不能严格代表所有相位。

---

### Sliding Mesh

同时保留：

\[
\text{周向空间变化}
+
\text{真实时间变化}
\]

优势是物理完整性最高。

代价是：

- 时间步要求；
- 计算量显著增加；
- 数据量增加；
- 多级全环计算成本很高。

---

## 9. 易混淆点

### 9.1 Mixing Plane 不是 MRF

- MRF：描述不同区域在哪个参考系中求解；
- Mixing Plane：描述转静接口如何传递平均信息。

二者经常组合，但不是同一概念。

### 9.2 Frozen Rotor 不是“整个流场用惯性系计算”

Frozen Rotor 通常仍然采用：

- 转子旋转参考系；
- 静子静止参考系；

只是把两者相对位置冻结。

### 9.3 Sliding Mesh 不等于 Overset

Sliding Mesh：

- 转静区域在接口处相邻；
- 通过动态滑移面交换通量。

Overset：

- 两套体网格在空间中重叠；
- 通过 donor/receptor 插值交换信息。

### 9.4 Frozen Rotor 保留更多信息，不代表平均性能一定更准

Frozen Rotor 的某个冻结相位可能具有偶然性。

Mixing Plane 虽然丢失周向结构，但对于：

- 压比；
- 效率；
- 流量；

这类平均总体性能，往往反而更加稳健。

---

## 10. 最终记忆框架

\[
\boxed{
\text{Mixing Plane}
=
\text{稳态}
+
\text{MRF}
+
\text{周向平均}
}
\]

\[
\boxed{
\text{Frozen Rotor}
=
\text{稳态}
+
\text{MRF}
+
\text{固定相位}
+
\text{保留周向非均匀性}
}
\]

\[
\boxed{
\text{Sliding Mesh}
=
\text{非定常}
+
\text{真实网格旋转}
+
\text{动态滑移接口}
}
\]

对于典型多级压气机：

> **总体性能与特性线：优先 Mixing Plane；  
> 局部转静干涉：可用 Frozen Rotor；  
> 真实非定常机理：使用 Sliding Mesh。**

---

*创建: 2026-10-14 | 修改: 2026-10-14*

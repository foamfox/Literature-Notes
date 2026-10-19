---
type: failure
subtype: experiment
status: resolved
created: "2026-09-23"
severity: medium
tags:
  - STAR-CCM+
  - Linux
  - VNC
  - OpenGL
  - Mesa
  - OpenSWR
  - VirtualGL
  - EGL
  - NVIDIA
  - RTX-A5000
domain:
  - CAE
  - CFD
  - HPC
source: "Linux 超算平台 STAR-CCM+ 远程可视化"
---

# STAR-CCM+ 在 Linux 超算 VNC 环境下三维交互卡顿排查与解决

> **状态**: ✅ resolved  
> **关联环境**: Linux 超算 + VNC/noVNC + NVIDIA RTX A5000 + STAR-CCM+ 2502  
> **结论**: 客户端场景默认回退到 Mesa/OpenSWR 软件渲染；最终通过 STAR-CCM+ 官方支持的服务器端 EGL 远程渲染解决。

---

## 1. 一句话摘要

在 Linux 超算 VNC 会话中，STAR-CCM+ 菜单响应正常，但三维模型旋转极慢。排查发现 GPU、驱动和 VirtualGL 均正常，但 STAR 客户端场景实际使用 Mesa/OpenSWR CPU 软件渲染；尝试 `vglrun + native OpenGL` 仍无法初始化 VTK OpenGL Context。最终根据 STAR-CCM+ 官方文档，改用服务器端远程渲染和 EGL：

```bash
starccm+ -rr server -rgraphics egl -rgpu 0
```

成功让 STAR 使用 NVIDIA RTX A5000 进行图形渲染，问题解决。

---

## 2. 现象

- STAR-CCM+ 菜单、按钮等普通 GUI 操作响应很快。
- 一旦拖动、旋转三维模型，帧率明显下降，远慢于本地 Windows。
- 本地 Windows STAR-CCM+ 三维交互流畅。
- 超算通过浏览器 noVNC/VNC 访问。

初始怀疑：

1. 网络/VNC 帧传输性能不足；
2. 超算没有可用 GPU；
3. STAR-CCM+ 没有真正使用 GPU；
4. OpenGL/VirtualGL 配置异常。

---

## 3. 环境

| 项目              | 内容                          |
| --------------- | --------------------------- |
| 软件              | Simcenter STAR-CCM+ 2502    |
| 本地对照            | Windows STAR-CCM+ 2510，交互流畅 |
| Linux 图形环境      | VNC / noVNC，`DISPLAY=:6`    |
| GPU             | NVIDIA RTX A5000 24 GB      |
| NVIDIA Driver   | 550.135                     |
| CUDA            | 12.4                        |
| OpenGL          | NVIDIA OpenGL 4.6           |
| VirtualGL       | 3.1.1                       |
| 物理 GPU X Server | `:0`                        |
| VNC 桌面 Display  | `:6`                        |

---

## 4. 排查过程

### 4.1 确认系统是否存在 GPU

执行：

```bash
nvidia-smi
```

结果确认：

```text
NVIDIA RTX A5000
24564 MiB
Driver Version: 550.135
```

同时 Xorg 及多个图形程序已经占用 GPU 显存。

**结论：GPU 真实存在，驱动工作正常。**

---

### 4.2 检查 VNC 会话中的 OpenGL

执行：

```bash
glxinfo -B
```

关键输出：

```text
direct rendering: Yes
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: NVIDIA RTX A5000/PCIe/SSE2
OpenGL version string: 4.6.0 NVIDIA 550.135
```

执行：

```bash
glxgears
```

服务器端可达到约：

```text
1865 FPS
```

**结论：当前 Linux 图形环境能够访问 A5000，不能简单归因于“没有显卡”或“驱动没装好”。**

---

### 4.3 观察 STAR 旋转时 GPU 利用率

执行：

```bash
nvidia-smi dmon -s u
```

持续旋转 STAR 模型时：

```text
sm = 0%
```

STAR 也未表现出明显 GPU 负载变化。

**结论：高度怀疑 STAR 三维场景并没有使用 A5000。**

---

### 4.4 检查 STAR 进程加载的图形库

对 STAR GUI 进程执行：

```bash
cat /proc/<PID>/maps | grep -Ei \
'libGL|libGLX|libEGL|nvidia|mesa|swrast|llvm'
```

发现 STAR 自带目录中加载了：

```text
.../STAR-CCM+.../mesa/...
libSWRAVX2.so
libLLVM-12.so
libglapi.so
```

初步说明 STAR 加载了 Mesa/OpenSWR 软件渲染组件。

---

### 4.5 使用 STAR 自带 Graphics Report 定案

在 STAR 场景中执行 Graphics Report/Test Graphics。

默认启动时得到：

```text
library: STAR Mesa OpenGL
window: OSMesaOffscreenRenderWindow
OpenGL vendor string: Intel Corporation
OpenGL renderer string: SWR (LLVM 12.0.0, 256 bits)
OpenGL version string: 3.3 (Core Profile) Mesa 21.1.8
```

这是决定性证据。

**结论：STAR 场景实际使用的是 Mesa + OpenSWR CPU 软件渲染，而不是 RTX A5000。**

---

## 5. VirtualGL 尝试链路

### 5.1 确认 VirtualGL 和物理 GPU X Server 正常

执行：

```bash
which vglrun
vglrun +v glxinfo -B
```

关键输出：

```text
[VGL] Opening connection to 3D X server :0
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: NVIDIA RTX A5000/PCIe/SSE2
```

直接执行：

```bash
DISPLAY=:0 glxinfo -B
```

同样得到 NVIDIA RTX A5000。

并通过：

```bash
ps -ef | grep Xorg
```

确认物理 Xorg 在：

```text
:0
```

**结论：VirtualGL、`:0` X Server、NVIDIA 驱动和 A5000 均正常。**

---

### 5.2 直接使用 `vglrun`

尝试：

```bash
vglrun starccm+
```

结果：

- STAR 仍然加载 Mesa/OpenSWR；
- Graphics Report 没有变为 NVIDIA。

**结论：仅用 `vglrun` 不能让 STAR 客户端渲染切换到 GPU。**

---

### 5.3 强制 `native` OpenGL

尝试：

```bash
vglrun -d :0 starccm+ -graphics native
```

Graphics Report 变为：

```text
library: native
window: vtkXOffscreenOpenGLRenderWindow
rendering: direct
OpenGL vendor string: VMware, Inc.
OpenGL renderer string: llvmpipe (LLVM 10.0.1, 256 bits)
```

说明：

- STAR 确实从 `STAR Mesa OpenGL` 切换到了 `native`；
- 但 native context 仍落在 VNC 虚拟 X Server 的 llvmpipe 软件路径，没有进入 A5000。

---

### 5.4 加入 VirtualGL preload

根据外部 HPC 使用经验，尝试：

```bash
vglrun -d :0 starccm+ \
-clientldpreload libdlfaker.so:libvglfaker.so \
-graphics native
```

结果：

```text
library: native
error: vtkOpenGLRenderWindow context not initialized
```

开启 verbose：

```bash
VGL_VERBOSE=1 vglrun -d :0 starccm+ \
-clientldpreload libdlfaker.so:libvglfaker.so \
-graphics native
```

VirtualGL 明确输出：

```text
[VGL] Opening connection to 3D X server :0
[VGL] Using pixel buffer objects for readback
[VGL] NOTICE: Replacing dlopen("libGL.so.1") with dlopen("libvglfaker.so")
[VGL] NOTICE: Replacing dlopen("libGLX.so.1") with dlopen("libvglfaker.so")
```

说明 VirtualGL 已经成功介入 STAR 的 GL/GLX 动态加载，但 STAR 的 VTK native offscreen OpenGL Context 仍无法初始化。

---

### 5.5 preload 相关附加检查

确认：

```bash
rpm -qf /usr/lib64/libvglfaker.so
```

得到：

```text
VirtualGL-3.1.1-20240228.x86_64
```

检查符号：

```bash
nm -D /usr/lib64/libvglfaker.so | grep _vgl_dlopen
```

得到：

```text
T _vgl_dlopen
```

而：

```bash
nm -D /usr/lib64/libdlfaker.so | grep _vgl_dlopen
```

得到：

```text
U _vgl_dlopen
```

同时 `ldd` 可正常解析两库依赖。

**结论：VirtualGL 两个库本身配套，不是 ABI/安装版本不匹配。**

尝试反转 preload：

```bash
-clientldpreload libvglfaker.so:libdlfaker.so
```

结果 STAR 卡在启动阶段，无法使用。

**结论：继续折腾 VirtualGL preload 顺序意义不大。**

---

## 6. 官方文档带来的关键转折

随后查阅 STAR-CCM+ 官方“图形故障排除 / 控制图形性能 / 本地和远程渲染图形”文档，得到几个关键信息：

### 6.1 Mesa fallback 是 STAR 的正常机制

官方说明：

- STAR 会自动检查可用硬件加速；
- 如果硬件 OpenGL 不可用或当前环境不满足要求，会自动回退到 Mesa；
- Mesa 驱动包括 LLVMpipe 和 OpenSWR；
- OpenSWR 通常比 LLVMpipe 更快；
- 软件渲染明显慢于硬件渲染。

这与实际 Graphics Report 完全一致：

```text
STAR Mesa OpenGL
→ OpenSWR
→ CPU 软件渲染
```

因此该现象不是 GPU 不存在，而是 **STAR 客户端的图形检查器/上下文无法在当前 VNC 客户端路径中使用硬件 OpenGL，于是自动 fallback 到 Mesa。**

---

### 6.2 官方更推荐服务器端/远程渲染

官方文档明确区分：

```text
-graphics
```

用于客户端 Java VM 内部渲染；

```text
-rgraphics
```

用于 STAR Server / Remote Renderer 渲染。

同时提供：

```text
-rr server
```

将渲染器从客户端 Java VM 中分离出来，在服务器侧作为独立 renderer 运行。

---

### 6.3 EGL 是 Linux GPU 远程渲染的首选路径

官方说明：

```text
-rgraphics egl
```

可让 Linux 服务器通过 EGL 使用 OpenGL，无需依赖：

```text
Windows / X / X11 / Xorg
```

并明确指出：

> EGL 是 STAR-CCM+ 远程渲染或专用渲染服务器的首选方式。

这意味着可以完全绕过此前失败的：

```text
VNC → X11/GLX → VirtualGL → VTK native context
```

链路。

---

## 7. 最终解决方案

最终使用：

```bash
starccm+ -rr server -rgraphics egl -rgpu 0
```

参数含义：

```text
-rr server
```

在 STAR Server 侧启动独立渲染器。

```text
-rgraphics egl
```

服务器端 renderer 使用 EGL 硬件 OpenGL，不依赖 VNC/X11/GLX。

```text
-rgpu 0
```

指定 GPU 0，即 NVIDIA RTX A5000。

最终架构：

```text
VNC / noVNC
    │
    │ GUI / 鼠标交互
    ↓
STAR Client
    │
    ↓
STAR Server Renderer
    │
    ↓
EGL
    │
    ↓
NVIDIA Driver
    │
    ↓
RTX A5000
```

相比失败路线：

```text
STAR Client
    ↓
VTK native OpenGL
    ↓
VirtualGL / GLX
    ↓
Xorg
    ↓
GPU
```

EGL 服务器端渲染直接绕开了客户端 VNC/X11 图形上下文问题。

---

## 8. 验证

最终命令：

```bash
starccm+ -rr server -rgraphics egl -rgpu 0
```

结果：

- STAR 场景恢复正常硬件加速；
- 三维模型旋转流畅；
- 原问题解决。

验证建议：

```bash
nvidia-smi
```

或：

```bash
nvidia-smi dmon -s u
```

同时可在 STAR 场景执行 Graphics Report，确认远程渲染状态。若报告结尾出现：

```text
***Report generated remotely***
```

说明远程 renderer 正在工作。

---

## 9. 根因

### 直接原因

STAR-CCM+ 在 VNC 客户端会话中无法正常使用 native GPU OpenGL，图形检查器自动回退到：

```text
Mesa OpenGL
→ OpenSWR
→ CPU 软件渲染
```

因此复杂三维场景交互非常卡顿。

### 深层原因

问题并不在：

- NVIDIA RTX A5000 硬件；
- NVIDIA 550.135 驱动；
- Xorg `:0`；
- VirtualGL 基础设施；
- 网络本身。

真正的冲突位于：

```text
STAR Client / VTK offscreen OpenGL
+
VNC/X11/GLX/VirtualGL
```

客户端 native OpenGL context 无法稳定建立。

STAR 官方本身提供了更适合 HPC/VNC 环境的架构：

```text
Server-side Renderer + EGL
```

因此继续强行修复客户端 VirtualGL 路径不是最优方案。

---

## 10. 关键教训

1. **不要仅根据 `glxinfo` 判断 STAR 是否使用 GPU。**  
   `glxinfo` 能看到 NVIDIA，只能证明当前 OpenGL 环境能访问 GPU，不能代表 STAR 场景实际 renderer。

2. **STAR Graphics Report 是判断实际渲染后端的最可靠工具。**  
   关键字段：
   ```text
   library
   OpenGL vendor
   OpenGL renderer
   OpenGL version
   ```

3. **`nvidia-smi dmon` 可作为辅助验证，但不能替代 Graphics Report。**

4. 在 Linux HPC + VNC 环境中，如果 STAR 默认显示：
   ```text
   STAR Mesa OpenGL
   SWR
   ```
   应优先考虑服务器端远程渲染，而不是只围绕 VNC 客户端调 GPU。

5. **对于 Linux NVIDIA GPU，STAR 官方支持的 `-rgraphics egl` 路径比 VirtualGL 客户端注入更直接、更稳定。**

6. VNC 只负责桌面显示并不等于三维渲染也必须发生在 VNC/X11 客户端内。

---

## 11. 可复用排查清单

- [ ] `nvidia-smi` 确认 GPU 和驱动存在
- [ ] `glxinfo -B` 确认系统 OpenGL 能看到 NVIDIA
- [ ] STAR 场景执行 Graphics Report
- [ ] 判断是否为 `STAR Mesa OpenGL / SWR / llvmpipe`
- [ ] 不要把 `glxinfo` 的 NVIDIA renderer 等同于 STAR renderer
- [ ] 若客户端 native OpenGL 在 VNC 中异常，优先尝试服务器端 renderer
- [ ] Linux NVIDIA 环境优先测试：
  ```bash
  starccm+ -rr server -rgraphics egl -rgpu 0
  ```
- [ ] 用 Graphics Report 检查是否出现：
  ```text
  ***Report generated remotely***
  ```
- [ ] 用 `nvidia-smi` / `dmon` 辅助确认 GPU 负载
- [ ] GPU 渲染正常后，再调整 STAR 中“GPU 使用”设置

---

## 12. 最终推荐启动方式

当前超算环境推荐直接使用：

```bash
starccm+ -rr server -rgraphics egl -rgpu 0
```

如果是需要在远程计算节点上运行仿真，同时在本地管理节点渲染图形，上述命令中的`server`需要改成具体的渲染节点名称，如：

```bash
starccm+ -rr gpu01 -rgraphics egl -rgpu 0
```

如需日常使用，可配置：

```bash
alias star_gpu='starccm+ -rr server -rgraphics egl -rgpu 0'
```

然后：

```bash
star_gpu
```

另外，如果开启GPU渲染后，图形操作时图形变糊，等个2s左右又变得清晰，可以在工具>选项>专家下，将“远程渲染压缩”设置为“无损”

---

## 13. 保底软件渲染方案

如果某节点确实无法使用 GPU，可使用：

```bash
starccm+ -graphics mesa_swr
```

OpenSWR 通常优于 LLVMpipe。

同时可在：

```text
工具 → 选项 → STAR-CCM+ → 可视化
```

调高交互刷新/降级相关设置，以改善软件渲染时的旋转响应。

---

## 14. 结论

本问题最终不是“超算 GPU 性能差”或“网络太慢”，而是 **STAR-CCM+ 客户端在 VNC 环境下回退到了 Mesa/OpenSWR CPU 软件渲染**。

VirtualGL 能正常访问 A5000，但 STAR 的 VTK native offscreen OpenGL context 无法通过该路径稳定初始化。查阅官方文档后，改为 STAR 自带的：

```text
Server-side Remote Rendering
+
EGL
+
RTX A5000
```

最终通过：

```bash
starccm+ -rr server -rgraphics egl -rgpu 0
```

解决问题。

---

*创建: 2026-09-22 | 关闭: 2026-09-23*

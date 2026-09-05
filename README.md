深入通信物理层（PHY）和协议栈的源码是一项极具挑战但也非常有成就感的工作。这要求你具备**数字信号处理（DSP）**、**通信原理**以及**C++/Python 混合编程**的基础。

三个项目：**`python_5gtoolbox`**（侧重 3GPP 标准算法验证）、**`gr-ieee802-11`**（侧重 SDR 硬件实战与 GNU Radio 可视化）、**`srsRAN`**（侧重工业级系统架构与底层性能优化），我为你制定了一套 **“从仿真到真实世界，再到工业级架构”的 4 阶段进阶实战方案**。

---

### 🛠️ 阶段零：基础设施准备（避坑指南）
通信项目的环境配置极其繁琐（依赖特定的数学库、SDR驱动等），强烈建议**不要在 Windows 下直接折腾**。
1. **操作系统**：使用 **WSL2 (Ubuntu 22.04)** 或原生 Ubuntu Linux。
2. **硬件准备**：购买一块入门级 SDR（软件定义无线电）开发板。推荐 **ADALM-PlutoSDR**（性价比极高，适合入门）或 **HackRF One**（带宽大，适合 WiFi 抓包）。
3. **必备工具**：
   - 抓包分析：**Wireshark**（分析 MAC/RRC 层信令）
   - 可视化：**GNU Radio Companion (GRC)**（查看眼图、星座图、时频图）

---

### 🟢 阶段一：深挖 `python_5gtoolbox`（吃透 5G 协议与算法）
**目标**：利用 Python 的易读性，搞懂 3GPP 标准中复杂的数学公式和矩阵运算。
**核心优势**：不需要 SDR 硬件，纯软件仿真，适合加断点单步调试。

#### 📌 具体行动路径：
1. **环境搭建与跑通 Demo**：
   - 解决克隆问题后，使用 `poetry install` 搭建环境。
   - 运行仓库 `tests/` 目录下的单元测试，确保 LDPC 和 Polar 编码解码能跑通。
2. **源码追踪：OFDM 调制与解调**：
   - 找到 `py5gphy/nr_pdsch/` 和 `py5gphy/nr_pusch/` 目录。
   - **断点调试**：在 `nr_ofdm.py`（或类似文件）中打断点，观察频域上的 QAM 符号是如何经过 `np.fft.ifft` 变成时域信号的。
   - **魔改实验**：尝试修改**CP（循环前缀）** 的长度，或者人为在时域信号中加入高斯白噪声（AWGN），观察接收端星座图的发散情况。
3. **源码追踪：信道编码（Polar & LDPC）**：
   - 深入 `py5gphy/polar/` 和 `py5gphy/ldpc/`。
   - 对比 3GPP TS 38.212 协议，看代码是如何生成**生成矩阵 (Generator Matrix)** 和进行**速率匹配 (Rate Matching)** 的。
4. **实战挑战**：写一个脚本，生成一段包含自定义字符串的 5G PDSCH 基带波形，保存为 `.bin` 文件，然后自己写一个简单的 Python 解调器把它还原出来。

---

### 🟡 阶段二：深挖 `gr-ieee802-11`（结合 SDR 玩转真实 WiFi 信号）
**目标**：掌握 GNU Radio 框架，理解真实世界中的同步、信道估计和均衡技术。
**核心优势**：高度可视化，能直接“听到”和“看到”空中的 WiFi 电磁波。

#### 📌 具体行动路径：
1. **环境搭建（使用 Docker 保平安）**：
   - 官方提供了 Docker 镜像，直接运行 `docker run -it gnuradio/gr-ieee802-11`，免去编译各种依赖的痛苦。
2. **无硬件仿真（GRC 流程图）**：
   - 打开 `apps/wifi_phy.grc`。这是一个完整的 WiFi 物理层收发机。
   - 将发射端和接收端直接连接，中间插入一个 **Channel Model（信道模型）**。
   - **观察重点**：打开接收端的 **Constellation Sink（星座图）** 和 **QT GUI Time Sink（时域图）**，观察同步前后的信号变化。
3. **源码追踪：帧同步机制（核心难点）**：
   - 在 GNU Radio 中定位到 `sync_short` 和 `sync_long` 模块（C++ 实现）。
   - **阅读代码**：WiFi 是如何利用 Short Training Field (STF) 的**自相关性**来做粗同步的？如何利用 Long Training Field (LTF) 做**细同步和频偏估计（CFO）**？
4. **源码追踪：信道估计与均衡**：
   - 找到 `frame_equalizer` 模块。看代码是如何提取 LTF 的导频，计算出信道响应 $H$，并使用 **Zero-Forcing (迫零)** 或 **MMSE** 算法对接收到的 QAM 符号进行除法补偿的。
5. **实战挑战**：插上你的 HackRF/PlutoSDR，运行 `wifi_transceiver.grc`。让你的电脑发出真实的 WiFi 信号，用 SDR 接收并在 GRC 中解出 MAC 层的 Beacon 帧，最后在 Wireshark 中查看抓到的包！

---

### 🔴 阶段三：深挖 `srsRAN`（剖析工业级 C++ 协议栈）
**目标**：学习高并发、内存管理、DSP 硬件加速（SIMD 指令集），理解基站到底是怎么调度的。
**核心优势**：这是目前最接近商用基站和终端的开源实现。

#### 📌 具体行动路径：
1. **环境搭建（ZMQ 虚拟化）**：
   - **不需要买昂贵的 USRP！** 编译 `srsRAN` 时开启 **ZeroMQ (ZMQ)** 支持。
   - 在一台 Linux 机器上，同时运行 `srsenb`（基站）和 `srsue`（终端），它们会通过 ZMQ 在内存中虚拟出射频线缆进行通信。
2. **源码追踪：MAC 层调度器（Scheduler）**：
   - 定位到 `srsenb/src/mac/scheduler_*.cc`。
   - **阅读代码**：基站是如何每 1ms（一个 TTI）决定把哪些资源块（RB）分配给哪个用户的？研究代码中的 **Round Robin（轮询）** 和 **Proportional Fair（比例公平）** 调度算法。
   - **魔改实验**：把调度算法改成 FIFO（先进先出），然后跑 iPerf 测速，观察吞吐量和不同用户间的公平性变化。
3. **源码追踪：DSP 物理层加速（性能密码）**：
   - 定位到 `lib/src/phy/` 目录。
   - 寻找带有 `avx2`、`sse` 或 `neon` 后缀的 C++ 文件。看开发者是如何使用 **SIMD 指令集** 来并行计算 FFT、LDPC 译码和信道均衡的。这是拉开业余代码和工业级代码性能差距的核心。
4. **实战挑战**：使用 Wireshark 抓取 `srsenb` 和 `srsue` 之间的 S1AP/NGAP 信令接口，结合源码中的 RRC 状态机（`rrc_ue.cc`），画出 5G 终端从开机到接入基站的完整信令交互时序图。

---

### 🏆 终极融合项目 (Capstone Project)
当你分别啃完这三个项目的部分源码后，尝试做以下一个融合挑战，这足以写进你的高级简历中：

**挑战：基于真实 SDR 的离线 5G 同步信号（SSB）解码器**
1. **抓信号**：使用 `gr-ieee802-11` 学到的 SDR 采集技术，用 HackRF/PlutoSDR 录制一段现实世界中中国移动/电信/联通的 5G 基站下行信号（Sub-6G 频段）。
2. **写 DSP**：使用 `python_5gtoolbox` 中提供的 PSS（主同步信号）生成算法，自己写一个 Python 脚本，对录制的 IQ 数据进行**滑动互相关**，找到 5G 帧的起始位置。
3. **解 MIB**：截取 SSB 信号，利用 `python_5gtoolbox` 中的 Polar 解码器，解出 MIB（Master Information Block），提取出该基站的物理小区 ID (PCI) 和系统帧号 (SFN)。

### 💡 学习建议与避坑指南
1. **不要试图读懂每一行代码**：通信代码极其庞大。一定要**带着问题去读**。比如：“WiFi 是怎么做频偏补偿的？” 然后全局搜索 `cfo` 或 `frequency offset`，只看相关的几十行代码。
2. **善用 GDB 和 Python Debugger**：在 GNU Radio 的 C++ OOT (Out-of-Tree) 模块中，使用 `printf` 或 `spdlog` 打印中间变量；在 Python 中使用 `pdb` 查看矩阵维度变化。
3. **结合论文阅读**：代码往往是论文的落地。在看 `gr-ieee802-11` 的同步代码时，去搜一下 Schmidl & Cox 同步算法的论文，会有醍醐灌顶的感觉。

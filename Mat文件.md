1、 .mat  文件是什么
 
 .mat  是 MATLAB 的二进制数据文件，MathWorks 定义的格式，用来保存矩阵、数组、结构体、参数配置。
在这个 5G‑PDSCH 项目里：
 
- 里面存两类东西：
1. 配置参数：SCS、带宽、RB数量、起始符号、PDSCH符号时长（这里  pdschsym_dur_8  = PDSCH占用8个OFDM符号）
2. 黄金参考数据：调制符号、OFDM时域IQ复数波形，作为标准基准向量，用来和 Python 实现做误差比对（pytest做一致性校验）
 
文件名含义拆解：
 nrPDSCH_short_testvec_105_scs_30khz_BW_20_StartSymbolIndex_0_pdschsym_dur_8.mat 
 
- scs_30khz：子载波间隔30kHz
- BW_20：20MHz带宽
- StartSymbolIndex_0：PDSCH从时隙第0个OFDM符号开始
- pdschsym_dur_8：连续占用8个OFDM符号（短PDSCH）
 
2、Python 解析 mat 文件（依赖  scipy ）
 
你的虚拟环境已经可以直接用，如果缺失：
 
bash
  
uv add scipy numpy
 
 
完整读取示例代码
 
python
  
import scipy.io
import numpy as np

# 文件路径
file_path = "tests/nr_pdsch/testvectors_short_pdsch/nrPDSCH_short_testvec_105_scs_30khz_BW_20_StartSymbolIndex_0_pdschsym_dur_8.mat"

# 读取mat字典
mat_dict = scipy.io.loadmat(file_path)

# 打印所有保存的变量名（知道文件里存了什么）
print("mat内部变量列表：", list(mat_dict.keys()))
 
 
运行后你会看到类似这些关键字段（不同版本命名略有区别）：
 
-  pdsch_symbols ：频域PDSCH调制复数符号
-  waveform ：OFDM调制后的时域复数IQ波形
-  cfg ：结构体，保存整套NR配置（SCS, NRB, MCS, 起始符号、符号长度）
 
读取复数波形
 
python
  
# 读取时域基带IQ信号（复数 I + jQ）
waveform = mat_dict["waveform"].flatten()

# 实部I，虚部Q
I = np.real(waveform)
Q = np.imag(waveform)

print("波形采样点数：", len(waveform))
print("前5个复数值：", waveform[:5])
 
 
读取配置参数示例
 
python
  
cfg = mat_dict["cfg"]
# matlab结构体取出参数（注意mat返回嵌套数组）
nrb = int(cfg["nrb"][0,0][0,0])
scs = int(cfg["scs"][0,0][0,0])
start_sym = int(cfg["StartSymbolIndex"][0,0][0,0])
sym_dur = int(cfg["pdschsym_dur"][0,0][0,0])

print(f"NRB={nrb}, SCS={scs}, StartSym={start_sym}, Duration={sym_dur}")
 
 
3、配合 Plotly 绘图（直接可视化这份标准参考波形）
 
python
  
import plotly.graph_objects as go

# 时域波形绘制
t = np.arange(len(waveform)) / (scs * 12 * nrb)
fig = go.Figure()
fig.add_trace(go.Scatter(x=t, y=I, name="I"))
fig.add_trace(go.Scatter(x=t, y=Q, name="Q"))
fig.update_layout(title="MATLAB标准短PDSCH时域波形", xaxis_title="Time(s)", yaxis_title="Amplitude")
fig.write_html("short_pdsch_ref.html")
fig.show()
 
 
4、版本坑点补充
 
1. v7.3超大mat文件： scipy.io.loadmat  读不了，改用  h5py 
 
bash
  
uv add h5py
 
 
2. MATLAB的二维矩阵一般是  [samples,1] ，Python需要  .flatten()  压成一维数组
3. pytest 的逻辑就是：用cfg在Python生成波形，和mat里waveform做误差对比
 
5、常用工程操作清单
 
1. 查看文件内全部变量名 →  list(mat_dict.keys()) 
2. 获取复数IQ数组 →  waveform = mat_dict["waveform"].ravel() 
3. 取出3GPP配置参数
4. 星座图 / PSD / 时域波形绘图对比自实现算法与MATLAB黄金向量
 
如果你需要，我可以写一段一键解析+自动绘图的完整脚本，专门适配这套short‑pdsch测试向量。

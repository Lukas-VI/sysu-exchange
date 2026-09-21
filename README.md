# sysu-exchange

> 2026 年度，为期一年的中山大学网络空间安全学院交流。
> 本仓库是你进入导师研究组前的**学习资料库**：`reference/` 以 git submodule 形式收录了导师在**图像复原（Image Restoration）**方向的全部参考代码仓库，每个仓库都已用**中文逐行注释**并附加了**中文导读 README**，供系统学习。

---

## 目录结构

```
sysu-exchange/
├── reference/          # 14 个图像复原研究仓库（git submodule）
│   ├── AdaIR/  ConvIR/  IRNeXt/  NDMamba/  ...   # 图像复原算法
│   └── MC-Blur-Dataset/  AwesomeDehazing/        # 数据集工具 / 论文清单
├── CV/                 # 个人简历等
└── docs/               # 其他文档
```

## submodule 使用说明

克隆主仓库后，一次性拉取全部子模块：

```bash
git clone --recurse-submodules https://github.com/Lukas-VI/sysu-exchange.git
# 或已克隆后执行：
git submodule update --init --recursive
```

日常更新：

```bash
git submodule update --remote   # 拉取所有子模块远端最新
```

---

## 跨项目总览

导师组工作在**单图 / 视频图像复原**方向，覆盖去雾、去雪、去雨、去模糊、低照度增强与通用复原。按任务方向分类：

| 类别 | 项目 | 说明 |
|------|------|------|
| **通用图像复原**（统一网络） | [AdaIR](reference/AdaIR) | 多尺度 U 形 Transformer，频率调制（FFT 高低频分解）贯穿解码 |
| | [ConvIR](reference/ConvIR) | 纯卷积复原，多尺度 U-Net + 频域损失 |
| | [IRNeXt](reference/IRNeXt) | 三尺度 CNN，动态滤波 + 三尺度 L1/FFT 监督 |
| | [NDMamba](reference/NDMamba) | Mamba/状态空间模型，时间步 scan 机制 |
| **频率/池化技巧复原**（单图多退化） | [SFNet](reference/SFNet) | 选择性频率网络，SFconv 高低频竞争注意力（ICLR'23） |
| | [FSNet](reference/FSNet) | 频率选择，动态滤波 + 高低频分离（TPAMI'23） |
| | [FocalNet](reference/FocalNet) | 焦点注意力，局部对比度调制 + 多尺度（去雾/去雪） |
| | [OKNet](reference/OKNet) | Omni-Kernel 大核分解 + 频域通道注意力（AAAI'24，去雾/去雪） |
| | [PoolNet](reference/PoolNet) | 池化金字塔（8 种池化并联），去雪/去雾/All-in-One |
| | [VIVNet](reference/VIVNet) | 视觉内注意力（BIM），通用图像/视频复原基线（TPAMI） |
| **视频复原** | [VIVNet](reference/VIVNet) | 见上：图像 + 视频通用基线 |
| | [Enhanced-Video-Deraining](reference/Enhanced-Spatio-Temporal-Interaction-Learning-for-Video-Deraining) | 粗→精两阶段时空建模，R-CLSTM 沿时间维递推（视频去雨） |
| **低照度增强** | [DI-retinex](reference/DI-retinex-low-light-enhancement) | Retinex 分解 + 亮度/对比度增强（无监督） |
| **数据/工具** | [MC-Blur-Dataset](reference/MC-Blur-Dataset) | MC-Blur 数据集下载/解压工具 |
| **论文清单** | [AwesomeDehazing](reference/AwesomeDehazing) | 去雾论文综述索引（无代码） |

---

## 建议阅读顺序

导师组代码高度同源，读通一套再去读其他仓库可触类旁通。多数仓库的
`layers.py + 模型名.py + train.py + data_load.py` 结构几乎一致，并共享一个
**核心范式**：

> **多尺度 U 形编解码 CNN + FAM（特征融合）/ SCM（条件引导）+ 空间域 L1 损失 + 0.1×频域 FFT 损失**

按由浅入深的顺序学习：

1. **IRNeXt**（或 FSNet）—— 结构最典型、最简单，先把"多尺度 + 动态滤波 + FFT 双监督"跑通。
2. **SFNet** —— 理解频率分解与选择性频率注意力。
3. **FocalNet / OKNet / PoolNet** —— 看注意力/池化/大核卷积等不同"插件"如何改进同一套骨架。
4. **AdaIR / ConvIR / NDMamba** —— 统一复原大模型（Transformer / Mamba），结构更复杂但思想一脉相承。
5. **VIVNet / Enhanced-Video-Deraining** —— 进入视频维度，重点理解 5 维张量 `(b,c,t,h,w)` 与时空建模。
6. **DI-retinex** —— 换一个子方向（低照度），了解传统 Retinex 假设怎么被建进网络。

## 学习导图（文字版）

```
图像复原 IR
├── 降质建模：退化生成（加噪/模糊/雾/雪/雨）
├── 网络骨架：多尺度 U 形编解码（EBlock×3 编码 + DBlock×3 解码 + 跳连）
│   ├── 特征融合 FAM / 条件引导 SCM
│   └── 各种"插件"：频率分解(SFNet/FSNet)、注意力(FocalNet/OKNet)、池化(PoolNet)、视觉内注意力(VIVNet)、状态空间(NDMamba)
├── 损失：空间 L1 + 频域 FFT L1（0.1 权重）双监督，三尺度输出
├── 训练：warmup + 余弦退火；多任务副本共享网络（去雾/去雪/去模糊）
└── 视频：粗精两阶段 + R-CLSTM 时间递推 / 3D 卷积
```

---

## 常见易踩坑提示（详见各仓库中文 README）

- 多处 `clip_grad_norm_` 阈值极小（0.001~0.01），训练时留意是否梯度被过度裁剪。
- 部分仓库存在代码笔误/冗余参数（如未使用的 `lamb_l/lamb_h`、未更新的 `best_psnr`、`eval.py` 引用未定义变量等），注释中已用"（理解存疑）"标注，不摄入原逻辑。
- 多任务副本（FocalNet/FSNet/SFNet/OKNet 等）内部是按任务复制出的相似目录，主看一套即可。
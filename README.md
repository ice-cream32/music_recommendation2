# 🎵 基于序列建模与图增强的音乐推荐系统研究

## 📌 一、项目背景与研究意义

在音乐推荐系统中，**用户行为序列稀疏与冷启动问题**始终是核心挑战之一。传统基于协同过滤的方法（如仅使用 ID embedding）在用户交互较少时表现显著下降。

本项目围绕以下关键问题展开：

> **如何在用户行为稀疏的情况下，提升推荐系统的鲁棒性与泛化能力？**

为此，本文从**序列建模 + 内容特征 + 图结构建模**三个维度出发，构建多模型对比实验体系。

---

## 🧠 二、核心贡献（Contributions）

本项目的主要创新点包括：

* 🔹 提出一种**双流序列建模结构（FDSA, Feature-level Dual Self-Attention）**

  * 同时建模 ID 信息与音频内容特征
* 🔹 引入**图增强机制（Graph-enhanced Embedding）**

  * 建模物品之间的高阶关系
* 🔹 构建**冷启动极限测试框架**

  * 精细划分短序列与中等序列用户
* 🔹 设计**多维评价体系**

  * 精度指标 + Beyond-Accuracy 指标（覆盖率、流行度偏置）

---

## 📊 三、数据集与特征说明

### 3.1 用户行为数据（Multi-Event Log）

| 字段                   | 含义                         |
| -------------------- | -------------------------- |
| uid                  | 用户 ID                      |
| item_id              | 音乐 ID                      |
| timestamp            | 行为时间                       |
| event_type           | 行为类型（listen / like / skip） |
| played_ratio_pct     | 播放完成比例                     |
| track_length_seconds | 音乐时长                       |

---

### 3.2 音频特征（Audio Embedding）

* 使用CNN模型提取音频向量（包含在数据集中）
* 向量维度：128
* 支持：

  * L2 归一化
  * 相似度计算（cosine similarity）

---

### 3.3 图结构信息

* 基于用户共现行为构建 item-item 图
* 邻接关系反映：

  * 共听关系
  * 语义相似性

---

### 3.4 数据集来源（Dataset Source）

本项目所使用的数据集来源于公开的音乐推荐数据集，包含用户行为日志与音频特征信息，具体说明如下：

#### 📌 数据来源

* 数据集名称：**Yandex Music Dataset**
* 发布平台：RecSys Challenge / Academic Dataset
* 发布网址：https://huggingface.co/datasets/yandex/yambda
* 数据类型：

  * 用户行为序列（listen / like / skip 等）
  * 音频内容特征（预计算 embedding）

---

#### 📊 数据组成

本项目使用的数据主要包括：

1. **用户交互数据（Multi-event logs）**

   * 记录用户与音乐的交互行为
   * 包含时间序列信息

2. **音频特征数据（Audio Embedding）**

   * 由 CNN 模型提取
   * 维度为 128
   * 用于增强内容理解能力

---

#### ⚠️ 数据使用声明

* 本数据集仅用于**学术研究与教学用途**  
* 若涉及商业使用，请遵循原数据集发布方的许可协议  

---

## 🏗 四、模型设计

### 4.1 基线模型：ID-Only

仅使用物品 ID embedding：

$$
h_i = \text{Embedding}(item_i)
$$

特点：

* 优点：简单高效
* 缺点：无法处理冷启动问题

---

### 4.2 FDSA 双流模型

模型结构如下：

* **ID 流（协同过滤）**
* **内容流（音频 embedding）**

融合方式：

$$
h_i = \text{Attention}(h_i^{id}, h_i^{content})
$$

优势：

* 融合行为与内容信息
* 提升泛化能力

---

### 4.3 FDSA + Graph 增强模型（最终模型）

在 FDSA 基础上，引入图嵌入：

$$
h_i^{final} = h_i^{fdsa} + h_i^{graph}
$$

其中：

* $h_i^{graph}$ 表示图结构学习得到的 embedding
* 捕捉高阶关系（multi-hop）

---

## ⚙️ 五、训练目标函数

采用对比学习形式的交叉熵损失：

$$
\mathcal{L} = -\log \frac{\exp(\text{sim}(h_i, e_i)/\tau)}{\sum_{j} \exp(\text{sim}(h_i, e_j)/\tau)}
$$

其中：

* $\text{sim}(\cdot)$：余弦相似度
* $\tau$：温度系数（temperature）
* 正样本：真实下一首歌曲
* 负样本：随机采样（1 vs 999）

---

## 🧪 六、实验设计

### 6.1 任务定义

* 任务：Next-item Recommendation
* 输入：用户历史序列
* 输出：下一首歌曲

---

### 6.2 评估设置

* 候选集：**1 正样本 + 999 负样本**
* 评价指标：

#### 🎯 精度类指标

* Recall@K
* NDCG@K

#### 🎵 Beyond-Accuracy 指标

* Coverage@K（覆盖率）
* Popularity Bias（流行度偏置）

---

### 6.3 冷启动划分

| 用户类型   | 定义           |
| ------ | ------------ |
| Short  | 序列长度 < 10    |
| Medium | 10 ≤ 长度 ≤ 30 |

---

## 📈 七、实验结果与分析

### 7.1 总体性能对比

| 模型           | Recall@100 | NDCG@100 |
| ------------ | --------- | ------- |
| ID-Only      | 0.7941        | 0.4043      |
| FDSA         | 0.8028      | 0.3888    |
| FDSA + Graph | 0.8781      | 0.5173    |

---

### 7.2 冷启动性能

实验表明：

* Graph 模型在短序列用户上显著优于基线
* 内容特征有效缓解冷启动问题
* 图结构增强进一步提升鲁棒性

---

### 7.3 关键结论

> **序列建模 + 内容特征 + 图结构的融合，是解决推荐系统冷启动问题的有效路径。**

---

## 📂 八、项目结构

```
├── music_idonly.ipynb           # 基线模型
├── music_FDSA.ipynb             # 双流模型
├── music_graph.ipynb            # 图增强模型
├── analysis_compare.ipynb       # 模型对比分析
├── drawings.ipynb               # 可视化与绘图
```

---

## ⚙️ 九、环境配置

```bash
Python >= 3.10
PyTorch >= 2.0
NumPy
Pandas
tqdm
```

---

## 🚀 十、运行方法

### 1️⃣ 模型训练与评估

```bash
依次运行：
- music_idonly.ipynb
- music_FDSA.ipynb
- music_graph.ipynb
```

### 2️⃣ 对比分析

```bash
运行 analysis_compare.ipynb
```

---

## 🤗 十一、预训练模型（Pretrained Models）

本项目训练得到的模型已上传至 Hugging Face，便于复现与使用：
* 🔗 模型主页（Model Hub）：https://huggingface.co/Pcpp/music-transformer
* 🎧 ID-Only 基线模型：https://huggingface.co/Pcpp/music-transformer/resolve/main/best_model_id_only.pth
* 🚀 FDSA + Graph 增强模型（最终模型）：https://huggingface.co/Pcpp/music-transformer/resolve/main/best_model_shared_gated_pro.pth

---

## 👨‍🎓 十二、作者信息

* 作者：彭乐
* 项目类型：本科毕业论文
* 研究方向：推荐系统 / 深度学习 / 序列建模

---


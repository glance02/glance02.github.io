# Vision Transformer（ViT）简介

## 1. ViT 是什么？

Vision Transformer（ViT）是一种将 Transformer 架构应用于图像理解的模型。Transformer 最初主要用于自然语言处理：它把句子拆成一个个词（token），再通过自注意力机制理解词与词之间的关系。

ViT 把这个思路迁移到图像上：将图像切分成许多固定大小的小图块（patch），把每个图块看作一个视觉 token，然后使用 Transformer Encoder 对这些 token 进行建模。

因此，ViT 可以简单理解为：

> 把图像切成小块，再像阅读一串词一样理解这些图块之间的关系。

## 2. 基本处理流程

以一张 `224 × 224` 的图像和 `16 × 16` 的 patch 为例：

```text
224 × 224 图像
      ↓
切分为 14 × 14 个 patch
      ↓
得到 196 个 patch token
      ↓
线性投影为 token embedding
      ↓
加入位置编码
      ↓
Transformer Encoder
      ↓
分类、分割或其他视觉任务
```

patch 数量为：

```text
(224 / 16) × (224 / 16) = 14 × 14 = 196
```

每个 `16 × 16` 的图块会被展平为一个向量，再通过线性层映射到 Transformer 所需的 embedding 维度。由于 Transformer 本身不直接知道图块的空间位置，因此还要加入位置编码。

在图像分类中，ViT 通常还会在序列开头加入一个特殊的 `[CLS]` token。经过多层 Transformer Encoder 后，使用 `[CLS]` token 的表示进行分类。

## 3. 自注意力在图像中做什么？

自注意力机制会计算不同 patch 之间的关联程度。一个 patch 在更新自己的表示时，可以参考图像中其他位置的 patch。

例如，在眼科图像中，模型可能学习到：

- 眼底血管与视盘之间的结构关系；
- OCT 中不同视网膜层之间的关系；
- 病灶区域与周围组织之间的关系；
- 相距较远的图像区域之间的整体形态关系。

这使 ViT 不仅能看局部纹理，还能较早地建立全局上下文。

## 4. ViT 与 CNN 的区别

| 方面 | CNN | ViT |
| --- | --- | --- |
| 基本单元 | 卷积核 | patch token 和自注意力 |
| 主要建模方式 | 逐层提取局部特征 | 建模不同 patch 之间的关系 |
| 空间先验 | 卷积天然包含局部性和平移等变性 | 主要依靠数据和位置编码学习 |
| 全局信息 | 通常需要多层卷积逐步扩大感受野 | 自注意力可以直接关联远距离 patch |
| 数据需求 | 相对较低 | 通常更依赖大规模预训练 |
| 典型用途 | 分类、检测、分割 | 分类，也可扩展到检测、分割等任务 |

ViT 并不意味着 CNN 已经没有价值。CNN 的局部归纳偏置在小数据集上仍然很有用；ViT 则在大规模数据和预训练条件下，具有较强的表示能力和迁移能力。

## 5. 为什么 VisionFM 使用 ViT？

VisionFM 为八种眼科影像模态分别设置了编码器，这些编码器都基于 Vision Transformer。不同影像的外观差异很大：眼底照片是彩色二维图像，OCT 是断层图像，MRI 和超声又有不同的灰度与纹理特征。

独立的模态编码器可以先学习每种影像自身的视觉规律，再将得到的表示交给任务专用解码器，完成诊断、预测、分割或关键点检测等任务。

大规模眼科图像预训练也有助于发挥 ViT 的优势。VisionFM 使用约 340 万张眼科图像进行预训练，并采用 iBOT 自监督学习方法，使编码器能够在缺少人工标注的情况下学习通用视觉特征。

需要注意的是，VisionFM 的“多模态”主要指多种眼科医学成像模态，并不等同于图像与自然语言联合输入的视觉语言模型。

## 6. 经典论文

### ViT 原始论文

Dosovitskiy et al. **An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale.** ICLR, 2021.

- 论文链接：[arXiv:2010.11929](https://arxiv.org/abs/2010.11929)
- 主要贡献：提出将图像切成 patch，并直接使用标准 Transformer Encoder 进行图像分类。

### 相关论文

1. Touvron et al. **Training Data-Efficient Image Transformers & Distillation through Attention.** ICML, 2021.
   - 提出 DeiT，研究在较少数据条件下训练视觉 Transformer。

2. Liu et al. **Swin Transformer: Hierarchical Vision Transformer using Shifted Windows.** ICCV, 2021.
   - 使用局部窗口和移位窗口构建层次化特征，适合检测和分割等任务。

3. Bao et al. **BEiT: BERT Pre-Training of Image Transformers.** ICLR, 2022.
   - 将类似 BERT 的掩码预训练思想应用到图像 Transformer。

4. Zhou et al. **iBOT: Image BERT Pre-Training with Online Tokenizer.** ICLR, 2022.
   - 结合掩码图像建模和自蒸馏，是 VisionFM 采用的自监督预训练方案之一。

## 7. 一句话总结

Vision Transformer 就是“把图像 patch 当成 token，用 Transformer 的自注意力机制理解整张图像”。在 VisionFM 中，ViT 充当不同眼科影像模态的基础编码器，负责把原始图像转换为可以用于多种临床任务的视觉表示。

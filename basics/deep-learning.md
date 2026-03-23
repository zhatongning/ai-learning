# 深度学习基础

> 神经网络和深度学习的核心原理

---

## 什么是深度学习？

深度学习是机器学习的一个子集，使用多层神经网络学习数据的表示。

---

## 神经网络基础

### 感知机（Perceptron）

最基础的神经网络单元。

```
输入 [x1, x2, ..., xn]
  │
  ▼
权重 [w1, w2, ..., wn]
  │
  ▼
加权和 + 偏置
  │
  ▼
激活函数
  │
  ▼
输出
```

### 常见激活函数

- ReLU（Rectified Linear Unit）
- Sigmoid
- Tanh
- Softmax

---

## 深度神经网络架构

### 1. CNN（卷积神经网络）

主要用于图像处理。

**关键组件**：
- 卷积层（Convolutional Layer）
- 池化层（Pooling Layer）
- 全连接层（Fully Connected Layer）

### 2. RNN（循环神经网络）

主要用于序列数据。

**变体**：
- LSTM（Long Short-Term Memory）
- GRU（Gated Recurrent Unit）

### 3. Transformer

革命性的架构，用于序列建模。

**核心概念**：
- 自注意力机制（Self-Attention）
- 多头注意力（Multi-Head Attention）
- 位置编码（Positional Encoding）

---

## 大语言模型（LLM）

基于 Transformer 架构的预训练模型。

### 预训练方法

- **自监督学习**
- **掩码语言建模（MLM）**（如 BERT）
- **因果语言建模（CLM）**（如 GPT）

### 微调（Fine-tuning）

在特定任务上调整预训练模型。

### 参数高效微调

- LoRA（Low-Rank Adaptation）
- Prefix Tuning
- Adapter Layers

---

## 待完善内容

- [ ] 每种架构的详细说明
- [ ] 代码示例（PyTorch/TensorFlow）
- [ ] 经典模型分析
- [ ] 实际项目案例

---

**上一章**: [机器学习入门](machine-learning.md) | **返回**: [首页](/README.md)

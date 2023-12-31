---
title: 分类模型评估（混淆矩阵, precision, recall, f1-score）的原理和Python实现
date: 2023-12-01 16:30:00 +0800
categories: [学习笔记]
tags: [dsml]     # TAG names should always be lowercase
img_path: /assets/img/
---

## 混淆矩阵

当我们已经获取到一个分类模型的预测值，可以通过不同指标来进行评估。

往往衡量二分类模型是基于以下的混淆矩阵概念：

- True Positive：真实值为正、预测值为正（真阳性）
- False Positive：真实值为负、预测值为正（假阳性）
- False Negative：真实值为正、预测值为负（假阴性）
- True Negative：真实值为负、预测值为负（真阴性）

但面对多个分类，比如40多个类别时无法单纯通过正负来混淆矩阵的每个值。在多个类别分类中，可以将每个类别视为应该独立的二元分类问题。对于每个类别A，其余不是类别A的样本可以临时合并为应该“非A”类别。我们将以上定义为：

- 真阳性 (TP)：对于特定类别A，TP是**正确标记为A**的样本数量。
- 假阳性 (FP)：对于特定类别A，FP是**错误地**标记为A的**其他**类别的样本数量。

- 假阴性 (FN)：对于特定类别A，FN是**实际为A**但**没有被标记为A**的样本数量。

- 真阴性 (TN)：对于特定类别A，TN是既不属于A也没有被标记为A的样本数量。

## 多分类指标

### 准确度 Accuracy

$$
Accuracy = \frac{TP+TN}{\text{No.samples}}
$$

- 准确率指的是所有预测准确的样本的占比。
- 适用于类别分布平衡的情况，但是再类别不平衡的数据集中可能不是非常靠谱。

### 精确度 Precision

$$
Precision = \frac{TP}{TP+FP}
$$

- 精确度指的是真阳性在所有正类中的比例，又叫测准率。对于某个类别A，相当于正确判断为A的数量在所有类别A的数量中的比例。
- 精确度高意味着较少的假阳性（误报）

### 召回率 Recall（灵敏度 Sensitivity）

$$
Recall = \frac{TP}{TP+FN}
$$

- 召回率指的是真阳性在**所有被预测为正类数据**中的比例，表示的是模型获取正类的能力。
- 召回率高表示漏报正确样本的情况少

### F1 Score

$$
F1 Score = 2 \times \frac{Precision \times Recall}{Presision+Recall}
$$

- F1 Score用于衡量精确度和召回率之间的平衡，作为评估标准更加全面。

- 适用于评估**类别不平衡**的情况。

- F1 Score相当于 Precision 和 Recall的调和平均数
  $$
  F1 Score = \frac {2TP}{2TP+FP+FN}
  $$

  - **调和平均数 (Harmonic mean)** 经常被用与分子相同、分母不同的场合，将分母调成平均数再当分母。
    $$
    H_n = \frac{n}{\sum_{i=1}^n \frac{1}{x_i}}
    $$

其中后三种measure在衡量整个数据时，通过以下方式汇总这些指标：

- **宏观平均 (Macro-average)**：对每个类别计算指标，然后计算这些指标的平均值。这种方法对所有类别给予了相同的重要性，即使它们的样本量不同。
- **加权平均 (Weighted-average)**：与宏观平均类似，但是在计算平均值时考虑到了每个类别的样本量。这对于不平衡的数据集特别有用。
- **微观平均 (Micro-average)**：将所有类别的TP、FP和FN累加起来，然后计算指标。在不平衡的数据集中，微观平均通常被认为更为公平。



## 在Python中绘制混淆矩阵

```python
import matplotlib.pyplot as plt
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay

y_true = [...]	# 正确的标签
y_pred = [...]	# 预测的标签

conf_mat = confusion_matrix(y_true, y_pred)
disp = ConfusionMatrixDisplay(confusion_matrix=conf_mat,
                              display_labels=np.unique(y_true)
                              )
disp.plot()
```



## 在Python中计算各分类指标

python中想要计算如上指标主要是使用 `sklearn` 包中预先写好的函数。可以使用以下代码进行计算：

```python
from sklearn.metrics import precision_score, recall_score, f1_score, accuracy_score

y_true = [...]	# 正确的标签
y_pred = [...]	# 预测的标签

# 计算正确率
accuracy = accuracy_score(y_true, y_pred)
# 计算精确度、召回率和F1分数
precision = precision_score(y_true, y_pred, average='macro')  # 'macro'表示未加权平均
recall = recall_score(y_true, y_pred, average='macro')
f1 = f1_score(y_true, y_pred, average='macro')
```

 或者可以一次性获取所有分类指标的报告。输出的是一个string，每一行为每个类别的统计指标。

```python
from sklearn.metrics import classification_report

# 如果使用`output_dict=True`将获得字典输出, 每个key为一个类别，value为这个类别的各指标dict。
report = classification_report(y_true, y_pred, output_dict=False)	
print(report)
```


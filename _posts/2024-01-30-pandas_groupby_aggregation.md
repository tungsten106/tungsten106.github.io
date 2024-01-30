---
title: Pandas tips - 分组计算
date: 2024-01-30 15:55:00 +0800
categories: [学习笔记]
tags: [pandas, python, pandas]     # TAG names should always be lowercase
img_path: /assets/img/
---

> 本文参考: [Pandas_Advanced_Exercise]: https://github.com/liuhuanshuo/Pandas_Advanced_Exercise	"第6章 数据分组与聚合"

分组后进行统计。`mean()` 可以替换成 `max()` ,  `min()` ,  `count()`  等

```python
# 计算分组后的平均值
df[["district", "salary"]].groupby("district").mean()

# 不将分组作为index
df[["district", "salary"]].groupby("district", as_index=False).mean()
```

对数量进行统计

```python
# 计算每个区出现公司的数量
df.groupby(["district"])["companySize"].count()

# 计算不同district不同companySize出现的次数
pd.DataFrame(df.groupby(["district"])["companySize"].value_counts())
```

查看 `groupby` 后的分组内容

```python
df.groupby(["district", "salary"]).groups
```

查看指定group的分组，如西湖区薪资为20000的工作：

```python
df.groupby(["district", "salary"]).get_group(("西湖区",20000))
```

## groupby+lambda 进行更复杂的计算

通过datetime计算不同发布日，不同行政区新增的岗位数量

```python
pd.DataFrame(df.groupby([
    df.createTime.apply(lambda x:x.day)
    ])["district"].value_counts())
```

统计各行政区力，industryField中字符含"电商"的总数

```python
df.groupby(["district"])["industryField"].apply(lambda x: x.str.contains("电商").sum())
```

根据变量的字符长度进行分组，并计算均值

```python
df.groupby([df.positionName.apply(lambda x: len(x))])["salary"].mean()
```

### 分组转换 transform

给原df新增一列作为行政区的平均薪资水平。为了应对不同长度，需要进行transform：

```python
df["avg_salary"] = df.groupby(["district"])["salary"].transform("mean")
```



## groupby+filter

过滤平均工资小于30000的行政区的全部数据：

```python
df.groupby(["district"]).filter(lambda x: x["salary"].mean() < 30000)
```



## groupby+plot

```python
import matplotlib.pyplot as plt
%config InlineBackend.figure_format = 'retina'
plt.rcParams['font.sans-serif'] = ['Microsoft YaHei']

df.groupby("district")["positionName"].count().plot(kind="bar")
plt.xlabel("行政区")
plt.ylabel("公司数量")
```



## groupby+agg

计算不同行政区中，salary的最小、最大和平均值

```python
df.groupby(["district"])["salary"].agg([min, max, np.mean])

# 带column命名
df.groupby(["district"]).agg(min=('salary', 'min'), max=('salary', 'max'), mean=('salary', 'mean')).rename_axis(["行政区"])

# 统计不同列: 通过dict
df.groupby("positionName").agg({'salary': np.median, 'score': np.mean})

# 对同一列做多个统计
df.groupby('district').agg({'salary': [np.mean, np.median, np.std], 'score': np.mean})
```


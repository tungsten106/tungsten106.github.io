---
title: Pandas tips - 数据文件读取
date: 2024-01-09 13:40:00 +0800
categories: [学习笔记]
tags: [pandas, python]     # TAG names should always be lowercase
img_path: /assets/img/
---

## 读取表格数据: `read_csv()`

Ref: [`pandas.read_csv()`](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html#pandas.read_csv)

### （在读取时）进行行列筛选

- 读取前xx行：`nrows`

  ```python
  pd.read_csv("data.csv", nrows=20)
  ```

- 将第xx行作为headers：`headers`

  ```python
  pd.read_csv("data.csv", headers=20)
  ```

- 跳过前xx行数据：`skiprows` （有headers的情况）

  ```python
  pd.read_csv("data.csv", skiprows=[i in range(1, 21)])
  ```

- 选择偶数/奇数/其他对index的选择，可以用 `skiprows` + `lambda` :

  ```python
  # 偶数行, x != 0为了保留headers
  pd.read_csv("data.csv", skiprows=lambda x: (x != 0) and not x % 2)
  ```

- 按照index/colname选择列（第1，3，5列）：`usecols`

  ```python
  pd.read_csv("data.csv", usecols=[1,3,5])			# 按照列index
  pd.read_csv("data.csv", usecols=['a', 'b', 'c'])	# 按照列名
  ```

- 要选择的colname不一定在df中，可以用一个lambda函数来避免报错。（想法相当于left join）

  ```python
  usecols = ['a', 'b', 'c', 'd']	# 'd'不在df的columns中
  pd.read_csv("data.csv", usecols=lambda col: col in usecols)
  ```

- 将某一列（某几列设置为index）：`index_col`

  ```python
  pd.read_csv("data.csv", index_col='a')		# 选择一行
  pd.read_csv("data.csv", index_col=['a','b'])	# 选择多行
  ```

### 缺失值处理

与 `na` 相关的几个参数：

- `na_values`：Additional strings to recognize as `NA`/`NaN`，有哪些额外应为识别成缺失值的内容

- `keep_default_na`：bool, 是否默认将默认缺失值：

  ```
  '-1.#IND', '1.#QNAN', '1.#IND', '-1.#QNAN', '#N/A N/A','#N/A', 'N/A', 'NA', '#NA', 'NULL', 'NaN', '-NaN', 'nan', '-nan', ''
  ```

  转换为 `NA`/`NaN`

- `na_filter ` **bool, default True**：是否检测缺失值

### 格式转换

- `dtype` ：例如`{'a': np.float64, 'b': np.int32, 'c': 'Int64'}` ，指定固定列为某数据类型

- `parse_dates` ：将日期转为timestamp，输入list（列表名称）

  ```python
  df1 = pd.read_csv("data.csv", 
                    parse_dates=["time"]
                    )
  type(df1["time"][0])	# pandas._libs.tslibs.timestamps.Timestamp
  ```

### 分块提取

- 在文件较大时，可以通过设置 `chunksize=10` 每次提取10个（或者更多）。这时 `read_csv()` 函数会返回一个iterable object（`pandas.io.parsers.readers.TextFileReader` ），每次通过`next()` 读取10行数据。

  ```python
  df = pd.read_csv("large_data.csv", chunksize=10)
  
  for data in df:	# 这里df为iterable object，可以用for循环查看每一个chunk
      print(data)
  ```

## 读取其他数据格式

```python
# 从JSON读取
pd.read_json("data.json")
# 从剪切板读取
pd.read_clipborad(sep="\t")
```

有一些参数是所有读取函数（ `pd.read_` 开头的函数）都共同的：

- `sep` ：调整分隔符，一般csv为 `sep=","` ，txt为 `sep="\t"`
- `encoding` ：文件编码格式，有时候中文文本需要调整编码模式为 `encoding="gbk"`

## 读取目录下多个/所有csv文件

```python
import os

df_list = []
data_dir = "./data"
for fn in os.listdir(data_dir):
    df_list.append(pd.read_csv(os.path.join(data_dir, fn)))
```


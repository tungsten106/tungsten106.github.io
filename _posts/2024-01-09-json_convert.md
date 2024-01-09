---
title: python中的json操作总结
date: 2024-01-09 10:43:00 +0800
categories: [踩坑总结]
tags: [data_structure]     # TAG names should always be lowercase
img_path: /assets/img/
---


`json.loads()` / `json.dumps()` vs `json.load()` / `json.dump()` 的区别：`s` 代表 `string` ，前两个用于字符串转换，后两个用于读取/写入.json文件

## json(dict) 与字符(str)转换

### str转json：`json.loads()` 

 `json.loads()` 函数主要用于转换字符串格式的JSON文件（或者dict）。用法例如

```python
d = '{"a": 1, "b": 2}'
json.loads(d)	# {"a": 1, "b": 2}
d1 = '[{"a": 1, "b": 2}, {"a": 1, "b": 2}]'
json.loads(d1)	# [{"a": 1, "b": 2}, {"a": 1, "b": 2}]
```

注意字典需要以双引号来引用key值，如果反着写：

```python
d2 = "{'a': 1, 'b': 2}"
json.loads(d2)
```

会报错 `JSONDecodeError: Expecting property name enclosed in double quotes: line 1 column 2 (char 1)` 

### json转str：`json.dumps()`

```python
d = {"a": 1, "b": 2}
json.dumps(d)	# '{"a": 1, "b": 2}'
d1 = {'a': 1, 'b': 2}
json.dumps(d1)	# '{"a": 1, "b": 2}'
```

注意如果有中文字符，可能需要添加 `ensure_ascii=False` 参数。（json.dumps()方法会将dict的数据先转换为string，然后将string写入到文本中，但是dumps()方法会默认将其中unicode码以ascii编码的方式输入到string。）

```python
d2 = {"a": "你好", "b": 2}
json.dumps(d2)						# '{"a": "\\u4f60\\u597d", "b": 2}'
json.dumps(d2, ensure_ascii=False)	# '{"a": "你好", "b": 2}'
```



## 读取.json文件：`json.load()`

这里主要讨论使用python的json包进行处理，有2种不同读取法。以最近处理的预训练数据集为例：

### 1. 存储为list格式

这个数据集格式如下：

```json
[{'instruction': '用一句话描述地球为什么是独一无二的。\\n',
  'input': '',
  'output': '地球上有适宜生命存在的条件和多样化的生命形式。'},
 {'instruction': '给出一段对话，要求GPT模型使用合适的语气和回答方式继续对话。',
  'input': '对话：\nA：你今天看起来很高兴，发生了什么好事？\nB：是的，我刚刚得到一份来自梅西银行的工作通知书。\nA：哇，恭喜你！你打算什么时候开始工作？\nB：下个月开始，所以我现在正为这份工作做准备。',
  'output': 'A: 这太好了！你的新工作听起来很令人兴奋。你对接下来的日子有什么期望吗？\nB: 是啊，我非常期待能在梅西银行工作。我希望我能够尽快适应新环境，并展示出我的所有技能和才能。'},
 {'instruction': '基于以下提示填写以下句子的空格。',
  'input': '提示：\n- 提供多种现实世界的场景\n- 空格应填写一个形容词或一个形容词短语\n句子:\n______出去享受户外活动，包括在公园里散步，穿过树林或在海岸边散步。',
  'output': '多种形容词可填，以下是其中一些例子：\n- 愉快的\n- 惬意的\n- 轻松的\n- 安静的\n- 美妙的'}
...
```

可以使用 `json.load()` 函数：

```python
with open("./data/xxx.json", 'r') as f:
    data = json.load(f)
```

### 2. 没有list，直接是多个json合并在一起

这个数据集格式如下：

```json
{"content": "类型#上衣*材质#牛仔布*颜色#白色*风格#简约*图案#刺绣*衣样式#外套*衣款式#破洞", "summary": "简约而不简单的牛仔外套，白色的衣身十分百搭。衣身多处有做旧破洞设计，打破单调乏味，增加一丝造型看点。衣身后背处有趣味刺绣装饰，丰富层次感，彰显别样时尚。"}
{"content": "类型#裙*材质#针织*颜色#纯色*风格#复古*风格#文艺*风格#简约*图案#格子*图案#纯色*图案#复古*裙型#背带裙*裙长#连衣裙*裙领型#半高领", "summary": "这款BRAND针织两件套连衣裙，简约的纯色半高领针织上衣，修饰着颈部线，尽显优雅气质。同时搭配叠穿起一条背带式的复古格纹裙，整体散发着一股怀旧的时髦魅力，很是文艺范。"}
{"content": "类型#上衣*风格#嘻哈*图案#卡通*图案#印花*图案#撞色*衣样式#卫衣*衣款式#连帽", "summary": "嘻哈玩转童年，随时<UNK>，没错，出街还是要靠卫衣来装酷哦！时尚个性的连帽设计，率性有范还防风保暖。还有胸前撞色的卡通印花设计，靓丽抢眼更富有趣味性，加上前幅大容量又时尚美观的袋鼠兜，简直就是孩子耍帅装酷必备的利器。"}
{"content": "类型#裤*风格#英伦*风格#简约", "summary": "裤子是简约大方的版型设计，带来一种极简主义风格而且不乏舒适优雅感，是衣橱必不可少的一件百搭单品。标志性的logo可以体现出一股子浓郁的英伦风情，轻而易举带来独一无二的<UNK>体验。"}
```

如果像上面这个方法一样直接用 `json.load()` 来处理的话会报错 `JSONDecodeError: Extra data: line 2 column 1 (char 151)` 。因此可以先逐行提取，再一一转换：

```python
file = open("./data/AdvertiseGen/dev.json", 'r', encoding='utf-8')
train_data = []
for line in file.readlines():
    dic = json.loads(line)
    train_data.append(dic)
```

先读取file的每一行成字符串，再进行转换。



## 写入.json文件: `json.dump()`

当我们要写入json文件时，可以用`json.dump()` 函数：

```python
formatted_data = [{'id': 'identity_0',
  'conversations': [{'from': 'user', 'value': '用一句话描述地球为什么是独一无二的。'},
   {'from': 'assistant', 'value': '地球上有适宜生命存在的条件和多样化的生命形式。'}]},
 {'id': 'identity_1',
  'conversations': [{'from': 'user',
    'value': '给出一段对话，要求GPT模型使用合适的语气和回答方式继续对话。对话：\nA：你今天看起来很高兴，发生了什么好事？\nB：是的，我刚刚得到一份来自梅西银行的工作通知书。\nA：哇，恭喜你！你打算什么时候开始工作？\nB：下个月开始，所以我现在正为这份工作做准备。'},
   {'from': 'assistant',
    'value': 'A: 这太好了！你的新工作听起来很令人兴奋。你对接下来的日子有什么期望吗？\nB: 是啊，我非常期待能在梅西银行工作。我希望我能够尽快适应新环境，并展示出我的所有技能和才能。'}]}]

with open("temp.json","w") as f:
     json.dump(formatted_data, f, ensure_ascii=False)
```

同样，由于有中文（unicode），我们可以使用 `ensure_ascii=False` 参数来保证输出编码正确。


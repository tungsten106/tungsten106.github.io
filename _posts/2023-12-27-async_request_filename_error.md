---
title: 使用aiohttp异步调用API+request上传文件中文文档名乱码解决方案
date: 2023-12-27 18:26:00 +0800
categories: [踩坑总结]
tags: [web, python, flask]     # TAG names should always be lowercase
img_path: /assets/img/
---

有时候在调用需要用异步调用API接口。在python中有很多框架，比如 `asyncio`， `Celery`，`Quart` 等。这里我选择了 `asyncio`。Python 3.5以上版本内置了`asyncio`库，可以用来编写单线程的并发代码。可以使用此库与`aiohttp`结合来发送异步HTTP请求。

## Python调用案例

### GET

```python
import asyncio
import aiohttp

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    # 指定要请求的URL
    url = "http://example.com"
    
    # 创建一个异步的HTTP会话
    async with aiohttp.ClientSession() as session:
        html = await fetch(session, url)  # 发送请求并获取响应
        print(html)  # 打印响应内容

# 运行异步主函数
asyncio.run(main())
```



### POST

#### 参数为JSON

```python
import asyncio
import aiohttp

async def fetch(session, url, data):
    # 使用session.post发送POST请求，data是POST请求的数据
    async with session.post(url, data=data) as response:
        return await response.text()  # 返回响应的文本内容

async def main():
    url = "http://example.com"  # 指定URL
    data = {'key': 'value'}  # 准备发送的数据

    async with aiohttp.ClientSession() as session:
        html = await fetch(session, url, data)  # 发送POST请求并获取响应
        print(html)  # 打印响应内容

# 运行异步主函数
asyncio.run(main())
```



#### 需要同时上传文件和JSON参数

```python
import asyncio
import aiohttp
from aiohttp import FormData

async def fetch(session, url, data):
    async with session.post(url, data=data) as response:
        return await response.text()

async def main():
    url = "http://example.com/upload"  # 模拟的文件上传URL

    # 准备文件字典
    files = {
        'file1': open('example1.txt', 'rb'),
        'file2': open('example2.txt', 'rb')
    }

    # 准备普通表单数据
    form_data = {
        'username': 'user1',
        'password': 'pass123'
    }

    data = FormData()
    # 添加普通表单数据
    for key, value in form_data.items():
        data.add_field(key, value)
    
    # 添加文件
    for file_name, file_obj in files.items():
        data.add_field(file_name,
                       file_obj,
                       filename=file_name,
                       content_type='text/plain'	# 这里可以不填或者根据自己上传的文件格式修改
                      )

    async with aiohttp.ClientSession() as session:
        html = await fetch(session, url, data)  # 发送文件和其他数据
        print(html)

    # 确保所有文件在发送后都已关闭
    for file in files.values():
        file.close()

# 运行异步主函数
asyncio.run(main())
```



## 中文文档名报错

在上传文档路径名文件路径包含了中文字符时，使用aiohttp传递参数会将文件名变成乱码，这通常是因为路径中的非ASCII字符被编码成了URL编码或者类似的格式。

在request端无法进行修改，我们可以在被调用的api中需要用到filename处增加这一行来进行更改：

```python
from urllib.parse import unquote

file.filename = unquote(file.filename)
```
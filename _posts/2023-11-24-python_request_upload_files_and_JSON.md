---
title: 如何用python request同时上传文件和JSON参数
date: 2023-11-24 13:43:21 +0800
categories: [踩坑总结]
tags: [web, python, flask]     # TAG names should always be lowercase
img_path: /assets/img/
---


一个http学习摸索过程中的记录，对http框架并不十分了解，如果有误欢迎指出。


假设我们目前有一些文件，和参数需要通过POST发送到请求服务端，我们可以通过content type为`multipart/form-data` 来同时传入这两个参数。

## 准备参数

我们先设置需要传入的参数，这里 `file_path` 需要改成自己的文件

```python
import requests

# 设置要上传的文件
file_path = "path/to/your/file"	# 这里替换成文件目录
files = {
    "file1": ("filename", open(file_path, "rb"))
}

# 设置要发送的JSON数据
params = {
    'key1': 'value1',
    'key2': 'value2'
}
```

## 编写service

在服务端要如何获取文件和JSON参数？我们首先要知道，通过如上方式传入数据，content-type是`multipart/form-data` 。所以我们在服务端应该使用 `request.form.to_dict()` 来获取表格里的参数内容。

我们新建一个命名为 `service.py` 的文件，写入一下脚本来启动命名为**"upload-endpoint"**的服务。我们这里服务没有做数据处理，只是把它们打印出来。

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/upload-endpoint', methods=['POST'])
def upload_endpoint():
    try:
        # 获取JSON数据
        params = request.form.to_dict()
        print(params)
        # 获取上传的文件
        file1 = request.files['file1']
        print(file1)
        
        # 处理JSON数据和文件
        # 在这里，你可以根据需要对JSON数据和文件进行操作
        # 例如，你可以保存文件到服务器，访问JSON数据等

        # 返回一个响应
        response_data = {
            'message': 'JSON参数和文件已成功接收和处理'
        }
        return jsonify(response_data), 200
    except Exception as e:
        error_message = str(e)
        return jsonify({'error': error_message}), 400

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

这里port（端口号）可以任意设定，我设定了5000。这里的端口号、服务名称需要与后面调用服务的网址对应。

## 发送请求

我们向布好的服务发送请求，我们向地址`http://127.0.0.1:5000/upload-endpoint` 发送请求。这里的网址是由端口号和上一部分的自定义命名生成的，`http://127.0.0.1` 为本地地址，可以不用更改。

我们先需要运行上一部分写好的`service.py` ，运行后放在后台。出现

```shell
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```

则正常运行。

```python
# 发送POST请求，同时传送JSON数据和文件
response = requests.post('http://127.0.0.1:5000/upload-endpoint', data=params, files=files)
```

可以看一下数据是怎么传的。JSON数据通过 `data` 传输，通过 `files` 上传文件附件。

然后对获取的内容进行解析，显示返回的内容

```python
# 检查响应
if response.status_code == 200:
    print('请求成功')
    print(response.json())
else:
    print('请求失败')
    print(response)
```

输出：

```shell
请求成功
{'message': 'JSON参数和文件已成功接收和处理'}
```

此时在运行着`service.py` 的输出会显示我们在服务端的操作，打印输入内容：

```shell
{'key1': 'value1', 'key2': 'value2'}
<FileStorage: 'test.docx' (None)>
127.0.0.1 - - [24/Nov/2023 13:09:11] "POST /upload_endpoint HTTP/1.1" 200 -
```

最后一行返回code为200，代表运行正常。
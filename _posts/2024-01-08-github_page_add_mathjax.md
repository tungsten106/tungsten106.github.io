---
title: "Github page数学公式无法正常显示解决方案(MathJax)"
date: 2024-01-08 23:02:00 +0800
categories: [踩坑总结]
tags: [javascript, markdown, latex]     # TAG names should always be lowercase
img_path: /assets/img/
---

在上传一篇文献阅读笔记到Github page时发现公式无法正常显示，之前在typora中能够正常显示的代码在网页上显示为纯latex格式于是进行了一些搜索。

- 我使用的Jekyll模板是chirpy，具体效果可能与使用的模板也有关系。


# 问题原因

这个问题的原因出在GitHub Page里的Jekyll虽然支持Markdown，但是不能正确显示公式 [[1](https://zhuanlan.zhihu.com/p/36302775)]。在检索中我发现比较通用的一种方式就是借用MathJax帮助渲染。

# 解决方法

首先以下所有方法都需要在 `_config.yml` 中设置 `markdown: kramdown`. 我使用的主题中有一段默认设置为：

```yaml
markdown: kramdown
kramdown:
  syntax_highlighter: rouge
  syntax_highlighter_opts: # Rouge Options › https://github.com/jneen/rouge#full-options
    css_class: highlight
    # default_lang: console
    span:
      line_numbers: false
    block:
      line_numbers: true
      start_line: 1
```

这里的配置可能要根据自己选择或者每个主题的固定配置，不需要一定和如上所写的一样。

## 方法1: 在文章开头加入js内容（不太ok）

搜索到的其中一种方式就是在每篇文章开头加入如下一段JS代码：

```javascript
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>
```

这个方法的问题在于设置了以后，部分字体能够正常显示，但是字体不是latex标准字体，并且有一些数学字体无法显示（如`\mathbb` , `\mathcal`）等。也能看但是无法显示一些关键notation，看着不舒服。

## 方法2: 在head文件汇总加入js内容（和方法1大同小异）

在 `_includes/head.html` 中添加如方法1同样的一段JS代码，原理一样，效果也一样。

## 方法3: 目前效果最好

这个方法我是在chirpy主题的issue 1140中找到答案的，链接在[这里](https://github.com/cotes2020/jekyll-theme-chirpy/issues/1140)；参考**[otzslayer](https://github.com/otzslayer)** 的答案，可以直接在`_layouts/default.html` 中添加如下一段：

```javascript
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
      TeX: {
        equationNumbers: {
          autoNumber: "AMS"
        }
      },
      extensions: ["tex2jax.js"],
      jax: ["input/TeX", "output/HTML-CSS"],
      tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
      processEscapes: true,
      "HTML-CSS": { fonts: ["TeX"] }
    }
  });
  MathJax.Hub.Register.MessageHook("Math Processing Error",function (message) {
        alert("Math Processing Error: "+message[1]);
      });
  MathJax.Hub.Register.MessageHook("TeX Jax - parse error",function (message) {
        alert("Math Processing Error: "+message[1]);
      });
</script>
<script
  type="text/javascript"
  async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"
></script>
```

其中 `src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"` 这一行不可以被替换。


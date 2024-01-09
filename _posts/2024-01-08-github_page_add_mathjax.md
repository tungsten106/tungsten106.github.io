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


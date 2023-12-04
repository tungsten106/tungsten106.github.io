---
title: 通过Jekyll Chirpy主题搭建Github Page记录
date: 2023-11-26 02:20:48 +0800
categories: [实践记录]
tags: [jekyll]     # TAG names should always be lowercase
img_path: /assets/img/
---

## 新建repo

我先是follow这个教程：
[keysaim教程](https://keysaim.github.io/post/blog/2017-08-15-how-to-setup-your-github-io-blog/)

它提供了如何从git repo建立自己的github.io，跟随这个教程知道新建了输出hello world的页面。

https://tungsten106.github.io/ 这个页面有了内容

但是我卡在了选择主题的部分，原博主选择了[Huxpro](https://github.com/Huxpro/huxpro.github.io) 作为主题，但我决定参考另外一个。


## 选择并clone jekyll主题

这个主题叫jekyll-theme-chirpy：[GitHub链接](https://github.com/cotes2020/jekyll-theme-chirpy)

这个主题的教程：[https://chirpy.cotes.page/posts/getting-started/](https://chirpy.cotes.page/posts/getting-started/)

在使用**Jekyll**主题之前先需要跟随 [Jekyll官网指导](https://jekyllrb.com/docs/installation/) 进行环境安装。

这里主要需要通过homebrew安装一些东西。brew install 通常会安装软件到 macOS 系统上的全局位置，而不是绑定到特定的 Python 环境或 Conda 环境。因此，不同 Anaconda 环境通常不会直接影响 brew install 安装的软件。

 

### 安装ruby报错 

!!! Failed to download ruby versions!

- 查了错误发现有可能是没有安装wget

在搜索后发现需要输入 `brew install ruby` 来进行安装

安装后出现：

![image-20231128015240869](image-20231128015240869.png)

是说要设置一些环境变量。

参考 [Mac升级ruby到最新版本](https://blog.csdn.net/a71468293a/article/details/104253813) 和 [macOS Monterey安装Jekyll](https://blog.csdn.net/wanghao_sh/article/details/128196126) 这两篇博文。

输入 `echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc` 后更新了zsh的配置文件

这样还不够，我们需要更新配置文件。

```zsh
source .zshrc
```

- 这里如果终端是bash则输入：`echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.bash_profile`，并用 `source .bash_profile` 来更新配置文件。

更新完再输入 `ruby -v` 后得到的是最新版本

```zsh
ruby 3.2.2 (2023-03-30 revision e51014f9c0) [arm64-darwin22]
```

然后输入gem install jekyll

- 安装卡顿，可能是源的问题
- 参考 [github使用Jekyll，在Rails bundle install卡住问题](https://blog.csdn.net/weixin_44512194/article/details/107053421) 进行删除/新增源

 

## 安装主题

教程中我选择option1，更佳jekyll新手友好。

~~但是在之前的教程中我已经完成过repo的建立，没办法按照教程的通过use the template来建立repo。参考教程1我打算尝试用`git clone`，尝试失败，还是老老实实重新安装了hh~~

参考[chirpy主题官方教程](https://chirpy.cotes.page/posts/getting-started/#option-1-using-the-chirpy-starter) 进行安装，或者可以根随以下内容。


在 [Github](https://github.com/cotes2020/chirpy-starter) 页面点击绿色Use this Template再选择Create a new repository，在新弹出的建立repository页面建立一个名称为USERNAME.github.io的repo。注意这里USERNAME必须和自己的用户名一样。

将这个新的repo clone到本地；然后在终端中输入：

```zsh
bundle
```

### 本地运行报错

在本地输入 `bundle exec jekyll s` 指令在http://localhost:4000/上可以查看网页部署。

有可能会报错：

```zsh
Could not find gem 'jekyll-theme-chirpy (~> 6.3, >= 6.3.1)' in locally installed gems.

Run `bundle install` to install missing gems.
```

或者          

```zsh
Could not find gem 'jekyll-theme-chirpy (~> 6.3, >= 6.3.1)' in locally installed gems.

The source contains the following gems matching 'jekyll-theme-chirpy':

 \* jekyll-theme-chirpy-6.2.3
```

第一个报错是因为gem没有安装主题，第二个是因为安装的版本不对（应该安装6.3.1，实际安装了6.2.3）运行`gem install jekyll-theme-chirpy -v 6.3.1` 安装合适的版本就好了，如果再不行的话运行`bundle install `。（参考 https://rubygems.org/gems/jekyll-theme-chirpy/versions/6.3.1 和 https://stackoverflow.com/questions/46380722/jekyll-theme-could-not-be-found ）

- 这里`6.3.1`也可以是其他版本，以报错版本为主


## 网页部署

到这一步网页其实没有部署好。需要在建立了Github Page的repo中选择Settings，然后选择左侧菜单栏的Pages页面，在Source中从 `Deploy from brance` 更改为 `Github Actions` .

请保证 `Gemfile.lock` 文件已commit到repo。如果操作系统不是Linux还需要在根目录下输入：

```zsh
bundle lock --add-platform x86_64-linux
```

- 这一步有可能遇到报错：

```zsh
Retrying fetcher due to error (2/4): Bundler::HTTPError Could not fetch specs from https://rubygems.org/ due to underlying error <Net::OpenTimeout: Failed to open TCP connection to rubygems.org:443 (execution expired) (https://rubygems.org/specs.4.8.gz)>
```

这里可以参考[stackoverflow](https://stackoverflow.com/questions/38410185/bundle-install-is-not-working) 上的一个回答，将 `Gemfile` （我这里修改成了 `Gemfile.lock` 也成功了）中的`source https://rubygems.org/`  改成 `source http://rubygems.org/`。这里我猜测可能是网络原因。

然后随便git commit一些内容进行激活，就可以在repo的Action页面中查看进程了。第一次加载网页可能会比较久，即使Action页面显示Deployed也有可能加载不出，耐心等待几分钟就好。


## Reference

一位同样适用Chirpy主题的教程: [链接](https://zjpzhao.github.io/posts/jekyll-githubpages/)
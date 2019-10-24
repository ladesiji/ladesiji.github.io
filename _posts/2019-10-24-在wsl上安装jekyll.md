---
layout: post
title:  "在wsl上安装jekyll"
categories: [tool]
---

# 在wsl上安装jekyll

最近在参观同事的github时，无意中发现github还支持个人博客。感觉是一个非常棒的功能，准备自己来搭一个尝试一下。

Github Page 基于jekyll来构建，简单的无需数据库的静态网站。将个人博客托管在github上，是完全免费的。[Jekyll中文网站](http://jekyllcn.com/)

下面开始安装Jekyll。

## 安装环境

Jekyll是在Linux 环境运行的，我的win10电脑中有ubuntu18.04。就用它了。

1. 安装前先更新一下。

    ```bash
    sudo apt-get update -y && sudo apt-get upgrade -y
    ```

2. 安装Ruby。

   ```bash
   sudo apt-get insall ruby
   sudo apt-get insall ruby-dev
   ```

3. 安装jekyll。

   ```bash
   gem install jekyll bundler
   ```
这里一定要强调一下，因为有一个大坑在这里。前面安装都很顺利，安装到这里时，提示没有`permissions for the /var/lib/gems/2.5.0 directory` 我输入sudo 后安装成功了，但jekyll 确总是跑不起来，提示有各种库没有安装。再仔细看安装说明，才发现有一行提示`(Note: no sudo here.)` 但是不用sudo 确实安装不上啊。看了好几个解决方案，也没有看懂究竟是啥原因，最后直接按人家的解决方案走吧：
```bash
echo 'export PATH="$HOME/.gem/bin:$PATH"' >> ~/.zshrc
echo 'export GEM_HOME=~/.gem' >> ~/.zshrc
echo 'export GEM_PATH=~/.gem' >> ~/.zshrc
source ~/.zshrc
```

## 主题
从[Jekyll Themes](http://jekyllthemes.org/) 找了一下简单的主题，下载下来，先用`jekyll serve` 跑一下，能成功渲染的话，就可以修改`_config.yml`文件，改成自己的名字，上传到`github-pages`的仓库中，就可以了。
这一步其实也有很多坑，有很多好看的主题，复杂的主题用了许多插件或框架，如果本地的`jekyll`配置不对，跑不起来，各种提示错误。找个简单的主题可以省一些事情，先把站点弄起来，花哨的东西以后再说。


资质愚钝，弄这个网站花了整整一天时间。目前这相主题中还有地方不会用，需要后续修正
+ 插入Python代码后 代码片段的高亮和主题比较丑陋，但不知道怎么修改。
+ 主题有文章标签分类功能，需要改为自己的标签，还不会修改。
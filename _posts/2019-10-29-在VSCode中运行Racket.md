---
layout: post
title: "Visual Studio Code 中运行Racket"
categories: [tools]
---


## 背景

在网上自学编程，总是想怎么正宗怎么来，极力向科班出身的靠拢，其中《SICP》这本书肯定是在各种推荐书单中的。以至于吹捧的人特别多，所以我也去网上找了个电子版的PDF来跟风。这本书虽然号称是计算机的入门书，但是真的非常难，勉强刷了一章后，就放下了。  
用了VScode后，看到网上有说VScode可以通过插件来支持Lisp，那当然不能错过。早就看Racket的界面不爽了，开始折腾起来。

## Racket安装

本机是WIn10，这里我使用的[DrRacket](https://racket-lang.org/) 安装后自带一个编辑器，除了背景是白色的，字体没有高亮外，还不错，支持括号提示。

## VScode插件

插件有两个，第一个是VScode-scheme，这个插件支持代码高亮，没什么好说的，安装了就能用。第二个是Code Runner，这个能将代码一键运行，主要的问题就是设置Code Runner。在VScode => settings => Extensions => Run Code Configuration 中选择 `Executor Map` 点击下面的 `Edit in settings.json` 进行编辑页面，如下图  
![示例](/image/Code-Runner-1.jpg)

在其中编辑Racket 的解释器，如下：

```json
"code-runner.executorMap": {
        "python": "set PYTHONIOENCODING=utf8 && python -u",
        "racket": "cd C:\\Program Files\\Racket && racket",
        "scheme": "cd C:\\Program Files\\Racket && racket"
    }
```

这里也有两个地方比较坑。

1. 我把Racket加入到环境变量了，直接运行 racket win10提示程序版本不对，无法运行。但在命令行中打开文件所在目录后，再执行，就没问题。
2. 使用Code Runner 后，Python 脚本运行按钮变了，之前是在Terminal中运行，很正常没问题，现在的输出位置是Output，中文老是乱码，网上给出了解决方法是在调用Python解释器的时候先声明输出是utf8。  

## 运行效果

示例的代码是SICP第二章的第一个课后题，使用`cons（）`来实现有理数的加减乘除运算。如下图：  
![Image text](/image/Code-Runner-2.jpg)
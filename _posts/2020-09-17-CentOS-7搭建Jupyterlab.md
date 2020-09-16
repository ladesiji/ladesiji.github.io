---
layout: post
title: 使用CentOS7搭建Jupyter服务
subtitle: 
categories: [tools]
---


## 购买VPS主机

最近对Linux越来越感兴趣，自己的机器上就装了两个虚拟机，一个kali，一个ubuntu。
但是笔记本实在太破了，而且开虚拟机比较慢。我这个初学者更想要一个命令行的环境。
看到腾讯的VPS十周年打折，对于新用户，只需要1.1折，95元1核2G内存50G硬盘包年，月均8元。
同学，你还在犹豫什么，赶快拿起手机订购吧。

## 安装Python

这个没有什么好说的，CentOS自带的Python为2.7版本的，yum里直接安装的也不是最新的。  

到Python官网下载一个3.85的最新版本，编译安装。

```bash
wget https://www.python.org/ftp/python/3.8.5/Python-3.8.5.tgz
tar -xvf Python-3.8.5.tgz
cd Python-3.8.5
./configure prefix=/usr/local/python3
make && make install
```

这里的命令稍微解释一下这一条命令。  
`./configure prefix=/usr/local/python3`
大致目的就是把python的安装目录指定一下，
里面的一些bin目录、lib目录就都会存放在这个目录下面。
如果不指定这个安装目录的话，最后python的安装文件将分散到linux的默认目录。

由于指定了目录，安装后还需要再创建两条软链接

```bash
ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

先升级一下pip，然后检查一下pip和python的版本

```bash
pip3 install --upgrade pip
pip3 --version
python3 --version
```

至此，Python3安装完成了。

## 安装Jupyterlab

首先要安装jupyterlab服务，然后生成配置文件

```bash
pip install jupyter jupyterlab
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3 # 由于之前是Python3不是默认安装，需要自己去创建链接
jupyter notebook --generate-config
```

再打开python3，生成密码

```python3
>>>from notebook.auth import passwd
>>>passwd('yourpasswd')
```

打开`~/.jupyter/jupyter_notebook_config.py` 配置文件，将刚才生成的密钥填写进去，修改其他参数：

```bash
c.NotebookApp.password ='' #秘钥，填写
c.NotebookApp.ip='*' # *允许任何ip访问
c.NotebookApp.open_browser = False # 默认不打开浏览器
c.NotebookApp.port =8888 # 可自行指定一个端口, 访问时使用该端口
c.NotebookApp.allow_root = True # 允许root远程登陆
c.NotebookApp.allow_remote_access = True 
```

配置完成了，可以打开服务了。

```bash
mkdir jupyter
cd jupyter
jupyter lab
```

输入vps IP地址和端口号，就可以打开服务了。
如果需要后台持续运行

```bash
nohub jupyter lab&
```

---
layout: post
date: 2017-01-11 21:55
status: public
tag: 'Python, notebook, SSH, remote, server, 远程服务, 教程'
title: '创建 IPython Notebook 远程服务'
categories: [Programming]
---

有的时候，我们需要连接到远程的服务器，使用上面的 Python 进行计算。除了使用 SSH 连接，然后使用 Python Shell 以外，我们可以用 IPython Notbook (Jupyter Notebook) 来获得更好的使用体验。


首先连接上远程服务器，然后进行如下的配置。

## 生成密钥
为了安全，需要为远程的 IPython Notebook 设置一个密码。在 Python Shell 中，可以用下面的命令生成密钥。其中在输入 `password()` 之后需要输入自己定义的密码。
```
In [1]: from IPython.lib import passwd
In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'sha1:57d70563a66b:9bc528b448032b6b73e49fc44473d992e5d4671e'
```
## 生成配置文件
直接在命令行中输入下面的命令，可以生成配置文件
```
 ipython profile create
```
会产生下面的输出
```
[ProfileCreate] Generating default config file: u'PATH\\TO\\YOUR\\CONFIG\\ipython_config.py'
[ProfileCreate] Generating default config file: u'PATH\\TO\\YOUR\\CONFIG\\ipython_kernel_config.py'
```
其中 `PATH\\TO\\YOUR\\CONFIG\\` 是基于你服务器的情况而产生的一个路径。

## 修改配置文件
修改刚才生成的`ipython_kernel_config.py`
```
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.password = u'sha1:xxxxxx'
c.NotebookApp.port = 9999
```
其中`c.NotebookApp.password = u'sha1:xxxxxx'` 要修改为在最开始生成的密码。

## 运行
```
 ipython notebook --config=PATH\\TO\\YOUR\\CONFIG\\ipython_kernel_config.py
```
然后在本地访问 `[server_ip]:9999`，输入在开始设置的密码，就可以愉快的使用 IPython notebook 了。
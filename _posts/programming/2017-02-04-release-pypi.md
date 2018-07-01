---
layout: post
date: 2017-02-04 20:31
status: public
tags: 'Python, PyPi, Release, Praise, Package, Pip'
title: '发布你自己的 PyPI 包'
categories: [Programming]
---

我们都知道，通过 pip 可以很容易的管理各种 Python 的包。那么当你有了一份好的代码时，如何能把它发布出去，让所有人都能以这样的方式下载呢？只需要通过下面几个步骤就可以完成。
## 新建项目
以项目 [praise](https://github.com/funtion/praise) 为例，目录结构应该至少是这样的
```bash
repo/
│  README.md
│  setup.cfg
│  setup.py
├─praise
│  │  __init__.py
```
其中`repo`是你项目的根目录，而其他文件的内容分别为
### README.md
项目的介绍文件，和一般的开源项目没什么太多区别
### setup.cfg
配置文件，内容为
```ini
[metadata]
description-file = README.md
```
### setup.py
项目元信息，内容为
```python
try:
    from setuptools import setup
except ImportError:
    from distutils.core import setup

setup(
    name='praise',
    packages=['praise'],
    version='0.1',
    description='Praise it',
    author='foo',
    author_email='foo@bar.com',
    url='',
    download_url='',
    keywords=['foo', 'bar'],
    classifiers=[],
)
```
其中大部分都是直接的字面意思。如果有发布在 GitHub 等地方的话，可以把 `url`写成项目地址，`download_url`写 release 里的下载地址。
### 项目代码
所有的代码都在以项目为名称的目录里面，这里用的是  `praise`。如果我们在 `__init__.py`里有这样的代码
```python
def foo():
    return "bar"
```
那么就可以这样使用
```python
import praise
result = praise.foo()
```
## 本地安装
在项目根目录，通过下面的命令（注意最后的点号）
```bash
pip install -e .
```
就可以安装你的代码并进行测试了。其中`-e`表示以编辑模式 (editable mode) 安装，可以即时更新代码的情况, 方便开发和调试。

## 发布
本地测试完成，就可以进行发布了，具体过程如下：
### 注册 PyPI 账号
分别在正式版和测试版网站上注册账号，路径为
* PyPI: [https://pypi.python.org/pypi?%3Aaction=register_form](https://pypi.python.org/pypi?%3Aaction=register_form)
* PyPI-test: [https://testpypi.python.org/pypi?%3Aaction=register_form](https://testpypi.python.org/pypi?%3Aaction=register_form)

注册完成以后会收到确认邮件。

### 本地配置
在 home 目录下（Linux 为`~`， windows 为`C:\Users\your_username\`)建立`.pypirc`文件，填入以下内容
```ini
[distutils]
index-servers =
  pypi
  pypitest

[pypi]
repository=https://pypi.python.org/pypi
username=username
password=password

[pypitest]
repository=https://testpypi.python.org/pypi
username=test_username
password=test_password
```
其中用户名和密码要换成刚才注册时设置的
### 发布到测试源
```bash
python setup.py register -r pypitest
python setup.py sdist upload -r pypitest
```
然后就可以通过
```bash
pip install -i https://testpypi.python.org/pypi <package name>
```
来进行安装，测试

### 发布到正式源
```bash
python setup.py register -r pypi
python setup.py sdist upload -r pypi
```
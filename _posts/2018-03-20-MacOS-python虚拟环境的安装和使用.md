---
layout: article
title: MacOS python3虚拟环境的安装和使用
key: 201803201
tags:
  - python
lang: zh-Hans
---
在使用macos时，因为其本身系统使用python2，所以在使用python3进行开发是会遇到各种奇奇怪怪的问题，这时候就要搬出前人造好的轮子，virtualenv 和 virtualenvwrapper，virtualenv就是python虚拟环境本身，virtualenvwrapper则像是对virtualenv的一种扩展，更方便使用和管理。

## install

### 安装和配置virtualenv and virtualenvwrapper

{% highlight shell %}
$ pip install virtualenv virtualenvwrapper
{% endhighlight %}

安装完成之后需要更新`~/.bash_profile`文件

{% highlight shell %}
$ vim ~/.bash_profile
{% endhighlight %}

添加如下内容

{% highlight shell %}
# Virtualenv/VirtualenvWrapper
source /Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh
VIRTUALENVWRAPPER_PYTHON="/Library/Frameworks/Python.framework/Versions/3.6/bin/python3"
export VIRTUALENVWRAPPER_PYTHON
{% endhighlight %}

![vim界面](http://ww1.sinaimg.cn/large/005L13Yhgy1fpj37lod4fj30fr09hmxy.jpg)

*PS：值得注意的是因为python安装目录和小版本号（如3.5，3.6）的不同，其中的路径可能需要自行修改成您实际的路径*

可以通过`which python3`查询路径

![which python3](http://ww1.sinaimg.cn/large/005L13Yhgy1fpj38c4ek0j30fu0a60t4.jpg)

## usage

### 新建虚拟环境
{% highlight shell %}
mkvirtualenv env_name -p python3 --system-site-packages
{% endhighlight %}

*PS:如非需要，此处不建议添加`--system-site-packages`，改参数后文有介绍*

![安装指令](http://ww1.sinaimg.cn/large/005L13Yhgy1fpj3w6uy2rj30fq02rdg9.jpg)

### 其他使用指令

{% highlight text %}

workon:列出虚拟环境列表

lsvirtualenv:同上

workon [envname]:切换虚拟环境

rmvirtualenv  [envname]:删除虚拟环境

deactivate: 离开虚拟环境

{% endhighlight %}

### 新建虚拟环境时virtualenv常用参数

{% highlight text %}
-p PYTHON_EXE, --python=PYTHON_EXE  # 选择python版本
--system-site-packages # 为虚拟环境添加系统本身拥有的包
{% endhighlight %}

### 测试

{% highlight shell %}
workon env_name
{% endhighlight %}

![测试](http://ww1.sinaimg.cn/large/005L13Yhgy1fpj43wuarwj30fu0a6t9a.jpg)

*PS：更多使用说明可以通过`mkvirtualenv --help`查看*

**谢谢您的阅读～**
---
layout: article
title: MacOS python3 import tensorflow1.6.0报错Illegal instruction:4
date: 2018-03-19
tags:
  - python
  - tensorflow
  - issue fixed
lang: zh-Hans
---
在我使用`pip3 install --upgrade tensorflow`命令将tensorflow升级到1.6.0后，出现了`Illegal instruction:4`的报错，在反复卸载安装，安装python虚拟环境无果后，发现是tensorflow 1.6.0版本问题，虽然不明白问题所在，但是版本回退后可以解决此问题。

{% highlight shell %}
$ pip3 uninstall tensorflow
$ pip3 install -Iv tensorflow==1.5
{% endhighlight %}

谢谢您的阅读🙏

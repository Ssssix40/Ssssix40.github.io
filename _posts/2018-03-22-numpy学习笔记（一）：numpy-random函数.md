---
layout: article
title: Numpy学习笔记（一）：numpy random函数
key: 201803221
tags:
  - python
  - numpy
  - learn note
lang: zh-Hans
---
感谢<a href="https://docs.scipy.org/doc/numpy-dev/reference/routines.random.html#random-generator" target="_blank">scipy.org</a>

在近期的tensorflow学习中，我发现，numpy作为python的数学运算库，学习tensorflow过程中经常需要用到，而numpy的random函数功能很多，每次用的时候都需要另行google，所以我决定将它的常用用法汇总一下。

## 0. first of all

{% highlight python %}
import numpy as numpy
{% endhighlight %}

既然是讲随机数，众所周知，计算机世界的随机数都是伪随机，都有一个叫做种子（seed）的东西

`numpy.random.seed(seed=None)`

可以通过输入int或arrat_like来使得随机的结果固定

{% highlight python %}
>>> np.random.rand(3, 3)

array([[0.43267997, 0.72368429, 0.72366367],
       [0.28496145, 0.44920635, 0.8924199 ],
       [0.31974178, 0.55658518, 0.01755763]])

>>> np.random.rand(3, 3)

array([[0.75196574, 0.33708946, 0.64345504],
       [0.85048542, 0.18109553, 0.69524277],
       [0.06390142, 0.30589554, 0.51643863]])

>>> np.random.seed(5)
>>> np.random.rand(3, 3)

array([[0.22199317, 0.87073231, 0.20671916],
       [0.91861091, 0.48841119, 0.61174386],
       [0.76590786, 0.51841799, 0.2968005 ]])

>>> np.random.seed(5)
>>> np.random.rand(3, 3)

rray([[0.22199317, 0.87073231, 0.20671916],
       [0.91861091, 0.48841119, 0.61174386],
       [0.76590786, 0.51841799, 0.2968005 ]])

{% endhighlight %}

## 1. numpy.random.rand()

`numpy.random.rand(d0,d1...dn)`

- rand函数根据给定维度生成半开区间[0,1)之间的数据，包含0，不包含1
- dn表示每个维度
- 返回值为指定纬度的numpy.ndarray

{% highlight python %}
>>> np.random.rand(3, 3) # shape: 3*3

array([[0.94340617, 0.96183216, 0.88510322],
       [0.44543261, 0.74930098, 0.73372814],
       [0.29233667, 0.3940114 , 0.7167332 ]])

>>> np.random.rand(3, 3, 3) # shape: 3*3*3

array([[[0.64794467, 0.17450186, 0.01016758],
        [0.36435826, 0.37682548, 0.19501414],
        [0.26438152, 0.28520726, 0.01617747]],

       [[0.43803165, 0.4096238 , 0.77309074],
        [0.42280405, 0.02623488, 0.82081416],
        [0.7611891 , 0.84823656, 0.64481959]],

       [[0.24420439, 0.62015463, 0.13258205],
        [0.87108689, 0.14997182, 0.43524276],
        [0.58190788, 0.32348629, 0.12158832]]])
{% endhighlight %}

## 2. np.random.randn()

`numpy.random.randn(d0,d1,…,dn)`

- randn函数返回一个或一组样本，具有<a href="https://baike.baidu.com/item/标准正态分布/552653?fr=aladdin" target="_blank">标准正态分布</a>。
- dn表示每个维度
- 返回值为指定维度的numpy.ndarray

{% highlight python %}
>>> np.random.randn() # 当没有输入参数时，仅返回一个值

-0.7377941002942127

>>> np.random.randn(3, 3)

array([[-0.20565666,  1.23580939, -0.27814622],
       [ 0.53923344, -2.7092927 ,  1.27514363],
       [ 0.38570597, -1.90564739, -0.10438987]])

>>> np.random.randn(3, 3, 3)

array([[[ 0.64235451, -1.64327647, -1.27366899],
        [ 0.69706885,  0.75246699,  2.16235763],
        [ 1.01141338, -0.19188666,  0.07684428]],

       [[ 1.34367043, -0.76837057,  0.27803575],
        [ 0.97007349,  0.41297538, -1.65008923],
        [-3.78282033,  0.67567421, -0.0753552 ]],

       [[-0.86540385,  0.14603592,  0.29318291],
        [-0.8167798 , -0.25492782, -0.58758   ],
        [ 0.02612474,  0.17882535, -0.95483945]]])

{% endhighlight %}

## 3. numpy.random.randint()

`numpy.random.randint(low, high=None, size=None, dtype=’l’)`

- 从区间[low,high）返回随机整形
- 参数：low为最小值，high为最大值，size为数组维度大小，dtype为数据类型，默认的数据类型是np.int
- high没有填写时，默认生成随机数的范围是[0，low)

{% highlight python %}
>>> np.random.randint(1, size = 10) # 返回[0, 1)之间的整数，所以只有0

array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0])

>>> np.random.randint(1, 5) # 返回[1, 5)之间随机的一个数字

2

>>> np.random.randint(-3, 3, size=(3, 3))

array([[-1, -2, -2],
       [-3, -1, -2],
       [ 2,  2,  2]])

{% endhighlight %}

## 4. numpy.random.random_sample()

`numpy.random.random_sample(size=None)`

- 从[0.0, 1.0)的半开区间返回浮点数

{% highlight python %}
>>> np.random.random_sample()

0.47108547995356098

>>> np.random.random_sample((5,))

array([ 0.30220482,  0.86820401,  0.1654503 ,  0.11659149,  0.54323428])

>>> 5 * np.random.random_sample((3, 2)) - 5

array([[-3.99149989, -0.52338984],
       [-2.99091858, -0.79479508],
       [-1.23204345, -1.75224494]])
{% endhighlight %}

类似功能的还有：
`numpy.random.random(size=None)`
`numpy.random.ranf(size=None)`
`numpy.random.sample(size=None)`

## 5. numpy.random.choice() ✡️

`numpy.random.choice(a, size=None, replace=True, p=None)`

- 从给定的一位数组中生成一个随机样本
- a要求输入一维数组类似数据或者是一个int；size是生成的数组纬度，要求数字或元组；replace为布尔型，决定样本是否有替换；p为样本出现概率

{% highlight python %}
>>> np.random.choice(5, 3) # 这个等同于np.random.randint(0,5,3)

array([0, 3, 4])

>>> np.random.choice(5, 3, p=[0.1, 0, 0.3, 0.6, 0])

array([3, 3, 0])

>>> np.random.choice(5, 3, replace=False)

array([3,1,0])

>>> np.random.choice(5, 3, replace=False, p=[0.1, 0, 0.3, 0.6, 0])

array([2, 3, 0])

>>> aa_milne_arr = ['pooh', 'rabbit', 'piglet', 'Christopher']
>>> np.random.choice(aa_milne_arr, 5, p=[0.5, 0.1, 0.1, 0.3])

array(['pooh', 'pooh', 'pooh', 'Christopher', 'piglet'],
      dtype='|S11')
{% endhighlight %}

**感谢您的阅读**🙏
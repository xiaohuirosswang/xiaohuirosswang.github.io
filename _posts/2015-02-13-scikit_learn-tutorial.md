---
layout: post
title: "scikit_learn 初体验"
description: ""
category: [Python, Deep learning]
---

## Why

机器学习在图像和语音识别领域已经有很多成熟的应用，比如：

- 图像识别，比如人脸识别
- 机器翻译
- 语音输入

那机器学习究竟是如何做到这些的呢？本文以图像识别中比较简单的数字识别为例来了解一下。

## What

[scikit-learn][sklearn_install_guide] 是一个用于数据挖掘和分析的 Python 库，完全开源并封装了很多机器学习的算法，我们可以很方便得对其提供的 __SVM__ 算法进行训练并将其用于实际应用场景，解决一些数据量不是很大的问题。

下面的例子将演示如何使用 __sklearn__ 库中提供的数字图片 `dataset` 来识别它们表示的实际数字。

最后画出的图形为：
	
![](/images/digitpng.png) 

## How

### 安装 scikit-learn

根据 [numpy 官方文档][numpy_install] 的说明，官方发布的 [源码和包在 SourceForge 上][sab]。那么分别从下面的地址下载安装最新 release 版本：

- [SciPy: Scientific Library for Python][sf_scipy]
- [Numerical Python][sf_numpy]

解决了依赖项问题之后，通过下面的命令安装 __sklearn__：

	pip install -U scikit-learn	


### 一个手写数字识别的例子

[scikit_learn 官方有个例子][completedemo]，使用 __sklearn__ 自带的 `dataset` 对算法进行训练并用于手写数字识别的例子。

### 准备工作

- 按照本文前半部分的步骤按照 __sklearn__。
- 安装 __matplotlib__ 的依赖库 __dateutil__ 
	
	pip install python-dateutil

- 安装 __matplotlib__ 的依赖库 __pyparsing__

	pip install pyparsing


- [安装科学计算库 matplotlib](http://matplotlib.org/downloads.html)


### Python 代码：

{% highlight python%}
# -*- coding: utf-8 -*-
"""
================================
Recognizing hand-written digits
================================

An example showing how the scikit-learn can be used to recognize images of
hand-written digits.

This example is commented in the
:ref:`tutorial section of the user manual <introduction>`.

"""

# 打印出上面的文档信息，Python 里面使用 """ 多行注释的信息被当作文档信息，可以对源代码、类、方法等进行注释从而生成文档，是自解释的机制
print(__doc__)

# 下面是 scikit——learn 官方例子的作者信息
# Author: Gael Varoquaux <gael dot varoquaux at normalesup dot org>
# License: BSD 3 clause

# 导入用于科学计算的库
import matplotlib.pyplot as plt

# 导入 sklearn 自带的手写数字 dataset，以及进行机器学习的模块
from sklearn import datasets, svm, metrics

# 加载 sklearn 自带的手写数字 dataset
digits = datasets.load_digits()

# 这里我们感兴趣的数据是不同灰度的 8x8 个小格子组成的图像
# 如果我们直接使用图像进行处理，就需要使用 pylab.imread 来加载图像数据，而且这些图像数据必须都是 8x8 的格式
# 对于这个 dataset 中的图像，dataset.target 给出了它们实际对应的数字
images_and_labels = list(zip(digits.images, digits.target))
for index, (image, label) in enumerate(images_and_labels[:4]):
    plt.subplot(2, 4, index + 1)
    plt.axis('off')
    plt.imshow(image, cmap=plt.cm.gray_r, interpolation='nearest')
    plt.title('Training: %i' % label)

# 为了使用分类器，需要将每个表示手写图像的 8x8 数字转换为一个数字数组
# 这样 digits.images 就变为了(采样，采样特性)的一个矩阵
n_samples = len(digits.images)
data = digits.images.reshape((n_samples, -1))
print(digits.images[0])
print(data[0])

# 创建一个分类器，这里 gamma 的值是给定的，可以通过 grid search 和 cross validation 等技术算出更好的值。
# 下面的链接有个例子是自己算 gamma：
# http://efavdb.com/machine-learning-with-wearable-sensors/
classifier = svm.SVC(gamma=0.001)

# 用前半部分数据训练分类器
classifier.fit(data[:n_samples / 2], digits.target[:n_samples / 2])

# 对后半部分数据使用训练好的分类器进行识别
expected = digits.target[n_samples / 2:]
predicted = classifier.predict(data[n_samples / 2:])

# 打印分类器的运行时信息以及期望值和实际识别的值
print("Classification report for classifier %s:\n%s\n"
      % (classifier, metrics.classification_report(expected, predicted)))
print("Confusion matrix:\n%s" % metrics.confusion_matrix(expected, predicted))

# 画出手写数字的图像并给出识别出的值
images_and_predictions = list(zip(digits.images[n_samples / 2:], predicted))
for index, (image, prediction) in enumerate(images_and_predictions[:4]):
    plt.subplot(2, 4, index + 5)
    plt.axis('off')
    plt.imshow(image, cmap=plt.cm.gray_r, interpolation='nearest')
    plt.title('Prediction: %i' % prediction)

plt.show()
{% endhighlight %}

下面的代码将输出注释中提到的将 8x8 数字矩阵转换为数组：

{% highlight python %}
n_samples = len(digits.images)
data = digits.images.reshape((n_samples, -1))
print("Number picture matrix:")
print(digits.images[0])
print("flatten array:")
print(data[0])
{% endhighlight %}

其输出为：

{% highlight text %}
Number picture matrix:
[[  0.   0.   5.  13.   9.   1.   0.   0.]
 [  0.   0.  13.  15.  10.  15.   5.   0.]
 [  0.   3.  15.   2.   0.  11.   8.   0.]
 [  0.   4.  12.   0.   0.   8.   8.   0.]
 [  0.   5.   8.   0.   0.   9.   8.   0.]
 [  0.   4.  11.   0.   1.  12.   7.   0.]
 [  0.   2.  14.   5.  10.  12.   0.   0.]
 [  0.   0.   6.  13.  10.   0.   0.   0.]]
flatten array:
[  0.   0.   5.  13.   9.   1.   0.   0.   0.   0.  13.  15.  10.  15.   5.
   0.   0.   3.  15.   2.   0.  11.   8.   0.   0.   4.  12.   0.   0.   8.
   8.   0.   0.   5.   8.   0.   0.   9.   8.   0.   0.   4.  11.   0.   1.
  12.   7.   0.   0.   2.  14.   5.  10.  12.   0.   0.   0.   0.   6.  13.
  10.   0.   0.   0.]
{% endhighlight %}

从结果看，识别率已经很高，如果有更多更多样化的训练数据并且自己算 `gamma` 参数的话，结果还能更好：

{% highlight text %}
             precision    recall  f1-score   support

          0       1.00      0.99      0.99        88
          1       0.99      0.97      0.98        91
          2       0.99      0.99      0.99        86
          3       0.98      0.87      0.92        91
          4       0.99      0.96      0.97        92
          5       0.95      0.97      0.96        91
          6       0.99      0.99      0.99        91
          7       0.96      0.99      0.97        89
          8       0.94      1.00      0.97        88
          9       0.93      0.98      0.95        92

avg / total       0.97      0.97      0.97       899
{% endhighlight %}

[sklearn_install_guide]: http://scikit-learn.org/stable/install.html
[numpy_install]: http://www.numpy.org/
[ployly]: https://plot.ly/python/getting-started/
[ploylysite]: https://plot.ly/
[charttype]: https://plot.ly/python/
[sab]: http://www.scipy.org/scipylib/download.html
[sf_scipy]: http://sourceforge.net/projects/scipy/files/scipy/
[sf_numpy]: http://sourceforge.net/projects/numpy/files/NumPy/
[completedemo]: http://scikit-learn.org/stable/auto_examples/plot_digits_classification.html#example-plot-digits-classification-py



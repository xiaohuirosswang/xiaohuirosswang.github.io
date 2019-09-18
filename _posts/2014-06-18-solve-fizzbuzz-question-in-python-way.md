---
layout: post
title: "FIZZBUZZ问题的Python解决方案"
description: ""
category: [Python]
---
# Solve FIZZBUZZ question in Python way


### 这几天看世界杯比赛，在德国对葡萄牙的比赛中，黄健翔他们在聊天中提到fizz buzz是德语中表示愉悦的欢呼声的意思，就是现场德国球迷的欢呼声。

### 我们知道ThoughtWorks有道非常有意思的笔试题FIZZBUZZ，输出从1到100之间的整数，遇到能被3整除的，输出FIZZ，能被5整除的，输出BUZZ，能被3和5同时整除的，输出FIZZBUZZ

### 这道题非常有意思，有经验的同学们都能做出来，但是如何用自己熟悉的语言优雅地实现就有挑战性，同时考察面试者的思路和对语言特性的熟悉度，代码风格等，在网上有很多不同语言的实现，大家兴致很高，其中用一行python代码的实现非常有意思，值得记录一下，在保证可读性的前提下使用python获取字符串子串的特殊语法实现：

{% highlight python %}
for num in range(1,101): print "FIZZ"[num % 3 *4 :] + "BUZZ"[num % 5 * 4 :] or num
{% endhighlight %}

### 语法很简单，主要是使用切片方式获取字符串子串的时候如果子串范围不在字符串长度范围时返回空，而空在布尔判断时会被转换为False的特性。




---
layout:     post
title:      "Python encode & decode总结"
subtitle:   "UTF-8 Unicode Ascii 等编码关系"
date:       2017-03-01
author:     "Mrtriste"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Python
    - 编码
---

## 声明

以上仅代表个人观点，欢迎探讨交流。

****

## 编码
以前接触过一些涉及编码的问题，但是总是以解决眼前的问题为主，如果遇到乱码的问题，一般解决的办法也都是改成UTF-8编码，很多时候这样就能解决问题。

我们最熟悉的应该是ASCII码，占一个字节，一共有256个字符，常用的是前128个。

Unicode编码的解释，如果在网上搜，有很多专业的解释，通常占两个字节，但如果要易于理解的话可以把它当成最基本的类，其他编码要互相转换可以借助Unicode编码当过渡。

UTF-8编码相对于Unicode的区别是，Unicode是定长的，而UTF-8是变长的，之所以这么做是由于有的字符只需要一位存储，但Unicode用两位就很浪费，所以有了UTF-8

那这样就产生一个问题，存储时都是放在一起的，如何判断哪几位是一个字符呢，简单来说就是通过一个UTF-8字符的前n位标1，第n+1位标0表示n位字节的字符。

本文重点在理清关系，原理性的东西可以另行搜索。
<br><br>



## Python的编码问题

### Python源代码中的编码问题
有时候运行程序时，会报```SyntaxError:None-SCCII character '\x..' in file xx.py```的错<br>
原因是源文件中含有中文等非ASCII字符的字符，而源文件默认解析的格式是ASCII，所以就会报错,解决办法是在文件第一/二行加上``` #-*- coding:utf-8 -*- ```


### Pythondecode、encode的编码问题
用Python写涉及编码的脚本时，可能经常会用到encode()和decode()函数，然后就会经常出现以下两种错误:

```py
UnicodeDecodeError: 'ascii' codec can't decode byte
UnicodeEncodeError: 'ascii' codec can't encode characters
```

我们先不管上面两个错误，我们先把刚才说的三个编码代表理一下，在Python中，所有的字符串都继承自basestring,basestring又分成两类，str和unicode

很显然刚才讲的Unicode是属于uniocde的，那么str是什么呢？

str就是除Unicode编码的其他编码方式的字符串，比如ascii,utf-8,gb2312等。

```py
#-*- coding:utf-8 -*-

def test(s1):
	s2 = s1.encode('utf-8')
	s3 = s1.encode('gbk')
	s4 = s3.encode('utf-8')
	s5 = s2.decode('gbk')
	s6 = s1.decode('ascii')
	s7 = s1.encode('ascii')
	print s2
	print s3
	print s4
	print s5
	print s6
	print s7

s = 'test'
test(s)
```

上面这段代码全部能打印出'test',所以针对ascii中的字符无论你怎么玩，一般都不会错,会出错的是含中文的字符串

```py
#-*- coding:utf-8 -*-

s = '中文'
s1 = s.decode('utf-8')

s2 = s.encode('utf-8')
s3 = s.decode('utf-8').encode('utf-8')
s4 = s.encode('gbk')
s5 = s.decode('utf-8').encode('gbk')

s6 = s.decode('utf-8').encode('ascii')
```

可以猜一下上面以UTF-8保存的源文件，哪些语句会报错？
答案是s1,s3,s5对，s2,s4,s6报错

我们首先要明白s现在的编码是UTF-8，因为``` -*- coding:utf-8 -*-```,就表示文件是以utf-8编码方式解析的，所以s是utf-8编码格式

1. 先看s1,既然s1是utf-8格式的，那么对s以utf-8格式解析，自然没错

2. 然后看s2,对utf-8进行utf-8编码，为啥会报错呢？

3. 先别急，看s3,为啥先解码再编码就对了呢？原因是在非ascii的字符串encode()时,会自动以ascii格式先解码成Unicode，再encode(),又由于ascii中没有中文，所以会报```UnicodeDecodeError: 'ascii' codec can't decode byte```的错

4. 然后，s4,s5与s2,s3类似，目的是将UTF-8编码的s转化成gbk的字符串

5. 最后，看s6,先将s以utf-8编码解码，再encode(),按刚才的流程讲应该没错啊，为啥还会报错呢？错误信息是```UnicodeEncodeError: 'ascii' codec can't encode characters ```,原因是ASCII字符集中没有中文，所以将中文encode()成ASCII时，ASCII接收不了，所以错误信息是ascii can't encode...
<br><br>



## 总结
有这么几个点可以总结（都是针对中文字符，因为英文字符玩不坏）

1. 所有的字符串都属于str类或unicode类，unicode只包含Unicode编码的字符串，str包含其他所有编码的字符串，而且这两个类都继承自basestring

2. encode()都是针对Unicode编码的字符串，如果对一个str(即非Unicode编码的字符串)直接encode()时，系统会对这个字符串自动调用decode('ascii'),尝试将其转化为Unicode，如果不能以ascii格式转化成Unicode，则会报```UnicodeDecodeError: 'ascii' codec can't decode byte```的错<br>

3. unicode是str改变编码格式的过渡类，可以把它看成一个垫脚石，如果对str执行encode(),不管是不是系统自动先decode(),还是显式执行，都要先转化为Unicode编码，再进行encode()

4. unicode可以看成一个最基本的编码，它可以转换为任何编码(只要不是目标编码中没有与源字符串对应的字符就好，比如ascii中就没有中文的字符，中文encode成ascii会报ascii can't encode...的错)

5. 不要对str直接使用encode(比如先decode再encode就不算),不要对unicode使用decode(因为Unicode已经是最基本的编码)
<br><br>


#### 最后引用一段别人在涉及编码时的处理流程

>把Python看做一个水池，一个入口，一个出口<br>
> 1. 入口处:全部decode转成unicode<br>
> 2. 池里:全部使用unicode处理(内部编码，统一unicode处理)<br>
> 3. 出口处，再encode转成目标编码，写到目标输出(文件或控制台)(当然，有例外，处理逻辑中要用到具体编码的情况)



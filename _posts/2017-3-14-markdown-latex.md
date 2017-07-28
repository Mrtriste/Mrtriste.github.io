---
layout:     post
title:      "markdown中使用LaTex"
subtitle:   ""
date:       2017-03-14
author:     "MrTriste"
header-img: "img/post-bg-js-module.jpg"
tags:
    - markdown
    - LaTex
---

## 关于如何在markdown里使用LaTeX格式的公式

首先推荐一个markdown的编辑神奇Typora.

1. 在每篇文章都加载的文件里添加脚本

   ```javascript
       <script type="text/x-mathjax-config">
           MathJax.Hub.Config({
             jax: ["input/TeX", "output/HTML-CSS"],
             tex2jax: {
               inlineMath: [ ['$', '$']],
               displayMath: [ ['$$', '$$']],
               processEscapes: true,
               skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
             }
             //,
             //displayAlign: "left",
             //displayIndent: "2em"
           });
       </script>
   	<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML" 				type="text/javascript">
       </script>
   ```

2. 以```$...$```包围的公式就会以inline的形式显示，以```$$...$$```包围的就会以block的形式显示


3. ```javascript
   	inlineMath: [ ['$', '$'], ["\(", "\)"] ],
   	displayMath: [ ['$$', '$$'], ["\[", "\]"] ]
   ```

   如果inlineMath和displayMath换成以上两种，```\(...\)```包围的也会以inline显示，```\[...\]```包围的以block形式显示。但在实际用的时候要用```\\(``` 和 ```\\)```以及```\\[```和```\\]```（因为有转移字符）。



#### PS

如果你发现用了```$$```发现没有换行居中显示，可能是因为$$上面一行没有空行，也就是需要

```
blablabla

$${x^2}$$

blablabla
```


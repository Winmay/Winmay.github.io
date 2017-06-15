---
layout: post
category: 前端
---

# 各浏览器下 scrollTop的差异 

**IE6/7/8：**

1. 对于没有doctype声明的页面里可以使用  document.body.scrollTop 来获取 scrollTop高度 ； 
2. 对于有doctype声明的页面则可以使用 document.documentElement.scrollTop； 

**Safari：**

safari 比较特别，有自己获取scrollTop的函数 ： window.pageYOffset ；

**Firefox：**

火狐等等相对标准些的浏览器就省心多了，直接用 document.documentElement.scrollTop ； 

# 获取scrollTop值 

完美的获取scrollTop 赋值短语 ： 

```
var scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
```

通过这句赋值就能在任何情况下获得scrollTop 值。 

仔细观察这句赋值，你会发现window.pageYOffset  (Safari)   被放置在 \|\| 的中间位置。

因为当 数字0 与 undefine 进行 或运算时，系统默认返回最后一个值。即或运算中 0 == undefine ; 

当页面滚动条刚好在最顶端，即scrollTop值为 0 时。  IE 下 window.pageYOffset  (Safari) 返回为 undefine ，此时将window.pageYOffset  (Safari) 放在或运算最后面时， scrollTop 返回 undefine ,  undefine 用在接下去的运算就会报错咯。 

而其他浏览器 无论 scrollTop 赋值或运算顺序如何都不会返回 undefine.  可以安全使用.. 

所以说到头还是IE的问题咯. 

# DTD相关说明：

1. 页面具有 DTD，或者说指定了 DOCTYPE 时，使用 document.documentElement。
2. 页面不具有 DTD，或者说没有指定了 DOCTYPE，时，使用 document.body。

在 IE 和 Firefox 中均是如此。

为了兼容，不管有没有 DTD，可以使用如下代码：

```
var scrollTop = window.pageYOffset  //用于FF
                || document.documentElement.scrollTop  
                || document.body.scrollTop  
                || 0;

```

# documentElement 和 body 相关说明：

1. body是DOM对象里的body子节点，即 body 标签；
2. documentElement 是整个节点树的根节点root，即 html 标签；
3. DOM把层次中的每一个对象都称之为节点，就是一个层次结构，你可以理解为一个树形结构，就像我们的目录一样，一个根目录，根目录下有子目录，子目录下还有子目录。
4. 以HTML超文本标记语言为例：整个文档的一个根就是,在DOM中可以使用document.documentElement来访问它，它就是整个节点树的根节点。而body是子节点，要访问到body标签，在脚本中应该写：document.body。

来源：
<http://www.cnblogs.com/xwgli/p/3490466.html>



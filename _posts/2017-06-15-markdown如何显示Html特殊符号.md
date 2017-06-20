---
layout: post
category: markdown
---

# markdown如何显示Html特殊符号

例如我们在编写markdown的时候，常常希望使用一些html标签来说明问题，例如&lt;div&gt;标签，可是如果直接在markdown中输入**&lt;&gt;**这两个符号的话，这两个符号，包括里面的内容都不会显示在页面中，就像丢失了一样。那么，我们到底需要使用什么方法才能把这两个符号在markdown中显示出来呢？

***答案是：使用字符实体！！！***

常用的字符实体如下：

显示结果 | 描述 | 实体名称 | 实体编号
------- | ---- | ------ | ------
&nbsp; | 空格 | &amp;nbsp; | &amp;#160;
&lt; | 小于号 | &amp;lt; | &amp;#60;
&gt; | 大于号 | &amp;gt; | &amp;#62;
&amp; | 和号 | &amp;amp; | &amp;#38;
&quot; | 引号 | &amp;quot; | &amp;#34;
&apos; | 撇号 | &amp;apos; | &amp;#39;

其他一些常用的字符实体：

显示结果 | 描述 | 实体名称 | 实体编号
------- | ---- | ------ | ------
&cent; | 分 | &amp;cent; | &amp;#162;
&pound; | 镑 | &amp;pound; | &amp;#163;
&yen; | 日圆 | &amp;yen; | &amp;#165;
&sect; | 节 | &amp;sect; | &amp;#167;
&copy; | 版权 | &amp;copy; | &amp;#169;
&reg; | 注册商标 | &amp;reg; | &amp;#174;
&times; | 乘号 | &amp;times; | &amp;#215;
&divide; | 除号 | &amp;divide; | &amp;#247;

***还有另外一个答案：使用转义字符&nbsp;&nbsp;&nbsp;&nbsp;“&nbsp;\\&nbsp;”&nbsp;&nbsp;&nbsp;&nbsp;！！！***

如：写\<div\>标签可以有三种方法

```
	//方法一： \<div\>
	//方法二： &lt;div&gt;
	//方法三： &#60;div&#62;
```




---
layout: post
category: 前端
---

# display

## display属性值

display 属性规定元素应该生成的框的类型。

属性值：

block：

/*表现为一个块级元素（一般情况下独占一行）*/

当display被设置为block(块)时，容器中所有的元素将会被当作一个单独的块，就像DIV元素一样，它会在那个点被放入到页面中。(实际上你可以设置span的display:block，使其可以像DIV一样工作。)

inline:

/*表现为一个行级元素（一般情况下不独占一行）*/

将display设置为inline，将使其行为和元素inline一样---即使它是普通的块元素如DIV，它也将会被组合成像span那样的输出流。

none:

/*元素不可见，并且不为其保留相应的位置*/

最后是display被设置：none,这时元素实际上就从页面中被移走，它下面所在的元素就会被自动跟上填充。

## display的使用

1、display默认属性值为块级的元素：

```
adress,quote,body,xmp,center,col,colgroup,dd,dtr,div,
dl,dt,fieldset,form,hn,hr,iframe,legend,listing,marquee,
menu,ol,p,plaintext,pre,table,td,th,tr,ul
```

2、display默认属性值为none的元素：

```
br,frame,nextid,tbody,tfoot,thead
```

3、li元素的display属性默认值为：list-item

4、其他元素display属性默认值都为inline

## display的特性

改变元素的display属性将对周围元素造成的影响有：

1. 在属性值设为block的元素后面添加新行
2. 从属性值设为inline的元素所在行中删除一行
3. 隐藏属性值设为none的元素并且释放该元素在文档中所占的物理空间，对于其他元素来说，相当于该元素不存在，因此，该元素的位置被其他元素顶替

# visibility

## visibility的属性值

visibility:visible

/*元素可见，默认值*/

visibility:hidden

/*元素不可见，但仍然为其保留相应的空间*/

visibility:collapse

/*只对table对象起作用，能移除行或列但不会影响表格的布局。如果这个值用在table以外的对象上则表现为hidden。*/

visibility:inherit

/*继承上级元素的visibility值。*/

## visibility特性

用来确定元素是显示还是隐藏，这用visibility="visible|hidden"来表示，visible表示显示，hidden表示隐藏。当visibility被设置为"hidden"的时候，元素虽然被隐藏了，但它仍然占据它原来所在的位置。

# visibility和display的区别

大多数人很容易将CSS属性display和visibility混淆，它们看似没有什么不同，其实它们的差别却是很大的。visibility和display两个属性都有隐藏元素的功能。visibility属性所控制的元素虽然不在浏览器里面显示出来，但他在浏览区里是存在的，只是不显示而已。而display属性设置为none，这个元素就变成了一个不显示的元素

display:none;

使用该属性后，HTML元素（对象）的宽度、高度等各种属性值都将“丢失”;

visibility:hidden;

使用该属性后，HTML元素（对象）仅仅是在视觉上看不见（完全透明），而它所占据的空间位置仍然存在，也即是说它仍具有高度、宽度等属性值。

# visibility和display的其他区别

其实visibility和display的区别主要有三点：

1. 空间占据
2. 回流与渲染
3. 株连性

回流与渲染：

display:none隐藏产生reflow和repaint(回流与重绘)，而visibility:hidden没有这个影响前端性能的问题；

株连性：

所谓“株连性”，就是如果祖先元素遭遇某祸害，则其子子孙孙无一例外也要遭殃。

display:none就是“株连性”明显的声明：一旦父节点元素应用了display:none，父节点及其子孙节点元素全部不可见，而且无论其子孙元素如何不屈地挣扎都无济于事。

在实际的web应用中，我们要经常实现一些显示隐藏的功能，由于display:none本身特性以及jQuery潜在的驱动，使得我们对display:none这种隐藏特性相当熟知。因此，久而久之会形成比较牢固的情感化认识，并无法避免地将这种认识迁移到其他类似表现属性(eg. visibility)的认识上，再加上一些常规经验……

举例来说吧，通常情况下，我们给一个父元素应用visibility:hidden，则其子孙后代也都会全部不可见。于是，我们就会有类似的认识迁移：应用了visibility:hidden声明下的子孙元素如何不屈地挣扎都摆脱不了不可见被抹杀的命运。而实际上却存在隐藏“失效”的情况。
何时隐藏“失效”？很简单，如果子孙元素应用了visibility:visible，那么这个子孙元素又会刘谦般地显现出来。

对比总结：

display:none是个相当惨无人道的声明，子孙后代全部搞死（株连性），而且连块安葬的地方都不留（不留空间），导致全体民众哗然（渲染与回流）。
 visibility:hidden则具有人道主义关怀，虽然不得已搞死子孙，但是子孙可以通过一定手段避免（伪株连性），而且死后全尸，墓地俱全（占据空间），国内民众比较淡然（无渲染与回流）。

# 其他隐藏元素

在CSS中，让元素隐藏（指屏幕范围内肉眼不可见）的方法很多，有的占据空间，有的不占据空间；有的可以响应点击，有的不能响应点击

```
{ display: none; /* 不占据空间，无法点击 */ } 
{ visibility: hidden; /* 占据空间，无法点击 */ } 
{ position: absolute; top: -999em; /* 不占据空间，无法点击 */ } 
{ position: relative; top: -999em; /* 占据空间，无法点击 */ } 
{ position: absolute; visibility: hidden; /* 不占据空间，无法点击 */ } 
{ height: 0; overflow: hidden; /* 不占据空间，无法点击 */ } 
{ opacity: 0; filter:Alpha(opacity=0); /* 占据空间，可以点击 */ } 
{ position: absolute; opacity: 0; filter:Alpha(opacity=0); /* 不占据空间，可以点击 */ } 
{ 
zoom: 0.001; 
-moz-transform: scale(0); 
-webkit-transform: scale(0); 
-o-transform: scale(0); 
transform: scale(0); 
/* IE6/IE7/IE9不占据空间，IE8/FireFox/Chrome/Opera占据空间。都无法点击 */ 
} 
{ 
position: absolute; 
zoom: 0.001; 
-moz-transform: scale(0); 
-webkit-transform: scale(0); 
-o-transform: scale(0); 
transform: scale(0); 
/* 不占据空间，无法点击 */ 
} 
```

## height:0和overflow:hidden的组合

overflow:hidden用中文理解就是“溢出隐藏”，也就是盒子以外的内容都咔嚓掉不可见的。加上height:0，只要是一般的非inline水平元素，则元素内部所有子孙都应该是不可见的。

height:0和overflow:hidden组合隐藏“失效”的条件如下：祖先元素没有position:relative/absolute/fixed声明，同时内部子元素应用了position:absolute/fixed声明。


# 什么时候使用Visibility或者Display属性？

Visibility和Display属性虽然都可以达到隐藏页面元素的目的，但它们的区别在于如何回应正常文档流。

如果你想隐藏某元素，但在页面上保留该元素的空间的话，你应该使用visibility：hidden。如果你想在隐藏某元素的同时让其它内容填充空白的话应该使用display：none。

在现实中我（作者）更多的倾向于使用display属性（相信这也是大多数人的习惯，bolo注）。当你决定用display：none来隐藏一个元素时，你必须知道其它内容将填充到该元素留下的空白位置，从而改变页面的布局。

# 使用Visibility或者Display属性的注意事项

display:none:

1、JS读取元素属性值

如果在样式文件或页面文件代码中直接用display:none对元素进行了隐藏，载入页面后，在没有通过js设置样式使元素显示的前提下，使用js代码会无法正确获得该元素的一些属性，比如offSetTop,offSetLeft等，返回的值会为0，通过js设置style.display来使元素显示后才能正确获得这些值。

2、SEO优化时需要注意

使用display:none隐藏的元素不会被百度等搜索网站检索，会影响到网站的SEO，某些情况下可以使用left:-100000px来达到同样效果。

3、样式文件

如果是通过样式文件或<style>css</style>方式来设置元素的display:none样式，用js设置style.display=""并不能使元素显示，可以使用block或inline等值来代替。通过style="display:none"直接在元素上进行的设置不会有这个问题

4、有些情况下可以使用style.visibility来代替style.display，但是要注意的是style.visibility隐藏元素时会保留元素在页面上所占的空间，而style.display隐藏元素且让出所占页面空间。

visibility:hidden:

如果想让某一段代码在前台不显示，最简单的方法是用css的display:none,这样，下边的内容就自动填补这个空隙。但是在一些特殊的情况下，我们只需要隐藏这个元素，但它的位置不能被其他元素占用了，那么，visibility:hidden就可以实现这个要求。

# 例子

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8" />
	<title>display属性值的使用</title>
	<style type="text/css">
	.test{
		height:100px;
		background-color:#ccc;
	}
	.displayWrap{
		display: none;
		height:100px;
		background-color:#E60C0F;
	}
	.displayChild{
		display: block;
		height:100px;
		width: 50%;
		background-color:#000;
	}
	.visibilityWrap{
		visibility: hidden;
		height:100px;
		background-color:#f5f00e;
	}
	.visibilityChild{
		visibility: visible;
		height:100px;
		width: 50%;
		background-color:#000;
	}
	</style>
</head>
<body>
	<!--株连性-->
	<div class='displayWrap'>
		<div class="displayChild">displayChild</div>
	</div>
	<div class='test'></div>
	<div class='visibilityWrap'>
		<div class="visibilityChild">visibilityChild</div>
	</div>
	<div class='test'></div>
	<!--空间占据-->
	<div class='displayWrap'></div>
	<div class='test'></div>
	<div class='visibilityWrap'></div>
	<div class='test'></div>
</body>
</html>
```




---
layout: post
category: 前端
---

# 定义

vertical-align 属性设置元素的垂直对齐方式。

# 值

1. 长度(length)：通过距离升高（正值）或降低（负值）元素。’0cm’等同于’baseline’。
2. 百分比(%)：通过距离（相对于1line-height1值的百分大小）升高（正值）或降低（负值）元素。’0%’等同于’baseline’。
3. baseline(默认)：元素的基线与父元素的基线对齐。
4. sub：降低元素的基线到父元素合适的下标位置。5. super：升高元素的基线到父元素合适的上标位置。6. top：把对齐的子元素的顶端与line box顶端对齐。7. text-top：把元素的顶端与父元素内容区域的顶端对齐。8. middle：元素的中垂点与 父元素的基线加1/2父元素中字母x的高度 对齐。9. bottom：把对齐的子元素的底端与line box底端对齐。10. text-bottom：把元素的底端与父元素内容区域的底端对齐。11. inherit：采用父元素相关属性的相同的指定值。

# 使用条件

只有一个元素属于inline或是inline-block（table-cell也可以理解为inline-block水平），vertical-align才会起作用。

# 作用对象

从各种取值的描述来看，它对于元素本身的表现并无影响，而是影响的在垂直方向的位置，而这个位置是其自身相对于上下文的元素的位置。

# 例子

使用一个内联元素span标签并设置属性font-size为30px，并使用一个input标签作为对比。

## top

![](/assets/image/14692444401542.jpg)

![](/assets/image/14692444561260.jpg)

![](/assets/image/14692444654819.jpg)

## bottom

![](/assets/image/14692445432783.jpg)

![](/assets/image/14692445471116.jpg)

![](/assets/image/14692445529231.jpg)

## text-top

![](/assets/image/14692445696948.jpg)

![](/assets/image/14692445731985.jpg)

![](/assets/image/14692445769043.jpg)

## text-bottom

![](/assets/image/14692445942868.jpg)

![](/assets/image/14692445979411.jpg)

![](/assets/image/14692446024882.jpg)

## middle

![](/assets/image/14692446173704.jpg)

![](/assets/image/14692446235191.jpg)

![](/assets/image/14692446290051.jpg)

## baseline

![](/assets/image/14692446436262.jpg)

![](/assets/image/14692446474110.jpg)

![](/assets/image/14692446506335.jpg)

## super

![](/assets/image/14692447253346.jpg)

![](/assets/image/14692447291297.jpg)

![](/assets/image/14692447337861.jpg)

## sub

![](/assets/image/14692447497105.jpg)

![](/assets/image/14692447531529.jpg)

![](/assets/image/14692447559375.jpg)

sub与super的说明：

有时候，设置super的时候，文字看起来比sub的位置还低，这是因为字体和行高的影响。

而这两个值实际上是html的两个用来负责设定上、下标的标签。

故上面两个例子，为了突出他们的效果，给父元素定了无单位数值的line-height，将font-size设置为16px。





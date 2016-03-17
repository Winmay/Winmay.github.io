# Underscore解析文档

## _.each(list, iterator, [context])（_.forEach）
遍历list中的所有元素，按顺序用遍历输出每个元素。如果传递了context参数，则把iteratee绑定到context对象上。每次调用iteratee都会传递三个参数：(element, index, list)。如果list是个JavaScript对象，iteratee的参数是 (value, key, list))。返回list以方便链式调用。（愚人码头注：如果存在原生的forEach方法，Underscore就使用它代替。）

![](/assets/image/14581993883849.jpg)

1. 首先执行“optimizeCb()”函数，得到迭代器iteratee的值。语句：

```
iteratee = optimizeCb(iteratee, context);
```
即如下图函数所示，argCount没有传入参数，即选择case 3.iteratee得到的值为：

```iteratee = function(value, index, collection) {	return func.call(context, value, index, collection);};
```
![](/assets/image/14581996360093.jpg)

2. 判断传入的对象obj是否为数组或对象,语句：

```
if (isArrayLike(obj))
```
即如下图函数所示，使用代码：

```length= property('length')(collection)
```

判断length是否有值。

![](/assets/image/14581996931981.jpg)

例：

```
var a = [1,2,3];
var b = {
	one:'one1',
	two:'two2',
	three:'three3'
};
console.log(a["length"]); //3
console.log(b["length"]); //undefined
```

1） 若obj为数组，则使用for循环，遍历从obj数组的第0为开始到，到obj的最后一位结束，并求出三个数值：obj[i]（obj数组对应的值）、i（obj数组对应的索引位置）、obj（obj数组列表本身），传进迭代器iteratee中，并运行iteratee函数。

![](/assets/image/14581999399598.jpg)

例：

```
var a = [1,2,3];
var text1 = _.each(a,function (element,index,list){
	console.log(element+','+index);
	console.log(list);
});
/*
1,0
[1,2,3]
2,1
[1,2,3]
3,2
[1,2,3]
*/
```
2） 若obj为对象a. 首先，检索object拥有的所有可枚举属性的名称，并返回数组keys。
		
```var keys = _.keys(obj);
```
		b. 使用for循环，遍历keys并返回它们的值（obj的属性），并求出三个值：obj[keys[i]]（obj对象属性对应的值）、keys[i]（obj对象对应的属性名称）、obj（obj对象列表本身），传进迭代器iteratee中，并运行iteratee函数。

![](/assets/image/14582001673368.jpg)

例：

```
var b = {
	one:'one1',
	two:'two2',
	three:'three3'
};
var text2 = _.each(b,function (element,index,list){
	console.log(element+','+index);
	console.log(list);
});
/*
one1,one
Object {one: "one1", two: "two2", three: "three3"}
two2,two
Object {one: "one1", two: "two2", three: "three3"}
three3,three
Object {one: "one1", two: "two2", three: "three3"}
*/
console.log(text2);//Object {one: "one1", two: "two2", three: "three3"}
```

3. 最后返回obj本身return obj;

## _.map(list, iterator, [context])（_. collect）
通过转换函数(iterator迭代器)映射列表中的每个值产生价值的新数组。iterator传递三个参数：value，然后是迭代 index(或 key 愚人码头注：如果list是个JavaScript对象是，这个参数就是key)，最后一个是引用指向整个list。

![](/assets/image/14582002899913.jpg)

1. 首先执行“cb()”函数，得到迭代器iteratee的值。语句：

```
iteratee = cb(iteratee, context);
```
即如下图函数所示：

![](/assets/image/14582003248371.jpg)

1） 若value为空值
返回与传入参数value相等的值 _.identity(value) 相当于数学里的: f(x) = x。

![](/assets/image/14582003520020.jpg)

2） 若value为函数
运行函数“optimizeCb()”函数，得到迭代器iteratee的值。argCount没有传入参数，即选择case 3.iteratee得到的值为：

```iteratee = function(value, index, collection) {	return func.call(context, value, index, collection);};
```

3） 若value为对象（数组或对象）
则返回一个函数（断言函数），这个函数会给你一个断言可以用来辨别给定的对象是否匹配value（attrs）指定键/值属性.

![](/assets/image/14582004212316.jpg)

4） 若value为其他值（数字，字符串，undefined，布尔值）,则返回一个函数，这个函数返回任何传入的对象的value属性值

```
obj(value):_.property= property (value);
```

![](/assets/image/14582004601542.jpg)

2. 判断obj是对象还是数组，若是对象，检索object拥有的所有可枚举属性的名称，并返回数组keys。3. 使用for循环，遍历keys（index）并返回它们的值（obj的属性或数组值），并求出三个值：obj[keys[i]／i]（obj对象属性（数组索引）对应的值）、keys[i]／i（obj对象对应的属性名称（数组对应的索引））、obj（obj对象（数组）列表本身），传进迭代器iteratee中，并运行iteratee函数后，返回新的数组列表值。

![](/assets/image/14582004907225.jpg)

例：

```
var a = [1,2,3];
var test1 = _.map(a,function (element,index,list){
	console.log(element+','+index);
	console.log(list);
	return element*3;
});
/*
1,0
[1,2,3]
2,1
[1,2,3]
3,2
[1,2,3]
*/
console.log(test1);//[3, 6, 9]
```

```
var b = {
	one:'one1',
	two:'two2',
	three:'three3'
};
var test2 = _.map(b,function (element,key,list){
	console.log(element+','+key);
	console.log(list);
	return element+':'+key;
});
/*
one1,one
Object {one: "one1", two: "two2", three: "three3"}
two2,two
Object {one: "one1", two: "two2", three: "three3"}
three3,three
Object {one: "one1", two: "two2", three: "three3"}
*/
console.log(test2);
//["one1:one", "two2:two", "three3:three"]
```

## _.reduce(list, iterator, [memo], [context])（_.foldl/_.inject）和_.reduceRight(list, iteratee, memo, [context])（_.foldr）Memo（备忘录）是reduce函数的初始值，reduce的每一步都需要由iterator返回。这个迭代传递4个参数：memo,value 和 迭代的index（或者 key）和最后一个引用的整个 list，最后一个是引用指向整个list。如果reduce的初始调用没有传递memo，iterator将会调用列表中的第一个元素。第一个元素通过memo参数在iterator中调用并传递到列表的下一个元素中。
reducRight是从右侧开始组合的元素的reduce函数，如果存在JavaScript 1.8版本的reduceRight，则用其代替。Foldr在javascript中不像其它有懒计算的语言那么有用（愚人码头注：lazy evaluation：一种求值策略，只有当表达式的值真正需要时才对表达式进行计算）。

![](/assets/image/14582006294352.jpg)

![](/assets/image/14582006343291.jpg)

![](/assets/image/14582006382114.jpg)

1. 判断是否有传memo的值，并得到initial的布尔值。
2. 运行optimizeCb（）函数，且argCount传入的参数值为4，故：
iteratee得到的值为：

```
iteratee = function(accumulator,value, index, collection) {	return func.call(context, accumulator ,value, index, collection);};```

3. 运行reducer（）函数，并得到最新的memo值，并返回
例：
```
var a = [1,2,3,4];
var test1 = _.reduce(a,function (memo,num,index,list){
	console.log(memo+','+num+','+index);
	console.log(list);
	return memo + num;
}, 3);
/*
3,1,0
[1, 2, 3, 4]
4,2,1
[1, 2, 3, 4]
6,3,2
[1, 2, 3, 4]
9,4,3
[1, 2, 3, 4]
*/
console.log(test1);//13
```

```
var b = [1,2,3,4];
var test2 = _.reduce(b,function (memo,num,index,list){
	console.log(memo+','+num+','+index);
	console.log(list);
	return num ;
}, 3);
/*
3,1,0
[1, 2, 3, 4]
1,2,1
[1, 2, 3, 4]
2,3,2
[1, 2, 3, 4]
3,4,3
[1, 2, 3, 4]
*/
console.log(test2);//4
```

```
var c = {
	one:'a',
	two:'b',
	three:'c',
};
var test3 = _.reduce(c,function (memo,value,key,list){
	console.log(value+','+key);
	console.log(memo);
	return value ;
}, 'd');
/*
a,one
d
b,two
a
c,three
b
*/
console.log(test3);//c
```

```
var list2 = [1,2,3,4];
var test2 = _.reduceRight(list2,function (memo,data,rightIndex,list){
	console.log( memo );
	console.log( data );
	console.log( rightIndex );
	console.log( list );
	return memo.concat(data);
}, [4]);
/*
[4]
4
3
[1, 2, 3, 4]
[4, 4]
3
2
[1, 2, 3, 4]
[4, 4, 3]
2
1
[1, 2, 3, 4]
[4, 4, 3, 2]
1
0
[1, 2, 3, 4]
*/
console.log(test2);// [4, 4, 3, 2, 1]
```
			




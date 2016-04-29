---
layout: post
category: js源码解析
---

# 集合(Collections)

## 1. _.each

 _.each(list, iterator, [context])（_.forEach）
遍历list中的所有元素，按顺序用遍历输出每个元素。如果传递了context参数，则把iteratee绑定到context对象上。每次调用iteratee都会传递三个参数：(element, index, list)。如果list是个JavaScript对象，iteratee的参数是 (value, key, list))。返回list以方便链式调用。（愚人码头注：如果存在原生的forEach方法，Underscore就使用它代替。）

```
_.each = _.forEach = function(obj, iteratee, context) {
	iteratee = optimizeCb(iteratee, context);
	var i, length;
	if (isArrayLike(obj)) {
	 for (i = 0, length = obj.length; i < length; i++) {
	   iteratee(obj[i], i, obj);
	 }
	} else {
	 // _.keys(object) 检索object拥有的所有可枚举属性的名称。返回数组
	 var keys = _.keys(obj);
	 for (i = 0, length = keys.length; i < length; i++) {
	   iteratee(obj[keys[i]], keys[i], obj);
	 }
	}
	return obj;
};
```

1、 首先执行“optimizeCb()”函数，得到迭代器iteratee的值。语句：

```
iteratee = optimizeCb(iteratee, context);
```
即如下图函数所示，argCount没有传入参数，即选择case 3.iteratee得到的值为：

```iteratee = function(value, index, collection) {	return func.call(context, value, index, collection);};
```

```
var optimizeCb = function(func, context, argCount) {
  if (context === void 0) return func;
  switch (argCount == null ? 3 : argCount) {
    case 1: return function(value) {
      return func.call(context, value);
    };
    case 3: return function(value, index, collection) {
      return func.call(context, value, index, collection);
    };
    case 4: return function(accumulator, value, index, collection) {
      return func.call(context, accumulator, value, index, collection);
    };
  }
  return function() {
    return func.apply(context, arguments);
  };
};
    // call函数和apply方法的第一个参数都是要传入给当前对象的对象，及函数内部的this。
    // 后面的参数都是传递给当前对象的参数。
    // 对于apply和call两者在作用上是相同的，但两者在参数上有区别的。
    // 对于第一个参数意义都一样，但对第二个参数：
    // apply传入的是一个参数数组，也就是将多个参数组合成为一个数组传入，
    // 而call则作为call的参数传入（从第二个参数开始）。
    // 如 func.call(func1,var1,var2,var3)
    // 对应的apply写法为：func.apply(func1,[var1,var2,var3])
    // 同时使用apply的好处是可以直接将当前函数的arguments对象作为apply的第二个参数传入
```

2、 判断传入的对象obj是否为类数组对象或对象,类数组对象是有一个数值length属性和对应非负整数属性的对象,如：{'0':'a', '1':'b', '2':'c', length:3}。语句：

```
if (isArrayLike(obj))
```
即如下图函数所示，使用代码：

```length= property('length')(collection)
```

判断length是否有值。

```
var property = function(key) {
  return function(obj) {
    return obj == null ? void 0 : obj[key];
  };
};

// Math.pow(底数,几次方)
var MAX_ARRAY_INDEX = Math.pow(2, 53) - 1;
var getLength = property('length');
var isArrayLike = function(collection) {
  var length = getLength(collection); 
  return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
};
```

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

1) 若obj为数组，则使用for循环，遍历从obj数组的第0为开始到，到obj的最后一位结束，并求出三个数值：obj[i]（obj数组对应的值）、i（obj数组对应的索引位置）、obj（obj数组列表本身），传进迭代器iteratee中，并运行iteratee函数。

```
for (i = 0, length = obj.length; i < length; i++) {
   iteratee(obj[i], i, obj);
}
```

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

2) 若obj为对象
a. 首先，检索object拥有的所有可枚举属性的名称，并返回数组keys。
		
```var keys = _.keys(obj);
```
		b. 使用for循环，遍历keys并返回它们的值（obj的属性），并求出三个值：obj[keys[i]]（obj对象属性对应的值）、keys[i]（obj对象对应的属性名称）、obj（obj对象列表本身），传进迭代器iteratee中，并运行iteratee函数。

```
var keys = _.keys(obj);
for (i = 0, length = keys.length; i < length; i++) {
	iteratee(obj[keys[i]], keys[i], obj);
}
```

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

3、 最后返回obj本身return obj;

## 2.  _.map

_.map(list, iterator, [context])（_. collect）
通过转换函数(iterator迭代器)映射列表中的每个值产生价值的新数组。iterator传递三个参数：value，然后是迭代 index(或 key 愚人码头注：如果list是个JavaScript对象是，这个参数就是key)，最后一个是引用指向整个list。

```
_.map = _.collect = function(obj, iteratee, context) {
	iteratee = cb(iteratee, context);
	var keys = !isArrayLike(obj) && _.keys(obj),
	   length = (keys || obj).length,
	   results = Array(length);
	for (var index = 0; index < length; index++) {
	 var currentKey = keys ? keys[index] : index;
	 results[index] = iteratee(obj[currentKey], currentKey, obj);
	}
	return results;
};
```

1、 首先执行“cb()”函数，得到迭代器iteratee的值。语句：

```
iteratee = cb(iteratee, context);
```
即如下图函数所示：

```
var builtinIteratee;
var cb = function(value, context, argCount) {
	if (_.iteratee !== builtinIteratee) return _.iteratee(value, context);
  // _.identity(value) 返回与传入参数相等的值. 相当于数学里的: f(x) = x,
  // 这个函数看似无用, 但是在Underscore里被用作默认的迭代器iterator.
  if (value == null) return _.identity;
  // _.isFunction(object) 如果object是一个函数（Function），返回true。
  if (_.isFunction(value)) return optimizeCb(value, context, argCount);
  // _.isObject(value) 如果object是一个对象，返回true。
  // 需要注意的是JavaScript数组和函数是对象，字符串和数字不是。
  // _.matcher(attrs) 返回一个断言函数，(matcher:匹配器)
  // 这个函数会给你一个断言可以用来辨别给定的对象是否匹配attrs指定键/值属性。
  if (_.isObject(value)) return _.matcher(value);
  // _.property(key) 返回一个函数，这个函数返回任何传入的对象的key属性。
  return _.property(value);
};
```

1) 若value为空值

返回与传入参数value相等的值 _.identity(value) 相当于数学里的: f(x) = x。

```
// 保持恒等函数默认迭代。
_.identity = function(value) {
  return value;
};
```

2) 若value为函数

运行函数“optimizeCb()”函数，得到迭代器iteratee的值。argCount没有传入参数，即选择case 3.iteratee得到的值为：

```iteratee = function(value, index, collection) {	return func.call(context, value, index, collection);};
```

3) 若value为对象（数组或对象）

则返回一个函数（断言函数），这个函数会给你一个断言可以用来辨别给定的对象是否匹配value（attrs）指定键/值属性.

```
_.matcher = _.matches = function(attrs) {
  // _.extendOwn(destination, *sources) Alias: assign 
  // 类似于 extend, 但复制source对象中自己的属性覆盖到destination目标对象。
  // 并且返回 destination 对象. 复制是按顺序的, 
  // 所以后面的对象属性会把前面的对象属性覆盖掉(如果有重复).
  // （愚人码头注：不包括继承过来的属性）
  attrs = _.extendOwn({}, attrs);
  return function(obj) {
    // _.isMatch(object, properties) 
    // 告诉你properties中的键和值是否包含在object中。
    return _.isMatch(obj, attrs);
  };
};
```

4) 若value为其他值（数字，字符串，undefined，布尔值）,则返回一个函数，这个函数返回任何传入的对象的value属性值

```
obj(value):_.property= property (value);
```

```
var property = function(key) {
  return function(obj) {
    return obj == null ? void 0 : obj[key];
  };
};

_.property = property;
```

2、 判断obj是不是类数组对象，检索object拥有的所有可枚举属性的名称，并返回数组keys。
3、 使用for循环，遍历keys（index）并返回它们的值（obj的属性或数组值），并求出三个值：obj[keys[i]／i]（obj对象属性（数组索引）对应的值）、keys[i]／i（obj对象对应的属性名称（数组对应的索引））、obj（obj对象（数组）列表本身），传进迭代器iteratee中，并运行iteratee函数后，返回新的数组列表值。

```
for (var index = 0; index < length; index++) {
 	var currentKey = keys ? keys[index] : index;
 	results[index] = iteratee(obj[currentKey], currentKey, obj);
}
return results;
```

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

## 3.  _.reduce和_.reduceRight_.reduce(list, iterator, [memo], [context])
（_.foldl/_.inject）
_.reduceRight(list, iteratee, memo, [context])
（_.foldr）Memo（备忘录）是reduce函数的初始值，reduce的每一步都需要由iterator返回。这个迭代传递4个参数：memo,value 和 迭代的index（或者 key）和最后一个引用的整个 list，最后一个是引用指向整个list。如果reduce的初始调用没有传递memo，iterator将会调用列表中的第一个元素。第一个元素通过memo参数在iterator中调用并传递到列表的下一个元素中。
reducRight是从右侧开始组合的元素的reduce函数，如果存在JavaScript 1.8版本的reduceRight，则用其代替。Foldr在javascript中不像其它有懒计算的语言那么有用（愚人码头注：lazy evaluation：一种求值策略，只有当表达式的值真正需要时才对表达式进行计算）。

```
_.reduce = _.foldl = _.inject = createReduce(1);
```

```
_.reduceRight = _.foldr = createReduce(-1);
```

```
var createReduce = function(dir) {
	var reducer = function(obj, iteratee, memo, initial) {
		 var keys = !isArrayLike(obj) && _.keys(obj),
		     length = (keys || obj).length,
		     index = dir > 0 ? 0 : length - 1;
		 if (!initial) {
		   	memo = obj[keys ? keys[index] : index];
		   	index += dir;
		 }
		 for (; index >= 0 && index < length; index += dir) {
			   var currentKey = keys ? keys[index] : index;
			   memo = iteratee(memo, obj[currentKey], currentKey, obj);
		 }
		 return memo;
	};
	
	return function(obj, iteratee, memo, context) {
	 	var initial = arguments.length >= 3;
	 	return reducer(obj, optimizeCb(iteratee, context, 4), memo, initial);
	};
};
```

1、 判断是否有传memo的值，并得到initial的布尔值。

2、 运行optimizeCb()函数，且argCount传入的参数值为4，故：
iteratee得到的值为：

```
iteratee = function(accumulator,value, index, collection) {	return func.call(context, accumulator ,value, index, collection);};```

3、 运行reducer()函数，并得到最新的memo值，并返回

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

## 4. _.find

_.find(list, predicate, [context])（_.detect）
在list中逐项查找，返回第一个通过predicate迭代函数真值检测的元素值，如果没有值传递给测试迭代器将返回undefined，如果找到匹配的元素，函数将立即返回，不会遍历整个list。

```
_.find = _.detect = function(obj, predicate, context) {
    var key;
    if (isArrayLike(obj)) {
	      // _.findIndex(array, predicate, [context]) 
	      // 类似于_.indexOf，当predicate通过真检查时，返回第一个索引值；否则返回-1。
	      key = _.findIndex(obj, predicate, context);
    } else {
	      // _.findKey(object, predicate, [context]) 
	      // 类似于_.findIndex，但是检测objects的属性，当predicate通过真检查时，返回属性名称；否则返回-1。
	      key = _.findKey(obj, predicate, context);
    }
    if (key !== void 0 && key !== -1) return obj[key];
};
```

1、于 _.indexOf，当predicate通过真检查时，返回第一个索引值；否则返回-1。

2、 若obj为对象，则执行函数 _.findKey()，此函数类似于 _.findIndex，但是检测objects的属性，当predicate通过真检查时，返回属性名称；否则返回undefined。

3、 若key存在且不等于-1，则返回对象中当前key（属性或索引）的值

4、 其中：

1) _.findIndex()

```
_.findIndex = createPredicateIndexFinder(1);
```

```
var createPredicateIndexFinder = function(dir) {
  return function(array, predicate, context) {
    predicate = cb(predicate, context);
    var length = getLength(array);
    var index = dir > 0 ? 0 : length - 1;
    for (; index >= 0 && index < length; index += dir) {
      if (predicate(array[index], index, array)) return index;
    }
    return -1;
  };
};
```

a. 运行cb()函数，得到迭代函数predicate()的值

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

b. index从0开始进行for循环，当满足迭代函数predicate的条件时，直接返回index（索引位置），若遍历完列表后，没有满足迭代函数predicate条件，则返回-1

2) _.findKey()

```
_.findKey = function(obj, predicate, context) {
  predicate = cb(predicate, context);
  var keys = _.keys(obj), key;
  for (var i = 0, length = keys.length; i < length; i++) {
    key = keys[i];
    if (predicate(obj[key], key, obj)) return key;
  }
};
```

a. 运行cb()函数，得到predicate的值

```predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

b. 检索object拥有的所有可枚举属性的名称，并返回数组keys。

c. index从0开始进行for循环，遍历obj的各个属性，并把属性名称值赋给key，当满足迭代函数predicate的条件时，直接返回key（属性名称值），若遍历完列表后，没有满足迭代函数predicate条件，则返回undefined。
例：

```
var list = [1, 2, 3, 4, 5, 6];
var test1 = _.find(list,function (data,index,list){
	console.log( data +','+ index );
	return data % 3 == 0;
});
/*
1,0
2,1
3,2
*/
console.log(test1);// 3
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
};
var test2 = _.find(list2,function (data,key,list){
	console.log( data +','+ key );
	return key == 'two';
});
/*
a,one
b,two
*/
console.log(test2);// b
```

## 5. _.filter

_.filter(list, predicate, [context])（_.select）
遍历list中的每个值，返回包含所有通过predicate迭代函数真值检测的元素值。（愚人码头注：如果存在原生filter方法，则用原生的filter方法。）

```
_.filter = _.select = function(obj, predicate, context) {
  var results = [];
  predicate = cb(predicate, context);
  _.each(obj, function(value, index, list) {
    if (predicate(value, index, list)) results.push(value);
  });
  return results;
};
```

1、 运行cb()函数，并得到迭代函数predicate()的值:

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

2、 运行_.each()函数，遍历obj对象，并判断是否满足迭代函数predicate的条件，满足，则把相应的值推进数组results中，不满足，则继续遍历，直到遍历完成为止，并返回results数组。

例：

```
var list = [1, 2, 3, 4, 5, 6];
var test1 = _.filter(list,function (data,index,list){
	console.log( data +','+ index );
	return data % 2 == 0;
});
/*
1,0
2,1
3,2
4,3
5,4
6,5
*/
console.log(test1);// [2, 4, 6]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
	five:'e',
	six:'f',
	seven:'g',
};
var test2 = _.filter(list2,function (data,key,list){
	return key == 'one' || key=='five' || key=='seven';
});
console.log(test2);// ["a", "e", "g"]
```

## 6. _.reject

_.reject(list, predicate, [context])
返回list中没有通过predicate真值检测的元素集合，与filter相反

```
_.reject = function(obj, predicate, context) {
  // negate_.negate(predicate) (对立面；反面)
  // 返回一个新的predicate函数的否定版本。
  return _.filter(obj, _.negate(cb(predicate)), context);
};
```

1、 运行cb()函数，并得到迭代函数predicate()的值:

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

2、 运行_.negate()函数，通过！非运算符，返回一个新的迭代函数predicate()的否定版本。

```
_.negate = function(predicate) {
  return function() {
    return !predicate.apply(this, arguments);
  };
};
```

例：

```
var list = [1, 2, 3, 4, 5, 6];
var test1 = _.reject(list,function (data,index,list){
	console.log( data +','+ index );
	return data % 2 == 0;
});
/*
1,0
2,1
3,2
4,3
5,4
6,5
*/
console.log(test1);// [1, 3, 5]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
	five:'e',
	six:'f',
	seven:'g',
};
var test2 = _.reject(list2,function (data,key,list){
	return key == 'one' || key=='five' || key=='seven';
});
console.log(test2);// ["b", "c", "d", "f"]
```## 7. _.every

_.every(list, [predicate], [context])（_.all）
如果list中的所有元素都通过predicate的真值检测就返回true。

```
_.every = _.all = function(obj, predicate, context) {
  predicate = cb(predicate, context);
  var keys = !isArrayLike(obj) && _.keys(obj),
      length = (keys || obj).length;
  for (var index = 0; index < length; index++) {
    var currentKey = keys ? keys[index] : index;
    if (!predicate(obj[currentKey], currentKey, obj)) return false;
  }
  return true;
};
```

1、 运行cb()函数，并得到迭代函数predicate()的值:

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

2、 判断obj不是类数组对象，检索object拥有的所有可枚举属性的名称，并返回数组keys。

3、 使用for循环，判断迭代函数 predicate()的条件，当有一条数据不满足条件时，返回false，当所有数据满足条件时，返回true。

例：

```
var list = [2, 4, 6, 8, 10, 12];
var test1 = _.every(list,function (data,index,list){
	return data % 2 == 0;
});
console.log(test1);// true
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'a',
	four:'a',
};
var test2 = _.every(list2,function (data,key,list){
	console.log(data);
	return data == 'a';
});
/*
a
b
*/
console.log(test2);// false
```

## 8. _.some

_.some(list, [predicate], [context])（_.any）
如果list中有任何一个元素通过 predicate 的真值检测就返回true。一旦找到了符合条件的元素, 就直接中断对list的遍历。

```
_.some = _.any = function(obj, predicate, context) {
  predicate = cb(predicate, context);
  var keys = !isArrayLike(obj) && _.keys(obj),
      length = (keys || obj).length;
  for (var index = 0; index < length; index++) {
    var currentKey = keys ? keys[index] : index;
    if (predicate(obj[currentKey], currentKey, obj)) return true;
  }
  return false;
};
```

1、 运行cb()函数，并得到迭代函数predicate()的值:

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

2、 判断obj不是类数组对象，检索object拥有的所有可枚举属性的名称，并返回数组keys。

3、 使用for循环，判断迭代函数 predicate()的条件，当有一条数据满足条件时，返回true，当所有数据不满足条件时，返回false。与_.every()函数相反。

例：

```
var list = [1, 2, 3, 4, 5, 6];
var test1 = _.some(list,function (data,index,list){
	console.log(data);
	return data % 2 == 0;
});
/*
1
2
*/
console.log(test1);// true
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
};
var test2 = _.some(list2,function (data,key,list){
	console.log(data);
	return data == 'b';
});
/*
a
b
*/
console.log(test2);// true
```

## 9. _.contains

_.contains(list, value, [fromIndex])（_.includes/_.include）
如果list包含指定的value则返回true（愚人码头注：使用===检测）。如果list 是数组，内部使用indexOf判断。使用fromIndex来给定开始检索的索引位置。

```
_.contains = _.includes = _.include = function(obj, item, fromIndex, guard) {
  // _.values(object) 
  // 返回object对象所有的属性值。
  if (!isArrayLike(obj)) obj = _.values(obj);
  if (typeof fromIndex != 'number' || guard) fromIndex = 0;
  // _.indexOf(array, value, [isSorted]) 
  // 返回value在该 array 中的索引值，如果value不存在 array中就返回-1。
  // 使用原生的indexOf 函数，除非它失效。
  // 如果您正在使用一个大数组，你知道数组已经排序，
  // 传递true给isSorted将更快的用二进制搜索..,或者，
  // 传递一个数字作为第三个参数，为了在给定的索引的数组中寻找第一个匹配值。
  return _.indexOf(obj, item, fromIndex) >= 0;
};
```

1、 判断obj若为对象，则运行_.values(obj)函数，返回一个数组，其数组为所有obj属性的值。

2、 确保fromIndex的值为数字。

3、 运行_.indexOf()函数，并检测返回值是否为-1，若是：返回false，若否：返回true。

例：

```
var list = [2, 4, 6, 8, 10, 12];
var test1 = _.contains(list,6);
console.log(test1);// true
```

```
var test3 = _.contains(list,6,3);
console.log(test3);// false
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
};
var test2 = _.contains(list2,'c');
console.log(test2);// true
```

## 10. _.invoke

_.invoke(list, methodName, *arguments)
在list的每个元素上执行methodName方法。任何传递给invoke的额外参数，invoke都会在调用methodName方法的时候传递给它。

```
_.invoke = restArgs(function(obj, method, args) {
  var isFunc = _.isFunction(method);
  return _.map(obj, function(value) {
    var func = isFunc ? method : value[method];
    return func == null ? func : func.apply(value, args);
  });
});
```

```
var restArgs = function(func, startIndex) {
  startIndex = startIndex == null ? func.length - 1 : +startIndex;
  return function() {
    var length = Math.max(arguments.length - startIndex, 0);
    var rest = Array(length);
    for (var index = 0; index < length; index++) {
      rest[index] = arguments[index + startIndex];
    }
    switch (startIndex) {
      case 0: return func.call(this, rest);
      case 1: return func.call(this, arguments[0], rest);
      case 2: return func.call(this, arguments[0], arguments[1], rest);
    }
    var args = Array(startIndex + 1);
    for (index = 0; index < startIndex; index++) {
      args[index] = arguments[index];
    }
    args[startIndex] = rest;
    return func.apply(this, args);
  };
};
```	

1、 运行函数restArgs()函数，并运行里面的function()函数

2、 遍历obj对象，判断method是否为函数对象，若是，直接把函数赋给func变量，若否，得到每一个遍历obj的值value，并读取value对象中的method属性的值，并赋给func变量。

3、 判断func是否为null，若存在，运行func函数并存在参数数组［args］，且作用域是value，若不存在，运行func函数。

例：

```
var list = [[5, 1, 7], [3, 2, 1]];
var test1 = _.invoke(list,'sort');
console.log(test1);// [[1,5,7],[1,2,3]]
```

## 11. _.pluck

_.pluck (list, propertyName)
pluck也许是map最常使用的用例模型的简化版本，即萃取数组对象中某属性值，返回一个数组。

```
_.pluck = function(obj, key) {
  return _.map(obj, _.property(key));
};
```

1、 运行 _.property函数，返回一个函数，这个函数返回任何传入的对象的key属性值obj(key): _.property= property (key); 

```
var property = function(key) {
  return function(obj) {
    return obj == null ? void 0 : obj[key];
  };
};

_.property = property;
```

即：

```
_.pluck = _.map(obj,function(key){
	return function(obj){
		return obj == null ? void 0 : obj[key];
	}
}
_.pluck = _.map(obj,function(objChild ,key){
	return objChild == null ? void 0 : objChild [key];
}
```

2、 运行_.map()函数，并得到返回值为obj属性值数组。
例：```
var list =  [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 50}, 
	{name: 'curly', age: 60}
];
var test1 = _.pluck(list,'name');
console.log(test1);// ["moe", "larry", "curly"]
```

## 12. _.where

_.where (list, properties)
遍历list中的每一个值，返回一个数组，这个数组包含properties所列出的属性的所有的 键 - 值对。

```
_.where = function(obj, attrs) {
  return _.filter(obj, _.matcher(attrs));
};
```

1、 运行_.matcher()函数，返回一个函数（断言函数），这个函数会给你一个断言可以用来辨别给定的对象是否匹配attrs指定键/值属性。

```
_.matcher = _.matches = function(attrs) {
  // _.extendOwn(destination, *sources) Alias: assign 
  // 类似于 extend, 但复制source对象中自己的属性覆盖到destination目标对象。
  // 并且返回 destination 对象. 复制是按顺序的, 
  // 所以后面的对象属性会把前面的对象属性覆盖掉(如果有重复).
  // （愚人码头注：不包括继承过来的属性）
  attrs = _.extendOwn({}, attrs);
  return function(obj) {
    // _.isMatch(object, properties) 
    // 告诉你properties中的键和值是否包含在object中。
    return _.isMatch(obj, attrs);
  };
};
```

即：

```
_.where = _.filter(obj,function(attrs){
	attrs = _.extendOwn({}, attrs);
	return function(obj){
		return _.isMatch(obj,attrs);
	}
});
```

2、 运行_.filter()函数，遍历obj中的每个值，返回包含所有通过function(attrs)迭代函数真值检测的元素值。

例：

```
var listOfPlays = [
	{title: 'Cymbeline', author: 'Shakespeare', year: 1611},
	{title: 'The Tempest',author: 'Shakespeare', },
	{title: 'The Tempest', author: 'Shakespeare', year: 1611},
];
var obj = {author: "Shakespeare", year: 1611};
var list = _.where(listOfPlays,obj);
console.log(list);
/*[Object]
0: Object
	author: "Shakespeare"
	title: "Cymbeline"
	year: 1611
	__proto__: Object
1: Object
	author: "Shakespeare"
	title: "The Tempest"
	year: 1611
	__proto__: Object
	length: 2
	__proto__: Array[0]
*/
```

## 13. _.findWhere 

_.findWhere (list, properties)
遍历整个list，返回匹配 properties参数所列出的所有 键 - 值 对的第一个值。如果没有找到匹配的属性，或者list是空的，那么将返回undefined。

```
_.findWhere = function(obj, attrs) {
  return _.find(obj, _.matcher(attrs));
};
```

1、 运行_.matcher()函数，返回一个函数（断言函数），这个函数会给你一个断言可以用来辨别给定的对象是否匹配attrs指定键/值属性.
即：```
_.findWhere = _.find(obj,function(attrs){
	attrs = _.extendOwn({}, attrs);
	return function(obj){
		return _.isMatch(obj,attrs);
	}
});
```

2、 运行_.find()函数，在obj中逐项查找，返回第一个通过function(attrs)迭代函数真值检测的元素值，如果没有值传递给测试迭代器将返回undefined，如果找到匹配的元素，函数将立即返回，不会遍历整个obj。
例：```
var publicServicePulitzers = [
	{year: 1348, reason: "For its public service in publishing."},
	{year: 1918,newsroom: "The New York Times",reason: "For its public"},
	{year: 1918, reason: "For its public service in publishing."},
	{year: 2008, newsroom: "The New York Times",reason: "For its public"},
];
var obj = {newsroom: "The New York Times"};
var list = _.findWhere(publicServicePulitzers,obj);
console.log(list);
/*
Object {
	year: 1918, 
	newsroom: "The New York Times", 
	reason: "For its public"
}
*/
```

## 14. _.max

_.max(list, [iteratee], [context])
返回list中的最大值。如果传递iteratee参数，iteratee将作为list中每个值的排序依据。如果list为空，将返回-Infinity，所以你可能需要事先用isEmpty检查 list 。

```
_.max = function(obj, iteratee, context) {
  var result = -Infinity, lastComputed = -Infinity,
      value, computed;
  if (iteratee == null || (typeof iteratee == 'number' && typeof obj[0] != 'object') && obj != null) {
  	//_.values(object) 
  	//返回object对象所有的属性值。
    obj = isArrayLike(obj) ? obj : _.values(obj);
    for (var i = 0, length = obj.length; i < length; i++) {
      value = obj[i];
      if (value != null && value > result) {
        result = value;
      }
    }
  } else {
    iteratee = cb(iteratee, context);
    _.each(obj, function(v, index, list) {
      computed = iteratee(v, index, list);
      if (computed > lastComputed || computed === -Infinity && result === -Infinity) {
        result = v;
        lastComputed = computed;
      }
    });
  }
  return result;
};
```

1、 判断迭代器iteratee是否为null或判断迭代器iteratee的类型为”number”、obj[0]的类型不为“object”，obj不为null。

若是:

a) 首先判断obj是否为类数组对象，是，则直接吧obj赋值给obj，否，则运行_.values(obj)函数,返回obj对象所有的属性值,并返回一个数组，赋值给obj。

b) 通过for循环，遍历新对象obj的每一个值，并赋值给value，判断value不为null，且value > result则把value赋值给result，直到找到最大值为止。并返回result值。

例：

```
var list2 = [3,45,23,67,43,21];
var test2 = _.max(list2);
console.log(test2);//67
```

```
var list3 = {
	age1:'34',
	age2:'42',
	age3:'56',
	age4:'43',
	age5:'35'
};
var test3 = _.max(list3);
console.log(test3);//56
```

2、 若否：

a) 运行cb()函数，并得到迭代函数iteratee()的值:

```
iteratee=optimizeCb(iteratee,context);			=function(value, index, collection) {      			return iteratee.call(context, value, index, collection);   		};
```

b) 运行迭代函数iteratee，并把返回值赋值给computed。

c) 运行_.each()函数，遍历obj。判断computed > lastComputed 或computed完全等于负无穷，result完全等于负无穷，则把v的值赋给result，把computed的值赋给lastComputed。直到找到computed的最大值为止，并返回result值。

例：

```
var list = [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 70}, 
	{name: 'curly', age: 60}
];
var test = _.max(list,function(value){
	return value.age;
});
console.log(test);//Object{name: 'larry', age: 70}
```

## 15. _.min

_.min(list, [iteratee], [context]) 
返回list中的最小值。如果传递iteratee参数，iteratee将作为list中每个值的排序依据。如果list为空，将返回-Infinity，所以你可能需要事先用isEmpty检查 list 。

```
_.min = function(obj, iteratee, context) {
  var result = Infinity, lastComputed = Infinity,
      value, computed;
  if (iteratee == null || (typeof iteratee == 'number' && typeof obj[0] != 'object') && obj != null) {
    obj = isArrayLike(obj) ? obj : _.values(obj);
    for (var i = 0, length = obj.length; i < length; i++) {
      value = obj[i];
      if (value != null && value < result) {
        result = value;
      }
    }
  } else {
    iteratee = cb(iteratee, context);
    _.each(obj, function(v, index, list) {
      computed = iteratee(v, index, list);
      if (computed < lastComputed || computed === Infinity && result === Infinity) {
        result = v;
        lastComputed = computed;
      }
    });
  }
  return result;
};
```

1、 判断迭代器iteratee是否为null或判断迭代器iteratee的类型为”number”、obj[0]的类型不为“object”，obj不为null。

若是:

a) 首先判断obj是否为类数组对象，是，则直接吧obj赋值给obj，否，则运行_.values(obj)函数,返回obj对象所有的属性值,并返回一个数组，赋值给obj。

b) 通过for循环，遍历新对象obj的每一个值，并赋值给value，判断value不为null，且value < result则把value赋值给result，直到找到最小值为止。并返回result值。

例：

```
var list2 = [3,45,23,67,43,21];
var test2 = _.min(list2);
console.log(test2);//3
```

```
var list3 = {
	age1:'34',
	age2:'42',
	age3:'56',
	age4:'43',
	age5:'35'
};
var test3 = _.min(list3);
console.log(test3);//34
```

2、 若否：

a) 运行cb()函数，并得到迭代函数iteratee()的值:

```
iteratee=optimizeCb(iteratee,context);			=function(value, index, collection) {      			return iteratee.call(context, value, index, collection);   		};
```

b) 运行迭代函数iteratee，并把返回值赋值给computed。

c) 运行_.each()函数，遍历obj。判断computed < lastComputed 或computed完全等于正无穷，result完全等于正无穷，则把v的值赋给result，把computed的值赋给lastComputed。直到找到computed的最小值为止，并返回result值。

例：

```
var list = [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 70}, 
	{name: 'curly', age: 60}
];
var test = _.min(list,function(value){
	return value.age;
});
console.log(test);//Object{name: 'moe', age: 40}
```

## 16. _.sample

_.sample(list, [n]) 
从 list中产生一个随机样本。传递一个数字表示从list中返回n个随机元素。否则将返回一个单一的随机项。

```
_.sample = function(obj, n, guard) {
  // _.random(min, max) 
  // 返回一个min 和 max之间的随机整数。
  // 如果你只传递一个参数，那么将返回0和这个参数之间的整数.
  if (n == null || guard) {
    if (!isArrayLike(obj)) obj = _.values(obj);
    return obj[_.random(obj.length - 1)];
  }
  // _.clone(object) 
  // 创建 一个浅复制（浅拷贝）的克隆object。
  // 任何嵌套的对象或数组都通过引用拷贝，不是复制。
  var sample = isArrayLike(obj) ? _.clone(obj) : _.values(obj);
  var length = getLength(sample);
  n = Math.max(Math.min(n, length), 0);
  var last = length - 1;
  for (var index = 0; index < n; index++) {
    var rand = _.random(index, last);
    var temp = sample[index];
    sample[index] = sample[rand];
    sample[rand] = temp;
  }
  return sample.slice(0, n);
};
```

1、 判断n是为null或guard存在。

a) 则判断obj若不为类数组对象，运行_.values(obj)函数，返回obj对象所有的属性值,并返回一个数组，赋值给obj。
b) 运行 _.random(obj.length - 1)函数，返回0与obj.length - 1之间的一个随机整数。从而，返回一个obj随机数。

2、 n存在

a) 判断obj是否为数组，若是，则运行_.clone(obj)，拷贝一份新的数组赋值给sample，若否，则运行 _.values(obj)函数，返回obj对象所有的属性值,并返回一个数组，赋值给sample。

b) 读取sample的长度，并赋值给length。

c) 运行Math.min()函数，得出length与n中的最小值，并运行Math.max()函数，得到其最小值与0之间的最大值。

d) 得出sample最后一个索引值，赋值给last。

e) 运行for循环函数，遍历n次，运行 _.random(index,last)函数，返回index与last之间的一个随机整数，赋值给rand，相当于一个随机索引值。添加中间变量temp存储sample[index]的值。最后得到sample[index]和sample[rand]的值，并相互交换他们所在的位置，重新得到一个随机数组。

f) 返回新数组中0至n的值。

例：

```
var list = [1, 2, 3, 4, 5, 6];
var test = _.sample(list);
console.log(test);//6
```

```
var list = [1, 2, 3, 4, 5, 6];
var test2 = _.sample(list,3);
console.log(test2);//[6, 5, 4]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
	five:'e'
};
var test2 = _.sample(list2);
console.log(test2);//c
```

## 17. _.shuffle

_.shuffle(list)
返回一个随机乱序的 list 副本, 使用 Fisher-Yates shuffle 来进行随机乱序。

```
_.shuffle = function(obj) {
  return _.sample(obj, Infinity);
};
```

1、 运行次函数_.shuffle()主要与运行 _.sample()函数同理，参数n为Infinity无穷大。故运行后将到一个随机的obj对象。

例：

```
var list = [1, 2, 3, 4, 5, 6];
var test = _.shuffle(list);
console.log(test);//[1, 5, 4, 2, 6, 3]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
	five:'e'
};
var test2 = _.shuffle(list2);
console.log(test2);//["c", "b", "e", "d", "a"]
```

## 18. _.sortBy

_.sortBy(list, iteratee, [context]) 
返回一个排序后的list拷贝副本。如果传递iteratee参数，iteratee将作为list中每个值的排序依据。迭代器也可以是字符串的属性的名称进行排序的(比如 length)。

```
_.sortBy = function(obj, iteratee, context) {
  var index = 0;
  iteratee = cb(iteratee, context);
  // _.pluck(list, propertyName)
  // pluck也许是map最常使用的用例模型的简化版本，
  // 即萃取数组对象中某属性值，返回一个数组。
  return _.pluck(_.map(obj, function(value, key, list) {
    return {
      value: value,
      index: index++,
      criteria: iteratee(value, key, list)
    };
  }).sort(function(left, right) {
    // arrayobj.sort(sortfunction)  
    // 返回一个元素已经进行了排序的 Array 对象。 
    // arrayObj 必选项。任意 Array 对象。  
    // sortFunction 可选项。是用来确定元素顺序的函数的名称。
    // 如果这个参数被省略，那么元素将按照 ASCII 字符顺序进行升序排列。  
    // 如果为 sortfunction 参数提供了一个函数，那么该函数必须返回下列值之一：
    // 负值，如果所传递的第一个参数比第二个参数小。
    // 零，如果两个参数相等。  
    // 正值，如果第一个参数比第二个参数大。
    var a = left.criteria;
    var b = right.criteria;
    if (a !== b) {
      if (a > b || a === void 0) return 1;	//降序排列
      if (a < b || b === void 0) return -1;	//升序排列
    }
    return left.index - right.index;
  }), 'value');
};
```

1、 运行cb()函数，

a) 若iteratee为迭代函数则:

```
iteratee=optimizeCb(iteratee,context);			=function(value, index, collection) {      			return iteratee.call(context, value, index, collection);   		};
```

b) 若iteratee为字符串或null，则

```
iteratee=_.property(iteratee);			=function(iteratee) {      			return function(obj) {
					   return obj == null ? void 0 : obj[iteratee];
					};   		};
```

2、 运行_.map()函数，返回一个新的数组，并运行sort()函数，对此数组按照一定规则进行排序。得到新的数组。

3、 运行_.pluck()函数，返回最终得到的数组。

例：

```
var list = [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 70}, 
	{name: 'curly', age: 60}
];
var test = _.sortBy(list,'name');
console.log(test);
/*
[
	{name: 'curly', age: 60}, 
	{name: 'larry', age: 70}, 
	{name: 'moe', age: 40}
];
*/
var test4 = _.sortBy(list,'age');
console.log(test4);
/*
[ 
	{name: 'moe', age: 40},
	{name: 'curly', age: 60}, 
	{name: 'larry', age: 70}
];
*/
```

```
var list2 = [3,45,23,67,43,21];
var test2 = _.sortBy(list2,function(num){ 
	//Math.sin(x)  
	//x 的正玄值。返回值在 -1.0 到 1.0 之间；
	return Math.sin(num); 
});
console.log(test2);//[67, 23, 43, 3, 21, 45]
```

```
var list3 = {
	age1:'34',
	age2:'42',
	age3:'56',
	age4:'43',
	age5:'35'
};
var test3 = _.sortBy(list3);
console.log(test3);//["34", "35", "42", "43", "56"]
```

## 19. _.groupBy

_.groupBy(list, iteratee, [context]) 
把一个集合分组为多个集合，通过 iterator 返回的结果进行分组。如果 iterator 是一个字符串而不是函数, 那么将使用 iterator 作为各元素的属性名来对比进行分组。

```
_.groupBy = group(function(result, value, key) {
  if (_.has(result, key)) result[key].push(value);
  else result[key] = [value];
});
```

1、 运行group()函数

```
var group = function(behavior, partition) {
  return function(obj, iteratee, context) {
    var result = partition ? [[], []] : {};
    iteratee = cb(iteratee, context);
    _.each(obj, function(value, index) {
      var key = iteratee(value, index, obj);
      behavior(result, value, key);
    });
    return result;
  };
};
```

2、 得到behavior的值为

```
behavior=function(result, value, key) {
  if (_.has(result, key)) result[key].push(value);
  else result[key] = [value];
}
```

3、 判断partition是否存在，若存在，则result=[[], []]，若不存在，则result={}。

4、 运行cb()函数，

a) 若iteratee为迭代函数则:

```
iteratee=optimizeCb(iteratee,context);			=function(value, index, collection) {      			return iteratee.call(context, value, index, collection);   		};
```

b) 若iteratee为字符串或null，则

```
iteratee=_.property(iteratee);			=function(iteratee) {      			return function(obj) {
					   return obj == null ? void 0 : obj[iteratee];
					};   		};
```

5、 运行_.each()函数，遍历obj对象。得到key的值。

6、 运行behavior()函数

a) 运行_.has(result, key)函数，判断对象result是否包含给定的键key。若result存在key，则把当前value值push到result[key]中；

b) 若result不存在key，则直接把value存到数组中并赋给result[key]；

7、 返回result的最终结果。

例：

```
var list = [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 40}, 
	{name: 'curly', age: 60}
];
var test = _.groupBy(list,'age');
console.log(test);
/*
{
	40:[
		{name: 'moe', age: 40}, 
		{name: 'larry', age: 40}
	],
	60:[
		{name: 'curly', age: 60}
	]
};
*/
```

```
var list2 = [1.3, 2.1, 2.4];
var test2 = _.groupBy(list2,function(num){ 
	//Math.floor(x)  
	//返回小于等于x的最大整数
	return Math.floor(num); 
});
console.log(test2);
/*
{
	1:[1.3],
	2:[2.1,2.4]
}
*/
```

## 20. _.indexBy

_.indexBy(list, iteratee, [context])  
给定一个list，和 一个用来返回一个在列表中的每个元素键 的iterator 函数（或属性名），返回一个每一项索引的对象。和groupBy非常像，但是当你知道你的键是唯一的时候可以使用indexBy 。

```
_.indexBy = group(function(result, value, key) {
  result[key] = value;
});
```

1、 与_.groupBy()函数一样，先运行group()函数

2、 得到behavior的值为

```
behavior=function(result, value, key) {
  result[key] = value;
}
```

3、 直接返回result的最终结果。

4、 此处与_.groupBy()函数最大的区别是， _.indexBy()函数中迭代函数iteratee得到的键值是唯一的，返回的result[key]结果为一个对象。而 _.groupBy()函数中迭代函数iteratee得到的键值不是唯一的，返回的result[key]结果为一个数组。

例：

```
var list = [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 70}, 
	{name: 'curly', age: 60}
];
var test = _.indexBy(list,'age');
console.log(test);
/*
{
	40:{name: 'moe', age: 40}, 
	60:{name: 'curly', age: 60},
	70:{name: 'larry', age: 70}
};
*/
```

```
var list2 = [1.3, 2.1, 3.4];
var test2 = _.indexBy(list2,function(num){ 
	//Math.floor(x)  
	//返回小于等于x的最大整数
	return Math.floor(num); 
});
console.log(test2);
/*
{
	1:1.3,
	2:2.1,
	3:3.4
}
*/
```

## 21. _.countBy

_.countBy(list, iteratee, [context])
排序一个列表组成一个组，并且返回各组中的对象的数量的计数。类似groupBy，但是不是返回列表的值，而是返回在该组中值的数目。

```
_.countBy = group(function(result, value, key) {
  if (_.has(result, key)) result[key]++; 
  else result[key] = 1;
});
```

1、 与_.groupBy()函数一样，先运行group()函数

2、 得到behavior的值为

```
behavior=function(result, value, key) {
  if (_.has(result, key)) result[key]++; 
  else result[key] = 1;
}
```

3、 运行behavior()函数

a) 运行 _.has(result, key)函数，判断对象result是否包含给定的键key。若result存在key，则result[key]++；

b) 若result不存在key，则result[key]=1；

4、 返回result的最终结果。

5、 此处与_.groupBy()函数最大的区别是， _.countBy()函数的结果是返回在该组中值的数目，而 _.groupBy()函数的结果是返回列表的值。

例：

```
var list = [
	{name: 'moe', age: 40}, 
	{name: 'larry', age: 40}, 
	{name: 'curly', age: 60}
];
var test = _.countBy(list,'age');
console.log(test);//{40: 2, 60: 1};
```

```
var list2 = [1.3, 2.1, 2.4];
var test2 = _.countBy(list2,function(num){ 
	//Math.floor(x)  
	//返回小于等于x的最大整数
	return Math.floor(num); 
});
console.log(test2);//{1: 1, 2: 2}
```

```
var list3 = [1, 2, 3,4,5];
var test3 = _.countBy(list3, function(num) {
  	return num % 2 == 0 ? 'even': 'odd';
});
console.log(test3);//{odd: 3, even: 2}
```

## 22. _.toArray

_.toArray(list) 
把list(任何可以迭代的对象)转换成一个数组，在转换 arguments 对象时非常有用。

```
// 这个正则按“|”分割，包含三个部分
// [^\ud800-\udfff] 是普通的 BMP 字符,表示不包含代理对代码点的所有字符
// [\ud800-\udbff][\udc00-\udfff] 是成对的代理项对,表示合法的代理对的所有字符
// [\ud800-\udfff] 是未成对的代理项字,表示代理对的代码点（本身不是合法的Unicode字符）
var reStrSymbol = /[^\ud800-\udfff]|[\ud800-\udbff][\udc00-\udfff]|[\ud800-\udfff]/g;
_.toArray = function(obj) {
  if (!obj) return [];
  if (_.isArray(obj)) return slice.call(obj);
  if (_.isString(obj)) {
    // Keep surrogate pair characters together
    // match() 方法可在字符串内检索指定的值，
    // 或找到一个或多个正则表达式的匹配。
    return obj.match(reStrSymbol);
  }
  if (isArrayLike(obj)) return _.map(obj, _.identity);
  return _.values(obj);
};
```

1、 判断obj不存在为null时，直接返回空数组[]。

2、 判断obj为数组时，直接返回obj。

例：

```
var test = (function(){ 
	return _.toArray(arguments).slice(1); 
})(1, 2, 3, 4);
console.log(test);//[2, 3, 4]
```

```
var list2 = [1,2,3,4,5,6];
var test2 = _.toArray(list2);
console.log(test2);//[1, 2, 3, 4, 5, 6]
```

3、 判断obj为字符串时，obj运行正则表达式obj.match(reStrSymbol)，并返回个字符串字母组成的数组。

```
var test4 = _.toArray('hello world!');
console.log(test4);
//["h", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d", "!"]
```

4、 判断obj为类数组对象时，运行_.map(obj)函数，并返回数组。

```
var list5 = {
	0:'a',
	1:'b',
	2:'c',
	3:'d',
	length:4
};
var test5 = _.toArray(list5);
console.log(test5);//["a", "b", "c", "d"]
```

5、 判断obj为对象时,运行_.values(obj)函数，返回obj对象所有的属性值。

```
var list3 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d'
};
var test3 = _.toArray(list3);
```

## 23. _.size

_.size(list) 
返回list的长度。

```
_.size = function(obj) {
  if (obj == null) return 0;
  return isArrayLike(obj) ? obj.length : _.keys(obj).length;
};
```

1、 判断obj为null，则返回0。

2、 判断obj为类数组对象，返回对象的长度。

例：

```
var list = [1,2,3,4,5,6];
var test = _.size(list);
console.log(test);//6
```

3、 判断obj为对象，运行_.keys(obj)函数，得到obj对象的属性名称，并返回此数组的长度。

例：

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d'
};
var test2 = _.size(list2);
console.log(test2);//4
```

## 24. _.partition

_.partition(array, predicate) 
拆分一个数组（array）为两个数组：第一个数组其元素都满足predicate迭代函数，而第二个的所有元素均不能满足predicate迭代函数。

```
_.partition = group(function(result, value, pass) {
	result[pass ? 0 : 1].push(value);
}, true);
```

1、 运行group()函数，得到behavior的值为

```
behavior=function(result, value, pass) {
 result[pass ? 0 : 1].push(value);
}
```

2、 由于partition的值为true，故result=[[], []]。

3、 运行behavior()函数，若满足pass条件，则把obj的值value传入数组索引为0的位置，若不满足pass条件，则把obj的值value传入数组索引为1的位置。

例：

```
var list = [1,2,3,4,5,6];
var test = _.partition(list,function (value){
	return value % 2 == 0;
});
console.log(test);//[[2, 4, 6],[1, 3, 5]]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d'
};
var test2 = _.partition(list2,function (value){
	return value == 'a' || value == 'c';
});
console.log(test2);//[['a', 'c'],['b', 'd']]
```






		




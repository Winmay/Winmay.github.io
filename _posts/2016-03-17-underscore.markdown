---
layout: post
category: 测试
---

# 1. _.each

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

1. 首先执行“optimizeCb()”函数，得到迭代器iteratee的值。语句：

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

2. 判断传入的对象obj是否为数组或对象,语句：

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
2) 若obj为对象a. 首先，检索object拥有的所有可枚举属性的名称，并返回数组keys。
		
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

3. 最后返回obj本身return obj;

# 2.  _.map

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

1. 首先执行“cb()”函数，得到迭代器iteratee的值。语句：

```
iteratee = cb(iteratee, context);
```
即如下图函数所示：

```
var cb = function(value, context, argCount) {
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

2. 判断obj是对象还是数组，若是对象，检索object拥有的所有可枚举属性的名称，并返回数组keys。3. 使用for循环，遍历keys（index）并返回它们的值（obj的属性或数组值），并求出三个值：obj[keys[i]／i]（obj对象属性（数组索引）对应的值）、keys[i]／i（obj对象对应的属性名称（数组对应的索引））、obj（obj对象（数组）列表本身），传进迭代器iteratee中，并运行iteratee函数后，返回新的数组列表值。

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

# 3.  _.reduce和_.reduceRight_.reduce(list, iterator, [memo], [context])
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

1. 判断是否有传memo的值，并得到initial的布尔值。
2. 运行optimizeCb()函数，且argCount传入的参数值为4，故：
iteratee得到的值为：

```
iteratee = function(accumulator,value, index, collection) {	return func.call(context, accumulator ,value, index, collection);};```

3. 运行reducer()函数，并得到最新的memo值，并返回
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

＃ 4. _.find

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

1. 若obj为数组，则执行函数_.findIndex()，此函数类似于_.indexOf，当predicate通过真检查时，返回第一个索引值；否则返回-1。
2. 若obj为对象，则执行函数_.findKey()，此函数类似于_.findIndex，但是检测objects的属性，当predicate通过真检查时，返回属性名称；否则返回undefined。
3. 若key存在且不等于-1，则返回对象中当前key（属性或索引）的值
4. 其中：
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
predicate=optimizeCb(value,context);			=function(value, index, collection) {      			return func.call(context, value, index, collection);   		};
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

```predicate=optimizeCb(value,context);			=function(value, index, collection) {      			return func.call(context, value, index, collection);   		};
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

# 5. _.filter

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

1. 运行cb()函数，并得到迭代函数predicate()的值:

```
predicate=optimizeCb(value,context);			=function(value, index, collection) {      			return func.call(context, value, index, collection);   		};
```

2. 运行_.each()函数，遍历obj对象，并判断是否满足迭代函数predicate的条件，满足，则把相应的值推进数组results中，不满足，则继续遍历，直到遍历完成为止，并返回results数组。
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

# 6. _.reject

_.reject(list, predicate, [context])
返回list中没有通过predicate真值检测的元素集合，与filter相反

```
_.reject = function(obj, predicate, context) {
  // negate_.negate(predicate) (对立面；反面)
  // 返回一个新的predicate函数的否定版本。
  return _.filter(obj, _.negate(cb(predicate)), context);
};
```

1. 运行cb()函数，并得到迭代函数predicate()的值:

```html
predicate=optimizeCb(value,context);			=function(value, index, collection) {      			return func.call(context, value, index, collection);   		};
```

2. 运行_.negate()函数，通过！非运算符，返回一个新的迭代函数predicate()的否定版本。

```javaScript
_.negate = function(predicate) {
  return function() {
    return !predicate.apply(this, arguments);
  };
};
```

例：

```js
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

```python
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
```
			




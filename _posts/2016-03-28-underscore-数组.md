---
layout: post
category: js源码解析
---

# 数组(Arrays)

## 1. _.first

_.first(array, [n]) Alias: head, take 
返回array（数组）的第一个元素。传递 n参数将返回数组中从第一个元素开始的n个元素（愚人码头注：返回数组中前 n 个元素.）。

```
_.first = _.head = _.take = function(array, n, guard) {
  if (array == null) return void 0;
  if (n == null || guard) return array[0];
  return _.initial(array, array.length - n);
};
```

1、 判断array为空时，返回void 0。

2 判断n为null或存在guard时，返回数组的第一个元素值。guard的存在是为了防止用户继续传入了第三个参数或以上无意义的参数。例：

“ _.map([[1, 2], [3, 4]], _.first)”

运行 _.map()函数时，将会有三个函数传进 _.first()函数：value，index，list。

3、 判断array存在且n存在时，运行_.initial()函数，返回数组0～（array.length - n）的元素值。

例：

```
var list = [3,5,7,2,8,2];
var test = _.first(list);
console.log(test);//3

var test2 = _.first(list,3);
console.log(test2);//[3, 5, 7]

var test3 = _.first(list,3,5);
console.log(test3);//3
```

## 2. _.initial

_.initial(array, [n]) 
返回数组中除了最后一个元素外的其他全部元素。 在arguments对象上特别有用。传递 n参数将从结果中排除从最后一个开始的n个元素（愚人码头注：排除数组后面的 n 个元素）。

```
_.initial = function(array, n, guard) {
  return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
};
```

1、 若n不存在，则运行slice.call()函数，返回一个0～（array.length - 1）的数组。

2、 若n存在，则运行slice.call()函数，返回一个0～（array.length - n）的数组。

例：

```
var list = [3,5,7,2,8,2];
var test = _.initial(list);
console.log(test);//[3, 5, 7, 2, 8]

var test2 = _.initial(list,2);
console.log(test2);//[3, 5, 7, 2]
```

## 3. _.last

_.last(array, [n]) 
返回array（数组）的最后一个元素。传递 n参数将返回数组中从最后一个元素开始的n个元素（愚人码头注：返回数组里的后面的n个元素）。

```
_.last = function(array, n, guard) {
  if (array == null) return void 0;
  if (n == null || guard) return array[array.length - 1];
  return _.rest(array, Math.max(0, array.length - n));
};
```

1、 判断array为空时，返回void 0。

2、 判断n为null或存在guard时，返回数组的最后一个元素值。

3、 判断array存在且n存在时，运行_.rest()函数，返回数组（array.length - n）～array.length的元素值。

例：

```
var list = [3,5,7,2,8,2];
var test = _.last(list);
console.log(test);//2

var test2 = _.last(list,2);
console.log(test2);//[8, 2]
```

## 4. _.rest

_.rest(array, [index]) Alias: tail, drop 
返回数组中除了第一个元素外的其他全部元素。传递 index 参数将返回从index开始的剩余所有元素 。

```
_.rest = _.tail = _.drop = function(array, n, guard) {
  return slice.call(array, n == null || guard ? 1 : n);
};
```

1、 若n不存在，则运行slice.call()函数，返回一个1～array.length的数组。

2、 若n存在，则运行slice.call()函数，返回一个n～array.length的数组。

例：

```
var list = [3,5,7,2,8,2];
var test = _.rest(list);
console.log(test);//[5, 7, 2, 8, 2]

var test2 = _.rest(list,2);
console.log(test2);//[7, 2, 8, 2]
```

## 5. _.compact

_.compact(array) 
返回一个除去所有false值的 array副本。 在javascript中, false, null, 0, "", undefined 和 NaN 都是false值.

```
_.compact = function(array) {
  return _.filter(array);
};
```

1、 根据上面的代码，可以认为 _.compact()与 _.filter()函数等价，即：

```
_.compact = _.filter(array);
			 ＝function(obj, predicate, context) {
  var results = [];
  predicate = cb(predicate, context);
  _.each(obj, function(value, index, list) {
    if (predicate(value, index, list)) results.push(value);
  });
  return results;
};
```

2、 运行 _.filter()函数，得到predicate= _.identity

3、 运行 _.each()函数，遍历array并返回array中的真值。

例：

```
var list = [0, 1, false, 2, '', 3, undefined , NaN, 'NaN', 'undefined'];
var test = _.compact(list);
console.log(test);//[1, 2, 3, 'NaN', 'undefined']
```

## 6. _.flatten

_.flatten(array, [shallow]) 
将一个嵌套多层的数组 array（数组） (嵌套可以是任何层数)转换为只有一层的数组。 如果你传递 shallow参数，数组将只减少一维的嵌套。

```
var flatten = function(input, shallow, strict, output) {
  output = output || [];
  var idx = output.length;
  for (var i = 0, length = getLength(input); i < length; i++) {
    // _.isArguments(object) 
    // 如果object是一个参数对象，返回true。
    var value = input[i];
    if (isArrayLike(value) && (_.isArray(value) || _.isArguments(value))) {
      // Flatten current level of array or arguments object.
      if (shallow) {
        var j = 0, len = value.length;
        while (j < len) output[idx++] = value[j++];
      } else {
        flatten(value, shallow, strict, output);
        idx = output.length;
      }
    } else if (!strict) {
      output[idx++] = value;
    }
  }
  return output;
};
```

```
_.flatten = function(array, shallow) {
  return flatten(array, shallow, false);
};
```

1、 运行flatten()函数。并传入三个参数：array，shallow，false。

2、 由于没有传入第四个参数，故output=[]，idx=output.length。

3、 运行for循环函数，value等于数组input[i]中的值。

4、 判断value为类数组对象且value为数组或者为参数对象。

a)若是，则判断shallow是否为真，若为真，则运行while函数，把数组的下一级中的所有元素都赋值给output，并结束运行flatten()函数，返回output值；若为假，则继续运行flatten()函数，直到output称为一维数组为止。

b)若否，则判断第三个参数strict为否，则直接把value的值赋给output。即，若value的值为对象时，直接把对象赋值给output。

例：

```
var list = [1, [2], [3, [[4]]]];
var test = _.flatten(list);
console.log(test);//[1, 2, 3, 4]

var test2 = _.flatten(list,true);
console.log(test2);//[1, 2, 3, [[4]]]
```

```
var list3 = [5, 6, 7, 8];
var test3 = _.flatten(list3);
console.log(test3);//[5, 6, 7, 8]
```

```
var list4 = [
	{one:'a',two:'b'}, 
	6, 
	{three:'c'}, 
	8 , 
	[1,5,3],
	[3,[5]]
];
var test4 = _.flatten(list4);
console.log(test4);//[{one:'a',two:'b'}, 6, {three:'c'}, 8, 1, 5, 3, 3, 5]
```

## 7. _.without

_.without(array, *values) 
返回一个删除所有values值后的 array副本。（愚人码头注：使用===表达式做相等测试。）

```
_.without = restArgs(function(array, otherArrays) {
  // _.difference(array, *others) 
  // 返回的值来自array参数数组，并且不存在于other 数组.
  return _.difference(array, otherArrays);
});
```

1、 运行restArgs()函数，并运行里面的function()函数。

2、 运行_.difference()函数，返回array参数中不存在otherArrays参数中的数组。

例：

```
var list = [1, 2, 1, 0, 3, 1, 4];
var test = _.without(list, 0, 1);
console.log(test);//[2, 3, 4]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
	five:'e'
};
var test2 = _.without(list2, 'b', 'd');
console.log(test2);//["a", "c", "e"]
```

## 8. _.uniq

_.uniq(array, [isSorted], [iteratee]) Alias: unique 
返回 array去重后的副本, 使用 === 做相等测试. 如果您确定 array 已经排序, 那么给 isSorted 参数传递 true值, 此函数将运行的更快的算法. 如果要处理对象元素, 传递 iteratee函数来获取要对比的属性.

```
_.uniq = _.unique = function(array, isSorted, iteratee, context) {
  // _.isBoolean(object) 
  // 如果object是一个布尔值，返回true，否则返回false。
  if (!_.isBoolean(isSorted)) {
    context = iteratee;
    iteratee = isSorted;
    isSorted = false;
  }
  if (iteratee != null) iteratee = cb(iteratee, context);
  var result = [];
  var seen = [];
  for (var i = 0, length = getLength(array); i < length; i++) {
    var value = array[i],
        computed = iteratee ? iteratee(value, i, array) : value;
    if (isSorted) {
      if (!i || seen !== computed) result.push(value);
      seen = computed;
    } else if (iteratee) {
      // _.contains(list, value, [fromIndex]) 
      // 如果list包含指定的value则返回true（
      // 如果list 是数组，内部使用indexOf判断。
      // 使用fromIndex来给定开始检索的索引位置。
      if (!_.contains(seen, computed)) {
        seen.push(computed);
        result.push(value);
      }
    } else if (!_.contains(result, value)) {
      result.push(value);
    }
  }
  return result;
};
```

1、 判断isSorted若不为布尔值，则把迭代函数iteratee赋值给context，isSorted赋值给iteratee，isSorted的值为false。这是为了防止用户习惯性把迭代函数iteratee作为第二个参数，把context作为第三个参数传进函数。

2、 若iteratee不为空，则运行cb()函数。

3、 运行for函数，遍历数组。

4、 若存在iteratee函数，则先把数组传入iteratee函数，运行完成后赋值给computed，若不存在，则直接把数组值赋值给computed。

5、 若isSorted为真，!i 判断i为0时为真，其他数组为假。即，i为0时一定运行push函数。其他数字则通过判断seen是否不等于computed时，运行push函数。最后把computed赋值给seen。但运行此段代码的前提是，已知array是按照顺序排列的。

6、 若isSorted为假，则判断iteratee函数是否存在，若存在，则运行 _.contains()函数，若seen不包含指定的computed值，则运行push函数。

7、 若不存在iteratee函数，则运行 _.contains()函数，若result不包含指定的value值，则运行push函数。

8、 最后返回result的值。

例：

```
var list = [1, 2, 1, 3, 1, 4];
var test = _.uniq(list);
console.log(test);//[1, 2, 3, 4]
```

```
var list2 = [1, 2, 3, 3, 4, 5, 5, 6, 6];
var test2 = _.uniq(list2,true);
console.log(test2);//[1, 2, 3, 4, 5, 6]

```

## 9. _.union

_.union(*arrays) 
返回传入的 arrays（数组）并集：按顺序返回，返回数组的元素是唯一的，可以传入一个或多个 arrays（数组）。

```
_.union = restArgs(function(arrays) {
  return _.uniq(flatten(arrays, true, true));
});
```

1、 运行restArgs()函数，并运行里面的function()函数。

2、 运行flatten()函数，传入数组arrays，并减少一维嵌套

3、 运行_.uniq()函数,把得到的array数组进行唯一值筛选，并返回最终值。

例：

```
var list = [1, 2, 3];
var list2 = [101, 2, 1, 10];
var list3 = [2, 1];
var test = _.union(list, list2, list3);
console.log(test);//[1, 2, 3, 101, 10]
```

```
var list5 = [1, [5,8], 3];
var list6 = [101, 2, 1, 10];
var list7 = [2, [1,7]];
var test2 = _.union(list5, list6, list7);
console.log(test2);//[1, [5,8], 3, 101, 2, 10, [1,7] ]
```

```
var list8 = [1, 2, 3];
var list9 = {one:'a'};
var list10 = [2, 1,7];
var test3 = _.union(list8, list9, list10);
console.log(test3);//[1, 2 , 3 ,7 ]
```

## 10. _.intersection

_.intersection(*arrays) 
返回传入 arrays（数组）交集。结果中的每个值是存在于传入的每个arrays（数组）里。

```
_.intersection = function(array) {
  var result = [];
  var argsLength = arguments.length;
  for (var i = 0, length = getLength(array); i < length; i++) {
    // _.contains(list, value, [fromIndex]) 
    // 如果list包含指定的value则返回true（
    // 如果list 是数组，内部使用indexOf判断。
    // 使用fromIndex来给定开始检索的索引位置。
    var item = array[i];
    if (_.contains(result, item)) continue;
    var j;
    for (j = 1; j < argsLength; j++) {
      if (!_.contains(arguments[j], item)) break;
    }
    if (j === argsLength) result.push(item);
  }
  return result;
};
```

1、 先读取arguments参数数组的长度，并赋值给argsLength。

2、 运行for函数，把每个传入的array的值赋值给item。

3、 运行 _.contains()函数，若result包含指定的item值，则继续运行运行第一层for循环。

4、 若result不包含指定的item值，运行第二层for函数，运行
_.contains()函数，若array中其中一个item值不存在于arguments[j]，则跳出第二层for循环。即证明item不是交集中的数据。

5、 若j完全与argsLength相等，则把item的值push到result中，即item的值都存在于其他的数组中。

例：

```
var list = [1, 2, 3];
var list2 = [101, 2, 1, 10];
var list3 = [2, 1];
var test = _.intersection(list, list2, list3);
console.log(test);//[1, 2]
```


## 11. _.difference

_.difference(array, *others) 
类似于without，但返回的值来自array参数数组，并且不存在于other 数组.

```
_.difference = restArgs(function(array, rest) {
  rest = flatten(rest, true, true);
  return _.filter(array, function(value){
    return !_.contains(rest, value);
  });
});
```

1、 运行restAargs()函数，把others参数变为数组传进rest中。

2、 运行flatten()函数，把rest中的各个数组变成一维数组，并且值返回类型为数组的值。

3、 运行_.filter()函数和 _.contains()函数，把value不包含于rest的值组成一个数组并返回得到的值。

例：

```
var list = [1, 2, 3, 4, 5];
var test = _.difference(list, [5, 2, 10]);
console.log(test);//[1, 3, 4]
```

```
var list2 = {
	one:'a',
	two:'b',
	three:'c',
	four:'d',
	five:'e'
};
var test2 = _.difference(list2, ['b'], ['d']);
console.log(test2);//["a", "c", "e"]
```

## 12. _.unzip

_.unzip(arrays) 
与zip功能相反的函数，给定若干arrays，返回一串联的新数组，其第一元素个包含所有的输入数组的第一元素，其第二包含了所有的第二元素，依此类推。通过apply用于传递数组的数组。 

```
_.unzip = function(array) {
  var length = array && _.max(array, getLength).length || 0;
  var result = Array(length);

  for (var index = 0; index < length; index++) {
  	  // _.pluck(list, propertyName) 
    // pluck也许是map最常使用的用例模型的简化版本，
    // 即萃取数组对象中某属性值，返回一个数组。
    result[index] = _.pluck(array, index);
  }
  return result;
};
```

1、 运行_.max()函数，并找出array中子数组的最大长度，赋值给length。

a) 其中， _.max()函数中的iteratee函数为：

```
var property = function(key) {
  return function(obj) {
    return obj == null ? void 0 : obj[key];
  };
};
var getLength = property('length');
iteratee = getLength;
```

b) _.max()函数将运行下面的函数求出最大值：

```
_.each(obj, function(v, index, list) {
 computed = iteratee(v, index, list);
 if (computed > lastComputed || computed === -Infinity && result === -Infinity) {
   result = v;
   lastComputed = computed;
 }
});
```

2、 运行for循环函数，并运行_.pluck()函数，得到array数组中各子数组的相应index的值，并赋值给result[index]。

3、 返回result的值。

例：

```
var list = [
	["moe", 30, true], 
	["larry", 40, false], 
	["curly", 50, false]
];
var test = _.unzip(list);
console.log(test);
/*
[
	['moe', 'larry', 'curly'], 
	[30, 40, 50], 
	[true, false, false]
]
*/
```

```
var list2 = [
	['moe', 'larry', 'curly'],
	[30, 40, 50],
	[true, false, false]
];
var test2 = _.unzip(list2);
console.log(test2);
/*
[
	["moe", 30, true], 
	["larry", 40, false], 
	["curly", 50, false]
]
*/
```

```
var list3 = [
	["moe", 30, true], 
	["larry", 40, false,'run'], 
	["curly", 50, false]
];
var test3 = _.unzip(list3);
console.log(test3);
/*
[
	['moe', 'larry', 'curly'], 
	[30, 40, 50], 
	[true, false, false],
	[undefined, 'run', undefined]
]
*/
```

## 13. _.zip

_.zip(*arrays) 
将 每个arrays中相应位置的值合并在一起。在合并分开保存的数据时很有用. 如果你用来处理矩阵嵌套数组时, _.zip.apply 可以做类似的效果。

```
_.zip = restArgs(_.unzip);
```

1、 先运行restArgs()函数，把多个数组合并为一个数组。

2、 运行_.unzip函数。得到result结果。

3、 _.zip()函数与 _.unzip()函数的最大区别是： _.zip()函数能传入的多个参数。 _.unzip()函数只能传入一个参数。

例：

```
var list = ['moe', 'larry', 'curly'];
var list2 = [30, 40, 50];
var list3 = [true, false, false];
var test = _.zip(list,list2,list3);
console.log(test);
/*
[
	["moe", 30, true], 
	["larry", 40, false], 
	["curly", 50, false]
]
*/
```

```
var list4 = ["moe", 30, true];
var list5 = ["larry", 40, false]; 
var list6 = ["curly", 50, false];
var test2 = _.zip(list4,list5,list6);
console.log(test2);
/*
[
	['moe', 'larry', 'curly'], 
	[30, 40, 50], 
	[true, false, false]
]
*/
```

## 14. _.object

_.object(list, [values]) 
将数组转换为对象。传递任何一个单独[key, value]对的列表，或者一个键的列表和一个值得列表。 如果存在重复键，最后一个值将被返回。

```
_.object = function(list, values) {
  var result = {};
  for (var i = 0, length = getLength(list); i < length; i++) {
    if (values) {
      result[list[i]] = values[i];
    } else {
      result[list[i][0]] = list[i][1];
    }
  }
  return result;
};
```

1、 运行getLength函数，得到list的长度。

2、 运行for函数，且运行次数为list的长度值。

3、 判断是否存在values：

a) 若存在，则把values[i]的值赋值给result[list[i]]。其中list[i]为result的key，values[i]为result相应key的值。

例：

```
var list1 = ['moe', 'larry', 'curly'];
var list2 = [30, 40, 50,60];
var test = _.object(list1,list2);
console.log(test);
/*
{moe: 30, larry: 40, curly: 50}
```

b) 若不存在，则把二维数组中，list[i][1]的值赋值给result[list[i][0]]。其中list[i][0]为result的key，list[i][1]为result相应key的值。

例：

```
var list3 = [
	['moe', 30],
	['larry', 40],
	['curly', 50]
];
var test2 = _.object(list3);
console.log(test2);
/*
{moe: 30, larry: 40, curly: 50}
*/
```

## 15. _.findIndex

_.findIndex(array, predicate, [context]) 
类似于 _.indexOf，当predicate通过真检查时，返回第一个索引值；否则返回-1。

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

```
_.findIndex = createPredicateIndexFinder(1);
```

1、 运行createPredicateIndexFinder()函数，并传入参数1。

2、 运行function，并传入参数array、predicate、context。

3、 得到迭代函数predicate的值：

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

4、 index从0开始进行for循环，当满足迭代函数predicate的条件时，直接返回index（索引位置），若遍历完列表后，没有满足迭代函数predicate条件，则返回-1。

例：

```
var list = [4, 6, 8, 12];
var test = _.findIndex(list,function(value){
	return value == 7;
});
console.log(test);//-1 // not found
```

```
var list2 = [4, 6, 7, 12];
var test2 = _.findIndex(list2,function(value){
	return value == 7;
});
console.log(test2);//2
```

```
var list = [
 	{'id': 1, 'name': 'Bob', 'last': 'Brown'},
 	{'id': 2, 'name': 'Ted', 'last': 'White'},
 	{'id': 3, 'name': 'Frank', 'last': 'James'},
 	{'id': 4, 'name': 'Ted', 'last': 'Jones'}
];
var test = _.findIndex(list,{
  	name: 'Ted'
});
console.log(test);//1
```

## 16. _.findLastIndex

_.findLastIndex(array, predicate, [context]) 
和 _.findIndex类似，但反向迭代数组，当predicate通过真检查时，最接近末端的索引值将被返回。

```
_.findLastIndex = createPredicateIndexFinder(-1);
```

1、 实现方法与 _.findIndex()函数一样，先运行createPredicateIndexFinder()函数，唯一的不同是传入了参数-1。其参数影响了for循环时是从左往右，或从右往左遍历数组。

例：

```
var list = [
 	{'id': 1, 'name': 'Bob', 'last': 'Brown'},
 	{'id': 2, 'name': 'Ted', 'last': 'White'},
 	{'id': 3, 'name': 'Frank', 'last': 'James'},
 	{'id': 4, 'name': 'Ted', 'last': 'Jones'}
];
var test = _.findLastIndex(list,{
  	name: 'Ted'
});
console.log(test);//3
```

```
var list2 = [
	{age:34,name:'aa'},
	{age:56,name:'bb'},
	{age:73,name:'cc'},
	{age:42,name:'dd'},
];
var test2 = _.findLastIndex(list2,function (value){
	return value.age == 73;
});
console.log(test2);//2
```

## 17. _.sortedIndex

_.sortedIndex(list, value, [iteratee], [context]) 
使用二分查找确定value在list中的位置序号，value按此序号插入能保持list原有的排序。如果提供iterator函数，iterator将作为list排序的依据，包括你传递的value 。iterator也可以是字符串的属性名用来排序(比如length)。

```
_.sortedIndex = function(array, obj, iteratee, context) {
  iteratee = cb(iteratee, context, 1);
  var value = iteratee(obj);
  var low = 0, high = getLength(array);
  while (low < high) {
  	  //math.floor(x)返回小于参数x的最大整数,即对浮点数向下取整
    var mid = Math.floor((low + high) / 2);
    if (iteratee(array[mid]) < value) low = mid + 1; else high = mid;
  }
  return low;
};
```

1、 运行cb()函数

a) 由于cb中的第三个参数argCount为1 ，故若iteratee为函数时，得到iteratee的值为：

```
iteratee = cb(iteratee, context, 1)
			 = function(value) {	return func.call(context, value);};
```

b) 若iteratee为字符串，则

```
iteratee=_.property(value);
```

2、 得到value的值，并定义low索引最小值与high索引最大值。

3、 运行Math.floor()函数向下取整，二分法求的索引的中间值mid。

4、 判断索引中间值所在的array位置中iteratee(array[mid])的值是否小于value的值，若是，则low赋值为mid+1；若否，则high赋值为mid。切通过while()函数实现循环，知道满足条件low < high为止。并返回low的值。

例：

```
var list = [10, 20, 30, 40, 50];
var test = _.sortedIndex(list,35);
console.log(test);//3
```

```
var list2 =  [
	[2,4,6], 
	[6,2]
];
var test2 = _.sortedIndex(list2,[7,8], function(value){
	return value[0];
});
console.log(test2);//2

var test3 = _.sortedIndex(list2,[5,8]);
console.log(test3);//1

var test4 = _.sortedIndex(list2,[9,1],1);
console.log(test4);//0
```

```
var list3 =  [
	{name: 'moe', age: 40}, 
	{name: 'curly', age: 60}
];
var test5 = _.sortedIndex(list3,{name: 'larry', age: 50}, 'age');
console.log(test5);//1

var test6 = _.sortedIndex(list3,{name: 'larry', age: 50}, function(value){
	return value.age;
});
console.log(test6);//1
```


## 18. _.indexOf

_.indexOf(array, value, [isSorted]) 
返回value在该 array 中的索引值，如果value不存在 array中就返回-1。使用原生的indexOf 函数，除非它失效。如果您正在使用一个大数组，你知道数组已经排序，传递true给isSorted将更快的用二进制搜索..,或者，传递一个数字作为第三个参数，为了在给定的索引的数组中寻找第一个匹配值。

```
var createIndexFinder = function(dir, predicateFind, sortedIndex) {
  return function(array, item, idx) {
    var i = 0, length = getLength(array);
    if (typeof idx == 'number') {
      if (dir > 0) {
        i = idx >= 0 ? idx : Math.max(idx + length, i);
      } else {
        length = idx >= 0 ? Math.min(idx + 1, length) : idx + length + 1;
      }
    } else if (sortedIndex && idx && length) {
      idx = sortedIndex(array, item);
      return array[idx] === item ? idx : -1;
    }
    if (item !== item) {
      // _.isNaN(object) 
      // 如果object是 NaN，返回true。 
      // 注意： 这和原生的isNaN 函数不一样，
      // 如果变量是undefined，原生的isNaN 函数也会返回 true 。
      // Array.prototype.slice.call(arguments)能将具有length属性的对象转成数组
      // 请使用 isNaN() 来判断一个值是否是数字。原因是 NaN 与所有值都不相等，包括它自己。
      idx = predicateFind(slice.call(array, i, length), _.isNaN);
      return idx >= 0 ? idx + i : -1;
    }
    for (idx = dir > 0 ? i : length - 1; idx >= 0 && idx < length; idx += dir) {
      if (array[idx] === item) return idx;
    }
    return -1;
  };
};
```

```
_.indexOf = createIndexFinder(1, _.findIndex, _.sortedIndex);
```

1、 运行createIndexFinder()函数。传入三个参数：1、_.findIndex、 _.sortedIndex。

2、 运行function函数，传入三个参数：array(array), value(item), isSorted(idx)。

3、 判断isSorted(idx)参数是否传入数字类型的值。

a) 若是，在这里dir为1大于0，判断idx大于等于0时，i的值为idx，若idx小于0，则取idx加上array的长度，与i的最大值。如idx为-3，length为6 ，i原始值为0 ，则i=3。

b) 若否，则判断isSorted(idx)参数是否传入true布尔变，且sortedIndex的值是否存在。若是，则运行sortedIndex()函数，用二分法的方式，把idx找出来，并判断array[idx]的值是否完全等于value(item)的值，若是，直接结束函数的运行，并返回idx的值，若否，则结束函数的运行，并返回-1值。

c) isSorted(idx)参数的传入，主要作用是运用了二分法，节省查找时间。

4、 若函数继续运行，则判断item是否为NaN，因为NaN与所有值都不相等，包括它自己。只能使用isNaN()来判断一个值是否是数字。然后运行predicateFind()(_.findIndex)函数。若满足 _.isNaN()函数条件，则把得到的索引值传给idx，若不满足，则idx等于-1，并结束函数的运行，并返回idx的值。

5、 若函数继续运行，则运行for函数，并通过条件array[idx] === item，找出idx，并返回。若不存在idx，则返回-1。

例：

```
var list = [1, 2, 3];
var test = _.indexOf(list,2);
console.log(test);//1
```

```
var list2 = [1, 2, 3, 4, 5, 6];
var test2 = _.indexOf(list2,2,3);
console.log(test2);//-1

var test3 = _.indexOf(list2,2,-5);
console.log(test3);//1

var test4 = _.indexOf(list2,4,true);
console.log(test4);//3
```

```
var list5 = {
	'0':'a',
	'1':'b',
	'2':'c',
	'3':'d',
	'4':'e',
	length:'4'
};
var test5 = _.indexOf(list5,'d');
console.log(test5);//3
```

```
var list6 = [1,5,6,undefined,2,6,NaN,2,5];
var test6 = _.indexOf(list6,NaN);
console.log(test6);//6

var test7 = _.indexOf(list6,undefined);
console.log(test7);//3
```

## 19. _.lastIndexOf

_.lastIndexOf(array, value, [fromIndex]) 
返回value在该 array 中的从最后开始的索引值，如果value不存在 array中就返回-1。如果支持原生的lastIndexOf，将使用原生的lastIndexOf函数。传递fromIndex将从你给定的索性值开始搜索。

```
_.lastIndexOf = createIndexFinder(-1, _.findLastIndex);
```

1、 运行createIndexFinder()函数。传入辆个参数：-1、_.findLastIndex。

2、 运行function函数，传入三个参数：array(array), value(item), fromIndex(idx)。

3、 判断fromIndex(idx)参数是否传入数字类型的值。

a) 若是，在这里dir为-1小于0，判断idx大于等于0时，i的值为idx+1，与array的长度的最小值。如idx为3，length为6 ，则i=4。若idx小于0，则i的值为idx+length+1。如idx为-4，length为6 ，则i=3。

b) 若否，这里与 _.indexOf不同，尽管fromIndex(idx)参数传入true布尔变量，当时sortedIndex不存在，故不使用二分法。

c) fromIndex(idx)参数的传入，主要作用是从你给定的索性值开始搜索，节省查找时间。

4、 若函数继续运行，则判断item是否为NaN，因为NaN与所有值都不相等，包括它自己。只能使用isNaN()来判断一个值是否是数字。然后运行predicateFind()(_.findLastIndex)函数。若满足 _.isNaN()函数条件，则把得到的索引值传给idx，若不满足，则idx等于-1，并结束函数的运行，并返回idx的值。

5、 若函数继续运行，则运行for函数，并通过条件array[idx] === item，找出idx，并返回。若不存在idx，则返回-1。

例：

```
var list = [1, 2, 3, 1, 2, 3];
var test = _.lastIndexOf(list,2);
console.log(test);//4

console.log('----------------------------------');

var test2 = _.lastIndexOf(list,2,true);
console.log(test2);//4
```

```
var list2 = [1, 2, 3, 4, 2, 6];
var test3 = _.lastIndexOf(list2,2,3);
console.log(test3);//1
```

```
var list3 = [1, 2, 3, 4, 4, 6];
var test4 = _.lastIndexOf(list3,4,-3);
console.log(test4);//3
```

## 20. _.range

_.range([start], stop, [step]) 
一个用来创建整数灵活编号的列表的函数，便于each 和 map循环。如果省略start则默认为 0；step 默认为 1.返回一个从start 到stop的整数的列表，用step来增加 （或减少）独占。值得注意的是，如果stop值在start前面（也就是stop值小于start值），那么值域为负增长。

```
_.range = function(start, stop, step) {
  if (stop == null) {
    stop = start || 0;
    start = 0;
  }
  if (!step) {
    step = stop < start ? -1 : 1;
  }
	// math.ceil(x)返回大于参数x的最小整数,即对浮点数向上取整.
  var length = Math.max(Math.ceil((stop - start) / step), 0);
  var range = Array(length);

  for (var idx = 0; idx < length; idx++, start += step) {
    range[idx] = start;
  }

  return range;
};
```

1、 运行function()函数，并传入三个参数：start、stop、step。

2、 判断第二个参数stop为空时，把第一个参数start或0赋值给stop，把0赋值给start。即相当于用户没有传入start的值，start的默认值为0。

3、 判断第三个参数没有传入，则若stop小于start，则step的值为-1 ，若stop大于等于start，则step的值为1。

4、 运行Math.ceil()函数，求出用户要求需要返回的值的长度，并与0比较，运行Math.max()函数，求出长度的最大值。

5、 运行for函数，start=start+step，并把start传递给range[idx]，最后返回range。

例：

```
var test = _.range(10);
console.log(test);
//[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

console.log('-----------------------');

var test = _.range(1, 11);
console.log(test);
//[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

console.log('-----------------------');

var test = _.range(0, 30, 5);
console.log(test);
//[0, 5, 10, 15, 20, 25]

console.log('-----------------------');

var test = _.range(0, -10, -1);
console.log(test);
//[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]

console.log('-----------------------');

var test = _.range(3, 0);
console.log(test);
//[3, 2, 1]

console.log('-----------------------');

var test = _.range(-1,-5);
console.log(test);
//[-1, -2, -3, -4]

console.log('-----------------------');

var test = _.range(0);
console.log(test);
//[]
```

## 21.  _.chunk

_.chunk(array,count)
传入数组array，并按照count传入的数字，每count个为一组，组成一个新的二维数组。 若不传count，或count小于1，则得到一个空的数组。

```
_.chunk = function(array, count) {
  if (count == null || count < 1) return [];

  var result = [];
  var i = 0, length = array.length;
  while (i < length) {
    result.push(slice.call(array, i, i += count));
  }
  return result;
};
```

1、 判断count是否为空，或是否小于1，则返回空数组。

2、 运行while()循环函数，当满足条件i < length时，运行slice.call()函数，并从i开始，至i+count的数组都push，到result中，且更新i的值。

3、 最后把返回result。

例：

```
var list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
var test = _.chunk(list,4);
console.log(test);
/*
[
	[0, 1, 2, 3], 
	[4, 5, 6, 7], 
	[8, 9]
]
*/

var test3 = _.chunk(list,-1);
console.log(test3);//[]
```

```
var list2 = [
	{name:'a'},
	{name:'b'},
	{name:'c'},
	{name:'d'},
	{name:'e'},
	{name:'f'},
	{name:'g'},
	{name:'h'},
	{name:'i'},
	{name:'j'},
];
var test2 = _.chunk(list2,3);
console.log(test2);
/*
[
	[{name:'a'},{name:'b'},{name:'c'}], 
	[{name:'d'},{name:'e'},{name:'f'}], 
	[{name:'g'},{name:'h'},{name:'i'}],
	[{name:'j'}]
]
*/
```




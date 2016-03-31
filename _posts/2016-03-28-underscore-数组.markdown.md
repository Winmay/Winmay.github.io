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

1. 判断array为空时，返回void 0。
2. 判断n为null或存在guard时，返回数组的第一个元素值。guard的存在是为了防止用户继续传入了第三个参数或以上无意义的参数。例：
	_.map([[1, 2], [3, 4]], _.first);
	运行 _.map()函数时，将会有三个函数传进 	_.first()函数：value，index，list。
3. 判断array存在且n存在时，运行_.initial()函数，返回数组0～（array.length - n）的元素值。
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

1. 若n不存在，则运行slice.call()函数，返回一个0～（array.length - 1）的数组。
2. 若n存在，则运行slice.call()函数，返回一个0～（array.length - n）的数组。
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

1. 判断array为空时，返回void 0。
2. 判断n为null或存在guard时，返回数组的最后一个元素值。
3. 判断array存在且n存在时，运行_.rest()函数，返回数组（array.length - n）～array.length的元素值。
例：

```
var list = [3,5,7,2,8,2];
var test = _.last(list);
console.log(test);//2

var test2 = _.last(list,2);
console.log(test2);//[8, 2]
```

## 3. _.rest

_.rest(array, [index]) Alias: tail, drop 
返回数组中除了第一个元素外的其他全部元素。传递 index 参数将返回从index开始的剩余所有元素 。

```
_.rest = _.tail = _.drop = function(array, n, guard) {
  return slice.call(array, n == null || guard ? 1 : n);
};
```

1. 若n不存在，则运行slice.call()函数，返回一个1～array.length的数组。
2. 若n存在，则运行slice.call()函数，返回一个n～array.length的数组。
例：

```
var list = [3,5,7,2,8,2];
var test = _.rest(list);
console.log(test);//[5, 7, 2, 8, 2]

var test2 = _.rest(list,2);
console.log(test2);//[7, 2, 8, 2]
```

## 4. _.compact

_.compact(array) 
返回一个除去所有false值的 array副本。 在javascript中, false, null, 0, "", undefined 和 NaN 都是false值.

```
_.compact = function(array) {
  return _.filter(array);
};
```

1. 根据上面的代码，可以认为 _.compact()与 _.filter()函数等价，即：

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

2. 运行_.filter()函数，得到predicate= _.identity
3. 运行_.each()函数，遍历array并返回array中的真值。
例：

```
var list = [0, 1, false, 2, '', 3, undefined , NaN, 'NaN', 'undefined'];
var test = _.compact(list);
console.log(test);//[1, 2, 3, 'NaN', 'undefined']
```

## 5. _.flatten

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

1. 运行flatten()函数。并传入三个参数：array，shallow，false。
2. 由于没有传入第四个参数，故output=[]，idx=output.length。
3. 运行for循环函数，value等于数组input[i]中的值。
4. 判断value为类数组对象且value为数组或者为参数对象。
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

## 6. _.without

_.without(array, *values) 
返回一个删除所有values值后的 array副本。（愚人码头注：使用===表达式做相等测试。）

```
_.without = restArgs(function(array, otherArrays) {
  // _.difference(array, *others) 
  // 返回的值来自array参数数组，并且不存在于other 数组.
  return _.difference(array, otherArrays);
});
```

1. 运行restArgs()函数，并运行里面的function()函数。
2. 运行_.difference()函数，返回array参数中不存在otherArrays参数中的数组。
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

## 7. _.uniq

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

1. 判断isSorted若不为布尔值，则把迭代函数iteratee赋值给context，isSorted赋值给iteratee，isSorted的值为false。这是为了防止用户习惯性把迭代函数iteratee作为第二个参数，把context作为第三个参数传进函数。
2. 若iteratee不为空，则运行cb()函数。
3. 运行for函数，遍历数组。
4. 若存在iteratee函数，则先把数组传入iteratee函数，运行完成后赋值给computed，若不存在，则直接把数组值赋值给computed。
5. 若isSorted为真，!i 判断i为0时为真，其他数组为假。即，i为0时一定运行push函数。其他数字则通过判断seen是否不等于computed时，运行push函数。最后把computed赋值给seen。但运行此段代码的前提是，已知array是按照顺序排列的。
6. 若isSorted为假，则判断iteratee函数是否存在，若存在，则运行_.contains()函数，若seen不包含指定的computed值，则运行push函数。
7. 若不存在iteratee函数，则运行_.contains()函数，若result不包含指定的value值，则运行push函数。
8. 最后返回result的值。
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

## 8. _.union

_.union(*arrays) 
返回传入的 arrays（数组）并集：按顺序返回，返回数组的元素是唯一的，可以传入一个或多个 arrays（数组）。

```
_.union = restArgs(function(arrays) {
  return _.uniq(flatten(arrays, true, true));
});
```

1. 运行restArgs()函数，并运行里面的function()函数。
2. 运行flatten()函数，传入数组arrays，并减少一维嵌套
3. 运行_.uniq()函数,把得到的array数组进行唯一值筛选，并返回最终值。
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

## 9. _.intersection

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

1. 先读取arguments参数数组的长度，并赋值给argsLength。
2. 运行for函数，把每个传入的array的值赋值给item。
3. 运行_.contains()函数，若result不包含指定的item值，则继续往下运行。
4. 继续运行for函数，运行_.contains()函数，若array中其中一个item值不存在arguments[j]，则跳出第一层for循环。


## 10. _.difference

_.difference(array, *others) 
类似于without，但返回的值来自array参数数组，并且不存在于other 数组.

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

```
_.difference = restArgs(function(array, rest) {
  rest = flatten(rest, true, true);
  return _.filter(array, function(value){
    return !_.contains(rest, value);
  });
});
```

## 11. _.unzip

_.unzip(*arrays) 
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


## 12. _.zip

_.zip(*arrays) 
将 每个arrays中相应位置的值合并在一起。在合并分开保存的数据时很有用. 如果你用来处理矩阵嵌套数组时, _.zip.apply 可以做类似的效果。

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

```
_.zip = restArgs(_.unzip);
```



## 13. _.object

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

## 14. _.findIndex

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

## 15. _.findLastIndex

_.findLastIndex(array, predicate, [context]) 
和 _.findIndex类似，但反向迭代数组，当predicate通过真检查时，最接近末端的索引值将被返回。

```
_.findLastIndex = createPredicateIndexFinder(-1);
```


## 16. _.indexOf

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




## 17. _.lastIndexOf

_.lastIndexOf(array, value, [fromIndex]) 
返回value在该 array 中的从最后开始的索引值，如果value不存在 array中就返回-1。如果支持原生的lastIndexOf，将使用原生的lastIndexOf函数。传递fromIndex将从你给定的索性值开始搜索。

```
_.lastIndexOf = createIndexFinder(-1, _.findLastIndex);
```


## 18. _.sortedIndex

_.sortedIndex(list, value, [iteratee], [context]) 
使用二分查找确定value在list中的位置序号，value按此序号插入能保持list原有的排序。如果提供iterator函数，iterator将作为list排序的依据，包括你传递的value 。iterator也可以是字符串的属性名用来排序(比如length)。

```
_.sortedIndex = function(array, obj, iteratee, context) {
  iteratee = cb(iteratee, context, 1);
  var value = iteratee(obj);
  var low = 0, high = getLength(array);
  while (low < high) {
    var mid = Math.floor((low + high) / 2);
    if (iteratee(array[mid]) < value) low = mid + 1; else high = mid;
  }
  return low;
};
```



## 19. _.range

_.range([start], stop, [step]) 
一个用来创建整数灵活编号的列表的函数，便于each 和 map循环。如果省略start则默认为 0；step 默认为 1.返回一个从start 到stop的整数的列表，用step来增加 （或减少）独占。值得注意的是，如果stop值在start前面（也就是stop值小于start值），那么值域会被认为是零长度，而不是负增长。-如果你要一个负数的值域 ，请使用负数step.

```
_.range = function(start, stop, step) {
  if (stop == null) {
    stop = start || 0;
    start = 0;
  }
  if (!step) {
    step = stop < start ? -1 : 1;
  }

  var length = Math.max(Math.ceil((stop - start) / step), 0);
  var range = Array(length);

  for (var idx = 0; idx < length; idx++, start += step) {
    range[idx] = start;
  }

  return range;
};
```

## 20.  _.chunk

_.chunk(array,[count])
返回一个初始化数组，传递count参数，

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


# 函数(Functions)



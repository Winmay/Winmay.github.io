---
layout: post
category: js源码解析
---

# 对象函数(Object Functions)

## 1. _.keys

_.keys(object) 
检索object拥有的所有可枚举属性的名称。

```
var ObjProto = Object.prototype;
// keys函数在小于IE9系统中不能使用`for key in ...`方法遍历对象属性。
var hasEnumBug = !{toString: null}.propertyIsEnumerable('toString');
var nonEnumerableProps = ['valueOf', 'isPrototypeOf', 'toString',
                    'propertyIsEnumerable', 'hasOwnProperty', 'toLocaleString'];

var collectNonEnumProps = function(obj, keys) {
  var nonEnumIdx = nonEnumerableProps.length;
  var constructor = obj.constructor;
  var proto = _.isFunction(constructor) && constructor.prototype || ObjProto;

  // Constructor is a special case.
  // 如果list包含指定的value则返回true（愚人码头注：使用===检测）。
  // 如果list 是数组，内部使用indexOf判断。使用fromIndex来给定开始检索的索引位置。
  // _.contains([1, 2, 3], 3); => true
  var prop = 'constructor';
  if (_.has(obj, prop) && !_.contains(keys, prop)) keys.push(prop);

  while (nonEnumIdx--) {
    prop = nonEnumerableProps[nonEnumIdx];
    if (prop in obj && obj[prop] !== proto[prop] && !_.contains(keys, prop)) {
      keys.push(prop);
    }
  }
};
```

```
var nativeKeys = Object.keys;
_.keys = function(obj) {
  if (!_.isObject(obj)) return [];
  if (nativeKeys) return nativeKeys(obj);
  var keys = [];
  // _.has(object, key) 对象是否包含给定的键吗？
  // 等同于object.hasOwnProperty(key)，
  // 但是使用hasOwnProperty 函数的一个安全引用，以防意外覆盖。
  for (var key in obj) if (_.has(obj, key)) keys.push(key);
  // Ahem, IE < 9.
  if (hasEnumBug) collectNonEnumProps(obj, keys);
  return keys;
};
```

1、 检测obj是否为对象类型，若不是，则返回空数组。

2、 判断是否存在函数nativeKeys(Object.keys)，若存在，则直接返回该对象obj的所有可枚举自身属性的属性名。

3、 若都不满足上面的两个条件，则继续往下运行。

4、 定义变量keys，用于存储空数组。

5、 运行for key in ...循环函数，判断若key存在于obj内，那么把key属性名push到keys中。

7、 为了兼容IE9以下的系统，则需要判段hasEnumBug是否为真。若为真，则运行collectNonEnumProps()函数。

注： 

1) 由于hasEnumBug等于函数{toString: null}.propertyIsEnumerable('toString')的假检测。

2) 而函数propertyIsEnumerable()的作用是用来检测属性是否属于某个对象的,如果检测到了,返回true,否则返回false。

3) 但函数propertyIsEnumerable()要返回true，还要满足两个条件：

a) 这个属性必须属于实例的,并且不属于原型. 

b) 这个属性必须是可枚举的,也就是自定义的属性,可以通过for..in循环出来的.

8、 由于IE9以下的系统不能通过for key in ...循环遍历对象属性。故hasEnumBug为true。

9、 运行collectNonEnumProps(obj, keys)函数。

1) 定义变量nonEnumIdx，用于存储nonEnumerableProps的长度。

2) 定义变量constructor，用于存储obj的构造函数obj.constructor。

3) 定义变量proto，用于存储对象的原型对象prototype。

4) 定义变量prop，用于存储需要运行的js原生函数名。

5) 当prop等于constructor时，判断prop是否存在与obj对象，且keys中包含props，则把prop的属性名push到keys中。

6） 运行while循环，从新赋值prop为nonEnumerableProps[nonEnumIdx]，并满足于判断句后，把prop的属性名push到keys中。

例：

```
var test = _.keys({one: 1, two: 2, three: 3});
console.log(test);
// ["one", "two", "three"]

console.log('--------------------------');

var test2 = _.keys({1: 'a', 2: 'b', 3: 'c', length:3});
console.log(test2);
// ["1", "2", "3", "length"]
```

## 2. _.allKeys

_.allKeys(object) 
检索object拥有的和继承的所有属性的名称。

```
_.allKeys = function(obj) {
  if (!_.isObject(obj)) return [];
  var keys = [];
  for (var key in obj) keys.push(key);
  // Ahem, IE < 9.
  if (hasEnumBug) collectNonEnumProps(obj, keys);
  return keys;
};
```

1、 检测obj是否为对象类型，若不是，则返回空数组。

2、 定义变量keys，用于存储空数组。

3、 运行for key in ...循环函数，把key属性名push到keys中。

4、 为了兼容IE9以下的系统，则需要判段hasEnumBug是否为真。若为真，则运行collectNonEnumProps()函数。

例：

```
function Stooge(name) {
  this.name = name;
}
Stooge.prototype.silly = true;
var test = _.allKeys(new Stooge("Moe"));
console.log(test);
// ["name", "silly"]
```

## 3. _.values

_.values(object)  
返回object对象所有的属性值。

```
_.values = function(obj) {
  var keys = _.keys(obj);
  var length = keys.length;
  var values = Array(length);
  for (var i = 0; i < length; i++) {
    values[i] = obj[keys[i]];
  }
  return values;
};
```

1、 运行_.keys()函数，把obj中得到的所有属性名称的集合赋值给keys。

2、 定义变量length，存储keys的长度值。

3、 定义空数组values。

4、 运行for循环函数，得出obj的各个值，并赋值给values[i]。

5、 返回values的值。

```
var test = _.values({one: 1, two: 2, three: 3});
console.log(test);
// [1, 2, 3]
```

## 4. _.mapObject

_.mapObject(object, iteratee, [context]) 
它类似于map，但是这用于对象。转换每个属性的值。

```
_.mapObject = function(obj, iteratee, context) {
  iteratee = cb(iteratee, context);
  var keys = _.keys(obj),
      length = keys.length,
      results = {};
  for (var index = 0; index < length; index++) {
    var currentKey = keys[index];
    results[currentKey] = iteratee(obj[currentKey], currentKey, obj);
  }
  return results;
};
```

1、 运行cb()函数，得到新的iteratee函数。

2、 运行_.keys()函数，得到obj属性的集合。

3、 定义length存在keys的长度值，定义results空对象。

4、 运行for循环，把keys对应的index值（obj的属性名）赋值给currentKey，并运行iteratee函数，并把新计算的结果返回给results[currentKey]。

5、 返回results的最终结果。

例：

```
var test = _.mapObject({start: 5, end: 12}, function(val, key) {
  return val + 5;
});
console.log(test);
// {start: 10, end: 17}
```

## 5. _.pairs

_.pairs(object) 
把一个对象转变为一个[key, value]形式的数组。

```
_.pairs = function(obj) {
  var keys = _.keys(obj);
  var length = keys.length;
  var pairs = Array(length);
  for (var i = 0; i < length; i++) {
    pairs[i] = [keys[i], obj[keys[i]]];
  }
  return pairs;
};
```

1、 运行_.keys()函数，得到obj属性的集合。

2、 定义length存在keys的长度值，定义pairs空数组。

3、 运行for循环，把obj的键和值组合为一个新的数组赋值给pairs对应的索引位置中。

4、 返回pairs的最终结果。

例：

```
var test = _.pairs({one: 1, two: 2, three: 3});
console.log(test);
// [["one", 1], ["two", 2], ["three", 3]]
```

## 6. _.invert

_.invert(object) 
返回一个object副本，使其键（keys）和值（values）对换。对于这个操作，必须确保object里所有的值都是唯一的且可以序列号成字符串。

```
_.invert = function(obj) {
  var result = {};
  var keys = _.keys(obj);
  for (var i = 0, length = keys.length; i < length; i++) {
    result[obj[keys[i]]] = keys[i];
  }
  return result;
};
```

1、 定义空对象result。

2、 运行_.keys()函数，得到obj属性的集合。

3、 运行for函数，以obj的键名作为result的值，obj的值作为result的键名，把obj的键名赋值给result。

4、 返回results的最终结果。

例：

```
var test = _.invert({Moe: "Moses", Larry: "Louis", Curly: "Jerome"});
console.log(test);
// {Moses: "Moe", Louis: "Larry", Jerome: "Curly"};
```

## 7. _.functions

_.functions(object)（.methods）
返回一个对象里所有的方法名, 而且是已经排序的 — 也就是说, 对象里每个方法(属性值是一个函数)的名称.

```
_.functions = _.methods = function(obj) {
  var names = [];
  for (var key in obj) {
    if (_.isFunction(obj[key])) names.push(key);
  }
  return names.sort();
};
```

1、 定义空数组names。

2、 运行for in函数，遍历对象中的各属性。

3、 运行_.isFunction()函数，判断obj的值是否为函数类型，若是，则把obj对应的键名push到names中。

4、 运行sort()函数，对names数组的元素进行排序。并返回结果。

例：

```
var obj = {
	all:function(){},
	one:'one',
	bindAll:function(){},
	bind:function(){},
	any:function(){}
};

var test = _.functions(obj);
console.log(test);
// ["all", "any", "bind", "bindAll"]
```



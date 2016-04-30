---
layout: post
category: js源码解析
---

# 对象函数(Object Functions)

## 21. _.isMatch

_.isMatch(object, properties) 
告诉你properties中的键和值是否包含在object中。

```
_.isMatch = function(object, attrs) {
  var keys = _.keys(attrs), length = keys.length;
  if (object == null) return !length;
  var obj = Object(object);
  for (var i = 0; i < length; i++) {
    var key = keys[i];
    if (attrs[key] !== obj[key] || !(key in obj)) return false;
  }
  return true;
};
```

1、 运行_.keys()函数，遍历对象attrs并把其属性名组成一个数组，把结果返回给keys.

2、 若object为空，则返回一个布尔值。

3、 定义空对象obj。

4、 运行for函数，得到每个key，并判断若attrs[key]的值不等于obj[key]的值，或者key不存在与obj，则返回false

5、 否则返回true

例：

```
var stooge = {name: 'moe', age: 32};
var test = _.isMatch(stooge, {age: 32});
console.log(test);
// true

var test2 = _.isMatch(null, {age: 32});
console.log(test2);
// false

var test3 = _.isMatch(null, {});
console.log(test3);
// true
```

## 22. _.isEqual

_.isEqual(object, other) 
执行两个对象之间的优化深度比较，确定他们是否应被视为相等。

```
// instanceof 运算符是用来在运行时指出对象是否是特定类的一个实例
var _ = function(obj) {
  if (obj instanceof _) return obj;
  if (!(this instanceof _)) return new _(obj);
  this._wrapped = obj;
};
```

```
// isEqual的内部递归比较函数
var eq, deepEq;
eq = function(a, b, aStack, bStack) {
  // 有时两个对象是相等的,但是它们并不是完全相同的，例如'0 === -0'
  if (a === b) return a !== 0 || 1 / a === 1 / b;
  // 一个严格的比较是必要的，例如'null == undefined'
  if (a == null || b == null) return a === b; 
  // NaN 与任何值（包括其自身）相比得到的结果均是 false,但是它们是等效的
  if (a !== a) return b !== b;
  // 使用原始的typeof方法检查类型。
  var type = typeof a;
  if (type !== 'function' && type !== 'object' && typeof b != 'object') return false;
  return deepEq(a, b, aStack, bStack);
};
```

```
// isEqual的内部递归比较函数
deepEq = function(a, b, aStack, bStack) {
  // 打开任何包装对象。
  if (a instanceof _) a = a._wrapped;
  if (b instanceof _) b = b._wrapped;
  // 比较className
  var className = toString.call(a);
  if (className !== toString.call(b)) return false;
  switch (className) {
    //通过className值来比较字符串，数字，正则表达式，日期，还有布尔值。
    case '[object RegExp]':
    //正则表达式强迫进行字符串比较，例如：('' + /a/i === '/a/i')
    case '[object String]':
      // 原字符串及其对应的对象包装器是等价的;因此'"5"'是等价于'new String("5")'。
      return '' + a === '' + b;
    case '[object Number]':
      // NaN 与任何值（包括其自身）相比得到的结果均是 false
      // 但是Object(NaN)是等价于NaN
      if (+a !== +a) return +b !== +b;
      // 对于其他的数字，需要执行一个公平的比较。
      return +a === 0 ? 1 / +a === 1 / b : +a === +b;
    case '[object Date]':
    case '[object Boolean]':
      // 强制日期和布尔运算数字原始值。
      // 比较日期的毫秒表示。注意,无效的日期毫秒表示为“NaN”并不是等价的。
      return +a === +b;
    case '[object Symbol]':
      return SymbolProto.valueOf.call(a) === SymbolProto.valueOf.call(b);
  }

  var areArrays = className === '[object Array]';
  if (!areArrays) {
    if (typeof a != 'object' || typeof b != 'object') return false;

    // 对象与不同的构造函数并不等价,且对象和数组来自不同的框架。
    var aCtor = a.constructor, bCtor = b.constructor;
    if (aCtor !== bCtor && !(_.isFunction(aCtor) && aCtor instanceof aCtor &&
                             _.isFunction(bCtor) && bCtor instanceof bCtor)
                        && ('constructor' in a && 'constructor' in b)) {
      return false;
    }
  }
  // 运行平等的循环结构。检测循环结构的算法是改编自ES 5.1节15.12.3,文摘提供“JO”
  // 遍历对象的初始化堆栈
  // 这里我们只需要比较对象和数组。
  aStack = aStack || [];
  bStack = bStack || [];
  var length = aStack.length;
  while (length--) {
    // 线性搜索。性能是成反比的独特的嵌套结构。
    if (aStack[length] === a) return bStack[length] === b;
  }

  // 添加第一个object对象到遍历对象的栈中
  aStack.push(a);
  bStack.push(b);

  // 递归比较对象和数组。
  if (areArrays) {
    // 比较数组长度来确定进一步的比较是必要的。
    length = a.length;
    if (length !== b.length) return false;
    // 深入比较内容,忽视非数字属性。
    while (length--) {
      if (!eq(a[length], b[length], aStack, bStack)) return false;
    }
  } else {
    // 深入比较对象
    var keys = _.keys(a), key;
    length = keys.length;
    // 确保两个对象包含相同数量的属性之前进一步比较他们是否相等。
    if (_.keys(b).length !== length) return false;
    while (length--) {
      // 深度比较每一个数字
      key = keys[length];
      if (!(_.has(b, key) && eq(a[key], b[key], aStack, bStack))) return false;
    }
  }
  // 从堆栈中删除第一个对象遍历对象。
  aStack.pop();
  bStack.pop();
  return true;
};
```

```
_.isEqual = function(a, b) {
  return eq(a, b);
};
```

1、 运行eq()函数

1) 若a全等于b，则返回满足于表达式(a !== 0 || 1 / a === 1 / b)的布尔值。由于'0 === -0'返回的是true，但其实这两个数字时不相等的，故需要以上表达式来判断。

2) 若a为空或b为空，则返回满足于表达式(a === b)的布尔值。由于'null == undefined'返回的事true，故需要严格的使用全等于符号(三个等于符号组成)来比较。

3) 若a不全等于本身，则返回满足于表达式(b !== b)的布尔值。由于NaN是不等于自己本身的，可是NaN实际上是与自身等效的。

4) 



## 23. _.isEmpty

_.isEmpty(object) 
如果object 不包含任何值(没有可枚举的属性)，返回true。 对于字符串和类数组（array-like）对象，如果length属性为0，那么 _.isEmpty检查返回true。





## 24. _.isElement

_.isElement(object) 
如果object是一个DOM元素，返回true。



## 25. _.isArray

_.isArray(object) 
如果object是一个数组，返回true。

## 26. _.isObject


## 27. _.isArguments/ _.isFunction/ _.isString/ _.isNumber/ _.isDate/ _.isRegExp/ _.isError


## 28. _.isFinite


## 29. _.isNaN


## 30. _.isBoolean


## 31. _.isNull


## 32. _.isUndefined




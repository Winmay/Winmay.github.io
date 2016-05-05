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
var SymbolProto = typeof Symbol !== 'undefined' ? Symbol.prototype : null;

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
      // 强制日期和布尔值运算数字原始值。
      // 比较日期的毫秒表示。注意,无效的日期毫秒表示为“NaN”并不是等价的。
      return +a === +b;
    case '[object Symbol]':
      // valueOf() 方法可返回 Boolean 对象的原始值。
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

4) 使用typeof判断a和b的数据类型，若a不为函数类型，且不为对象类型，b不为对象类型，则返回false。

5) 若a和b都不满足以上条件，则运行deepEq()函数，并返回布尔值。

2、 deepEq()函数：

1) 若a是_对象的一个实例，那么a = a._wrapped

2) 若b是_对象的一个实例，那么b = b._wrapped

3) 把a对象转换为字符串，赋值给className，并判断className与b对象转换为字符串的值不相等，则返回false。

4) 通过运行switch...case...函数，来判断className的值，并分别运行不同的函数。

a) className为[object RegExp]正则表达式时，则强迫其a和b进行字符串比较，例：'' + /a/i === '/a/i'，返回满足于表达式('' + a === '' + b)的布尔值。

b) className为[object String]字符串时，则返回满足于表达式('' + a === '' + b)的布尔值。

c) className为[object Number]数字时，首先判断对象a是否为NaN，若是则返回满足于表达式(+b !== +b)的布尔值。若不是则返回满足于表达式(+a === 0 ? 1 / +a === 1 / b : +a === +b)的布尔值。

d)  className为[object Date]日期时，强制日期运算数字原始值，则返回满足于表达式(+a === +b)的布尔值。

e)  className为[object Boolean]布尔值时，强制布尔值运算数字原始值，则返回满足于表达式(+a === +b)的布尔值。

f)  className为[object Symbol]象征对象时，返回满足于表达式(SymbolProto.valueOf.call(a) === SymbolProto.valueOf.call(b))的布尔值。由于每次运行Symbol()都是创建一个新的对象，故Symbol("foo") === Symbol("foo")为false。

5) 若className不是数组类型

a) 若a或b的数据类型不属于对象，则返回false。

b) 把a的构造函数赋值给aCtor，b的构造函数赋值给bCtor，并判断当aCtor !== bCtor且!(_.isFunction(aCtor) && aCtor instanceof aCtor && _.isFunction(bCtor) && bCtor instanceof bCtor)且('constructor' in a && 'constructor' in b)，则返回false。

6) 对传入的第三第四个参数aStack，bStack进行操作。遍历对象的初始化堆栈，这里我们只需要比较对象和数组。进行while循环，若满足条件aStack[length] === a，则返回满足于表达式(bStack[length] === b)的布尔值。若不满足条件，则继续往下运行，添加第一个object对象到遍历对象的栈中。

7) 若className为数组类型，则

a) 先比较长度是否一样，若不一样，则返回false

b) 若长度一样，则运行while循环，从最后一个元素开始，运行eq()函数，若返回结果为false，则返回false。

8) 若className不是数组类型，则

a) 深入比较对象，同样先比较其长度是否一样，若不一样，则返回false。

b) 若一样则运行while循环函数，从最后一个元素开始，判断a中的key不存在与b中，则返回false，运行eq()返回结果为false，则同样返回false。

9) 从堆栈中删除第一个对象遍历对象。最后返回true。

例：

```
var stooge = {name: 'moe', luckyNumbers: [13, 27, 34]};
var clone  = {name: 'moe', luckyNumbers: [13, 27, 34]};
console.log(stooge == clone);
// false
var test = _.isEqual(stooge, clone);
console.log(test);
// true

var test2 = _.isEqual(0, -0);
console.log(test2);
// false

var test3 = _.isEqual(3, 3);
console.log(test3);
// true

var test4 = _.isEqual(null, undefined);
console.log(test4);
// false

var test5 = _.isEqual(null, null);
console.log(test5);
// true

var test6 = _.isEqual('name', 'name');
console.log(test6);
// true

var test7 = _.isEqual(NaN, NaN);
console.log(test7);
// true
```

## 23. _.isEmpty

_.isEmpty(object) 
如果object 不包含任何值(没有可枚举的属性)，返回true。 对于字符串和类数组（array-like）对象，如果length属性为0，那么 _.isEmpty检查返回true。

```
_.isEmpty = function(obj) {
  if (obj == null) return true;
  if (isArrayLike(obj) && (_.isArray(obj) || _.isString(obj) || _.isArguments(obj))) return obj.length === 0;
  return _.keys(obj).length === 0;
};
```

1、 若obj为null，则返回true

2、 若obj不是null，则判断是非为类数组对象，且判断obj为数组或字符串或参数对象时，返回满足条件(obj.length === 0)的布尔值。

3、 若obj为对象类型，则返回满足条件(_.keys(obj).length === 0)的布尔值。

例：

```
var test = _.isEmpty([1, 2, 3]);
console.log(test);
// false

var test2 = _.isEmpty([]);
console.log(test2);
// true

console.log('------------------');

var test3 = _.isEmpty({length:0});
console.log(test3);
// false

var test4 = _.isEmpty({});
console.log(test4);
// true

console.log('------------------');

var test5 = _.isEmpty('name');
console.log(test5);
// false

var test6 = _.isEmpty('');
console.log(test6);
// true
```

## 24. _.isElement

_.isElement(object) 
如果object是一个DOM元素，返回true。

```
_.isElement = function(obj) {
  return !!(obj && obj.nodeType === 1);
};
```

1、 当obj存在，且obj.nodeType的值为1时，通过两个感叹号!!强制转换为布尔类型的数据，并返回true

2、 由于nodeType的值为1时说明其DOM节点为元素element节点。

3、 双感叹号!!一般用来将后面的表达式强制转换为布尔类型的数据（boolean），也就是只能是true或者false;
因为javascript是弱类型的语言（变量没有固定的数据类型）所以有时需要强制转换为相应的类型。

例：

```
var test = _.isElement(jQuery('body')[0]);
console.log(test);
// true
```

## 25. _.isArray

_.isArray(object) 
如果object是一个数组，返回true。

```
_.isArray = nativeIsArray || function(obj) {
  return toString.call(obj) === '[object Array]';
};
```

1、 先用toString把obj转换为字符串，然后判断得到的字符串值是否全等于[object Array]，若等于则返回true，否则返回false。

例：

```
var test = _.isArray([1,2,3]);
console.log(test);
// true

var test2 = _.isArray({0:'a', 1:'b' ,length:2});
console.log(test2);
// false
```

## 26. _.isObject

_.isObject(value) 
如果object是一个对象，返回true。需要注意的是JavaScript数组和函数是对象，字符串和数字不是。

```
_.isObject = function(obj) {
  var type = typeof obj;
  return type === 'function' || type === 'object' && !!obj;
};
```

1、 通过typeof函数判断obj的数据类型。

2、 若type的值为'function'或'object'，且存在obj，则返回true。

例：

```
var test = _.isObject({});
console.log(test);
// true

var test2 = _.isObject(function(){});
console.log(test2);
// true

var test3 = _.isObject([]);
console.log(test3);
// true

var test4 = _.isObject(1);
console.log(test4);
// false

var test5 = _.isObject('a');
console.log(test5);
// false

var test6 = _.isObject(null);
console.log(test6);
// false
```

## 27. _.isArguments

_.isArguments(object) 
如果object是一个参数对象，返回true。

```
_.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error', 'Symbol', 'Map', 'WeakMap', 'Set', 'WeakSet'], function(name) {
  _['is' + name] = function(obj) {
    return toString.call(obj) === '[object ' + name + ']';
  };
});
```

1、 通过_.each()函数，在 _对象中添加相应的对象，并运行函数。

2、 此处将生产函数

```
_.isArguments = function(obj) {
  return toString.call(obj) === '[object Arguments]';
};
```

3、 使用toString把obj转换为字符串类型，若得到的值全等于'[object Arguments]'，则返回true，否则返回false。

4、 为了兼容IE9以下的浏览器，则运行以下代码

```
if (!_.isArguments(arguments)) {
  _.isArguments = function(obj) {
    return _.has(obj, 'callee');
  };
}
```

若_.isArguments(arguments)为false，则运行 _.has(obj, 'callee')函数，判断字符串'callee'是否存在与obj中，若存在则返回true，否则返回false。

例：

```
var fun = function(){
	return _.isArguments(arguments); 
}

var test = fun(1,2,3)
console.log(test);
//true

var test2 = _.isArguments([1,2,3]);
console.log(test2)
//false
```

其中callee 属性是 arguments 对象的一个成员，该属性仅当相关函数正在执行时才可用。callee 属性的初始值是正被执行的 Function 对象。 这将允许匿名函数成为递归的。

例：

```
function factorial(n) {
   if (n <= 0)
      return 1;
   else
      return n * arguments.callee(n - 1)
}
print(factorial(4));
// Output: 24
```

## 28. _.isFunction

_.isFunction(object) 
如果object是一个函数（Function），返回true。

```
_.isFunction = function(obj) {
  return toString.call(obj) === '[object Function]';
};
```

1、 使用toString把obj转换为字符串类型，若得到的值全等于'[object Function]'，则返回true，否则返回false。

2、 为了兼容IE11，Safari8，PhantomJS浏览器，则运行以下代码

```
var nodelist = root.document && root.document.childNodes;
if (typeof /./ != 'function' && typeof Int8Array != 'object' && typeof nodelist != 'function') {
  _.isFunction = function(obj) {
    return typeof obj == 'function' || false;
  };
}
```

把DOM中的子节点复制给node list，并判断此类型不为函数类型。且判断/./不为函数类型，Int8Array不为对象类型，则返回满足条件(typeof obj == 'function' || false)的布尔值。

例：

```
var test = _.isFunction(alert);
console.log(test);
//true

var test2 = _.isFunction('alert');
console.log(test2)
//false
```

## 29. _.isString

_.isString(object)
如果object是一个字符串，返回true。

```
_.isString = function(obj) {
  return toString.call(obj) === '[object String]';
};
```

1、 使用toString把obj转换为字符串类型，若得到的值全等于'[object String]'，则返回true，否则返回false。

例：

```
var test = _.isString("moe");
console.log(test);
//true

var test2 = _.isString(1);
console.log(test2)
//false
```

## 30. _.isNumber

_.isNumber(object) 
如果object是一个数值，返回true (包括 NaN)。

```
_.isNumber = function(obj) {
  return toString.call(obj) === '[object Number]';
};
```

1、 使用toString把obj转换为字符串类型，若得到的值全等于'[object Number]'，则返回true，否则返回false。

例：

```
var test = _.isNumber(8.4 * 5);
console.log(test);
//true

var test2 = _.isNumber(NaN);
console.log(test2)
//true

var test3 = _.isNumber('1');
console.log(test3)
//false
```

## 31. _.isDate

_.isDate(object) 
如果object是一个日期，返回true。

```
_.isDate = function(obj) {
  return toString.call(obj) === '[object Date]';
};
```

1、 使用toString把obj转换为字符串类型，若得到的值全等于'[object Date]'，则返回true，否则返回false。

例：

```
var test = _.isDate(new Date());
console.log(test);
//true

var test2 = _.isNumber('2016-03-13');
console.log(test2)
//false
```

## 32. _.isRegExp

_.isRegExp(object) 
如果object是一个正则表达式，返回true。

```
_.isRegExp = function(obj) {
  return toString.call(obj) === '[object RegExp]';
};
```

1、 使用toString把obj转换为字符串类型，若得到的值全等于'[object RegExp]'，则返回true，否则返回false。

例：

```
var test = _.isRegExp(/moe/);
console.log(test);
//true
```

## 33. _.isError

_.isError(object) 
如果object继承至 Error 对象，那么返回 true。

```
_.isError = function(obj) {
  return toString.call(obj) === '[object Error]';
};
```

1、 使用toString把obj转换为字符串类型，若得到的值全等于'[object Error]'，则返回true，否则返回false。

例：

```
try {
  	throw new TypeError("Example");
} catch (o_O) {
  	var test = _.isError(o_O);
	console.log(test);
	//true
}
```

## 34. _.isNaN

_.isNaN(object) 
如果object是 NaN，返回true。 
注意： 这和原生的isNaN 函数不一样，如果变量是undefined，原生的isNaN 函数也会返回 true 。

```
_.isNaN = function(obj) {
  return _.isNumber(obj) && isNaN(obj);
};
```

1、 先运行_.isNumber()函数，若obj为数组类型，则运行isNaN()，若obj为NaN则返回true，否则返回false。

例：

```
var test = _.isNaN(NaN);
console.log(test);
//true

var test2 = isNaN(undefined);
console.log(test2);
//true

var test3 = _.isNaN(undefined);
console.log(test3);
//false
```

## 35. _.isBoolean

_.isBoolean(object) 
如果object是一个布尔值，返回true，否则返回false。

```
_.isBoolean = function(obj) {
  return obj === true || obj === false || toString.call(obj) === '[object Boolean]';
};
```

1、 先判断obj是否存在值，若不存在，则判断obj是否完全等于false，若不相等，则运行toString把obj转换为字符串类型，若得到的值全等于'[object Boolean]'，则返回true，否则返回false。

例：

```
var test = _.isBoolean(null);
console.log(test);
//false

var test2 = _.isBoolean(undefined);
console.log(test2);
//false
```

## 36. _.isNull

_.isNull(object) 
如果object的值是 null，返回true。

```
_.isNull = function(obj) {
  return obj === null;
};
```

1、 当obj对象完全等于null时，返回true，反则返回false。

例：

```
var test = _.isNull(null);
console.log(test);
//true

var test2 = _.isNull(undefined);
console.log(test2);
//false
```

## 37. _.isUndefined

_.isUndefined(value) 
如果value是undefined，返回true。

```
_.isUndefined = function(obj) {
  return obj === void 0;
};
```

1、 当obj对象完全等于undefined时，返回true，反则返回false。

例：

```
var test = _.isUndefined(window.missingVariable);
console.log(test);
//true

var test2 = _.isUndefined(undefined);
console.log(test2);
//true

var test3 = _.isUndefined(null);
console.log(test3);
//false
```


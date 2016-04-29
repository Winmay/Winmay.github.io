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

## 8. _.extend

```
var createAssigner = function(keysFunc, defaults) {
  return function(obj) {
    var length = arguments.length;
    if (defaults) obj = Object(obj);
    if (length < 2 || obj == null) return obj;
    for (var index = 1; index < length; index++) {
      var source = arguments[index],
          keys = keysFunc(source),
          l = keys.length;
      for (var i = 0; i < l; i++) {
        var key = keys[i];
        if (!defaults || obj[key] === void 0) obj[key] = source[key];
      }
    }
    return obj;
  };
};

_.extend = createAssigner(_.allKeys);
```

1、 运行createAssigner()函数，传入参数_.allKeys。

2、 运行function(obj)函数，定义length，用于存储传入参数的长度。

3、 判断defaults是否存在，若存在，则返回与obj对应类型的对象，并赋值给obj，相当于new一个新的对象出来。

4、 判断length传入参数的长度是否小于2或者obj为空，那么直接返回结果obj。

5、 运行for循环函数，注意index是从1开始：

1) 定义source，用于存储第index个传入的参数数据。

2) 运行keysFunc(source)函数，即这里运行的函数为 _.allKeys(source)，把所有的属性名，包括原型中的属性名都组成一个数组传入keys中。

3) 定义变量l，用于存储keys的长度。

4) 运行第二级的for循环

a) 把数组keys中的每个key值遍历一次并赋值给变量key。

b) 判断defaults不存在，或obj中key的值为undefined时，则把source中对应的键值对添加到obj中。

6、 返回最终结果obj。

例：

```
function baseCar(){
	this.color = 'red';
	this.size = 'big';
	this.name = 'lily';
}

function extendCar(){
    this.number = 'aa';
}

extendCar.prototype = new baseCar();
var instance = new extendCar();

var test = _.extend({name: 'moe'}, instance);
console.log(test);
// Object {name: "lily", number: "aa", color: "red", size: "big"}
```

## 9. _.extendOwn

_.extendOwn(destination, *sources) Alias: assign
类似于 extend,但只复制自己的属性覆盖到目标对象。（愚人码头注：不包括继承过来的属性）

```
_.extendOwn = _.assign = createAssigner(_.keys);
```

1、 运行createAssigner()函数，传入参数_.keys。

2、 运行function(obj)函数，定义length，用于存储传入参数的长度。

3、 判断defaults是否存在，若存在，则返回与obj对应类型的对象，并赋值给obj，相当于new一个新的对象出来。

4、 判断length传入参数的长度是否小于2或者obj为空，那么直接返回结果obj。

5、 运行for循环函数，注意index是从1开始：

1) 定义source，用于存储第index个传入的参数数据。

2) 运行keysFunc(source)函数，即这里运行的函数为 _.keys(source)，把所有属于自己的属性名，其中不包括原型中的属性名，组成一个数组传入keys中。此处是与 _.extend()函数的最大区别。

3) 定义变量l，用于存储keys的长度。

4) 运行第二级的for循环

a) 把数组keys中的每个key值遍历一次并赋值给变量key。

b) 判断defaults不存在，或obj中key的值为undefined时，则把source中对应的键值对添加到obj中。

6、 返回最终结果obj。

例：

```
function baseCar(){
	this.color = 'red';
	this.size = 'big';
}

function extendCar(){
    this.number = 'aa';
}

extendCar.prototype = new baseCar();
var instance = new extendCar();

var test = _.extendOwn({name: 'moe'}, instance);
console.log(test);
// Object {name: "moe", number: "aa"}
```

## 10. _.defaults

_.defaults(object, *defaults)
用defaults对象填充object 中的undefined属性。 并且返回这个object。一旦这个属性被填充，再使用defaults方法将不会有任何效果。

```
_.defaults = createAssigner(_.allKeys, true);
```

1、 运行的方式与_.extend函数一样，唯一的不同是 _.defaults函数会传入第二个参数true。

2、 此参数的作用是：当obj对象和source对象存在相同的key时，那么source对象中的key值将不会对obj对象相应的key值作出任何改变。

3、 但是，在_.extend函数中，若obj对象与source对象存在相同的key，那么source对象中的key值将会覆盖obj对象的key值。

例：

```
var iceCream = {flavor: "chocolate"};

var test = _.defaults(iceCream, {flavor: "vanilla", sprinkles: "lots"});
console.log(test);
// {flavor: "chocolate", sprinkles: "lots"}
```

## 11. _.findKey

_.findKey(object, predicate, [context]) 
它类似于 _.findIndex，但是这用于检测对象的键。当predicate通过真检查时，返回第一个键名；否则返回undefined。

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

1、 运行cb()函数，得到迭代函数predicate()的值

```
predicate=optimizeCb(predicate,context);			=function(value, index, collection) {      			return predicate.call(context, value, index, collection);   		};
```

2、 运行 _.keys(obj)函数，把obj的属性名组成数组传给keys。

3、 运行for循环，判断predicate()函数返回的结果是否为真，若为真，则返回此属性名。

例：

```
var obj = {
	one:'a',
	two:'b',
	three:'c',
	four:'b',
	five:'d',
	six:'e'
};

var test = _.findKey(obj,function(value,key){
	return value == 'b';
});
console.log(test);
// two
```

## 12. _.pick

_.pick(object, *keys) 
返回一个object副本，只过滤出keys(有效的键组成的数组)参数指定的属性值。或者接受一个判断函数，指定挑选哪个key。

```
var keyInObj = function(value, key, obj) {
    return key in obj;
};


_.pick = restArgs(function(obj, keys) {
  var result = {}, iteratee = keys[0];
  if (obj == null) return result;
  if (_.isFunction(iteratee)) {
    if (keys.length > 1) iteratee = optimizeCb(iteratee, keys[1]);
    keys = _.allKeys(obj);
  } else {
    iteratee = keyInObj;
    keys = flatten(keys, false, false);
    obj = Object(obj);
  }
  for (var i = 0, length = keys.length; i < length; i++) {
    var key = keys[i];
    var value = obj[key];
    if (iteratee(value, key, obj)) result[key] = value;
  }
  return result;
});
```

1、 运行restArgs()函数，由于function传入了两个参数：obj、keys，故restArgs()函数中的参数startIndex=1。所以运行返回的函数：

func.call(this, arguments[0], rest);

2、 定义空数组result，定义变量iteratee，初始值为rest的第一个值keys[0]。

3、 判断obj是否为空，若为空则直接返回result。

4、 判断iteratee是否为函数类型，若是则

1) 判断数组keys的长度是否大于1，若是则运行optimizeCb()函数。

2) 运行 _.allKeys(obj)函数，把obj的所有属性名，包括原型中的属性全部组成一个数组赋值给keys。

5、 若iteratee不是函数类型，则

1) 把iteratee设置为keyInObj函数，此函数作用是判断属性key是否存在与obj对象中。

2) 运行flatten()函数，把keys数组减少到一维数组，且保留非数组类型的数据。

3) 返回与obj对应类型的对象，并赋值给obj。

6、 运行for函数：

1) 遍历数组keys，并把相应索引i的值赋值给key

2) 定义变量存储obj对应key的值。

3) 判断iteratee()函数返回的结果是否为真，若为真，则更新result的值。

7、 返回最终结果result。

例：

```
var obj = {name: 'moe', age: 50, userid: 'moe1'};

var test = _.pick(obj, 'name', 'age');
console.log(test);
// {name: 'moe', age: 50}

var test2 = _.pick(obj, function(value, key, object) {
  	return _.isNumber(value);
});
console.log(test2);
// {age: 50}
```

## 13. _.omit

_.omit(object, *keys) 
返回一个object副本，只过滤出除去keys(有效的键组成的数组)参数指定的属性值。 或者接受一个判断函数，指定忽略哪个key。

```
_.omit = restArgs(function(obj, keys) {
  var iteratee = keys[0], context;
  if (_.isFunction(iteratee)) {
    // _.negate(predicate) 
    // 返回一个新的predicate函数的否定版本。
    iteratee = _.negate(iteratee);
    if (keys.length > 1) context = keys[1];
  } else {
    keys = _.map(flatten(keys, false, false), String);
    iteratee = function(value, key) {
      return !_.contains(keys, key);
    };
  }
  return _.pick(obj, iteratee, context);
});
```

1、 运行restArgs()函数，由于function传入了两个参数：obj、keys，故restArgs()函数中的参数startIndex=1。所以运行返回的函数：

func.call(this, arguments[0], rest);

2、 判断iteratee是否为函数类型，若是则

1) 运行_.negate(iteratee)函数，把iteratee的运行结果取反后重新赋值给iteratee。

2) 判断数组keys的长度是否大于1，若是则把keys[1]的值赋值给context。

3、 若iteratee不是函数类型，则

1) 运行map()遍历函数，先运行flatten()函数，把keys数组减少到一维数组，且保留非数组类型的数据。再遍历keys数组，把保留字符串类型的数据并组成新的数组赋值给keys。

2) 定义函数iteratee，检测在keys中不包含key的数据，并返回布尔变量。

3) 运行_.pick()函数。并返回最终结果。

例：

```
var obj = {name: 'moe', age: 50, userid: 'moe1'};

var test = _.omit(obj, 'name', 'age');
console.log(test);
// {userid: 'moe1'}

var test2 = _.omit(obj, function(value, key, object) {
  	return _.isNumber(value);
});
console.log(test2);
// {name: 'moe', userid: 'moe1'}
```

## 14. _.create

_.create(prototype, props) 
创建具有给定原型的新对象， 可选附加props 作为 own的属性。 基本上，和Object.create一样， 但是没有所有的属性描述符。

```
var Ctor = function(){};
var nativeCreate = Object.create;

var baseCreate = function(prototype) {
  if (!_.isObject(prototype)) return {};
  if (nativeCreate) return nativeCreate(prototype);
  Ctor.prototype = prototype;
  var result = new Ctor;
  Ctor.prototype = null;
  return result;
};

_.create = function(prototype, props) {
  var result = baseCreate(prototype);
  if (props) _.extendOwn(result, props);
  return result;
};
```

1、 运行baseCreate()函数，传入参数prototype。

1) 若prototype不是对象类型，则直接返回空对象。

2) 若nativeCreate存在，即Object.create存在，则返回nativeCreate(prototype)对象。

其中Object.create(prototype, descriptors)函数是创建一个具有指定原型且可选择性地包含指定属性的对象。第一个参数是要继承的原型，如果不是一个子函数，可以传一个null，第二个参数是对象的属性描述符，这个参数是可选的。

3)若上面两个添加都不满足，则往下运行，定义Ctor的原型为传入参数prototype。定义result为新的Ctor函数。返回result。

2、 此处由于存在nativeCreate为函数function create() { [native code] }，故

```
result = nativeCreate(prototype)
		 = Object.create(prototype)
		 = {}
```

3、 若有传入第二个参数props，则运行_.extendOwn()，把props对象扩展到result中。

4、 返回最终结果result。

例：

```
function baseCar(){
	this.color = 'red';
	this.size = 'big';
}

function extendCar(){
    this.number = 'aa';
}

extendCar.prototype = new baseCar();

var test = _.create(extendCar.prototype, {name: "Moe"});
console.log(test);		// baseCar {name: "Moe"}
console.log(test.name); // Moe
```

## 15. _.clone

_.clone(object) 
创建 一个浅复制（浅拷贝）的克隆object，即副本。任何嵌套的对象或数组都通过引用拷贝，而不是复制原对象。

```
_.clone = function(obj) {
  if (!_.isObject(obj)) return obj;
  return _.isArray(obj) ? obj.slice() : _.extend({}, obj);
};
```

1、 若obj不是对象类型，则直接返回obj对象。

2、 若obj为数组类型，则通过slice()函数，返回obj数组。若obj不为数组类型，则运行_.extend()函数，把obj对象扩展到空对象中。

例：

```
var test = _.clone({name: 'moe'});
console.log(test);
// {name: 'moe'}
```

```
var test2 = _.clone([1, 2, 3, 4, 5]);
console.log(test2);
// [1, 2, 3, 4, 5]
```

## 16. _.tap

_.tap(object, interceptor) 
用 object作为参数来调用函数interceptor，然后返回object。这种方法的主要意图是作为函数链式调用 的一环, 为了对此对象执行操作并返回对象本身。

```
_.tap = function(obj, interceptor) {
  interceptor(obj);
  return obj;
};
```

1、 以obj作为为参数，运行interceptor函数，并返回obj本身。

例：

```
var test = _.chain([1,2,3,200])
  .filter(function(num) { return num % 2 == 0; })
  .tap(alert) // [2, 200] (alerted)
  .map(function(num) { return num * num })
  .value();
console.log(test);
//[4, 40000]
```

```
var test2 = _.tap({one:'a',two:'b'},_.keys);
console.log(test2);
//{one:'a',two:'b'}
```

## 17. _.has

_.has(object, key) 
对象是否包含给定的键？等同于object.hasOwnProperty(key)，但是使用hasOwnProperty 函数的一个安全引用，以防意外覆盖。

```
var ObjProto = Object.prototype;
var hasOwnProperty = ObjProto.hasOwnProperty;

_.has = function(obj, key) {
  return obj != null && hasOwnProperty.call(obj, key);
};
```

1、 hasOwnProperty函数方法是返回一个布尔值，指出一个对象是否具有指定名称的属性。 

使用方法：

object.hasOwnProperty(proName

其中参数object是必选项。一个对象的实例。proName是必选项。一个属性名称的字符串值。此方法无法检查该对象的原型链中是否具有该属性；该属性必须是对象本身的一个成员。 

2、 判断obj对象不为空且使用hasOwnProperty.call方法，检测key属性存在于obj对象中，则返回true。

3、 若不满足这两个条件中的任一个条件，则返回false。

例：

```
var test = _.has({a: 1, b: 2, c: 3}, "b");
console.log(test);
//true
```

## 18. _.property

_.property(key) 
返回一个函数，这个函数返回任何传入的对象key的属性值。

```
var property = function(key) {
  return function(obj) {
    return obj == null ? void 0 : obj[key];
  };
}; 
_.property = property;
```

1、 判断obj为null时，返回undefined，若不为空，则返回obj对应的key值。

例：

```
var obj = {name: 'moe'};
var test = _.property('name')(obj)
console.log('moe' === test);
//true
```

## 19. _.propertyOf

_.propertyOf(object) 
和 _.property相反。需要一个对象，并返回一个函数,这个函数将返回一个提供的属性的值。

```
_.propertyOf = function(obj) {
  return obj == null ? function(){} : function(key) {
    return obj[key];
  };
};
```

1、 判断obj是否为null，若是则返回空的函数，若否则运行function并返回obj对应的key值。

例：

```
var stooge = {name: 'moe'};
var test = _.propertyOf(stooge)('name');
console.log(test);
//'moe'
```

## 20. _.matcher

_.matcher(attrs) 
返回一个断言函数，这个函数会给你一个断言可以用来辨别给定的对象是否匹配attrs指定键/值属性。

```
_.matcher = _.matches = function(attrs) {
  attrs = _.extendOwn({}, attrs);
  return function(obj) {
    // _.isMatch(object, properties) 
    // 告诉你properties中的键和值是否包含在object中。
    return _.isMatch(obj, attrs);
  };
};
```

1、 运行 _.extendOwn()函数，把arrts对象扩展到空对象中。并重新赋值给attrs。

2、 运行 _.isMatch()函数，判断attrs是否存在与obj中。若存在则返回true，否则返回false。

例：

```
var list = [
	{selected: true, visible: true},
	{selected: true, visible: true, unable: false},
	{selected: false, visible: true, unable: false},
];
var obj = {selected: true, visible: true, unable: false}

var ready = _.matcher({selected: true, visible: true});
var test = _.matcher({selected: true, visible: true})(obj);

var readyToGoList = _.filter(list, ready);

console.log(test); //true
console.log(readyToGoList);
/*
[
	{selected: true, visible: true},
	{selected: true, visible: true, unable: false},
]
*/
```




---
layout: post
category: js源码解析
---

# 函数(Functions)

## 1. _.bind

_.bind(function, object, *arguments) 
绑定函数 function 到对象 object 上, 也就是无论何时调用函数, 函数里的 this 都指向这个 object.任意可选参数 arguments 可以传递给函数 function , 可以填充函数所需要的参数,这也被称为 partial application。对于没有结合上下文的partial application绑定，请使用partial。 
(愚人码头注：partial application翻译成“部分应用”或者“偏函数应用”。partial application可以被描述为一个函数，它接受一定数目的参数，绑定值到一个或多个这些参数，并返回一个新的函数，这个返回函数只接受剩余未绑定值的参数。)

```
// 一个内部函数创建一个新的对象,继承自另一个。
// Object.create(proto [, propertiesObject ]) 是E5中提出的一种新的对象创建方式，
// 第一个参数是要继承的原型，如果不是一个子函数，可以传一个null，
// 第二个参数是对象的属性描述符，这个参数是可选的。
var nativeCreate = Object.create;
var Ctor = function(){};
var baseCreate = function(prototype) {
  if (!_.isObject(prototype)) return {};
  if (nativeCreate) return nativeCreate(prototype);
  Ctor.prototype = prototype;
  var result = new Ctor;
  Ctor.prototype = null;
  return result;
};
```

```
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
	// instanceof 用于判断一个变量是否某个对象的实例
  if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);
  var self = baseCreate(sourceFunc.prototype);
  var result = sourceFunc.apply(self, args);
  if (_.isObject(result)) return result;
  return self;
};

_.bind = restArgs(function(func, context, args) {
  if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');
  var bound = restArgs(function(callArgs) {
    return executeBound(func, bound, context, this, args.concat(callArgs));
  });
  return bound;
});
```

1、 运行restArgs()函数，由于function传入了三个参数：func,context,args。故restArgs()函数中的参数startIndex=2。所以运行返回的函数：

func.call(this, arguments[0], arguments[1], rest);

2、 判断func不是函数类型的话，抛出异常。

3、 定义变量bound，并运行restArgs()函数，由于function只传入了一个参数callArgs，故restArgs()函数中的参数startIndex=0。所以运行返回的函数：

func.call(this, rest);

4、 运行executeBound()函数。传入5个参数：func、bound、context、this、args.concat(callArgs)。

5、 判断callingContext是否为变量boundFunc对象的一个实例，若不是，则返回函数：

sourceFunc.apply(context, args);

即，如果当前的this已经指向的一个function的实例，就不需要再改变this的指向，因为此时的function已经作为一个构造函数在使用。_.bind()函数传入的参数this为window,已经指向的一个function的实例，参数bound为restArgs()函数，故callingContext不是变量boundFunc对象的一个实例，满足上面的条件，故运行上面的函数。

6、 否则boundFunc(function)视为构造函数使用，要保证callingContext(this)为构造函数的实例，即运行baseCreate()函数，将sourceFunc(function)强制转换成一个类的构造函数。

7、 若result为对象类型，则返回result，否则返回self。

例：

```
var func = function(greeting){ 
	console.log( greeting + ': ' + this.name );
};
var func1 = _.bind(func, {name: 'moe'}, 'hi');
func1();
//hi: moe
```

```
var func2 = function(greeting,mood){ 
	console.log( greeting + ': ' + this.name + ',' + mood);
};
var func2 = _.bind(func2, {name: 'moe'}, 'hi','I am find!');
func2();
//hi: moe,I am find!
```

## 2. _.bindAll

_.bindAll(object, *methodNames) 
把methodNames参数指定的一些方法绑定到object上，这些方法就会在对象的上下文环境中执行。绑定函数用作事件处理函数时非常便利，否则函数被调用时this一点用也没有。methodNames参数是必须的。

```
_.bindAll = restArgs(function(obj, keys) {
  keys = flatten(keys, false, false);
  var index = keys.length;
  if (index < 1) throw new Error('bindAll must be passed function names');
  while (index--) {
    var key = keys[index];
    obj[key] = _.bind(obj[key], obj);
  }
});
```

1、 运行restArgs()函数，由于function传入了两个参数：obj、keys，故restArgs()函数中的参数startIndex=1。所以运行返回的函数：

func.call(this, arguments[0], rest);

2、 运行flatten()函数，这里传入了三个参数：keys、false、false。第一个参数keys为一个一维或多维的数组，第二个参数false的意思是把一个多维的数组最终转换为剩下一层的一维数组，第三个参数false的意思是保留非数组类型的数据，并直接赋值到相应位置。把新得到的数组重新赋值给keys。

3、 把keys的长度赋值给index。

4、 判断index < 1时，抛出异常。

5、 运行while()函数，满足条件时运行里面的代码块。

a) 把keys[index]得到的函数名赋值给key。

b) 运行_.bind()函数，并以obj[key]作为函数参数，obj作为作用域参数，传进函数里面。并把得到的值赋给obj[key]。

例：

```
var divView = {  
    ele: '#divTip',  
    tip: 'hello,underscore!',  
    onClick: function(){ console.log('clicked: ' + this.tip); },
    onHover: function(){ console.log('hovering: ' + this.tip); }
};  
_.bindAll(divView, 'onClick','onHover'); 
$(divView.ele).bind('click', divView.onClick); 
//若不运行_.bindAll()函数，则打印clicked: undefined
//clicked: hello,underscore! 
$(divView.ele).bind('click', divView.onHover); 
//若不运行_.bindAll()函数，则打印hovering: undefined
//hovering: hello,underscore!
```

## 3. _.partial

_.partial(function, *arguments) 
局部应用一个函数填充在任意个数的 arguments，不改变其动态this值。和bind方法很相近。你可以传递 _ 给arguments列表来指定一个不预先填充，但在调用时提供的参数。

```
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
	// instanceof 用于判断一个变量是否某个对象的实例
  if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);
  var self = baseCreate(sourceFunc.prototype);
  var result = sourceFunc.apply(self, args);
  if (_.isObject(result)) return result;
  return self;
};

_.partial = restArgs(function(func, boundArgs) {
  var placeholder = _.partial.placeholder;
  var bound = function() {
    var position = 0, length = boundArgs.length;
    var args = Array(length);
    for (var i = 0; i < length; i++) {
      args[i] = boundArgs[i] === placeholder ? arguments[position++] : boundArgs[i];
    }
    while (position < arguments.length) args.push(arguments[position++]);
    return executeBound(func, bound, this, this, args);
  };
  return bound;
});

_.partial.placeholder = _;
```

1、 运行restArgs()函数，由于function传入了两个参数：obj、keys，故restArgs()函数中的参数startIndex=1。所以运行返回的函数：

func.call(this, arguments[0], rest);

2、 把下划线 _ 赋值给placeholder。

3、 定义变量bound，并运行function()函数。

a) 定义变量position，初始值为0。定义变量length，初始值为boundArgs的长度。定义数组新的args。

b) 运行for函数，判断boundArgs[i]是否完全等于placeholder，若是，则把arguments[position++]赋值给args[i]，否者把boundArgs[i]的值赋值给args[i]。直到for循环结束。其中arguments为用户运行函数时，自己传进去的参数。

c) 运行while()函数，当position < arguments.length时，把argument[position++]的值push到args里面。

d) 运行executeBound()函数。传入5个参数：func、bound、this、this、args。

e) 判断callingContext是否为变量boundFunc对象的一个实例,若不是，则返回函数：

sourceFunc.apply(context, args);

_.partial()函数传入的参数this为window,参数bound为bound()函数本身，故callingContext不是变量boundFunc对象的一个实例，满足上面的条件，故运行上面的函数。

例：

```
var subtract = function(a, b) { return b - a; };
sub5 = _.partial(subtract, 5);
var test = sub5(20);
console.log(test);
// 15

// Using a placeholder
subFrom20 = _.partial(subtract, _, 20);
var test2 = subFrom20(5);
console.log(test2);
// 15
```

## 4. _.memoize

_.memoize(function, [hashFunction]) 
Memoizes方法可以缓存某函数的计算结果。对于耗时较长的计算是很有帮助的。如果传递了 hashFunction 参数，就用 hashFunction 的返回值作为key存储函数的计算结果。hashFunction 默认使用function的第一个参数作为key。memoized值的缓存可作为返回函数的cache属性。

```
_.memoize = function(func, hasher) {
  var memoize = function(key) {
    var cache = memoize.cache;
    var address = '' + (hasher ? hasher.apply(this, arguments) : key);
    // _.has(object, key) 
    // 对象是否包含给定的键吗？
    //等同于object.hasOwnProperty(key)，
    //但是使用hasOwnProperty 函数的一个安全引用，以防意外覆盖。
    if (!_.has(cache, address)) cache[address] = func.apply(this, arguments);
    return cache[address];
  };
  memoize.cache = {};
  return memoize;
};
```

1、 运行memoize()函数

2、 定义变量cache，存储缓存对象。

3、 定义变量address，存储key。判断hasher是否有值，若有则运行函数hasher.apply(this, arguments) ，否则把key赋值给address。

4、 判断cache是否存在address，若存在，则返回cache[address]，若不存在，则运行func.apply(this, arguments)，并赋值给cache[address]。

5、 清空存储对象memoize.cache。

6、 返回memoize的值。

例：

```
var fibonacci = _.memoize(function(n) {
  return n < 2 ? n: fibonacci(n - 1) + fibonacci(n - 2);
});

var test = fibonacci(5); 
console.log(test);
// 5
/*
Object {1: 1}
Object {0: 0, 1: 1}
Object {0: 0, 1: 1, 2: 1}
Object {0: 0, 1: 1, 2: 1}
Object {0: 0, 1: 1, 2: 1, 3: 2}
Object {0: 0, 1: 1, 2: 1, 3: 2}
Object {0: 0, 1: 1, 2: 1, 3: 2, 4: 3}
Object {0: 0, 1: 1, 2: 1, 3: 2, 4: 3}
Object {0: 0, 1: 1, 2: 1, 3: 2, 4: 3, 5: 5}
*/
```

## 5. _.delay


## 6. _.defer


## 7. _.throttle


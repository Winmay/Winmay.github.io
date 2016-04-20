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

_.delay(function, wait, *arguments) 
类似setTimeout，等待wait毫秒后调用function。如果传递可选的参数arguments，当函数function执行时， arguments 会作为参数传入。

```
_.delay = restArgs(function(func, wait, args) {
  return setTimeout(function() {
    return func.apply(null, args);
  }, wait);
});
```

1、 运行restArgs()函数，由于function传入了三个参数：func、wait、args，故restArgs()函数中的参数startIndex=2。所以运行返回的函数：

func.call(this, arguments[0], arguments[1], rest);

2、 运行setTimeout()函数，当等待wait毫秒后调用function。

3、 运行func.apply(null, args);

例：

```
var log = _.bind(console.log, console);
_.delay(log, 1000, 'logged later');
//'logged later' 
```

## 6. _.defer

_.defer(function, *arguments) 
延迟调用function直到当前调用栈清空为止，类似使用延时为0的setTimeout方法。对于执行开销大的计算和无阻塞UI线程的HTML渲染时候非常有用。 如果传递arguments参数，当函数function执行时， arguments 会作为参数传入。

```
_.defer = _.partial(_.delay, _, 1);
```

1、 运行_.partial()函数，则函数如下：

_.defer = _.delay.apply(this, 1);

2、 运行_.delay()函数。则函数如下：

```
_.defer = setTimeout(function() {
    return func.apply(null, args);
  }, 1);
```

例：

```
_.defer(function(){ alert('deferred'); });
```

## 7. _.throttle

_.throttle(function, wait, [options]) 
创建并返回一个像节流阀一样的函数，当重复调用函数的时候，至少每隔 wait毫秒调用一次该函数。对于想控制一些触发频率较高的事件有帮助。（愚人码头注：详见：javascript函数的throttle和debounce）

默认情况下，throttle将在你调用的第一时间尽快执行这个function，并且，如果你在wait周期内调用任意次数的函数，都将尽快的被覆盖。如果你想禁用第一次首先执行（开始边界）的话，传递{leading: false}，还有如果你想禁用最后一次执行（结尾边界）的话，传递{trailing: false}。

```
_.throttle = function(func, wait, options) {
  var timeout, context, args, result;
  // 上次执行时间点
  var previous = 0;
  if (!options) options = {};

  // 延迟执行函数
  var later = function() {
    // _.now() 
    // 一个优化的方式来获得一个当前时间的整数时间戳。可用于实现定时/动画功能。
    // 若设定了开始边界不执行选项，上次执行时间始终为0
    previous = options.leading === false ? 0 : _.now();
    timeout = null;
    result = func.apply(context, args);
    if (!timeout) context = args = null;
  };

  var throttled = function() {
    var now = _.now();
    // 首次执行时，如果设定了开始边界不执行选项，将上次执行时间设定为当前时间。
    if (!previous && options.leading === false) previous = now;
    // 延迟执行时间间隔
    var remaining = wait - (now - previous);
    context = this;
    args = arguments;
    // 延迟时间间隔remaining小于等于0，表示上次执行至此所间隔时间已经超过一个时间窗口
    // remaining大于时间窗口wait，表示客户端系统时间被调整过
    if (remaining <= 0 || remaining > wait) {
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
      //如果延迟执行不存在，且没有设定结尾边界不执行选项
    } else if (!timeout && options.trailing !== false) {
      timeout = setTimeout(later, remaining);
    }
    return result;
  };

  throttled.cancel = function() {
    clearTimeout(timeout);
    previous = 0;
    timeout = context = args = null;
  };

  return throttled;
};
```

1、 定义变量timeout(用于判断计时是否完成)、context(用于作用域的存储)、args(用于传入变量的存储)、result(用于存储函数运行后的结果)。

2、 定义变量previous，设定上次执行时间点，初始值为0。

3、 定义延时执行函数later()。

1) 判断是否禁用第一次首先执行，即options.leading为false，若是，则上次执行时间始终为0。若否，则上次执行时间为新生成的整数时间戳 _.now()。

2) 把变量timeout设置为空null。

3) 使用apply的方法，以context为作用域，args为参数，运行func函数，并把返回的结果存到result中。

4) 判断若timeout为空时，context和args都设置为null。

4、 定义主函数throttled()。

1) 定义变量now，存储当前时间的整数时间戳。

2) 若上次执行时间点为0，且否禁用第一次首先执行，即options.leading为false，则把上次执行时间点设为当前时间的整数时间戳。

3) 设置延迟执行时间间隔remaining。并把this作用域赋给context，arguments参数赋给args。

4) 若延迟时间间隔remaining小于等于0(表示上次执行至此所间隔时间已经超过一个时间窗口)，或remaining大于时间窗口wait(表示客户端系统时间被调整过)

a、 若timeout不为空，则运行clearTimeout()函数，清除计时器setTimeout()，并把timeout设置为空。

b、 把上次执行时间点设为当前时间的整数时间戳。

c、 使用apply的方法，以context为作用域，args为参数，运行func函数，并把返回的结果存到result中。

d、 判断若timeout为空时，context和args都设置为null。

5) 如果延迟执行为空，且没有禁用最后一次执行，则以remaining为时间间隔，执行setTimeout()函数并执行延迟函数later()。

6) 返回结果result。

5、 定义throttled.cancel函数，把运行clearTimeout()函数，清除计时器setTimeout()并对于变量previous、timeout、context、args重新赋初始值。

例：

```
var updatePosition = function(){
	console.log(document.body.scrollTop);
};

//开始滚动时，不输出结果，两秒后才开始输出结果，结束滚动后，经过两秒再次输出结果。
var throttled = _.throttle(updatePosition, 2000, {leading: false});

//开始滚动时，先输出第一个结果，两秒后继续输出结果，但滚动结束后不输出最后一个结果
var throttled = _.throttle(updatePosition, 2000, {trailing: false});

//开始滚动时，不输出结果，两秒后才开始输出结果，但滚动结束后不输出最后一个结果
var throttled = _.throttle(updatePosition, 2000, {leading: false,trailing: false});

//开始滚动时，先输出第一个结果，两秒后继续输出结果，结束滚动后，经过两秒再次输出结果
var throttled = _.throttle(updatePosition, 2000);
$(window).scroll(throttled);
```

## 8. _.debounce

_.debounce(function, wait, [immediate]) 
返回 function 函数的防反跳版本, 将延迟函数的执行(真正的执行)在函数最后一次调用时刻的 wait 毫秒之后. 对于必须在一些输入（多是一些用户操作）停止到达之后执行的行为有帮助。 例如: 渲染一个Markdown格式的评论预览, 当窗口停止改变大小之后重新计算布局, 等等.

传参 immediate 为 true， debounce会在 wait 时间间隔的开始调用这个函数 。（愚人码头注：并且在 waite 的时间之内，不会再次调用。）在类似不小心点了提交按钮两下而提交了两次的情况下很有用。

```
_.debounce = function(func, wait, immediate) {
  var timeout, result;

  var later = function(context, args) {
    timeout = null;
    if (args) result = func.apply(context, args);
  };

  var debounced = restArgs(function(args) {
    var callNow = immediate && !timeout;
    if (timeout) clearTimeout(timeout);
    if (callNow) {
      timeout = setTimeout(later, wait);
      result = func.apply(this, args);
    } else if (!immediate) {
      // _.delay(function, wait, *arguments) 
      // 类似setTimeout，等待wait毫秒后调用function。
      // 如果传递可选的参数arguments，当函数function执行时， 
      // arguments 会作为参数传入。
      timeout = _.delay(later, wait, this, args);
    }

    return result;
  });

  debounced.cancel = function() {
    clearTimeout(timeout);
    timeout = null;
  };

  return debounced;
};
```

1、 定义变量timeout(用于判断计时是否完成)、result(用于存储函数运行后的结果)。

2、 定义延时执行函数later()。

1) 把timeout变量设置为null

2) 若存在args参数，则使用apply的方法，以context为作用域，args为参数，运行func函数，并把返回的结果存到result中。

3、 定义主函数debounced()。

1) 运行restArgs()函数，把传入的args生成数组类型。

2) 定义变量callNow，用于判断是否现在执行。当immediate为true且timeout为空时，callNow为true。

3) 判断timeout存在时，运行clearTimeout()函数，清除计时器setTimeout()。

4) 当callNow为真时，运行setTimeout()函数，把函数添加到栈中。并使用apply的方法，以this为作用域，args为参数，运行func函数，并把返回的结果存到result中。wait毫秒后，运行later函数。

5) 当callNow为假且immediate为假时，运行 _.delay()函数，等待wait毫秒后调用later函数。

6) 定义debounced.cancel函数，把运行clearTimeout()函数，清除计时器setTimeout()并对于变量timeout重新赋初始值null。

例：

```
var updatePosition = function(){
	console.log(document.body.scrollTop);
};

// 滚动鼠标时，不输出任何值，当鼠标停下并过两秒后，输出值。
// 其中，当鼠标停下后，当两秒内继续滚动鼠标，则重新计算时间。
var debounce = _.debounce(updatePosition, 2000);

// 当滚动开始时，输出值，当鼠标停下并过两秒后，也不会输出任何值，即只输出一次值。
var debounce = _.debounce(updatePosition, 2000, true);
$(window).scroll(debounce);
```

## 9. _.wrap

_.wrap(function, wrapper)
将第一个函数 function 封装到函数 wrapper 里面, 并把函数 function 作为第一个参数传给 wrapper. 这样可以让 wrapper 在 function 运行之前和之后 执行代码, 调整参数然后附有条件地执行.

```
_.wrap = function(func, wrapper) {
  // _.partial(function, *arguments)
  // 局部应用一个函数填充在任意个数的 
  // arguments，不改变其动态this值。和bind方法很相近。
  // 你可以传递_ 给arguments列表来指定一个不预先填充，但在调用时提供的参数。
  return _.partial(wrapper, func);
};
```

1、 运行 _.partial()函数，即将func函数绑定到wrapper函数中并调用。

例：

```
var hello = function(name) { 
	return "hello: " + name; 
};
var wrap = _.wrap(hello, function(func) {
  return "before, " + func("moe") + ", after";
});
var test = wrap();
console.log(test);
//'before, hello: moe, after'
```

## 10. _.negate

_.negate(predicate) 
返回一个新的predicate函数的否定版本

```
_.negate = function(predicate) {
  return function() {
    return !predicate.apply(this, arguments);
  };
};
```

1、 使用apply的方法，以this为作用域，arguments为参数，运行predicate函数后，并返回得到的false值。

```
var isFalsy = _.negate(Boolean);
var test = _.find([-2, -1, 0, 1, 2], isFalsy);
console.log(test);
//0

var aa = _.negate(function(value){
	return value % 3 != 0;
});
var test2 = _.filter([1, 2, 3, 4, 5, 6], aa);
console.log(test2);
//[3, 6]
```

## 11. _.compose

_.compose(*functions) 
返回函数集 functions 组合后的复合函数, 也就是一个函数执行完之后把返回的结果再作为参数赋给下一个函数来执行. 以此类推. 在数学里, 把函数 f(), g(), 和 h() 组合起来可以得到复合函数 f(g(h()))。

```
_.compose = function() {
  var args = arguments;
  var start = args.length - 1;
  return function() {
    var i = start;
    var result = args[start].apply(this, arguments);
    while (i--) result = args[i].call(this, result);
    return result;
  };
};
```

1、 定义变量args，用于存储functions数组参数。

2、 定义变量start，用于存储第一个开始运行的function位置。

3、 运行function()

1) 定义变量i，用于存储第i个运行的function位置。

2) 定义result，用于存储function的运行结果。

3) 使用apply的方法，以this为作用域，arguments为参数，运行args[start]函数，并把返回的结果赋值给result。

4) 运行while循环函数，以i-- 作为条件，即函数将从右往左运行。使用call的方法，以this为作用域，result为参数，运行args[i]函数，并把返回的结果赋值给result。

5) 运行完毕后，返回result。

例：

```
var greet    = function(name){ return "hi: " + name; };
var exclaim  = function(statement){ return statement.toUpperCase() + "!"; };
var welcome = _.compose(greet, exclaim);
var text = welcome('moe');
console.log(text);
//'hi: MOE!'
```

## 12. _.after

_.after(count, function) 
创建一个函数, 只有在运行了 count 次之后才有效果. 在处理同组异步请求返回结果时, 如果你要确保同组里所有异步请求完成之后才 执行这个函数, 这将非常有用。

```
_.after = function(times, func) {
  return function() {
    if (--times < 1) {
      return func.apply(this, arguments);
    }
  };
};
```

1、 运行function，先把times的值减1，然后判断times的值是否小于1，若是，使用apply的方法，以this为作用域，arguments为参数，运行func函数，并返回结果。

例：

```
function a(arg){
    console.log(arg);
}

var afterA = _.after(3,a);
afterA('a');//不调用
afterA('b');//不调用
afterA('c');//调用  c
afterA('d');//调用  d
```

## 13. _.before

_.before(count, function) 
创建一个函数,调用不超过count 次。 当count已经达到时，最后一个函数调用的结果将被记住并返回。

```
_.before = function(times, func) {
  var memo;
  return function() {
    if (--times > 0) {
      memo = func.apply(this, arguments);
    }
    if (times <= 1) func = null;
    return memo;
  };
};
```

1、 定义memo，永存存储运行结果。

2、 先把times的值减1，然后判断times的值是否大于0，若是，则使用apply的方法，以this为作用域，arguments为参数，运行func函数，并把返回的结果赋值给memo。

3、 判断times是否小于等于1，若是，则把func设置为null

4、 返回memo的值。

例：

```
function a(arg){
    console.log(arg);
}

var beforeA = _.before(3,a);
beforeA('a');//调用  a
beforeA('b');//调用  b
beforeA('c');//不调用
beforeA('d');//不调用
```

## 14. _.once

_.once(function) 
创建一个只能调用一次的函数。重复调用改进的方法也没有效果，只会返回第一次执行时的结果。 作为初始化函数使用时非常有用, 不用再设一个boolean值来检查是否已经初始化完成.

```
_.once = _.partial(_.before, 2);
```

1、 运行 _.partial()函数，即将参数2绑定到 _.before函数中并调用。

2、 运行 _.before函数，即 

_.once =  _.before(2, function);

例：

```
function a(arg){
    console.log(arg);
}

var afterA = _.once(a);
afterA('a');//调用  a
afterA('b');//不调用
afterA('c');//不调用
afterA('d');//不调用
```







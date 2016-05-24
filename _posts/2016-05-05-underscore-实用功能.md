---
layout: post
category: js源码解析
---

# 实用功能(Utility Functions)

## 1. _.noConflict

_.noConflict() 
放弃Underscore的控制变量" _ "。返回Underscore 对象的引用。

```
var root = typeof self == 'object' && self.self === self && self ||
          typeof global == 'object' && global.global === global && global ||
          this;

// 保存变量“_”的初始值，为undefined。          
var previousUnderscore = root._;

// 创建一个安全参考下划线对象在下面使用
// instanceof 运算符是用来在运行时指出对象是否是特定类的一个实例
var _ = function(obj) {
  if (obj instanceof _) return obj;
  if (!(this instanceof _)) return new _(obj);
  this._wrapped = obj;
};

_.noConflict = function() {
  root._ = previousUnderscore;
  return this;
};
```

1、 把root._重新赋值为underscore的原始值previousUnderscore。即"_"对象不起作用

2、 返回this，直接运行this对象的引用

例：

```
var underscore = _.noConflict();
console.log(underscore);
/*
function(obj) {
    if (obj instanceof _) return obj;
    if (!(this instanceof _)) return new _(obj);
    this._wrapped = obj;
}
*/
```

## 2. _.identity

_.identity(value) 
返回与传入参数相等的值. 相当于数学里的: f(x) = x。
这个函数看似无用, 但是在Underscore里被用作默认的迭代器iterator.

```
//保持恒等函数默认迭代。
_.identity = function(value) {
  return value;
};
```

1、 把传入的value原值返回。

例：

```
var stooge = {name: 'moe'};
console.log(stooge === _.identity(stooge));
//true
```

## 3. _.constant

_.constant(value)
创建一个函数，这个函数 返回相同的值 用来作为 _.constant的参数。

```
//常量生产函数，通常在Underscore以外的地方使用
_.constant = function(value) {
  return function() {
    return value;
  };
};
```

1、 返回一个函数，此函数将返回用户自己传入的值。

例：

```
var stooge = {name: 'moe'};
console.log(stooge === _.constant(stooge)());
//true
```

## 4. _.noop

_.noop() 
返回undefined，不论传递给它的是什么参数。 可以用作默认可选的回调参数。

```
_.noop = function(){};
```

1、 用于定义一个没有数据的变量，方便进行下一步操作。且节省内存。若运行此函数将返回undefined。

例：

```
var obj={};
obj.initialize = _.noop;
console.log(obj.initialize);
//function(){}
```

## 5. _.times

_.times(n, iteratee, [context]) 
调用给定的迭代函数n次,每一次调用iteratee传递index参数。生成一个返回值的数组。 

```
_.times = function(n, iteratee, context) {
  var accum = Array(Math.max(0, n));
  iteratee = optimizeCb(iteratee, context, 1);
  for (var i = 0; i < n; i++) accum[i] = iteratee(i);
  return accum;
};
```

1、 定义accum变量，用于存储新数组，且数组长度为0～n中的最大值。

2、 运行optimizeCb函数，并把返回值赋值给iteratee迭代函数。

3、 运行for循环，运行n次迭代函数，并把相应值赋值给accum的相应位置

4、 返回最终结果accum。

例：

```
var test = _.times(3,function(n){ 
	return n*2;
});
console.log(test);
//[0, 2, 4]
```

## 6. _.random

_.random(min, max) 
返回一个min 和 max之间的随机整数。如果你只传递一个参数，那么将返回0和这个参数之间的整数。

```
_.random = function(min, max) {
  if (max == null) {
    max = min;
    min = 0;
  }
  return min + Math.floor(Math.random() * (max - min + 1));
};
```

1、 若max为空时，则把min的值赋值给max，把min的值设置为0。

2、 运行Math.random()函数，随机选取大于等于 0.0 且小于 1.0 的伪随机 double 值。

其中，返回指定范围的随机数(m-n之间)的公式

```
Math.random()*(n-m)+m；
```

3、 运行Math.floor(x)函数，返回小于参数x的最大整数，即对浮点数向下取整。所以max减去min后需要再+1。

例：

```
var test = _.random(0, 100);
console.log(test);
//30

var test2 = _.random(70);
console.log(test2);
//6
```

## 7. _.now

_.now() 
一个优化的方式来获得一个当前时间的整数时间戳。可用于实现定时/动画功能。

```
_.now = Date.now || function() {
  return new Date().getTime();
};
```

1、 先运行Date.now函数，即返回从 1970 年 1 月 1 日至今的毫秒数。

2、 或者运行以下新的Date().getTime()函数，返回的数据与Date.now函数相同。

例：

```
var test = _.now();
console.log(test);
//1462501151316
```

## 8. _.escape

_.escape(string) 
转义HTML字符串，替换&, <, >, ", ', 和 `字符。

```
var escapeMap = {
  '&': '&amp;',
  '<': '&lt;',
  '>': '&gt;',
  '"': '&quot;',
  "'": '&#x27;',
  '`': '&#x60;'
};

var createEscaper = function(map) {
  var escaper = function(match) {
    return map[match];
  };
  // Regexes for identifying a key that needs to be escaped.
  var source = '(?:' + _.keys(map).join('|') + ')';
  var testRegexp = RegExp(source);
  var replaceRegexp = RegExp(source, 'g');
  return function(string) {
    string = string == null ? '' : '' + string;
    return testRegexp.test(string) ? string.replace(replaceRegexp, escaper) : string;
  };
};

_.escape = createEscaper(escapeMap);
```

1、 运行createEscaper(map)函数。其中map对象为escapeMap。

2、 运行 _.key(map)函数，取map对象中的属性名并组成一个数组，运行join()方法，把数组中的所有元素放入一个字符串'|'，每个元素之间通过指定的分隔符'|'进行分隔。用于接下来的正则表达式。例：(red|blue|green)，表示查找任何指定数据的选项。

其中，正则表达式中使用了符号(?:)，代表 非捕获型括号，只是分组的作用不捕获。

3、 运行function(string)，检测若string为空时，则string为空字符串，否则把传入的字符串赋值给string。

4、 运行test() 方法，检测一个字符串是否匹配testRegexp正则表达式，若匹配，则运行escaper(match)函数，把符号属性match相应的值map[match]赋值给escaper。并运行replace()方法，把escaper得到的值代替匹配正则表达式replaceRegexp的值，并返回string的最终结果。若不匹配，则直接返回string字符串。

例：

```
var test = _.escape('Curly, Larry & Moe');
console.log(test);
//Curly, Larry &amp; Moe

var test2 = _.escape('2 < 1 > 0');
console.log(test2);
//2 &lt; 1 &gt; 0

var test3 = _.escape('a,"b",c');
console.log(test3);
//a,&quot;b&quot;,c

var test4 = _.escape("a,'b',c");
console.log(test4);
//a,&#x27;b&#x27;,c

var test5 = _.escape("`n");
console.log(test5);
//&#x60;n
```

## 9. _.unescape

_.unescape(string) 
和escape相反。转义HTML字符串，替换'&amp;', '&lt;', '&gt;', '&quot;', '&#x60;', 和 '&#x2F;'字符。

```
var escapeMap = {
  '&': '&amp;',
  '<': '&lt;',
  '>': '&gt;',
  '"': '&quot;',
  "'": '&#x27;',
  '`': '&#x60;'
};
var unescapeMap = _.invert(escapeMap);

_.unescape = createEscaper(unescapeMap);
```

1、 运行_.invert()函数，把escapeMap对象中的键（keys）和值（values）对换。

2、 与 _.escape()函数一样，运行createEscaper()函数，并得到返回的结果。

例：

```
var test = _.unescape('Curly, Larry &amp; Moe');
console.log(test);
//Curly, Larry & Moe

var test2 = _.unescape('2 &lt; 1 &gt; 0');
console.log(test2);
//2 < 1 > 0

var test3 = _.unescape('a,&quot;b&quot;,c');
console.log(test3);
//a,"b",c

var test4 = _.unescape("a,&#x27;b&#x27;,c");
console.log(test4);
//a,'b',c

var test5 = _.unescape("&#x60;n");
console.log(test5);
//`n
```

## 10. _.result

_.result(object, property, [defaultValue]) 
如果指定的property 的值是一个函数，那么将在object上下文内调用它;否则，返回它。如果提供默认值，并且属性不存在，那么默认值将被返回。如果设置defaultValue是一个函数，它的结果将被返回。

```
_.result = function(object, prop, fallback) {
  var value = object == null ? void 0 : object[prop];
  if (value === void 0) {
    value = fallback;
  }
  return _.isFunction(value) ? value.call(object) : value;
};
```

1、 若object为空，则value的值为undefined，否则为object[prop]。

2、 若value的值为undefined，则把fallback的值赋值给value。

3、 若value为函数类型，则以object对象为作用域运行value，并返回得到的结果，若否，则直接返回value的值。

例：

```
var object = {
	cheese: 'crumpets', 
	stuff: function(){ return 'nonsense'; }
};
var test = _.result(object, 'cheese');
console.log(test);
//"crumpets"

var test2 = _.result(object, 'stuff');
console.log(test2);
//"nonsense"

var test3 = _.result(object, 'cheese', 'ham');
console.log(test3);
//"crumpets"

var test4 = _.result(object, 'meat', 'ham');
console.log(test4);
//"ham"

```

## 11. _.uniqueId

_.uniqueId([prefix]) 
为需要的客户端模型或DOM元素生成一个全局唯一的id。如果prefix参数存在， id 将附加给它。

```
var idCounter = 0;
_.uniqueId = function(prefix) {
  var id = ++idCounter + '';
  return prefix ? prefix + id : id;
};
```

1、 idCounter的初始值为0，运行uniqueId函数，先把idCounter变量自加1，再加上一个空的字符串，把其转换为字符串赋值给id变量。

2、 若有传入prefix的值，则返回prefix加id的字符串值，若没有传入prefix的值，则直接返回id字符串。

例：

```
var test = _.uniqueId('contact_');
console.log(test);
//contact_1

var test2 = _.uniqueId('unique_');
console.log(test2);
//unique_2

```

## 12. _.iteratee

_.iteratee(value, [context]) 
一个重要的内部函数用来生成可应用到集合中每个元素的回调， 返回想要的结果 - 无论是等式，任意回调，属性匹配，或属性访问。 
通过 _.iteratee转换判断的Underscore 方法的完整列表是 map, find, filter, reject, every, some, max, min, sortBy, groupBy, indexBy, countBy, sortedIndex, partition, 和 unique.

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

_.iteratee = builtinIteratee = function(value, context) {
  return cb(value, context, Infinity);
};
```

1、 运行cb()函数，并返回得到的值。

例：

```
var stooges = [
	{name: 'curly', age: 25}, 
	{name: 'moe', age: 21}, 
	{name: 'larry', age: 23}
];

var test = _.map(stooges, _.iteratee('age'));
console.log(test);
//[25, 21, 23]

```

## 13. _.mixin

_.mixin(object) 
允许用您自己的实用程序函数扩展Underscore。传递一个 {name: function}定义的哈希添加到Underscore对象，以及面向对象封装。

```
var chainResult = function(instance, obj) {
  return instance._chain ? _(obj).chain() : obj;
};

// 添加自己的函数到underscore对象中。
_.mixin = function(obj) {
  // _.functions(object) 
  // 返回一个对象里所有的方法名, 而且是已经排序的 — 也就是说, 对象里每个方法(属性值是一个函数)的名称.
  _.each(_.functions(obj), function(name) {
    var func = _[name] = obj[name];
    _.prototype[name] = function() {
      var args = [this._wrapped];
      push.apply(args, arguments);
      return chainResult(this, func.apply(_, args));
    };
  });
  return _;
};

// 添加所有的underscore函数到wrapper对象中。
_.mixin(_);
```

1、 运行函数_.functions(object)，按顺序返回以函数名组成的数组。

2、 运行函数_.each()，遍历新添加的每个函数名

1) 新建underscore函数 _[name]，且吧obj中对应name的key值赋给此函数。

2） 定义运行underscore的原型prototype中的name函数。

a) 定义args，且其值为数组[this._wrapped]。

b) 不断push更新args数组。

c) 运行chainResult()函数，判断instance._chain存在时，返回_(obj).chain()的运行结果，若不存在，则直接返回结果obj。其中obj为运行func.apply( _, args))后的结果。

3、 若没有新的函数传入，则直接返回underscore中的所有函数。

例：

```
_.mixin({
  	capitalize: function(string) {
    	return string.charAt(0).toUpperCase() + string.substring(1).toLowerCase();
  	}
});

var test = _("fabio").capitalize();
console.log(test);
// "Fabio"


var test2 = _.capitalize("fabio");
console.log(test2);
// "Fabio"
```

在underscore中，其把一些常用的函数也添加到wrapper中。

```
var ArrayProto = Array.prototype;

// 添加所有增减数组函数到wrapper变量。
_.each(['pop', 'push', 'reverse', 'shift', 'sort', 'splice', 'unshift'], function(name) {
  var method = ArrayProto[name];
  _.prototype[name] = function() {
    var obj = this._wrapped;
    method.apply(obj, arguments);
    if ((name === 'shift' || name === 'splice') && obj.length === 0) delete obj[0];
    return chainResult(this, obj);
  };
});

// 添加所有访问数组函数到wrapper变量。
_.each(['concat', 'join', 'slice'], function(name) {
  var method = ArrayProto[name];
  _.prototype[name] = function() {
    return chainResult(this, method.apply(this._wrapped, arguments));
  };
});
```

1、 pop() 方法用于删除并返回数组的最后一个元素。

2、 push() 方法可向数组的末尾添加一个或多个元素，并返回新的长度。

3、 reverse() 方法用于颠倒数组中元素的顺序。

4、 shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。

5、 sort() 方法用于对数组的元素进行排序。

6、 splice() 方法可删除从 index 处开始的零个或多个元素，并且用参数列表中声明的一个或多个值来替换那些被删除的元素。

7、 unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。

8、 concat() 方法用于连接两个或多个数组。

9、 join() 方法用于把数组中的所有元素放入一个字符串。

10、 slice() 方法可从已有的数组中返回选定的元素。

## 14. _.templateSettings

此为模版符号设置对象，主要用于修改 _.template函数中插入变量的符号。具体举例请看 _.template函数的解释。

```
_.templateSettings = {
  evaluate: /<%([\s\S]+?)%>/g,
  interpolate: /<%=([\s\S]+?)%>/g,
  escape: /<%-([\s\S]+?)%>/g
};
```

## 15. _.template

_.template(templateString, [settings]) 
将 JavaScript 模板编译为可以用于页面呈现的函数, 对于通过JSON数据源生成复杂的HTML并呈现出来的操作非常有用。 模板函数可以使用 <%= … %>插入变量, 也可以用<% … %>执行任意的 JavaScript 代码。 如果您希望插入一个值, 并让其进行HTML转义,请使用<%- … %>。 当你要给模板函数赋值的时候，可以传递一个含有与模板对应属性的data对象 。 如果您要写一个一次性的, 您可以传对象 data 作为第二个参数给模板 template 来直接呈现, 这样页面会立即呈现而不是返回一个模板函数. 参数 settings 是一个哈希表包含任何可以覆盖的设置 _.templateSettings.

```
var noMatch = /(.)^/;

var escapes = {
  "'": "'",
  '\\': '\\',
  '\r': 'r',
  '\n': 'n',
  '\u2028': 'u2028',
  '\u2029': 'u2029'
};

var escapeRegExp = /\\|'|\r|\n|\u2028|\u2029/g;

var escapeChar = function(match) {
  return '\\' + escapes[match];
};
```

```
_.template = function(text, settings, oldSettings) {
  if (!settings && oldSettings) settings = oldSettings;
  // _.defaults(object, *defaults) 
  // 用defaults对象填充object 中的undefined属性。 
  settings = _.defaults({}, settings, _.templateSettings);

  // source 属性用于返回模式匹配所用的文本。
  var matcher = RegExp([
    (settings.escape || noMatch).source,
    (settings.interpolate || noMatch).source,
    (settings.evaluate || noMatch).source
  ].join('|') + '|$', 'g');

  // 编译模板源,适当转义字符串。
  var index = 0;
  var source = "__p+='";
  // replace() 方法用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。
  text.replace(matcher, function(match, escape, interpolate, evaluate, offset) {
    source += text.slice(index, offset).replace(escapeRegExp, escapeChar);
    index = offset + match.length;

    if (escape) {
      source += "'+\n((__t=(" + escape + "))==null?'':_.escape(__t))+\n'";
    } else if (interpolate) {
      source += "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'";
    } else if (evaluate) {
      source += "';\n" + evaluate + "\n__p+='";
    }

    // Adobe VMs need the match returned to produce the correct offset.
    return match;
  });
  source += "';\n";

  // If a variable is not specified, place data values in local scope.
  if (!settings.variable) source = 'with(obj||{}){\n' + source + '}\n';

  source = "var __t,__p='',__j=Array.prototype.join," +
    "print=function(){__p+=__j.call(arguments,'');};\n" +
    source + 'return __p;\n';

  var render;
  try {
    render = new Function(settings.variable || 'obj', '_', source);
  } catch (e) {
    e.source = source;
    throw e;
  }

  var template = function(data) {
    return render.call(this, data, _);
  };

  // Provide the compiled source as a convenience for precompilation.
  var argument = settings.variable || 'obj';
  template.source = 'function(' + argument + '){\n' + source + '}';

  return template;
};
```

1、 若settings不存在，且oldSettings存在，则吧oldSettings赋值给settings。

2、 运行join('|')函数，把数组中的分隔符更换为|。并运行正则表达式RegExp()函数。

3、 定义index的初始值为0，source的初始值为字符串"__p+='"。

4、 运行replace()方法：

1) 取text中index到offset位置之间的字符串，并使用replace()函数，替换一些字符后，把得到的text结果加上原来的source数据，把最后结果返回给source。

2) 更新index的值。

3) 若escape，则运行以下表达式：

```
source += "'+\n((__t=(" + escape + "))==null?'':_.escape(__t))+\n'";
```

4) 若interpolate，则运行以下表达式：

```
source += "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'";
```

5) 若evaluate，则运行以下表达式：

```
source += "';\n" + evaluate + "\n__p+='";
```

6) 返回match的值。

5、 在source数据后面添加换行标志。

6、 若settings.variable不存在，则运行以下代码：

```
source = 'with(obj||{}){\n' + source + '}\n';
```

7、 继续对source变量进行修改。

8、 定义变量render，并且运行try...catch...函数。若没有报错，则新建function函数，并赋值给source。否则把source赋值给e.source，并抛出异常e。

9、 运行template函数。

10、 定义argument的值为settings.variable或字符串'obj'。

11、 把template.source赋值为function的字符串。

12、 返回template的值。

例：

```
var compiled = _.template("hello: <%= name %>");
var test = compiled({name: 'moe'});
console.log(test);
// hello: moe


var template = _.template("<b><%- value %></b>");
var test2 = template({value: '<script>'});
console.log(test2);
// <b>&lt;script&gt;</b>
```

您也可以在JavaScript代码中使用 print. 有时候这会比使用 <%= ... %> 更方便。

```
var compiled = _.template("<% print('Hello ' + epithet); %>");
var test3 = compiled({epithet: "stooge"});
console.log(test3);
// Hello stooge
```

如果ERB式的分隔符您不喜欢, 您可以改变Underscore的模板设置, 使用别的符号来嵌入代码.定义一个 interpolate 正则表达式来逐字匹配嵌入代码的语句, 如果想插入转义后的HTML代码则需要定义一个 escape 正则表达式来匹配,还有一个 evaluate 正则表达式来匹配您想要直接一次性执行程序而不需要任何返回值的语句.您可以定义或省略这三个的任意一个.例如, 要执行Mustache.js类型的模板:

```
_.templateSettings = {
  	interpolate: /\{\{(.+?)\}\}/g
};

var template = _.template("Hello {{ name }}!");
var test4 = template({name: "Mustache"});
console.log(test4);
// "Hello Mustache!"
```

默认的, template 通过 with 语句来取得 data 所有的值. 当然, 您也可以在 variable 设置里指定一个变量名. 这样能显著提升模板的渲染速度.

```
var test6 = _.template("Using 'with': <%= data.answer %>", {variable: 'data'})({answer: 'no'});
console.log(test6);
// Using 'with': no
```

预编译模板对调试不可重现的错误很有帮助. 这是因为预编译的模板可以提供错误的代码行号和堆栈跟踪, 有些模板在客户端(浏览器)上是不能通过编译的 在编译好的模板函数上, 有 source 属性可以提供简单的预编译功能.

```
<script>
  JST.project = <%= _.template(jstText).source %>;
</script>
```

# 链式语法(Chaining)

对一个对象使用 chain 方法, 会把这个对象封装并 让以后每次方法的调用结束后都返回这个封装的对象, 当您完成了计算, 可以使用 value 函数来取得最终的值. 以下是一个同时使用了 map/flatten/reduce 的链式语法例子, 目的是计算一首歌的歌词里每一个单词出现的次数.

```
var lyrics = [
  {line: 1, words: "I'm a lumberjack and I'm okay"},
  {line: 2, words: "I sleep all night and I work all day"},
  {line: 3, words: "He's a lumberjack and he's okay"},
  {line: 4, words: "He sleeps all night and he works all day"}
];

var test = _.chain(lyrics)
  .map(function(line) { return line.words.split(' '); })
  .flatten()
  .reduce(function(counts, word) {
    counts[word] = (counts[word] || 0) + 1;
    return counts;
  }, {})
  .value();

console.log(test);

/*
{
	He:1,
	He's:1,
	I:2,
	I'm:2,
	a:2,
	all:4,
	and:4,
	day:2,
	he:1,
	he's:1,
	lumberjack:2,
	night:2,
	okay:2,
	sleep:1,
	sleeps:1,
	work:1,
	works:1
}
*/
```

## 1. _.chain

返回一个封装的对象. 在封装的对象上调用方法会返回封装的对象本身, 直到 value 方法调用为止.

```
var _ = function(obj) {
  if (obj instanceof _) return obj;
  if (!(this instanceof _)) return new _(obj);
  this._wrapped = obj;
};

_.chain = function(obj) {
  var instance = _(obj);
  instance._chain = true;
  return instance;
};
```

1、 运行_(obj)下划线函数，得到instance._wrapped=obj。

2、 设置instance._chain为true，以此判断函数是否为链式结构。

3、 返回instance的结果。

例：

```
var stooges = [
	{name: 'curly', age: 25}, 
	{name: 'moe', age: 21}, 
	{name: 'larry', age: 23}
];
var youngest = _.chain(stooges)
  .sortBy(function(stooge){ return stooge.age; })
  .map(function(stooge){ return stooge.name + ' is ' + stooge.age; })
  .first()
  .value();

console.log(youngest);
// moe is 21
```

## 2. _.value

_(obj).value() 
获取封装对象的最终值.

```
// 从wrapped和链接chained对象提取结果。
_.prototype.value = function() {
  return this._wrapped;
};

// 提供一些方法用于打开代理引擎操作,比如JSON stringification和算法。
_.prototype.valueOf = _.prototype.toJSON = _.prototype.value;

_.prototype.toString = function() {
  return String(this._wrapped);
};
```

1、 返回最后得到的this._wrapped结果。

例：

```
var test = _([1, 2, 3]).value();
console.log(test);
//[1, 2, 3]
```


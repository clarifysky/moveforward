# 学习ES2015

一个ECMAScript 2015特性的细节概览。基于Luke Hoban的es6features 仓库。

> ## es6features
> 这个文档原搬自Luke Hoban的牛逼的[es6features](https://git.io/es6features)，去GitHub上给他打个星！

> ## REPL
> 确保在在线的[REPL](https://babeljs.io/repl)上测试这些特性

## 介绍

ECMAScript 2015是一个在2015年六月正式批准的ECMAScript标准。ES2015是对JS的一次重大更新，自ES5以来第一次重要更新是在2009年的标准化。这些特性在主JS引擎中的实现[正在路上](https://kangax.github.io/es5-compat-table/es6/)。

## ECMAScript 2015特性

### 箭头和关键词This

箭头是方法的简写，它使用`=>`语法。他们在语句构成上和C#，Java 8 还有CoffeScript中的相关特性很相似。它们同时支持表达式和声明体。不同于方法，箭头共享相同的关键词`this`作为它们的包含代码：

```js
// 表达式体
var odds = events.map(v => v + 1);
var nums = events.map((v, i) => v + i);

// 声明体
nums.forEach(v => {
  if (v % 5 === 0 )
    fives.push(v);
});

// 关键词this
var bob = {
  _name: "Bob",
  _friends: [],
  printFriends() {
    this._friends.forEach(f => console.log(this._name + "knows " + f));
  }
};
```

### 类

ES2015中的类是一个简单的在原形基础的OO模型上的语法糖。具有一个单一的简便陈述形式使类模型简单易用，并且鼓励互通性。类支持原形基础的继承，父类调用，实例和静态方法还有构造器。

```js
class SkinnerMesh extends THREE.Mesh {
  constructor(geometry, materials) {
    super(geometry, materials);

    this.idMatrix = SkinnerMesh.defaultMatrix();
    this.bones = [];
    this.boneMatrices = [];
    //...
  }

  update(camera) {
    //...
    super.update();
  }

  static defaultMatrix() {
    return new THREE.Matrix4();
  }
}
```

### 增强的对象字面量

对象字面量被扩展来在构造，`foo: foo`指派速记，定义方法和发起父类调用时支持设置原形。同时，这些还将对象字面量和类声明放得更近，并使基于对象的设计从一些相似的便利中获益。

```js
var obj = {
  // 设置原形. "__proto__"或'__proto__'都可以。
  __proto__: theProtoObj,
  // 对于重复的__proto__属性，计算属性名字不会设置原形或触发早期错误。
  ['__proto__']: somethingElse,
  // 'handler: handler'的简写
  handler,
  // 方法
  toString() {
    // 父调用
    return "d " + super.toString();
  },
  // 计算（动态）属性名
  ["prop_" + (() => 42)() ]: 42
};
```

> `__proto__`属性需要原生支持，并在早期的ECMAScript版本中被废弃了。多数引擎已经支持了这个属性，但有一些没有。同时，注意，只有[web浏览器](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-additional-ecmascript-features-for-web-browsers)需要实现它，就像在[Annex B](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-object.prototype.__proto__)中一样。它同时在Node中可用。

### 模版字符串

模版字符串提供语法糖用来构建字符串。这类似于Perl，Python和其他语言中的字符串篡改特性。有个可选的情况，可以加入标签来允许自定义字符串构建，避免注入攻击或从字符串内容中构建更高级的数据结构。

```js
// 基本的字面量创建
`This is a pretty little template string.`

// 多行字符串
`In ES5 this is
not legal`

// 篡改变量绑定
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

// 转义模版字符串
String.raw`In ES5 "\n" is a line-feed.`

// 构建一个HTTP请求前缀用来诠释替换和构造。
GET`http://foo.org/bar?a=${a}&b=${b}
Content-Type: application/json
X-Credentials: ${credentials}
{ "foo": ${foo},
  "bar": ${bar}}`(myOnReadyStateChangeHandler);
```

### 变性

变性允许绑定使用模式匹配，支持匹配数组和对象。变性是故障弱化的，与标准的对象查阅类似，例如`foo["bar"]`，在没有找到时产出一个`undefined`值。

```js
// 列表匹配
var [a, ,b] = [1,2,3];
a === 1;
b === 3;

// 对象匹配
var { op: a, lhs: { op: b }, rhs: c } = getASTNode()

// 对象匹配简写，在域内绑定`op`，`lhs`和`rhs`
var { op, lhs, rhs } = getASTNode()

// 可被用于参数位置
function g({name: x}) {
  console.log(x);
}
g({name: 5});

// 软故障变性
var [a] = [];
a === undefined;

// 带有默认值的软失误变性
var [a = 1] = [];
a === 1;

// 变性+默认参数
function r({ x, y, w = 10, h = 10 }) {
  return x + y + w + h;
}
r({ x: 1, y: 2 }) === 23
```

### 默认＋剩余＋传播

被调用者评估的默认参数值。在一个功能调用中，将数组转为连续的参数。将实验参数绑定到数组上。参数其余地方直接对应。

```js
function f(x, y = 12) {
  // 如果没有传递，y就是12，或者y被作为undefined传递
  return x + y;
}
f(3) == 15

function f(x, ...y) {
  // y是一个数组
  return x * y.length;
}
f(3, "hello", true) == 6

function f(x, y, z) {
  return x + y + z;
}
// 将数组的每个元素作为参数传递
f(...[1, 2, 3]) == 6
```

### Let+Const

代码块作用域的绑定结构。`let`是新的`var`。`const`是单一指派的。静态限制阻止在分派前使用。

```js
function f() {
  let x;
  {
    // 正确，代码块作用于的name
    const x =  "sneaky";
    // 错误，常量
    x = "foo";
  }
  // 可以，这个被声明为`let`
  x = "bar";
  // 错误，已经在作用于内声明过了
  let x = "inner";
}
```

### 迭代器＋For...Of

迭代器对象允许自定义迭代，就像CLR IEnumerable或Java Iterable。归纳`for..in`为自定义的基于迭代器的迭代`for..of`，不要要求意识为一个数组，开启惰性设计模式，像LINQ那样。

```js
let fibonacci = {
  [Symbol.iterator]() {
    let pre = 0, cur = 1;
    return {
      next() {
        [pre, cur] = [cur, pre + cur];
        return { done: false, value: cur }
      }
    }
  }
}

for (var n of fibonacci) {
  // 在1000的时候阶段序列
  if (n > 1000)
    break;
  console.log(n);
}
```

迭代基于这些鸭形的接口（仅使用[TypeScript](http://typescriptlang.org/)类型语法来阐述):

```js
interface IteratorResult {
  done: boolean;
  value: any;
}

interface Iterator {
  next(): IteratorResult;
}

interface Iterable {
  [Symbol.iterator](): Iterator
}
```

> ### 由polyfill支持

> 要使用迭代器，你必须包含Babel [polyfill](https://babeljs.io/docs/usage/polyfill).

### 生成器

生成器使用`function*`和`yield`简化迭代器著作。声明为function*的方法会返回一个生成器实例。生成器是迭代器的子类型，它包含有额外的`next`和`throw`。这允许值流回到生成器中，因此`yield`是一个表达形式，它将返回一个值（或是抛出）。

注意：也可以用来启用'await'——像异步编程那样，参见ES7 `await`[提议](https://github.com/lukehoban/ecmascript-asyncawait)

```js
var fibonacci = {
  [Symbol.iterator]: function*() {
      var pre = 0, cur = 1;
      for (;;) {
        var temp = pre;
        pre = cur;
        cur += temp;
        yield cur;
      }
  }
}

for (var n of fibonacci) {
  // 在1000的时候截断序列
  if (n > 1000)
    break;
  console.log(n);
}
```

生成器接口是（仅使用[TypeScript](http://typescriptlang.org/)类型语法来阐述）：

```js
interface Generator extends Iterator {
  next(value?: any): IteratorResult;
  throw(exception: any);
}
```

> ### 由polyfill支持

> 为了使用生成器，你必须包含Babel [polyfill](https://babeljs.io/docs/usage/polyfill).

### Comprehensions

在Babel 6.0中移除了

### Unicode

没有中断的附加物支持完整的Unicode，包括新的来自字符串的的unicode字面量和新的处理代码点的正则表达式`u`模式，如同新API在21位代码点处理字符串一样。这些附加物支持构建全局应用：

```js
// 和ES5.1一样
"𠮷".length == 2

// 新的正则表达行为，'u'
"𠮷".match(/./u)[0].length == 2

// 新形式
"\u{20BB7}" == "𠮷" == "\uD842\uDFB7"

// 新的字符串ops
"𠮷".codePointAt(0) == 0x20BB7
// for-of 迭代代码点
for (var c of "𠮷") {
  console.log(c);
}
```

### 模块

语言层级的对模块的支持，为了组件定义。组编自流行的JavaScript模块领导者（AMD, CommonJS）。运行时行为由一个寄存定义的默认载入器定义。不明确的异步模型——没有代码会执行知道请求的模块可用并被处理。

```js
// lib/math.js
export function sum(x, y) {
  return x + y;
}
export var pi = 3.141593;

// app.js
import * as math from "lib/math";
console.log("2π = " + math.sum(math.pi, math.pi));

//  ohterApp.js
import {sum, pi} from "lib/math";
console.log("2π = " + sum(pi, pi));
```
一些额外的特性，包括`export default`和`export *`:

```js
// lib/mathplusplus.js
export * from "lib/math";
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}

// app.js
import exp, {pi, e} from "lib/mathplusplus";
console.log("e^π = " + exp(pi));
```

> ### 模块格式化器

> Babel可以将ES2015模块编译成不同的格式，包括Common.js， AMD，System，和UMD，你甚至可以创建你自己的。更多细节请查看[模块文档](https://babeljs.io/docs/usage/modules)。

### 模块载入器

> ### 不是ES2015的一部分

> 这个作为ECMAScript 2015规格书中的实现定义遗留。最终的标准会在WHATWG的[载入器规格书](https://whatwg.github.io/loader/)中，但那现正在处理中。下面的是来自之前的ES2015草稿：

模块载入器支持：
- 动态载入
- 状态隔离
- 全局命名空间隔离
- 编纂钩子
- 嵌套虚拟化

默认的模块载入器可以被配置，新的载入器可以被构建来评估和载入代码，在隔离或受限情况下。

```js
// 动态加载——'System'是默认的载入器
System.import("lib/math").then(function(m) {
  alert("2π = " + m.sum(m.pi, m.pi));
});

// 创建一个执行沙盒——新载入器
var loader = new Loader({
  global: fixup(window)  // 替换'console.log'
});
loader.eval("console.log(\"hello world!\");");

// 直接操纵模块缓存
System.get("jquery");
System.set("jquery", Module({$: $})); // 警告：没有最终定案
```

> ### 需要额外的polyfill

> 由于Babel默认使用common.js模块，所以他没有包含polyfill用于模块载入器API。可以从[这里](https://github.com/ModuleLoader/es6-module-loader)获取它。

> ### 使用模块载入器

> 为了使用这个，你将需要告诉Babel使用`system`模块格式化器。同样请查看[System.js](https://github.com/systemjs/systemjs)

### Map+Set+WeakMap+WeakSet

常见算法的有效的数据结构。WeakMaps提供leak-free object-key'd side tables.

```js
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Maps
var m = new Map();
m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;

// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });
// 因为被添加的对象没有其他引用，他不在集合中被持有。
```

> ### 由polyfill支持

> 为了在所有环境中支持Maps，Sets，WeakMaps，和WeakSets，你必须包含Babel [plyfill](https://babeljs.io/docs/usage/polyfill).


### Proxies

Proxies允许针对宿主对象的带有全部可用行为的对象创建。可被用于拦截，对象虚拟化，日志／简要等。

```js
// 代理一个正常对象
var target = {};
var handler = {
  get: function (receiver, name) {
    return `Hello, ${name}`;
  }
};

var p = new Proxy(target, handler);
p.world === "Hello, world!";

// 代理一个功能对象
var target = function() { return "I am the target"; };
var handler = {
  apply: function(receiver, ...args) {
    return "I am the proxy";
  }
};

var p = new Proxy(target, handler);
p() === "I am the proxy";
```

有一些对运行时元操作可用的捕捉器：

```js
var handler =
{
  // target.prop
  get: ...,
  // target.prop = value
  set: ...,
  // 'prop' in target
  has: ...,
  // delete target.prop
  deleteProperty: ...,
  // target(...args)
  apply: ...,
  // new target(...args)
  construct: ...,
  // Object.getOwnPropertyDescriptor(target, 'prop')
  getOwnPropertyDescriptor: ...,
  // Object.defineProperty(target, 'prop', descriptor)
  defineProperty: ...,
  // Object.getPrototypeOf(target), Reflect.getPrototypeOf(target),
  // target.__proto__, object.isPrototypeOf(target), object instanceof target
  getPrototypeOf: ...,
  // Object.setPrototypeOf(target), Reflect.setPrototypeOf(target)
  setPrototypeOf: ...,
  // for (let i in target) {}
  enumerate: ...,
  // Object.keys(target)
  ownKeys: ...,
  // Object.preventExtensions(target)
  preventExtensions: ...,
  // Object.isExtensible(target)
  isExtensible :...
}
```

> ### 不受支持的特性

> 由于ES6的限制，代理不能被编译或填补。在[多种JavaScript引擎](https://kangax.github.io/compat-table/es6/#Proxy)中查看支持情况。

### 符号

符号允许对对象状态开启访问控制。符号允许属性被键为`string`（如在ES5中）或是`symbol`。符号是新的原始类型。可选的`name`参数用于调试——但那不是身份的一部分。符号是唯一的（像gensym），但不是私有的，因为它们被通过如`Object.getOwnPropertySymbols`的反射特性释放出去了。

```js
(function() {

  // 模块域符号
  var key = Symbol("key");

  function MyClass(privateData) {
    this[key] = privateData;
  }

  MyClass.prototype = {
    doStuff: function() {
      ... this[key] ...
    }
  };

  // Babel的受限支持，完整支持需要原始实现
  typeof key === "symbol"
})();

var c = new MyClass("hello")
c["key"] === undefined
```

> ### 来自polyfill的受限支持

> 受限的支持需要Babel polyfill，由于语言的限制，一些特性不能被编译或填充。查看core.js的[caveats section])(https://github.com/zloirock/core-js#caveats-when-using-symbol-polyfill)获取更多细节。

### 子类内置

在ES2015中，内置的如`Array`，`Date`和DOM`Element`s可以被子类化。

```js
// User code of Array subclass
class MyArray extends Array {
    constructor(...args) { super(...args); }
}

var arr = new MyArray();
arr[1] = 12;
arr.length == 2
```
> ### 部分支持

> 内置的子类化能力应当按具体情况评估，因为如`HTMLElement`这样的类可以被子类化，而`Date`，`Array`，`Error`这些不能，这是因为ES5引擎的限制。

### Math + Number + String + Object APIs

多个新库附加物，包括核心数学库，数组转换助手和用于拷贝的对象指派：

```js
Number.EPSILON
Number.isInteger(Infinity) // false
Number.isNaN("NaN") // false

Math.acosh(3) // 1.762747174039086
Math.hypot(3, 4) // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2

"abcde".includes("cd") // true
"abc".repeat(3) // "abcabcabc"

Array.from(document.querySelectorAll("*")) // Returns a real Array
Array.of(1, 2, 3) // Similar to new Array(...), but without special one-arg behavior
[0, 0, 0].fill(7, 1) // [0,7,7]
[1,2,3].findIndex(x => x == 2) // 1
["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys() // iterator 0, 1, 2
["a", "b", "c"].values() // iterator "a", "b", "c"

Object.assign(Point, { origin: new Point(0,0) })
```

> ### 来自polyfill的受限的支持

> 多数这些APIs都被Babel polyfill支持，然而，由于多种原因，一些确定的特性被忽略了（如，`String.prototype.normalize`需要大量附加代码来支持）。你可以在[这里](https://github.com/addyosmani/es6-tools#polyfills)找到更多的填充物。

### 二进制和八进制字面量

为二进制（'b'）和八进制（'o'）添加了两种新的数字字面量。

```js
0b111110111 === 503 // true
0o767 === 503 // true
```

> ### 仅支持字面量形式

> Babel只能转换`0o767`而不是`Number("0o767")`。

### 承诺

承诺是个异步编程库。承诺是个或许在未来可用的值的一类展现。承诺被用在多个JavaScript库中。

```js
function timeout(duration = 0) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, duration);
    })
}

var p = timeout(1000).then(() => {
    return timeout(2000);
}).then(() => {
    throw new Error("hmm");
}).catch(err => {
    return Promise.all([timeout(100), timeout(200)]);
})
```

> ### 由polyfill支持

> 为了支持承诺，你必须包含Babel [polyfill](https://babeljs.io/docs/usage/polyfill)。

### 反射API

全反射API释放了对向上的运行时级别的元操作。这是代理API效果上的反转，并允许发起对应于同样的作为代理捕捉器的元操作的调用。尤其对实现代理有效。

```js
var O = {a: 1};
Object.defineProperty(O, 'b', {value: 2});
O[Symbol('c')] = 3;

Reflect.ownKeys(O); // ['a', 'b', Symbol(c)]

function C(a, b){
  this.c = a + b;
}
var instance = Reflect.construct(C, [20, 22]);
instance.c; // 42
```

> ### 由polyfill支持

> 要使用反射API，你必须包含Babel [polyfill](https://babeljs.io/docs/usage/polyfill).

### 尾部调用

在尾部的调用被确保不会使栈无界增长。使得递归算法在无界输入面前是安全的。

```js
function factorial(n, acc = 1) {
    "use strict";
    if (n <= 1) return acc;
    return factorial(n - 1, n * acc);
}

// 会在今天多数的实现中形成栈溢出，
// 但在ES2015中，针对武断的输入是安全的
factorial(100000)
```

> ### 在Babel 6中被临时移除了

> 由于全局支持尾调用的复杂性和性能冲击，只有明确的自引用是受支持的。因为别的bugs被移除了，将来会重新实现。

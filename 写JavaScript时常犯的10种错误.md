---
date: 2015-12-24 18:12
status: public
tag: JavaScript
title: 写JavaScript时常犯的10种错误
url: /10-most-common-javascript-mistakes
---

时至今日，JavaScript 无疑已经几乎是所有web应用开发的核心技术了。在过去的几年中，随着大量强大的JavaScript库， 单页面应用框架， 数据可视化，动画，甚至服务端Javsscript 平台的激增，JavaScript 在 Web APP 开发 中变得越来越常见，理所当然的成为了一个大神成长之路上必点的技能点。

乍一看, JavaScript 貌似很简单。 但事实是，如果仅仅是用JavaScript在网页上实现一些基本功能的话对任何一位有经验的开发者来说都是很简单的，哪怕他之前没有写过 JavaScript。**但是，JavaScript 确实要比人们普遍所认为的要更加琐碎，强大，复杂。实际上，JavaScript的很多不被注意的细节经常让代码无法按照预期正常运行， - 我们在这里挑出了10个常见问题来讨论 - 如果你想成为一个 JavaScript 大神，那么你应该知道并且避免这些问题。**

##常见问题1： this 的错误引用

我曾经听过这样一个笑话：

> “I’m not really here, because what’s here, besides there, without the ‘t’?”

这个笑话在某些方面形象的说明了当引用 `this` 这个关键字的那种困惑。我的意思是，`this` 真的是 `this` 么？ 或者完全是其他的东西？或者是 `undefined` ?

最近这几年，随着 JavaScript 编码技术和设计模式越来越复杂，在回调函数和闭包中自我调用用的越来越多，也常常造成我们下面要讲的关于 this/that 的困惑。

看一下这段代码 Demo:
```js
Game.prototype.restart = function () {
	this.clearLocalStorage();
	this.timer = setTimeout(function() {
        this.clearBoard();    // 这里的 "this" 指向哪里?
	}, 0);
};
```
运行那段代码会报错，错误为：
```js
	Uncaught TypeError: undefined is not a function
```
为啥嘞？

全是作用域闹的，报错的原因是因为当你调用 `setTimeout()` 的时候，你实际上在调用 `window.setTimeout()`。结果, `setTimeout()` 里面的匿名函数的上下文是 `window` 对象。`window` 对象中没有 `clearBoard()` 方法。

在低级浏览器中的解决方案是简单地把引用存一下以便于让闭包里可以“找到”那个对应的 this 值; 示例:
```js
Game.prototype.restart = function () {
    this.clearLocalStorage();
    var self = this;   // 趁作用域还没变之前，保存当前 'this' 的引用!
    this.timer = setTimeout(function(){
    	self.clearBoard();    // 在这里就很清楚 'self' 指向的是谁了!
    }, 0);
};
```
在高级浏览器中可以用 `bind()` 来绑定合适的 `this` 值：
```js
Game.prototype.restart = function () {
	this.clearLocalStorage();
	this.timer = setTimeout(this.reset.bind(this), 0);  // 绑定到正确的 'this'
};

Game.prototype.reset = function(){
	this.clearBoard();    // 哈哈, 这样 'this' 就对了!
};
```
##常见问题2： 以为有块级作用域
在我们的[JavaScript 招聘指南](http://www.toptal.com/javascript#hiring-guide)中有这样的一个讨论，在 JavaScript 开发者中有一个困惑的源头(也是 bug 的源头)是假定了 JavaScript 在每个块中都有一个作用域。即使在其他语言中是这样的，然而在 JavaScript 并不是这样。考虑一下下边的例子：
```js
for (var i = 0; i < 10; i++) {
	/* ... */
}
console.log(i);  // 猜猜看这里会输出什么?
```
你猜一下 `console.log()` 结果? undefined 还是 报错? 不管你信不信，它将会输出 10. 为啥嘞？

在其他的大多数语言中，这个代码会报错，因为变量 i 的生命周期被限定到了 for 的作用域中。在 JavaScript 中，则不存在这种情况，变量 i 的生命周期延续到了 for 循环以外的地方，在它循环结束之后依然保存着它最后的值。（这个行为是已知的，附上相关[文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting)）

但是，值得注意的是，JavaScript 已经通过新增加的 `let` 关键字 支持块级作用域了。`let` 关键字早在 JavaScript 1.7 就可以使用了，并且已经成为官方标准 [ECMAScript 6](https://people.mozilla.org/~jorendorff/es6-draft.html)中的一部分。

JavaScript 新手？ 移步至[作用域，原型，更多.](http://www.toptal.com/javascript/javascript-prototypes-scopes-and-performance-what-you-need-to-know)


## 常见问题3: 造成内存泄漏

如果你在敲代码的时候不刻意的去避免内存泄漏，那么它将是 JavaScript 不可避免的一个问题。可能造成内存泄漏的情况的太多了，我们着重挑出几个最常见的情况。

###内存泄漏 例1 在已经被销毁的对象中保存着引用:

思考以下代码：
```js
var theThing = null;
var replaceThing = function () {
	var priorThing = theThing;  // hold on to the prior thing
	var unused = function () {
    // 'unused' 是唯一引用了 'priorThing' 的地方，
    // 但是 'unused' 永远不会被调用
    	if (priorThing) {
    		console.log("hi");
    	}
	};
	theThing = {
    	longStr: new Array(1000000).join('*'),  // 创建一个 1MB 大小的对象
    	someMethod: function () {
    		console.log(someMessage);
    	}
	};
};
setInterval(replaceThing, 1000);    // 每秒调用 `replaceThing` 一次
```

如果你运行上面的那个代码然后观察内存使用情况，你将会发现你的内存被大量泄漏了，每秒泄漏1MB, 也不能人工手动回收。好像是我们每调用一次 replaceThing, 就会泄漏掉 longstr 的内存。 但是为什么呢？

让我们再仔细观察一下：

每个 theThing 对象中包含 1MB 的 longStr 对象。 每秒，当我们调用 replaceThing 方法时，他将会在 priorThing 中保存着上一个 theThing 的引用。
但是我们仍然认为这样不会造成问题。因为每次调用的时候，上次的引用 priorThing 将会被反向引用（当 priorThing 通过 priorThing = the Thing 被重置)。此外，只有 replaceThing 的主体被引用了, unused 函数 从来没有被引用。

所以让我们再想一下为什么会有内存泄漏？

为了弄清楚到底发生了什么，我们需要更好的去理解JavaScript内部的工作原理。闭包实现的一个最典型的特点是？每个函数对象都有一个字典类型的链接对象?来代表它的词法作用域。如果在replaceThing中定义两个都使用priorThing的函数，很重要的一点是他们获取到的是同一个对象，即使priorThing被多次的分配，所以这两个函数共享相同的词法环境。但只要一个变量被任意一个闭包函数使用，那么它最终还是会存在于在这范围内的所有闭包函数共享的词法环境中。而也正是这一细微的差别导致了令人困惑不已的内存泄漏问题。(详细信息请点击[这里](https://www.meteor.com/blog/2013/08/13/an-interesting-kind-of-javascript-memory-leak))

###内存泄漏 例二: 循环调用

思考以下代码：
```js
function addClickHandler(element) {
    element.click = function onClick(e) {
        alert("Clicked the " + element.nodeName)
    }
}
```
在这里，onclick 有一个保存在 element 的必包（通过 element.nodeName）,通过声明 onclick ＝ element.click, 循环调用就被建立了，element -> onClick -> element -> onClick -> element...

有趣的是， 即使将 element 从 DOM 树中移除，这个循环的自我调用将会阻止 element 和 onClick 被回收，继续造成内存泄漏。

###避免内存泄漏： 你需要知道的事

JavaScript 的内存管理（就是常说的 gc ), 以下的对象被认为是可达到的，被称为 根：

* 被当前栈中引用的任何地方的对象（就是全部的局部变量和本函数所引用的参数，和全部在必包作用域里的变量）
* 全部的全局变量

在内存中保存着的变量，至少可以通过一个引用或者引用链可以到达。

在浏览器中存在垃圾处理器来清除不可达到的对象，如果垃圾处理器认为这个对象不可达到，那么它将会移除。不幸的是，残留一个事实上不再使用的但是垃圾处理器仍认为可达到的对象是特别容易的一件事。


##常见问题 4：相等性的混淆

JavaScript有这样一个便利，他会自动将boolean上下文中引用的任意值强行转化为boolean型的值。但在有的情况下，我们反而会因为这种便利而觉得很迷惑。例如下面几种从很早以前开始就困扰着许多JavaScript开发者的情况：  
```js
//得到的值均为'true'  
console.log(false == '0');
console.log(null == undefined );
console.log("\t\r\n" == 0);
console.log('' == 0);

//以下的判断也成立
if({}) // ...
if([]) // ...
```
说说最后的两个if判断，尽管{}、[]是空的（这可能导致我们认为他们等效于false）,但实际上他们都是对象，在JavaScript中任意对象均会被强行转化为true布尔型数据，正如[ECMA-262 specification](http://www.ecma-international.org/ecma-262/5.1/#sec-9.2)中所提到的。  

这些列子证明，类型强制转换的规则有时候并不清晰。因此，在除了很明确的需要类型强制转换的情况之外，我们通常最好是使用 === 和 !===（而不是 == 和 !=），以此来避免类型强制转换过程中任何意想不到的副作用。（在比较时，== 和 != 会自动进行类型转换，而 === 和 !==是在不进行类型转换的前提下进行对比。）

另外有一个虽然是比较偏的知识点 —但由于我们对强制类型转换和对比作了讨论—因此也很值得一提：NaN和任何值（甚至是NaN自身！）比较都会返回false。因此你不能使用等式运算符（==、===、!=、!==）去确定一个值是否是NaN，要使用内置的isNaN全局函数来代替。
```js
console.log(NaN == NaN); //false
console.log(NaN === NaN); //false
console.log(isNaN(NaN)); //true
```
##常见问题 5：进行低效率的DOM操作

JavaScript相对来说很很容易对DOM进行操作（即添加，修改和移除元素），但并不能保证操作都能很高效的进行。

一个常见的例子是依次添加一系列的DOM元素的代码。添加一个DOM元素是一个高代价的操作。持续添加多个DOM元素的代码效率很低并且有可能不会很好地运行。

一个高效的替代方案是：当需要添加多个DOM元素时，使用[document fragments](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment),以此来同时提高效率和性能。

例如
```js
var div = document.getElementsByTagName("my_div");

var fragment = document.createDocumentFragment();

for(var e = 0; e < elems.length; e++){ // elems预先设有元素列表
	fragment.appendChild(elems[e]);
}
div.appendChild(fragment.cloneNode(true));
```
这种方法除了能内在的提高效率，对性能也有影响。修改已经添加的DOM元素的代价很高，而先创建元素并在添加之前进行修改，然后再添加到DOM中会产生更好的性能。

##常见问题 6：在for循环内部错误的使用函数

思考以下代码：
```js
var elements = document.getElementsByTagName('input');
var n = elements.length; // 我们约定这个例子有10个元素
for(var i = 0; i < n; i++){
	elements[i].onclick = function(){
		console.log("This is element #" + i);
	}
}
```
基于以上的代码，如果有10个input元素，点击任何一个都会显示“This is element #10”！这是因为在这些元素的onclick被调用的时候，上面的for循环已经结束了，i的值已经是10（所有的元素均是）。

我们在实现期望功能的情况下，改正了上面代码的问题：
```js
var elements = document.getElementsByTagName('input');
var n = elements.length;
var makeHandler = function(num) {
	return function() {
		console.log("This is element #" + num);
	}
};
for (var i = 0; i<n; i++) {
	elements[i].onclick = makeHandler(i+1);
}
```
在修改版本的代码中，每次循环时makeHandler会立即执行，每次运行都会接受当前的i+1的值并将它绑定到一个作用域内的num变量。外层函数返回内层函数（由它来使用作用域中的num变量），并且元素的onclick被设置为外层函数。这确保了每次onclick能接收和使用合适的i值（通过作用域中的num变量）。



## 常见问题 7: 原型继承的不恰当使用

因为绝大部分的JavaScript开发者并没能完全理解原型继承的特性，所以他们也没能尽可能利用它。

这里有个简单实例，

```js
BaseObject = function(name) {
    if(typeof name !== "undefined") {
        this.name = name;
    } else {
        this.name = 'default'
    }
};
```

上面的实例看起来相当的简单，如果你提供一个name值，你将可以使用它，否则name值则被重置为'default'。

例如：

```js
var firstObj = new BaseObject();
var secondObj = new BaseObject('unique');

console.log(firstObj.name);  // -> 结果是 'default'
console.log(secondObj.name); // -> 结果是 'unique'
```

但是，如果我们执行以下操作：

```js
delete secondObj.name;
```

我们将会得到：

```js
console.log(secondObj.name); // -> 结果是 'undefined'
```

但是，这似乎并不符合name值会被重置为'default'的需求？这其实很容易实现，如果我们将源代码以原型继承的方式重构，如下所示：

```js
BaseObject = function (name) {
    if(typeof name !== "undefined") {
        this.name = name;
    }
};

BaseObject.prototype.name = 'default';
```

在上面的代码中，`BaseObject`对象从其原型对象`prototype`上继承了被默认设置的`name`属性`default`。所以，如果未传入`name`值就调用构造函数，则其`name`属性将会默认为`default`。同样地，如果`baseobeject`的一个实例的`name`属性被删除了，这时该实例将通过原型链去搜索原型对象上的属性，即该实例的`name`属性值将会被重置为`default`，这样我们就实现了之前的需求。
``` javascript
var thirdObj = new BaseObject('unique');
console.log(thirdObj.name);  // -> 结果是 'unique'

delete thirdObj.name;
console.log(thirdObj.name);  // -> 结果是 'default'
```

## 常见错误 8：对创建实例方法的错误引用

让我们定义一个简单的对象，并创建其实例对象，如下所示：

```js
var MyObject = function() {}

MyObject.prototype.whoAmI = function() {
    console.log(this === window ? "window" : "MyObj");
};

var obj = new MyObject();
```

为了方便起见，现在让我们对实例对象的`whoAmI`方法创建一个引用，想必我们应该能够只通过`whoAmI()`而不是更长的`obj.whoAmI()`调用该方法了。

```js
var whoAmI = obj.whoAmI;
```

为了要确定一切顺利，让我们输出打印一下新变量whoAmI的值:

```js
console.log(whoAmI);
```

输出为:

```js
function () {
    console.log(this === window ? "window" : "MyObj");
}
```

OK,很好，看起来一切顺利.
但是现在，当我们调用`obj.whoAmI()`和其快速引用`whoAmI()`时，请看两者的区别:

```js
obj.whoAmI();  // 输出 "MyObj" (跟我们预期的一样)
whoAmI();      // 输出 "window" (啊哦!)
```

到底哪里出了问题呢？
这里的障眼法在于当我们执行语句`var whoAmI = obj.whoAmI;`时，这里的新变量`whoAmI`是在全局`global`命名空间内定义的，因此其函数内`this`指向`window`而不是`MyObject`的实例对象`obj`。

因此，如果我们真的需要对一个对象的方法创建引用，我们一定要在该对象的命名空间下去操作，以保持`this`的值都不发生改变。以下的例子是实现该情况的一种方法:

```js
var MyObject = function() {}

MyObject.prototype.whoAmI = function() {
    console.log(this === window ? "window" : "MyObj");
};

var obj = new MyObject();
obj.w = obj.whoAmI;   // 保证仍然在 obj 命名空间内

obj.whoAmI();  // 输出 "MyObj" (符合预期)
obj.w();       // 输出 "MyObj" (符合预期)
```

## 常见错误9: 给`setTimeout`或`setInterval`函数的第一个参数传一个字符串

首先，让我们在最开始明确清楚，给`setTimeout`或`setInterval`函数的第一个参数提供一个字符串的行为本质上并不是一个错误。这完全是合法的JavaScript代码,这里更多的是执行性能和效率的问题。很少有人给你解释的是在底层如果你给`setTimeout`或`setInterval`函数的第一个参数传入一个字符串执行，这会导致其构造函数将会被转换为一个新的函数。这个过程会是缓慢和低效的，该过程也是不必要的。

给这两个方法的第一个参数传入字符串的另一个可以代替的选择是传入一个函数。让我们来看一个例子。

这里是给`setTimeout`和`setInterval`函数的第一个参数提供一个字符串的典型用法。

```js
setInterval("logTime()", 1000);
setTimeout("logMessage('" + msgValue + "')", 1000);
```

更好的选择是给初始参数传入一个函数，如下所示：

```js
setInterval(logTime, 1000);   // 直接传递logTime函数给 setInterval

setTimeout(function() {       // 传递一个匿名函数给 setTimeout
    logMessage(msgValue);     // (作用域内的 msgValue 变量依然是有效的 )
  }, 1000);
```


##常犯错误 #10：不使用“严格模式”

正如在[JavaScript Hiring Guide](http://www.toptal.com/javascript#hiring-guide)中所解释的，“严格模式”（即在你的JavaScript源文件开始的地方添加上'use strict';）是一种在你的JavaScript代码运行时，自动执行更严格的解析和错误处理的方法，同时这也会使代码更加安全。

其实，不使用严格模式本身并不算一个“错误”，但是它的使用越来越受到鼓励，省略严格模式越来越被认为是一种糟糕的形式。

这儿列出了严格模式的几个主要的好处：

* **更容易调试**。那些容易被忽略或者是没有自动生成错误或者抛出异常的代码错误，能够提醒你更早的去发现你代码中的问题并且引导你更快的去发现错误来源。

* **防止出现意外的全局变量**。在非严格模式中，给一个未申明的变量赋值会自动创建一个同名的全局变量。这也是JavaScript中最常见的一类错误。而如果你尝试在严格模式中这样操作将会抛出一个错误。

* **消除了this的强制转化**。在非严格模式下，使用this来引用一个null或者undefined类型的值，this会自动强制指向全局对象。这会导致很多挠破头皮也找不出来的bug。在严格模式中，使用this引用一个null或者undefined类型的值会抛出一个错误。

* **不允许重复定义属性名或者是参数值**。当检测到一个对象里有重复命名的属性名（例如：`var object = {foo:"bar",foo:"baz"};`）或者是一个函数有重复命名的参数（例如：`function foo(val1,val2,val1){}`）时，严格模式将会抛出一个错误。我们应该捕获那些几乎确定是代码中的bug，否则你将会浪费掉很多的时间去追踪一些不必要的问题。

* **eval()更加安全**严格模式和非严格模式的eval()表现有一些不同。最显著的是在严格模式下，一个eval()语句内部申明的变量和函数不会在eval（）所处的作用域中进行创建，他们只用于eval（）内部（在非严格模式下会创建，这也是很多问题的常见原因）。

* **delete无效使用时抛出错误**delete操作（用于移除对象的属性）不能用于对象未配置的属性。当企图去删除一个未配置的属性时，非严格代码只会默默地失败，而严格模式在这样的情况下会抛出一个错误。

## 总结
任何技术都是这样的，你对为什么和怎么样会让 JavaScript 工作或者不工作理解的越好，你的代码就越健壮，你对你代码的安全性就越有把握
相反的，缺少对 JavaScript 代码示例和概念的深入理解将会导致你的 JavaScript 代码出问题。

彻底理解你所使用的语言的细节是你提升你工作效率和提升你的熟练度的最有效的方式。在你的 JavaScript 代码出问题的时候，避免这些常见的 JavaScript 问题将会对你有帮助。

---

译自：[http://www.toptal.com/javascript/10-most-common-javascript-mistakes](http://www.toptal.com/javascript/10-most-common-javascript-mistakes)
本文首发：[http://wx.h5.vc/post/10-most-common-javascript-mistakes](http://wx.h5.vc/post/10-most-common-javascript-mistakes)
本文通过Github PR 完成 [https://github.com/h5vc/wx.h5.vc/pull/1](https://github.com/h5vc/wx.h5.vc/pull/1)
欢迎有兴趣的一起上来玩儿啊。
## 关注我们
![微信扫一扫关注我们](http://7xp3fo.com1.z0.glb.clouddn.com/_image/yufe2015/h5.vc.jpg)
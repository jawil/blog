## Call、Apply、Bind
### 一、起因
今天看博客时，看到了这样的一段js代码：
`var bind = Function.prototype.call.bind(Function.prototype.bind);`

上面那段代码涉及到了call、bind，所以我想先区别一下call、apply、bind的用法。这三个方法的用法非常相似，将函数绑定到上下文中，即用来改变函数中this的指向。举个例子：

```
var jawil = {
    name: "jawil",
    sayHello: function (age) {
         console.log("hello, i am ", this.name + " " + age " years old");
     }
};

var  lulin = {
    name: "lulin",
};

jawil.sayHello(24);

// hello, i am jawil 24 years old
```
下面看看call、apply方法的用法：

```
1、每个函数都包含两个非继承而来的方法：apply()和call()。 
2、他们的用途相同，都是在特定的作用域中调用函数。 
3、接收参数方面不同，apply()接收两个参数，一个是函数运行的作用域(this)，另一个是参数数组。
4、call()方法第一个参数与apply()方法相同，但传递给函数的参数必须列举出来。
```
```
jawil.sayHello.call(lulin, 24);
// hello, i am lulin 24 years old
jawil.sayHello.apply(lulin, [24]);
// hello, i am lulin 24 years old
```
`结果都相同。从写法上我们就能看出二者之间的异同。相同之处在于，第一个参数都是要绑定的上下文，后面的参数是要传递给调用该方法的函数的。不同之处在于，call方法传递给调用函数的参数是逐个列出的，而apply则是要写在数组中。`

我们再来看看bind方法的用法：
###### 工作原理

```
Function.prototype.bind = function (scope) {
    var fn = this;
    return function () {
        return fn.apply(scope);
    };
}
```

```
jawil.sayHello.bind(lulin, 24)(); //hello, i am lulin 24 years old
jawil.sayHello.bind(lulin, [24])(); //hello, i am lulin 24 years old
```
bind方法传递给调用函数的参数可以逐个列出，也可以写在数组中。bind方法与call、apply最大的不同就是前者返回一个绑定上下文的函数，而后两者是直接执行了函数。由于这个原因，上面的代码也可以这样写:

```
jawil.sayHello.bind(lulin)(24); //hello, i am lulin 24 years old
jawil.sayHello.bind(lulin)([24]); //hello, i am lulin 24 years old
```
bind方法还可以这样写 `fn.bind(obj, arg1)(arg2)`.

`用一句话总结bind的用法：该方法创建一个新函数，称为绑定函数，绑定函数会以创建它时传入bind方法的第一个参数作为this，传入bind方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。
`
### 二、bind在实际中的应用
实际使用中我们经常会碰到这样的问题

```
js 代码:
function Person(name){
 this.nickname = name;
 this.distractedGreeting = function() {
 
   setTimeout(function(){
     console.log("Hello, my name is " + this.nickname);
   }, 500);
 }
}
 
var alice = new Person('Alice');
alice.distractedGreeting();
//Hello, my name is undefined
```
`这个时候输出的this.nickname是undefined，原因是this指向是在运行函数时确定的，而不是定义函数时候确定的，再因为setTimeout在全局环境下执行，所以this指向setTimeout的上下文：window。`


以前解决这个问题的办法通常是缓存this，例如：

```
js 代码:
function Person(name){
  this.nickname = name;
  this.distractedGreeting = function() {
    var self = this; // <-- 注意这一行!
    setTimeout(function(){
      console.log("Hello, my name is " + self.nickname); // <-- 还有这一行!
    }, 500);
  }
}
 
var alice = new Person('Alice');
alice.distractedGreeting();
// after 500ms logs "Hello, my name is Alice"
```
`这样就解决了这个问题，非常方便，因为它使得setTimeout函数中可以访问Person的上下文。但是看起来稍微一种蛋蛋的忧伤。`

但是现在有一个更好的办法！您可以使用bind。上面的例子中被更新为：

```
js 代码:
function Person(name){
  this.nickname = name;
  this.distractedGreeting = function() {
    setTimeout(function(){
      console.log("Hello, my name is " + this.nickname);
    }.bind(this), 500); // <-- this line!
  }
}
 
var alice = new Person('Alice');
alice.distractedGreeting();
// after 500ms logs "Hello, my name is Alice"
```
`bind() 最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的 this 值。JavaScript新手经常犯的一个错误是将一个方法从对象中拿出来，然后再调用，希望方法中的 this 是原来的对象。（比如在回调中传入这个方法。）如果不做特殊处理的话，一般会丢失原来的对象。从原来的函数和原来的对象创建一个绑定函数，则能很漂亮地解决这个问题：`


```
js 代码:
this.x = 9; 
var module = {
  x: 81,
  getX: function() { return this.x; }
};
 
module.getX(); // 81
 
var getX = module.getX;
getX(); // 9, 因为在这个例子中，"this"指向全局对象
 
// 创建一个'this'绑定到module的函数
var boundGetX = getX.bind(module);
boundGetX(); // 81
```
`很不幸，Function.prototype.bind 在IE8及以下的版本中不被支持，所以如果你没有一个备用方案的话，可能在运行时会出现问题。bind 函数在 ECMA-262 第五版才被加入；它可能无法在所有浏览器上运行。你可以部份地在脚本开头加入以下代码，就能使它运作，让不支持的浏览器也能使用 bind() 功能。
幸运的是，MDN为没有自身实现 .bind() 方法的浏览器提供了一个绝对可靠的替代方案：`

```
js 代码:
if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      // closest thing possible to the ECMAScript 5 internal IsCallable function
      throw new TypeError("Function.prototype.bind - what is trying to be bound 
      is not callable");
    }
 
    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(this instanceof fNOP && oThis
                                 ? this
                                 : oThis || window,
                               aArgs.concat(Array.prototype.slice.call(arguments)));
        };
 
    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
 
    return fBound;
  };
}
```

### 三、深入理解Function.prototype.call.bind(Function.prototype.bind)
现在回到开始的那段代码：
`var bind = Function.prototype.call.bind(Function.prototype.bind);`

我们可以这样理解这段代码：
`var bind = fn.bind(obj)`
 
`fn `相当于 `Function.prototype.call` ， `obj `相当于 `Function.prototype.bind` 。而 `fn.bind(obj) `一般可以写成这样 `obj.fn` ，为什么呢？因为 `fn` 绑定了 `obj` ， `fn` 中的 `this `就指向了 `obj` 。我们知道，函数中 this 的指向一般是指向调用该函数的对象。所以那段代码可以写成这样:
`var bind = Function.prototype.bind.call;`

大家想一想`Function.prototype.call.bind(Function.prototype.bind)` 返回的是什么？

`console.log(Function.prototype.call.bind(Function.prototype.bind)) // call()`

返回的是 call 函数，但这个 call 函数中的上下文的指向是 Function.prototype.bind 。这个 call 函数可以这样用

```
var bind = Function.prototype.call.bind(Function.prototype.bind);

var lulin = {
 name: "lulin" 
};

function hello () {
  console.log("hello, I am ", this.name);
}

bind(hello, lulin)() // hello, I am lulin
```

大家可能会感到疑惑，为什么是这样写 `bind(hello, lulin) `而不是这样写 `bind(lulin, hello)` ？既然 `Function.prototype.call.bind(Function.prototype.bind) `相当于 `Function.prototype.bind.call` ，那么先来看下 `Function.prototype.bind.call` 怎么用。 call 的用法大家都知道：

`var bind = Function.prototype.call.bind(Function.prototype.bind);`

而不直接这样写呢：
`var bind = Function.prototype.bind.call;`

先来看一个例子：

```
var name = "lulin";
var jawil = {
    name: "jawil" 
    hello: function () {
        console.log(this.name);
    }
};
jawil.hello(); // jawil

var hello = jawil.hello;
hello(); // lulin
```

有些人可能会意外，` hello()` 的结果应该是 `jawil` 才对啊。其实，将 `jawil.hello` 赋值给变量 `hello` ，再调用 `hello()` ， `hello` 函数中的 `this` 已经指向了 `window` ，与 `jawil.hello` 不再是同一个上下文，而全局变量 `name` 是 `window `的一个属性，所以结果就是` lulin` 。再看下面的代码：


```
var hello = jawil.hello.bind(jawil);
hello(); // jawil
```
结果是 `jawil` ，这时 `hello` 函数与 `jawil.hello` 是同一个上下文。其实上面的疑惑已经解开了，直接这样写：
`var bind = Function.prototype.bind.call;`

 `bind` 函数中的上下文已经与` Function.prototype.bind.call` 中的不一样了，所以使用 `bind` 函数会出错。而这样写
`var bind = Function.prototype.call.bind(Function.prototype.bind);`

 `bind` 函数中的上下文与 `Function.prototype.call.bind(Function.prototype.bind)` 中是一样的。
### 四、注意,易误解的细节
`bind所做的就是自动封装函数在函数自己的闭包中，这样我们可以捆绑上下文（this关键字）和一系列参数到原来的函数。
 你最终得到的是另一个函数指针。
`

```
function add(a,b){ 
   return a + b;
}
var newFoo = add.bind(this,3,4);
```
`请注意，我们不仅捆绑this到foo()函数，而且我们也捆绑了两个参数。所以，当我们调用newFoo()的时候，返回值将是7。
 但是，如果我们在调用之前newFoo更改的参数的话，会发生什么？ `

```
function add(a,b){    
    return a + b;
}
var a = 3;
var b = 4;
var newFoo = add.bind(this,a, b);
a = 6;
b = 7;
console.log(newFoo());
```
`返回值仍然是7，因为bind()绑定的是参数的值，而不是实际变量的值。
 这是好消息，就像我说的，我们可以在代码中利用这个巨大的优势。但是，对我而言它最有用的地方是在callbacks中。 `

### 五、捷径
bind()也可以为需要特定this值的函数创造捷径。
例如要将一个类数组对象转换为真正的数组，可能的例子如下：

```
var slice = Array.prototype.slice;
// ...
slice.call(arguments);
```
如果使用bind()的话，情况变得更简单：

```
var unboundSlice = Array.prototype.slice;
var slice = Function.prototype.call.bind(unboundSlice);
// ...
slice(arguments);
```


### 六、实现
上面的几个小节可以看出bind()有很多的使用场景，但是bind()函数是在 ECMA-262 第五版才被加入；它可能无法在所有浏览器上运行。这就需要我们自己实现bind()函数了。

首先我们可以通过给目标函数指定作用域来简单实现bind()方法：

```
Function.prototype.bind = function(context){
  self = this;  //保存this，即调用bind方法的目标函数
  return function(){
      return self.apply(context,arguments);
  };
};
```
考虑到函数柯里化的情况，我们可以构建一个更加健壮的bind()：

```
Function.prototype.bind = function(context){
  var args = Array.prototype.slice.call(arguments, 1),
  self = this;
  return function(){
      var innerArgs = Array.prototype.slice.call(arguments);
      var finalArgs = args.concat(innerArgs);
      return self.apply(context,finalArgs);
  };
};
```
这次的bind()方法可以绑定对象，也支持在绑定的时候传参。

继续，Javascript的函数还可以作为构造函数，那么绑定后的函数用这种方式调用时，情况就比较微妙了，需要涉及到原型链的传递：

```
Function.prototype.bind = function(context){
  var args = Array.prototype.slice(arguments, 1),
  F = function(){},
  self = this,
  bound = function(){
      var innerArgs = Array.prototype.slice.call(arguments);
      var finalArgs = args.concat(innerArgs);
      return self.apply((this instanceof F ? this : context), finalArgs);
  };

  F.prototype = self.prototype;
  bound.prototype = new F();
  retrun bound;
};
```
这是《JavaScript Web Application》一书中对bind()的实现：通过设置一个中转构造函数F，使绑定后的函数与调用bind()的函数处于同一原型链上，用new操作符调用绑定后的函数，返回的对象也能正常使用instanceof，因此这是最严谨的bind()实现。

对于为了在浏览器中能支持bind()函数，只需要对上述函数稍微修改即可：

```
Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not 
      callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(
              this instanceof fNOP && oThis ? this : oThis || window,
              aArgs.concat(Array.prototype.slice.call(arguments))
          );
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
```
#### apply,call,bind的一些常用用法
1、 匿名函数使用bind创建多个函数

`循环中处理callbacks的解决方案之一就是，围绕我们想要调用的函数创建匿名函数。`

```
for(var i = 0;i < 10;i++){
    (function(ii){
        setTimeout(function(){            
            console.log(ii);
        },1000);
    })(i);
```
`ES6 下能用 let：`

```
for(let i = 1; i <= 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, i * 1000);
}
```
`但是我们可以使用bind，大大简化代码。`

``` 
for(var i = 1; i <= 5; i++) {
  setTimeout(console.log.bind(console, i),1000);
}
```

`这是因为每次调用bind就会给出一个新的函数指针，并且保留原来的函数不变。
同时，我们还删除了linting错误“不要在循环写函数”，因为我们不是在循环中创造的函数，我们只是指向了我们在循环外创建的函数。`

2、bind改变this指向

```
var bar = function(){
console.log(this.x);
}
var foo = {
x:3
}
bar(); // undefined
var func = bar.bind(foo);
func(); // 3
```
`这里我们创建了一个新的函数 func，当使用 bind() 创建一个绑定函数之后，它被执行的时候，它的 this 会被设置成 foo ， 而不是像我们调用 bar() 时的全局作用域。`
`有个有趣的问题，如果连续 bind() 两次，亦或者是连续 bind() 三次那么输出的值是什么呢？像这样：`

```
var bar = function(){
  console.log(this.x);
}
var foo = {
  x:3
}
var sed = {
  x:4
}
var func = bar.bind(foo).bind(sed);
func(); //?
 
var fiv = {
  x:5
}
var func = bar.bind(foo).bind(sed).bind(fiv);
func(); //?
```
`答案是，两次都仍将输出 3 ，而非期待中的 4 和 5 。原因是，在Javascript中，多次 bind() 是无效的。更深层次的原因， bind() 的实现，相当于使用函数在内部包了一个 call / apply ，第二次 bind() 相当于再包住第一次 bind() ,故第二次以后的 bind 是无法生效的。`

3、bind 能让调用变的更加简单。

```
// same as "slice" in the previous example
var unboundSlice = Array.prototype.slice;
var slice = Function.prototype.call.bind(unboundSlice);

// ...

slice(arguments);
// slice(arguments, 1);
```
`再举个类似的例子，比如说我们要添加事件到多个节点，for 循环当然没有任何问题，我们还可以 “剽窃” forEach 方法：`

```
Array.prototype.forEach.call(document.querySelectorAll('input[type="button"]'), function(el){
  el.addEventListener('click', fn);
});
```
`更进一步，我们可以用 bind 将函数封装的更好：`

```
var unboundForEach = Array.prototype.forEach;
var forEach = Function.prototype.call.bind(unboundForEach);

forEach(document.querySelectorAll('input[type="button"]'), function (el) {
  el.addEventListener('click', fn);
});
```
`同样类似的，我们可以将 x.y(z) 变成 y(x,z) 的形式：`

```
var obj = {
  num: 10,
  getCount: function() {
    return this.num;
  }
};

var unboundBind = Function.prototype.bind
  , bind = Function.prototype.call.bind(unboundBind);

var getCount = bind(obj.getCount, obj);
console.log(getCount());  // 10
```

4、 数组之间追加

```
var array1 = [12 , "foo" , {name:"Joe"} , -2458]; 
var array2 = ["Doe" , 555 , 100]; 
第一种:
array1.concat(array2);
第二种:
Array.prototype.push.apply(array1, array2); 
第三种:
Array.prototype.concat.apply([], [array1, array2]);
/* array1 值为 [12 , "foo" , {name "Joe"} , -2458 , "Doe" , 555 , 100] */
```

5、获取数组中的最大值和最小值

```
var numbers = [5, 458 , 120 , -215 ]; 
var maxInNumbers = Math.max.apply(Math, numbers),  //458
  maxInNumbers = Math.max.call(Math,5, 458 , 120 , -215); //458
```
`number 本身没有 max 方法，但是 Math 有，我们就可以借助 call 或者 apply 使用其方法。`

6、验证是否是数组（前提是toString()方法没有被重写过）

```
function isArray(obj){ 
  return Object.prototype.toString.call(obj) === '[object Array]' ;
}
```
7、类（伪）数组使用数组方法

```
var domNodes = Array.prototype.slice.call(document.getElementsByTagName("*"));
```
`Javascript中存在一种名为伪数组的对象结构。比较特别的是 arguments 对象，还有像调用 getElementsByTagName , document.childNodes 之类的，它们返回NodeList对象都属于伪数组。不能应用 Array下的 push , pop 等方法。
但是我们能通过 Array.prototype.slice.call 转换为真正的数组的带有 length 属性的对象，这样 domNodes 就可以应用 Array 下的所有方法了。`

8、深入理解运用apply、call(综合实例)
`定义一个 log 方法，让它可以代理 console.log 方法，常见的解决方法是：`

```
function log(msg)　{
 console.log(msg);
}
log(1);  //1
log(1,2);  //1
```
`上面方法可以解决最基本的需求，但是当传入参数的个数是不确定的时候，上面的方法就失效了，这个时候就可以考虑使用 apply 或者 call，注意这里传入多少个参数是不确定的，所以使用apply是最好的，方法如下：`

```
function log(){
 console.log.apply(console, arguments);
};
log(1);  //1
log(1,2);  //1 2
```
`接下来的要求是给每一个 log 消息添加一个"(app)"的前辍，比如：`

```
log("hello world");  //(app)hello world
```
`该怎么做比较优雅呢?这个时候需要想到arguments参数是个伪数组，通过 Array.prototype.slice.call 转化为标准数组，再使用数组方法unshift，像这样：`

```
function log(){
 var args = Array.prototype.slice.call(arguments);
 args.unshift('(app)');
 console.log.apply(console, args);
};
```


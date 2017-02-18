# Extend

#### 1.拼接继承(拷贝继承,原型式继承)
###### 深拷贝与浅拷贝

```
eg：有A、B两个对象，且都有子对象

深拷贝：将B对象拷贝到A对象中，包括B里面的子对象;

浅拷贝：将B对象拷贝到A对象中，但不包括B里面的子对象;
```
`首先深复制和浅复制只针对像 Object, Array 这样的复杂对象的。简单来说，浅复制只复制一层对象的属性，而深复制则递归复制了所有层级。`
###### 简单实现浅拷贝
下面是一个简单的浅复制实现:

```
var obj = { a:1, arr: [2,3] };
var shadowObj = shadowCopy(obj);

function shadowCopy(src) {
  var dst = {};
  for (var prop in src) {
    if (src.hasOwnProperty(prop)) {
      dst[prop] = src[prop];
    }
  }
  return dst;
}
```
`因为浅复制只会将对象的各个属性进行依次复制，并不会进行递归复制，而 JavaScript 存储对象都是存地址的，所以浅复制会导致 obj.arr 和 shadowObj.arr 指向同一块内存地址，大概的示意图如下。`
<img src="https://pic4.zhimg.com/v2-39761dfd012733879e0d100ec260a5d7_b.png"/>
导致的结果就是：

```
shadowObj.arr[1] = 5;
obj.arr[1]   // = 5
```
###### 简单实现深拷贝
`而深复制则不同，它不仅将原对象的各个属性逐个复制出去，而且将原对象各个属性所包含的对象也依次采用深复制的方法递归复制到新对象上。这就不会存在上面 obj 和 shadowObj 的 arr 属性指向同一个对象的问题。`
实现方法一:

```
var cloneObj = function(obj){
    var str, newobj = obj.constructor === Array ? [] : {};
    if(typeof obj !== 'object'){
        return;
    } else if(window.JSON){
        str = JSON.stringify(obj), //系列化对象
        newobj = JSON.parse(str); //还原
    } else {
        for(var i in obj){
            newobj[i] = typeof obj[i] === 'object' ? 
            cloneObj(obj[i]) : obj[i]; 
        }
    }
    return newobj;
};
var obj = { a:1, arr: [1,2] };
var obj2 = cloneObj(obj);
```
实现方法二:
`下面是jQuery.extend的源码,可以看看,不是很长,其实和jQuery.fn.extend是同一个方法:`

```
jQuery.extend = jQuery.fn.extend = function() {
    var src, copyIsArray, copy, name, options, clone,
        target = arguments[0] || {},
        i = 1,
        length = arguments.length,
        deep = false;

    // Handle a deep copy situation
    if ( typeof target === "boolean" ) {
        deep = target;

        // skip the boolean and the target
        target = arguments[ i ] || {};
        i++;
    }

    // Handle case when target is a string or something (possible in deep copy)
    if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
        target = {};
    }

    // extend jQuery itself if only one argument is passed
    if ( i === length ) {
        target = this;
        i--;
    }

    for ( ; i < length; i++ ) {
        // Only deal with non-null/undefined values
        if ( (options = arguments[ i ]) != null ) {
            // Extend the base object
            for ( name in options ) {
                src = target[ name ];
                copy = options[ name ];

                // Prevent never-ending loop
                if ( target === copy ) {
                    continue;
                }

                // Recurse if we're merging plain objects or arrays
                if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = 
                jQuery.isArray(copy)) ) ) {
                    if ( copyIsArray ) {
                        copyIsArray = false;
                        clone = src && jQuery.isArray(src) ? src : [];

                    } else {
                        clone = src && jQuery.isPlainObject(src) ? src : {};
                    }

                    // Never move original objects, clone them
                    target[ name ] = jQuery.extend( deep, clone, copy );

                // Don't bring in undefined values
                } else if ( copy !== undefined ) {
                    target[ name ] = copy;
                }
            }
        }
    }

    // Return the modified object
    return target;
};
```
实现方法三:
`自己献丑写了一个,还望多多包涵!`

```
   //对象的深拷贝继承
    if (!Object.prototype.extend) {
        Object.prototype.extend = function(parent) {
            if (Object.prototype.toString.call(parent) === '[object Object]') {
                for (var attr in parent) {
                    parent.hasOwnProperty(attr) &&
                        this[attr] == void 0 &&
                        (function() {
                            var type = Object.prototype.toString.call(parent[attr]);
                            switch (type) {
                                //属性值为Object对象
                                case '[object Object]':
                                    this[attr] = {}.extend(parent[attr]);
                                    break;
                                    //属性值为Function对象
                                case '[object Function]':
                                    var fn = function() {};
                                    fn = parent[attr];
                                    this[attr] = fn;
                                    break;
                                    //属性值为Array对象
                                case '[object Array]':
                                    this[attr] = Array.prototype.concat.call([], 
                                    parent[attr]);
                                    break;
                                    //属性值为普通属性
                                default:
                                    this[attr] = parent[attr];
                                    break;
                            }
                        }.bind(this)());
                }
                return this;
            }
        }
    }

    var par1 = {
        a: 1,
        b: 2,
        c: {d: [1, 2, 3, 4, 5, 6]},
        e: function() {
            console.log(666666666);
        }
    };
    var child = {}.extend(par1);
    child.e();
    console.log(child.c.d[2]); //3

    var par2 = {
        a: 1,
        b: 2,
        c: { d: 3,e: 4, f: {g: 5,h: 6}   
        }
    };
    var child = {}.extend(par2);
    console.log(child.c.f.g); //5
```
结果如下面的示意图所示：
<img src="https://pic1.zhimg.com/6604224933c95787764d941432a1f968_b.jpg"/>
`需要注意的是，如果对象比较大，层级也比较多，深复制会带来性能上的问题。在遇到需要采用深复制的场景时，可以考虑有没有其他替代的方案。在实际的应用场景中，也是浅复制更为常用。`

#### 2.document.create(原型式继承,原型代理)
`利用[[Prototype]]特性，可以实现原型继承；对于字面量形式的对象，会隐式指定Object.prototype为其[[Prototype]]，也可以通过Object.create()显示指定，其接受两个参数：第一个是[[Prototype]]指向的对象(原型对象)，第二个是可选的属性描述符对象。`
这种继承借助原型并基于已有的对象创建新对象，同时还不用创建自定义类型的方式称为`原型式继承`.
```
var book = {
    title:"这是书名";
};
//和下面的方式一样
var book = Object.create(Object.prototype,{
    title:{
        configurable:true,
        enumerable:true,
        value:"这是书名",
        wratable:true
    }
});
```
`字面量对象会默认继承自Object，更有趣的用法是，在自定义对象之间实现继承。`

```
var book1 = {
    title:"JS高级程序设计",
    getTitle:function(){
        console.log(this.title);
    }
};
var book2 = Object.create(book1,{
    title:{
        configurable:true,
        enumerable:true,
        value:"JS权威指南",
        wratable:true
    }
});
book1.getTitle();  //"JS高级程序设计"
book2.getTitle();  //"JS权威指南"
console.log(book1.hasOwnProperty("getTitle"));  //true
console.log(book1.isPrototypeOf("book2"));  //false
console.log(book2.hasOwnProperty("getTitle"));  //false
```
`当访问book2的getTitle属性时，JavaScript引擎会执行一个搜索过程：现在book2的自有属性中寻找，找到则使用，若没有找到，则搜索[[Prototype]]，若没有找到，则继续搜索原型对象的[[Prototype]]，直到继承链末端。末端通常是Object.prototype，其[[Prototype]]被设置为null。
实现继承的另外一种方式是利用构造函数。每个函数都具有可写的prototype属性，默认被自懂设置为继承自
Object.prototype,可以通过改写它来改变原型`
###### document.create的实现原理

```
 var parent = {
        name: 'jawil',
        age: '23',
        getInfo: function() {
            console.log(this.name, this.age);
        }
    }
    var child = Object.create(parent);

    child.hasOwnProperty('age'); //false

```
   ` 为什么是false而不是true, document.create的继承原理并不是通过拷贝来继承的, 而是通过原型链, 所以
    child并没有name和age属性, 在浏览器输入child.__proto__会发现输出的是parent对象, 由此可知:
    child的模板就是parent, 看下面我们经常写的一个例子:`

  
```  
function Person() {}

    Person.prototype = {
        constructor: Person,
        name: jawil,
        age: 23,
        getInfo: function() {
            console.log(this.name, this.age);
        }
    }
```
   ` 我们创建的每个函数都有一个prototype属性， 这个属性是一个指针， 指向一个对象， 这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。 那么， prototype就是通过调用构造函数而创建的那个对象实例的原型对象。`

   ` var boy = new Person();`

  `  我们知道boy这个实例是以Person.prototype为模板创造出来的, boy.__proto__ === Person.prototype;
    一样的, boy.hasOwnProperty('age'); //false`

   
```
我们再回头看看var child = Object.create(parent);
是不是能看出一些端倪, 我们可以这么对比,
某个构造函数充当Person的角色
parent(某个构造函数指的prototype向了parent而已) 充当Person.prototype的角色
child 充当new Person() 的角色
对于这段代码:
  var parent = {
      name: 'jawil',
      age: '23',
      getInfo: function() {
          console.log(this.name, this.age);
      }
  }
var child = Object.create(parent);
我们在chrome浏览器输入child.__proto__ === parent; //true
我们知道__proto__是对象中一个指向相关原型的神秘链接;
```

`我们想要实现document.create的功能, 那么我们就需要实现child.__proto__这个地址指向parent, 有人说我直接让child.__proto__ = parent, 就是这么简单暴力, 话是这么说, 但是并不是所有浏览器都认识__proto__这个私有属性, 那我们该怎么实现这个方法呢 ? 我们一步一步来 :
首先根据定义我们知道: child.__proto__本来是指向构造函数的原型对象, 可是却指向了parent这个实例对象, 由此
可以知道, 这个原型对象指向了parent, 我们假设child是Function F() {}构造出来的, 那么可以得出以下结论:`
  
``` 
var child = new F();
F.prototype = parent;
弄清楚这两点写起来就很容易了, 下面就是主要代码:
   var parent = {
       name: 'jawil',
       age: '23',
       getInfo: function() {
           console.log(this.name, this.age);
       }
   };
var F = function() {};
F.prototype = parent;
var child = new F();
这三句话就相当于var child = Object.create(parent);
```

#### 3.构造函数(类继承)
##### ①纯原型链继承

``` 
function Person() {
        this.color=['black','blue','red'];
        this.name="jawil";

    }
    Person.prototype.say = function() {
        console.log(this.name);
    }

   function Child(){

   }
    Child.prototype=new Person();
    Child.prototype.constructor=Child;

    var child1=new Child();
    console.log(child1.color);//['black','blue','red']
    child1.say();//jawil
```
   `优点：正确设置原型链实现继承
    优点：父类实例属性得到继承，原型链查找效率提高，也能为一些属性提供合理的默认值
    缺点：父类实例属性为引用类型时，不恰当地修改会导致所有子类被修改
    缺点：创建父类实例作为子类原型时，可能无法确定构造函数需要的合理参数，这样提供的参数继承给子类没有实际意义， 当子类需要这些参数时应该在构造函数中进行初始化和设置
    总结：继承应该是继承方法而不是属性，为子类设置父类实例属性应该是通过在子类构造函数中调用父类构造函数进行初始化` 
###### 分析一下出现这种缺点的原因:
```   
child1.color.push('yellow');//["black", "blue", "red", "yellow"]
var child2=new Child();
console.log(child2.color);//["black", "blue", "red", "yellow"]
```
`关于这行代码Child.prototype=new Person();
    我们可以拆成这样理解colors:
        Child.prototype={
            color:['black','blue','red'],
        }
        因为this就是Child.prototype,因此也就是相当于Child.prototype.colors=['black','blue','red'];
    所以所有的Child实例会共享这一个color属性.
原型链的第二个问题就是:在创建子类型的实例时,不能向超类型的构造函数中传递参数.实际上,应该说是没有办法在不影响所有对象实例的情况下,给超类型的构造函数传递参数.有鉴于此,再加上前面刚刚讨论的由于原型中包含引用类型值所带来的问题,实际中很少会单独使用原型链继承.
    `

##### ②借用构造函数继承
`在解决原型中包含引用类型值所带来的问题的过程中,开发人员开始使用一种叫做借用构造函数(constructor stealing)的技术(有时候也叫做伪造对象或经典继承).这种技术的基本思想相当简单,即在子类型构造函数的内部调用超类型构造函数.别忘了,函数只不过是在特定环境中执行代码的对象,因此通过使用apply()和call()方法也可以在(将来)新创建的对象上执行构造函数,如下所示:`
    
```
function Person(name,age) {
        this.color = ['black', 'blue', 'red'];

    }
    Person.prototype.say = function() {
        console.log(this.color);
    }

    function Child(name,age) {
        Person.call(this,name,age);
    }
    Child.prototype = new Person();
    var child1 = new Child();
    child1.color.push('yellow');
    var child2 = new Child();
    console.log(child2.color); //["black", "blue", "red"]
```

`这行代码Person.call(this)实际上是"借调"了超类型的构造函数.通过使用call()方法(或者apply()方法),我们实际上在(未来将要)新创建的Child实例的环境中调用了Person的构造函数.这样一来,就会在新的Child对象上执行Person()函数中定义的所有对象初始化代码.结果Child的每个实例就会有自己的color属性的副本了.
`

```
接下来我们详细的分析这句话:Person.call(this);
首先我们要弄懂call函数的机制:
js关于call的这份文档容易让人迷糊。而《javascript权威指南》对call的描述就比较容易理解了。
```
<img src="http://images2015.cnblogs.com/blog/794452/201511/794452-20151109174953494-406212384.png"/>
` 注意红色框中的部分，f.call(o)其原理就是先通过 o.m = f 将 f作为o的某个临时属性m存储，然后执行m，执行完毕后将m属性删除。`

```
    如
    function f() {
        var a = "name";
        this.b = "test1";
        this.add = function() {
            return "add"
        }
    }

    function o() {
        this.c = "c";
    }
    f.call(o);
    
    图1中 f.call(o) 其实相当于：
    function o() {
        this.c = "c";
        var a = "name";
        this.b = "test1";
        this.add = function() {
            return "add"
        }
    }
```
` 说白了，就是把f的方法在o中走一遍，但不做保存。既然不做保存，那么如何通过call实现继承和修改函数运行时的this指针等妙用？关键在于this，对，关键还是在于this的作用域。在之前的文章反复说过，this的作用域不是定义它的函数的作用域，而是执行时的作用域。
`
##### ③寄生式继承
`这种继承方式是把原型式+工厂模式结合起来，目的是为了封装创建的过程`

```
 function obj(o){
        function F(){}
        F.prototype = o;
        return new F();
    }
 function create(o){
        var f= obj(o);
        f.run = function () {
            return this.arr;//同样，会共享引用
        };
        return f;
    }
```
###### 组合式继承的小问题:
`组合式继承是js最常用的继承模式，但组合继承的超类型在使用过程中会被调用两次；一次是创建子类型的时候，另一次是在子类型构造函数的内部`

```
 function Parent(name){
        this.name = name;
        this.arr = ['哥哥','妹妹','父母'];
    }

    Parent.prototype.run = function () {
        return this.name;
    };

    function Child(name,age){
        Parent.call(this,age);//第二次调用
        this.age = age;
    }

    Child.prototype = new Parent();//第一次调用
```
`以上代码是之前的组合继承，那么寄生组合继承，解决了两次调用的问题。`
###### 寄生组合式继承(寄生继承拓展版)

```
 function obj(o){
        function F(){}
        F.prototype = o;
        return new F();
    }
    function create(parent,test){
        var f = obj(parent.prototype);//创建对象
        f.constructor = test;//增强对象
    }

    function Parent(name){
        this.name = name;
        this.arr = ['brother','sister','parents'];
    }

    Parent.prototype.run = function () {
        return this.name;
    };

    function Child(name,age){
        Parent.call(this,name);
        this.age =age;
    }

    inheritPrototype(Parent,Child);//通过这里实现继承

    var test = new Child('trigkit4',21);
    test.arr.push('nephew');
    alert(test.arr);//
    alert(test.run());//只共享了方法

    var test2 = new Child('jack',22);
    alert(test2.arr);//引用问题解决
```

  
##### ④目前最完美的继承方法
###### 父集  
``` 
function Person(name, age) {
        this.age = age;
        this.name = name;

    }
    Person.prototype.say = function() {
        console.log(this.name, this.age);
    }
    Person.type = 'fn';
    Person.num = {
        a: 1,
        b: 2
    }
    Person.test = function() {
            console.log(666666);
        }
```
###### 子集继承父集属性  
``` 
//继承属性
    function Child(name, age) {
        Person.call(this, name, age);
    }
     /*function Child() {
        Person.apply(this, arguments);
    }*/
    //继承原型方法:
    function F() {}
    F.prototype = Person.prototype;
    Child.prototype = new F();
    这三行代码相当于:Child.prototype=Object.create(Person.prototype);

    //继承静态方法和属性,最好进行深拷贝
    for (var attr in Person) {
        console.log(attr);
        if (Person.hasOwnProperty(attr)) {
            if (Child[attr] == void 0) {
                var type = Object.prototype.toString.call(Person[attr]);
                if (type=='[object Object]' || type=='[object Array]') {
                    var str = JSON.stringify(Person[attr]);
                    var obj = JSON.parse(str);
                    Child[attr] = obj;
                } else {
                    Child[attr] = Person[attr];
                }
            }
        }
    }

```
###### 输出  
```
Child.test();
var child = new Child('jawil', 22);
child.say();
Child.num.a=22;
console.log(Child.num.a);//22
console.log()Person.num.a);//1
```

###  总结:
##### 1.类继承的缺点

```
紧耦合问题（在面向对象设计中，类继承是耦合最严重的一种设计），紧耦合还会引发另一个问题：
脆弱基类问题
层级僵化问题（新用例的出现，最终会使所有涉及到的继承层次上都出现问题）
必然重复性问题（因为层级僵化，为了适应新用例，往往只能复制，而不能修改已有代码）
大猩猩-香蕉问题（你想要的是一个香蕉，但是最终到的却是一个拿着香蕉的大猩猩，还有整个丛林）
```
##### 2.各种继承的特点    
```
拷贝继承: 通用型的 有new或无new的时候都可以

类式继承: new构造函数

原型继承: 无new的对象

继承: 子类不影响父类， 子类可以继承父类的一些功能(代码复用)

属性的继承: 调用父类的构造函数 call

方法的继承: for in : 拷贝继承(jquery也是采用拷贝继承extend)
```


# Prototype

#### 一、理解prototype相关的重要概念:
1.万物皆对象。 

```
在JS里，万物皆对象。方法（Function）是对象，方法的原型(Function.prototype)是对象。因此，它们都会具有对
象共有的特点。
即：对象具有属性__proto__，可称为隐式原型，一个对象的隐式原型指向构造该对象的构造函数的原型，这也保证了实例
能够访问在构造函数原型中定义的属性和方法。
```
2.方法(Function)

```
方法这个特殊的对象，除了和其他对象一样有上述_proto_属性之外，还有自己特有的属性——原型属性（prototype），
这个属性是一个指针，指向一个对象，这个对象的用途就是包含所有实例共享的属性和方法（我们把这个对象叫做原型对
象）。原型对象也有一个属性，叫做constructor，这个属性包含了一个指针，指回原构造函数。
```
3.私有变量和函数
`在函数内部定义的变量和函数，如果不对外提供接口，外部是无法访问到的，也就是该函数的私有的变量和函数。`

```
 function Box(){
        var color = "blue";//私有变量
        var fn = function() //私有函数
        {
            
        }
    }
```
`这样在函数对象Box外部无法访问变量color和fn，他们就变成私有的了：`

```
var obj = new Box();
    alert(obj.color);//弹出 undefined
    alert(obj.fn);//同上
```
4.静态变量和函数
`当定义一个函数后通过点号 “.”为其添加的属性和函数，通过对象本身仍然可以访问得到，但是其实例却访问不到，这样的变量和函数分别被称为静态变量和静态函数。`

```
 function Obj(){};

    Obj.num = 72;//静态变量
    Obj.fn = function()  //静态函数
    {
        
    }  
    
    alert(Obj.num);//72
    alert(typeof Obj.fn)//function
    
    var t = new Obj();
    alert(t.name);//undefined
    alert(typeof t.fn);//undefined
```
5.实例变量和函数
`在面向对象编程中除了一些库函数我们还是希望在对象定义的时候同时定义一些属性和方法，实例化后可以访问，js也能做到这样`

```
 function Box(){
                this.a=[]; //实例变量
                this.fn=function(){ //实例方法
                    
                }
            }
            
            console.log(typeof Box.a); //undefined
            console.log(typeof Box.fn); //undefined
            
            var box=new Box();
            console.log(typeof box.a); //object
            console.log(typeof box.fn); //function
```
`为实例变量和方法添加新的方法和属性`

```
function Box(){
                this.a=[]; //实例变量
                this.fn=function(){ //实例方法
                    
                }
            }
            
            var box1=new Box();
            box1.a.push(1);
            box1.fn={};
            console.log(box1.a); //[1]
            console.log(typeof box1.fn); //object

            var box2=new Box();
            console.log(box2.a); //[]
            console.log(typeof box2.fn); //function
```
`在box1中修改了a和fn，而在box2中没有改变，由于数组和函数都是对象，是引用类型，这就说明box1中的属性和方法与box2中的属性与方法虽然同名但却不是一个引用，而是对Box对象定义的属性和方法的一个复制。
这个对属性来说没有什么问题，但是对于方法来说问题就很大了，因为方法都是在做完全一样的功能，但是却又两份复制，如果一个函数对象有上千和实例方法，那么它的每个实例都要保持一份上千个方法的复制，这显然是不科学的，这可肿么办呢，prototype应运而生。
`

6.原型:prototype

```
我们创建的每个函数都有一个prototype属性，这个属性是一个指针，指向一个对象，这个对象的用途是包含可以由特
定类型的所有实例共享的属性和方法。那么，prototype就是通过调用构造函数而创建的那个对象实例的原型对象。

使用原型的好处是可以让对象实例共享它所包含的属性和方法。也就是说，不必在构造函数中添加定义对象信息，而是
可以直接将这些信息添加到原型中。使用构造函数的主要问题就是每个方法都要在每个实例中创建一遍。

在JavaScript中,一共有两种类型的值,原始值和对象值。每个对象都有一个内部属性 prototype ,我们通常称之为
原型。原型的值可以是一个对象,也可以是null。如果它的值是一个对象，则这个对象也一定有自己的原型。这样就形
成了一条线性的链，我们称之为原型链。

每个函数都有一个prototype属性;值为对象;

function fn(){};  //  声明一个prop函数;

typeof  fn.prototype;  //object;
```
`函数可以用来作为构造函数来使用。另外只有函数才有prototype属性并且可以访问到，但是对象实例不具有该属性，只有一个内部的不可访问的__proto__属性。__proto__是对象中一个指向相关原型的神秘链接。按照标准，__proto__是不对外公开的，也就是说是个私有属性，但是Firefox的引擎将他暴露了出来成为了一个共有的属性，我们可以对外访问和设置。`

```
 var Browser = function(){};
    Browser.prototype.run = function(){
        alert("I'm Gecko,a kernel of firefox");
    }
    
    var Bro = new Browser();
    Bro.run();
```
`当我们调用Bro.run()方法时，由于Bro中没有这个方法，所以，他就会去他的__proto__中去找，我们知道,这个也就是Browser.prototype，所以最终执行了该run()方法。（在这里，函数首字母大写的都代表构造函数，以用来区分普通函数）
当调用构造函数创建一个实例的时候，实例内部将包含一个内部指针（__proto__）指向构造函数的prototype，这个连接存在于实例和构造函数的prototype之间，而不是实例与构造函数之间。`

```
function Person(name){                             //构造函数
                this.name=name;
            }
            
            Person.prototype.printName=function() //原型对象
            {
                alert(this.name);
            }
            
            var person1=new Person('Byron');//实例化对象
            console.log(person1.__proto__);//Person
            console.log(person1.constructor);//自己试试看会是什么吧
            console.log(Person.prototype);//指向原型对象Person
            var person2=new Person('Frank');
```
`Person的实例person1中包含了name属性，同时自动生成一个__proto__属性，该属性指向Person的prototype，可以访问到prototype内定义的printName方法`

7.构造函数;
`构造函数有什么用处?首先来说明一下,构造函数首字母大写,可以通过New关键字创建一个新的实例,此时它就是构造函数,也就会重新创建一个作用域;每个实例化的对象的作用域是不相同的,this就会指向新创建的实例化对象;但是要是直接在全局中调用,它只是一个普通的函数,是直接挂在window下;举个例子;`

```
//构造函数;
function Person(name){
          this.name = name;
         this.sayName = function(){
                console.log(this.name);
         }
};

//当成构造函数调用;
var person1 = new Person('lily');
person1.sayName();//"lily"

//当成普通函数调用;
Person('lily');
window.sayName();//"lily"

//在其他对象的作用域中调用
var o = new Object();
Person.call(o,"lily");


/*或者用apply调用,这里不再研究call和apply的区别,
   但是要知道call的性能要比apply好
*/
Person.apply(o,["lily"]);
o.sayName();  //"lily"
```
8.构造函数的属性
`我说过,函数都有一个prototype属性,每创建一个函数,同时就会创建它的prototype对象,该对象会自动获取
constructor属性,那么构造函数也不例外,依然用上面的Person函数为例; 
function Person(){};
Person.prototype被用作新对象的原型,Person.prototype指向Person原型对象;这个原型对象有一个
constructor属性,这个属性又会指向构造函数Person; 
.protype是构造函数下的一个属性,值是一个对象,原型对象是定义的一个叫做原型方法的函数,这里是强制叫
它为prototype`

```
//举个例子;

function Person(){};

var f = Person.prototype;  //声明一个和Person相关连的原型对象

var  p = f.constructor; //和 原型相关的函数

console.log(Person.prototype.constructor == Person);//true

```

```
//再来个例子

function Person(){};

Person.prototype = {

       age: 21,

        sayAge:function(){

                 alert(this.age);

        }

}

//console.log(Person.prototype.constructor == Person);//fasle

```

`在这里使用对象字面量的形式创建了prototype对象,实质上重写了缺省的prototype对象,constructor属性也就变成新对象的constructor属性(指向Object构造函数)
所以必须要让constructor指向正确的构造函数,在prototype对象中加入constructor: Person,这样就需要自己手动把constructor属性指向了Person
console.log(Person.prototype.constructor == Object);//true`

#### 二、prototype、__proto__和原型链
###### (1)prototype和__proto__的区别
<img src="https://dn-myg6wstv.qbox.me/2e7817d676e605e54e62.png" alt="prototype与__proto__" align=center />

```
var a = {};
console.log(a.prototype); //undefined
console.log(a.__proto__);  //Object {}

var b = function(){}
console.log(b.prototype); //b {}
console.log(b.__proto__);  //function() {}
```
###### (2)构造函数、实例和原型对象的区别
`实例就是通过构造函数创建的。实例一创造出来就具有constructor属性（指向构造函数）和__proto__属性（指向原型对象），
构造函数中有一个prototype属性，这个属性是一个指针，指向它的原型对象。
原型对象内部也有一个指针（constructor属性）指向构造函数:Person.prototype.constructor = Person;
实例可以访问原型对象上定义的属性和方法。
在这里person1和person2就是实例，prototype是他们的原型对象。`
###### 再举个栗子：

```
 function Animal(name)   //积累构造函数
    {
        this.name = name;//设置对象属性
    }
    
    Animal.prototype.behavior = function() //给基类构造函数的prototype添加behavior方法
    {  
        alert("this is a "+this.name);
    }
    
    var Dog = new Animal("dog");//创建Dog对象
    var Cat = new Animal("cat");//创建Cat对象
    
    Dog.behavior();//通过Dog对象直接调用behavior方法
    Cat.behavior();//output "this is a cat"
    
    alert(Dog.behavior==Cat.behavior);//output true;
```
`可以从程序运行结果看出，构造函数的prototype上定义的方法确实可以通过对象直接调用到，而且代码是共享的。（可以试一下将Animal.prototype.behavior 中的prototype属性去掉，看看还能不能运行。）在这里，prototype属性指向Animal对象。`
######  (3)__proto__属性指向谁?
<img src="https://dn-myg6wstv.qbox.me/414693e5821245adeb86.png" alt="prototype与__proto__" align=center />

```
/*1、字面量方式*/
var a = {};
console.log(a.constructor); //function Object() { [native code] } (即构造器Object）
console.log(a.__proto__ === a.constructor.prototype); //true

/*2、构造器方式*/
var A = function (){}; var a = new A();
console.log(a.constructor); // function(){}（即构造器function A）
console.log(a.__proto__ === a.constructor.prototype); //true

/*3、Object.create()方式*/
var a1 = {a:1} 
var a2 = Object.create(a1);
console.log(a2.constructor); //function Object() { [native code] } (即构造器Object)
console.log(a2.__proto__ === a1);// true 
console.log(a2.__proto__ === a2.constructor.prototype); //false（此处即为图1中的例外情况）
```
######  (4)构造器Function和Object的关系
<img src="https://user-gold-cdn.xitu.io/2016/11/29/ce8bff088a74cda6789a11f6075b7411" alt="prototype与__proto__" align=center />

* 我们再配合代码来看一下就明白了：

```
//①构造器Function的构造器是它自身
Function.constructor=== Function;//true

//②构造器Object的构造器是Function（由此可知所有构造器的constructor都指向Function）
Object.constructor === Function;//true

//③构造器Function的__proto__是一个特殊的匿名函数function() {}
console.log(Function.__proto__);//function() {}

//④这个特殊的匿名函数的__proto__指向Object的prototype原型。
Function.__proto__.__proto__ === Object.prototype//true

//⑤Object的__proto__指向Function的prototype，也就是上面③中所述的特殊匿名函数
Object.__proto__ === Function.prototype;//true
Function.prototype === Function.__proto__;//true

总结:
1. 所有的构造器的 constructor 都指向 Function
2. Function 的 prototype 指向一个特殊匿名函数，而这个特殊匿名函数的 __proto__ 指向 Object.prototype
```

###### 理解 Object.create()继承
```
var a = {} 
var b = Object.create(a);
可以得出:a.__proto__===Object.prototype因为a是function Object(){}构造函数以Object.prototype
对象为模板创造出来的,所以a.__proto__指向了Object.prototype,而创造b是以a为模板创造出来的,所以
b.__proto__指向了a,因此b.__proto__==a,而不再指向Object.prototype,反正记住,创造这个对象的模
板是谁,也就是谁创造了我,我的__proto__就指向谁.
```
###### 什么是原型链?
<img src="https://user-gold-cdn.xitu.io/2016/11/29/877d6c73f3b810ddd1692fffd06c290b" alt="prototype与__proto__" align=center />

```
var A = function(){};
var a = new A();
console.log(a.__proto__); //Object {}（即构造器function A 的原型对象）
console.log(a.__proto__.__proto__); //Object {}（即构造器function Object 的原型对象）
console.log(a.__proto__.__proto__.__proto__); //null
```



#### 三、实例化对象与构造函数和原型对象的关系?

######  (1)实例化对象与原型对象的关系

```
当我们使用new操作符实例化一个对象出来后,会有一个内部属性[[prototype]],这个属性只读不写,在FF,chrome,

safari每个对象都支持一个_proto_实质上是一个指针,就是一个地址,指向构造函数的原型对象,所以无论创建多少个

实例,他们本质上都是继承同一个类.

```

###### (2)实例化对象与构造函数的关系

```
创建实例,关键是要理解这个new操作符在这里做了什么?

<1>创建了一个新对象;

<2>将构造函数的作用域赋值给新的对象;

<3>执行构造函数中的代码(这里就相当于给创建的新的对象添加属性);

<4>返回新对象;(函数要有返回值,对比工厂模式和构造函数发现后者没有返回值,具体原因还在研究);

function Person(){};

var person = new Person();

console.log(person.constructor ==Person);

实例化的对象有一个constructor(构造函数)属性,指向构造函数;

new操作符缺点:无法共享属性和方法;

原型链:对于原型链我只有一句话要说:任何原型链的最后一个对象就是Object,测试的方法就不在这里多做赘述;

prototype的思想:由于所有的实例对象共享同一个prototype对象，那么从外界看起来，prototype对象就好像
是实例对象的原型，而实例对象则好像"继承"了prototype对象一样
```

*  1.分析一个例子

```function B(){};

function A(){};

A.prototype = new B();

console.log(A.prototype.constructor == B); //true

console.log(A.constructor == B);//false

```

* 分析解答: 

```
B函数,我们可以显而易见的知道:

B.prototype.constructor==B;
new B().__proto__==B.prototype;

因此,对于A函数,我们已知的是:A.prototype = new B();
根据new B().__proto__==B.prototype我们可以得出一个推论:
A.prototype.__proto__ == new B().__proto__==B.prototype;
A.prototype.constructor == new B().constructor==B;


而实际上我们知道:
A.constructor==Function;
还有一点就是虽然
new B().constructor==B;
B.prototype.constructor==B;
但是new B().constructor!=B.prototype.constructor;
这个也很容易理解,比如你妈生了你,你妈也生了你弟,你们都是你妈生的,但是你跟你弟是一样的吗?

但是A.prototype.constructor == new B().constructor==B是因为
A.prototype=new B(),这时候的A.protptype已经被重写了,也就是A.prototype是new B()克隆来的,
他们是一模一样的,这时候的A.prototype.constructor他的创造已经不是A了,它已经跟B姓了,所以就得
出:A.prototype.constructor==B;

```
*  2.再来分析一个例子,子类继承父类的静态方法

```
function A(){}

A.say=function(){console.log(1)};

function B(){}

B.__proto__=A;

B.say();
//最后输出1
```

* 分析解答: 

```
首先我们可以看出这个继承的核心就在于B.__proto__=A;那么我们就来分析和拆分这句话B.__proto__到底是什么意

思,指向谁?

每个对象都支持一个_proto_实质上是一个指针,就是一个地址,指向构造函数的原型对象.函数B是由Function构造出来

的,所以B.__proto__==Function.prototype,当B没有say属性时候沿着原型链找到B.__proto__指向的对象,也就

是最初那个Function.prototype有没有say方法,由于B.__proto__=A;改变了此时B.__proto__的指向,此时会找到

A,由于A函数有say这个静态方法,所以B.say()理所当然的输出了1.


```


#### 四、总结,6个关键点

```
1.对象有属性__proto__,指向该对象的构造函数的原型对象。
2.方法除了有属性__proto__,还有属性prototype，prototype指向该方法的原型对象。
3.Object.constructor===Function；本身Object就是Function函数构造出来的。
4.所有的构造器的 constructor 都指向 Function
5.Function 的 prototype 指向一个特殊匿名函数，而这个特殊匿名函数的 __proto__ 指向 Object.prototype      
6.如何查找一个对象的constructor，就是在该对象的原型链上寻找碰到的第一个constructor属性所指向的对象。
```
#### 五、放几张图压压惊
* 1.Function、Object、Prototype、__proto__内存关系图
<img src="http://www.blogjava.net/images/blogjava_net/heavensay/web-front/8164530.png" alt="prototype与__proto__" align=center />

`上图中，红色箭头表示函数对象的原型的constructor所指向的对象。
`


* 堆区图说明：

<img src="http://www.blogjava.net/images/blogjava_net/heavensay/web-front/35166462.png" alt="prototype与__proto__" align=center />

```
Function.prototype函数对象图内部表示prototype属性的红色虚框，只是为了说明这个属性不存在。

通过上图Function、Object、Prototype关系图中，可以得出一下几点：
所有对象所有对象，包括函数对象的原型链最终都指向了Object.prototype，而
Object.prototype.__proto__===null，原型链至此结束。

Animal.prototype是一个普通对象。

Object是一个函数对象，也是Function构造的，Object.prototype是一个普通对象。
Object.prototype.__type__指向null。

Function.prototype是一个函数对象，前面说函数对象都有一个显示的prototype属性，但是Function.prototype
却没有prototype属性，即Function.prototype.prototype===undefined，所有Function.prototype函数对象
是一个特例，没有prototype属性。

Object虽是Function构造的一个函数对象，但是Object.prototype没有指向Function.prototype，即
Object.prototype!==Function.prototype。

```
* 2.prototype、constructor内存关系图(在Function、Object、Prototype关系图上加入constructor元素)：
<img src="http://www.blogjava.net/images/blogjava_net/heavensay/web-front/8199006.png" alt="prototype与__proto__" align=center />

######  Prototype跟Constructor关系介绍
`   在 JavaScript 中，每个函数对象都有名为“prototype”的属性(上面提到过Function.prototype函数对象是个例外，没有prototype属性)，用于引用原型对象。此原型对象又有名为“constructor”的属性，它反过来引用函数本身。这是一种循环引用（i.e. Animal.prototype.constructor===Animal）。
通过以下例子跟内存效果图来分析Prototype、constructor间的关系。`


```
    console.log('**************constructor****************'); 

    console.log('anim.constructor===Animal:'+(anim.constructor===Animal))    ;    //true
    console.log('Animal===Animal.prototype.constructor:'+
    (Animal===Animal.prototype.constructor))    ;    //true
    console.log('Animal.constructor===Function.prototype.constructor:'+
    (Animal.constructor===Function.prototype.constructor));   //true
    console.log('Function.prototype.constructor===Function:'+
    (Function.prototype.constructor===Function));    //true
    console.log('Function.constructor===Function.prototype.constructor:'+
    (Function.constructor===Function.prototype.constructor));    //true

    console.log('Object.prototype.constructor===Object:'+
    (Object.prototype.constructor===Object));    //true
    console.log('Object.constructor====Function:'+(Object.constructor===Function));    //true
```
#### 完结图
<img src="http://images0.cnblogs.com/blog2015/683809/201508/201728184569708.jpg" alt="prototype与__proto__" align=center />




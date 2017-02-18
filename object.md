# Object
## JavaScript对象的基本认识和创建
#### 一、什么是对象？
`面向对象（Object-Oriented，OO）的语言有一个标志，那就是都有类的概念，例如C++、Java等；但是ECMAScript没有类的概念。ECMAScript-262把对象定义为：无序属性的集合，其属性可以包含基本值、对象或者函数。通俗一点的理解就是，ECMAScript中的对象就是一组数据和功能的集合，通过new操作符后跟要创建的对象类型的名称来创建。每个对象都基于一个引用类型创建。引用可以是原生类型（相关介绍：引用类型），或者开发人员自定义的类型。`

#### 二、Object对象
`跟其他OO语言一样，Object对象类型是所有它的实例的基础。Object对象类型所具有的任何属性和方法同样也存在于更具体的对象中。`

```
//创建object对象
var o = new Object();
//若没有参数传递，可以省略圆括号，但不推荐使用
var o = new Object;
```
Object的每个实例都具有共同的基本属性和方法

| 属性或者方法 | 说明 |
| --- | --- |
| constructor | 指向创建当前对象的构造函数 |
| hasOwnProperty(name) | 检测给定属性name在实例对象（不是原型对象）中是否存在。 name以字符串形式指定 |
| isPropertyOf(object) | 检测传入的对象object是否该方法调用者的原型对象。一般格 式：Class.prototype.isPropertyOf(object) |
| propertyIsEnumerable(pr) | 检测属性pr能否用for-in循环枚举。属性pro用字符串形式指定 |
| toLocaleString() | 返回对象的字符串表示。与地区和环境对应 |
| toString() | 返回对象的字符串表示 |
| valueOf() | 返回对象的字符串、数值或布尔值表示 |


#### 三、对象的属性类型
`在ECMAScript 5中，定义了用于描述属性的各种特征的内部特性，为实现JavaScript引擎服务，所以这些特性在JavaScript中不能直接访问。属性类型有两种：数据属性和访问器属性。`
##### 1、数据属性
`数据属性包含一个数据值的位置，在这个位置对数据进行读取和写入。ECMAScript定义了4个特性来描述数据属性的行为：
`

| 特性 | 说明 |
| --- | --- |
| [[Configurable]] | 表示能否通过delete删除属性从而重新定义属性、能否修改属性的特性、能 否把属性修改为访问器属性。默认值是true |
| [[Enumerable]] | 表示能都通过for-in循环返回属性。默认值是true |
| [[Writable]] | 表述属性的数据值。默认值是undefined |
| [[Value]] | 表述属性的数据值。默认值是undefined |

##### 2、访问器属性
`访问器属性不包含数据值，不能直接定义。读取访问器属性时，调用getter函数，该函数负责返回有效值；写入访问器属性时，调用setter函数并传入新值。访问器也有4个属性：`

| 特性 | 说明 |
| --- | --- |
| [[Configurable]] | 表示能否通过delete删除属性从而重新定义属性、能否修改属性的特性、能 否把属性修改为访问器属性。默认值是true |
| [[Enumerable]] | 表示能都通过for-in循环返回属性。默认值是true |
| [[Get]] | 读取属性时调用的函数，默认是undefined |
| [[Set]] | 写入属性时调用的函数，默认是undefined |

怎么获取对象属性的默认特性呢？

```
var person = {
    name:"dwqs",
    age:20,
    interesting:"coding",
    blog:"www.ido321.com"
};
var desc = Object.getOwnPropertyDescriptor(person,"age");
alert(desc.value);   //20
alert(desc.configurable);   //true
alert(desc.enumerable);     //true
alert(desc.writable);       //true
```
`使用ECMAScript 5的Object.getOwnPropertyDescriptor(object,prop)获取给定属性的描述符：object是属性所在的对象，prop是给定属性的字符串形式，返回一个对象。若prop是访问器属性，返回的对象包含configurable、enumerable、set、get;若prop是数据属性，返回的对象包含configurable、enumerable、writable、value。
属性的特性都不能直接定义，ECMAScript提供了Object.defineProperty(obj,prop,desc)：obj是属性所在的对象，prop是给定属性的字符串形式，desc包含特性集合的对象。`

```
var person = {
name:“dwqs”,
age:20,
interesting:“coding”,
blog:“www.ido321.com”
};
//定义sex属性，writable是false,所以不能修改
Object.defineProperty(person,“sex”,{
writable:false,
value:“male”
});
alert(person.sex); //male
//在严格模式下出错，非严格模式赋值被忽略
person.sex=“female”; //writable是false,所以不能修改
alert(person.sex); //male
```
`可以多次调用Object.defineProperty()修改特性，但是，若将configurable定义为false,则不能再调用Object.defineProperty()方法将其修改为true。此外，调用Object.defineProperty()时若不指定特性的值，则configurable、enumerable和writable的默认值是false。`

```
var person = {
    name:"dwqs",
    age:20,
    interesting:"coding",
    blog:"www.ido321.com"
};
Object.defineProperty(person,"sex",{
    configurable:false,
    value:"male"
});
alert(person.sex); //male
delete person.sex;  //configurable是false，严格模式下出错，非严格模式下忽略此操作
alert(person.sex); //male

//抛出Cannot redefine property: sex错误
Object.defineProperty(person,"sex",{
    configurable:true,
    value:"male"
});
alert(person.sex);   //不能弹框
delete person.sex;
alert(person.sex);//不能弹框
```
`也可以用Object.defineProperties(obj,props)同时定义多个属性：obj是属性所在的对象，props是一个包含多个属性定义的对象`

```
var book={};
Object.defineProperties(book,{
_year:{
value:2014
},
edition:{
value:1
},
year:{
get:function()
{
return this._year;
},
set:function(newValue)
{
if(newValue > 2014)
{
this._year = newValue;
this.edition += newValue – 2014
}
}
}
});
var descs = Object.getOwnPropertyDescriptor(book,“_year”);
alert(descs.value); //2014
//调用Object.defineProperty()时若不指定特性的值，则configurable、enumerable和writable的默认
值是false。
alert(descs.configurable); //false
alert(typeof descs.get); //undefined
get和set可以不指定，也可以值指定二者之一。只有get表示属性是只读的，只有set表示属性只能写入不能读取
```

#####  3、枚举属性
` ECMAScript 5定义了两个常用的方法用于枚举属性：
  Object.keys(obj)：列举出obj能够枚举的所有属性，即对应的属性特性[[Enumerable]]是true的属性。
  Object.getOwnPropertyNames(obj)：列举出obj能够枚举的和不能枚举所有属性，即不管对应的属性特性[[Enumerable]]是true还是false。`

```
var person = {
    name:"dwqs",
    age:20,
    interesting:"coding",
    blog:"www.ido321.com"
};
Object.defineProperty(person,"sex",{
    configurable:false,
    value:"male"
});
alert(Object.keys(person));  //name,age,interesting,blog
alert(Object.getOwnPropertyNames(person)); //name,age,interesting,blog,sex
```
`因为sex属性是用defineProperty()方法定义，而对于未指定的enumerable的值默认是false。所以第一次弹框没有返回sex属性，而第二次返回了。`

#### 四、创建对象的方式
##### 1、工厂模式

```
function Person(name,age,blog)
{
    var o = new Object();
    o.name = name;
    o.age = age;
    o.blog = blog;
    o.interest = function()
    {
        alert("I like coding and writing blog!!!");
    }
    return o;
}
var per1 = Person("dwqs",20,"www.ido321.com");
alert(per1.name);  //dwqs
```
`优点：节省代码量，防止冗余。
缺点：不能识别对象，即创建的每个实例都有相同的属性，不能反应对象的类型。`

##### 2、构造函数模式

```
function Person(name,age,blog)
{
    this.name = name;
    this.age = age;
    this.blog = blog;
    this.interest = function()
    {
        alert("I like coding and writing blog!!!");
    }
}
var per1 = new Person("dwqs",20,"www.ido321.com");
alert(per1.name);
```
`优点：创建的每个实例都不同，可以识别不同对象。
缺点：创建一个实例，实例共有的方法都会重新创建。对应的解决方案可以把方法定义到全局上去，或者定义在原型对象上。`

```
function Person(name,age,blog)
{
    this.name = name;
    this.age = age;
    this.blog = blog;
    this.interest = my;
}
 function my()
{
    alert("I like coding and writing blog!!!");
```
这样，Person的所有实例均共享了my()方法。
##### 3、原型模式

```
function Person()
{
    
}
Person.prototype.name = "dwqs";
Person.prototype.age = 20;
Person.prototype.blog = "www.ido321.com";
Person.prototype.interest = function()
{
    alert("I like coding and writing blog!!!");
}
var per1 = new Person();
alert(per1.name);  //dwqs
var per2 = new Person();
alert(per1.interest == per2.interest);  //true
```
`per1和per2共享了所有原型上定义的属性和方法，显然，不能保持每个实例的独立性了，不是我们想要的。具体关于原型，后续笔记在解释。`

##### 4、原型模式+构造函数模式（推荐使用）

```
function Person(name,age,blog)
{
    this.name = name;
    this.age = age;
    this.blog = blog;
}
Person.prototype.interest = function()
{
    alert("I like coding and writing blog!!!");
}
var per1 = new Person("dwqs",20,"www.ido321.com");
alert(per1.name);  //dwqs
var per2 = new Person("i94web",22,"www.i94web.com");
alert(per2.blog);  //www.i94web.com
```
`构造函数定义每个实例的不同属性，原型共享共同的方法和属性，最大限度的节省内存。`
##### 5、动态原型模式

```
function Person(name,age,blog)
{
    this.name = name;
    this.age = age;
    this.blog = blog;
    if(typeof this.interest != "function")
    {
        Person.prototype.interest = function()
        {
            alert("I like coding and writing blog!!!");
        };
    }
}
var per1 = new Person("dwqs",20,"www.ido321.com");
per1.interest();  //I like coding and writing blog!!!
```
`将所有信息均封装在构造函数中，在需要的时候，初始化原型。`
##### 6、寄生构造函数模式

```
function Person(name,age,blog)
{    
var o = new Object();   
o.name = name;    
o.age = age;    
o.blog = blog;    
o.interest = function()   
{        
alert("I like coding and writing blog!!!");    
}    
return o;
}
var per1 = Person("dwqs",20,"www.ido321.com");
alert(per1.name);  //dwqs
```
`形式跟工厂模式一样，但这个可以在特殊情况下用来为对象创建构造函数，并且使用new操作符创建对象，而工厂模式没有使用new，直接使用返回的对象。例如，在不能修改原生对象Array时，为其添加一个新方法`

```
function SpecialArray()
{
    var values = new Array();
    values.push.apply(values,arguments);
    values.toPipedString = function()
    {
        return this.join("|");
    };
    return values;
}
var colors = new SpecialArray("red","blue","yellow");
alert(colors.toPipedString());  //red|blue|yellow

```
`需要说明的是，寄生构造函数模式返回的对象与构造函数或者构造函数的原型对象没有关系，所以不能用instanceof操作符来确定对象类型。`
##### 7、稳妥构造函数模式

```
function Person(name,age,blog)
{
     var o = new Object();
     o.sayName = function()
     {
         alert(name);
     }
     return o;
}

var per1 = Person("dwqs");
per1.sayName();  //dwqs
```
`以这种方式创建的对象,除了sayName()方法之外，没有其他办法访问name的值。per1称为稳妥对象。稳妥对象指没有公共属性，其方法也不引用this的对象。
稳妥构造函数模式与寄生构造函数有两点区别：新建对象的实例方法不引用this；不使用new操作符调用构造函数。`

## JavaScript面向对象精要(一)
#### 1.数据类型
在JavaScript中，数据类型分为两类：

```
原始类型：保存一些简单数据，如true，5等。JavaScript共有5中原始类型： 
boolean：布尔，值为true或false
number：数字，值为任何整型会浮点数值
string：字符串，值为由单引号或双引号括出的单个字符或连续字符（JavaScript不区分字符类型）
null：空类型，其仅有一个值：nulll
undefined：未定义，其仅有一个值：undefined
var name = "Pomy";
var blog = "http://www.ido321.com";
var age = 22;
alert(typeof blog); //"string"
alert(typeof age); //"number
```
`原始类型的值是直接保存在变量中，并可以用typeof进行检测。但是typeof对null的检测是返回object,而不是返回null`

```
//弹出Not null
if(typeof null){
    alert("Not null");   
}else{
    alert("null");
}
```
`所以检测null时，最好用全等于(===),其还能避免强制类型转换`

```
console.log("21" === 21);  //false
console.log("21" == 21);  //true
console.log(undefined == null);  //true
console.log(undefined === null);  //fals
```
`对于字符串、数字或者布尔值，其都有对应的方法，这些方法来自于对应的原始封装类型：String、Number和Boolean。原始封装类型将被自动创建。`


```var name = "Pomy";
var char = name.charAt(0);
console.log(char);  //"P"
在JavaScript引擎中发生的事情：

var name = "Pomy";
var temp = new String(name);
var char = temp.charAt(0);
temp = null;
console.log(char);  //"P"
```
`字符串对象的引用在用完之后立即被销毁，所以不能给字符串添加属性，并且instanceof检测对应类型时均返回false：`


```var name = "Pomy";
name.age = 21;
console.log(name.age);   //undefined
console.log(name instanceof String);  //false
引用类型：保存为对象，实质是指向内存位置的引用，所以不在变量中保存对象。除了自定义的对象，JavaScript提供了6中内建类型：
Array：数组类型，以数字为索引的一组值的有序列表
Date：日期和时间类型
Error：运行期错误类型
Function：函数类型
Object：通用对象类型
RegExp：正则表达式类型 
```
可以用new来实例化每一个对象，或者用字面量形式来创建对象：

```var obj = new Object;
var own = {
            name:"Pomy",
            blog:"http://www.ido321.com",
            "my age":22
        };
console.log(own.blog);    //访问属性
console.log(own["my age"]); 
obj = null;  //解除引用
```
`obj 并不包含对象实例，而是一个指向内存中实际对象所在位置的指针（或者说引用）。因为typeof对所有非函数的引用类型均返回object，所以需要用instanceof来检测引用类型。`

#### 2.函数
`在JavaScript中，函数就是对象。使函数不同于其他对象的决定性特性是函数存在一个被称为[[Call]]的内部属性。内部属性无法通过代码访问而是定义了代码执行时的行为。`

#### 3.创建形式
`1、函数声明：用function关键字，会被提升至上下文 
2、函数表达式：不能被提升 
3、实例化Function内建类型`

```
sayHi();    //函数提升
function sayHi(){
    console.log("Hello");
}
//其他等效等效方式
/*
var sayHi = function(){
     console.log("Hello");
}
var sayHi = new Function(" console.log(\"Hello\");");
*/
```
#### 4.参数
`JavaScript函数的另外一个独特之处在于可以给函数传递任意数量的参数。函数参数被保存在arguments类数组对象中，其自动存在函数中，能通过数字索引来引用参数，但它不是数组实例：`

```
alert(Array.isArray(arguments));   //false
```
`类数组对象arguments 保存的是函数的实参，但并不会忽略形参。因而，arguments.length返回实参列表的长度，arguments.callee.length返回形参列表的长度。`

```
function ref(value){
    return value;
}
console.log(ref("Hi"));
console.log(ref("Hi",22));
console.log(ref.length);  //1
```
#### 5.函数中的this
`关于this的问题，可参考下篇文章：JavaScript中的this。 
JavaScript提供了三个方法用于改变this的指向：call、apply和bind。三个函数的第一个参数都是指定this的值，其他参数均作为参数传递给函数。
`
#### 6.对象
`对象是一种引用类型，创建对象常见的两种方式：Object构造函数和对象字面量形式`

```
var per1 = {
    name:"Pomy",
    blog:"http://www.ido321.com"
};
var per2 = new Object;
per2.name = "不写代码的码农";
```
#### 7.属性操作
在JavaScript中，可以随时为对象添加属性：

```
per1.age = 0;
per1.sayName = function(){
    alert(this.name);   //"Pomy"
}
```
因而，在检测对象属性是否存在时，常犯的一个错误是：

```
//结果是false
if(per1.age){
    alert(true)
}else{
    alert(false);
}

```
`per1.age 是存在的，但是其值是0，所以不能满足if条件。if判断中的值是一个对象、非空字符串、非零数字或true时，判断会评估为真；而当值是一个null、undefined、0、false、NaN或空字符串时评估为假。 
因而，检测属性是否存在时，有另外的两种方式：in和hasOwnProperty()，前者会检测原型属性和自有(实例)属性，后者只检测自有(实例)属性。`

```
console.log("age" in per1);  //true
console.log(per1.hasOwnProperty("age"));  //true
console.log("toString" in per1);  //true
console.log(per1.hasOwnProperty("toString"));  //false
```
`对象per1并没有定义toString，该属性继承于Object.prototype，所以in和hasOwnProperty()检测该属性时出现差异。如果只想判断一个对象属性是不是原型，可以利用如下方法：`

```
function isPrototypeProperty(obj,name){
    return name in obj && !obj.hasOwnProperty(name);
}
```
`若要删除一个属性，用delete操作符，用于删除自有属性，不能删除原型属性。`

```
per1.toString = function(){
    console.log("per1对象");
};
console.log(per1.hasOwnProperty("toString"));   //true
per1.toString();   //"per1对象"
delete per1.toString;
console.log(per1.hasOwnProperty("toString"));   //false
console.log(per1.toString());  //[object Object]
```
`有时需要枚举对象的可枚举属性，也有两种方式：for-in循环和Object.keys()，前者依旧会遍历出原型属性，后者只返回自有属性。所有可枚举属性的内部属性[[Enumerable]]的值均为true。`

```
var per3 = {
    name:"Pomy",
    blog:"http://www.ido321.com",
    age:22,
    getAge:function(){
        return this.age;
    }
};

```
`实际上，大部分原生属性的[[Enumerable]]的值均为false，即该属性不能枚举。可以通过propertyIsEnumerable()检测属性是否可以枚举：`

```
console.log(per3.propertyIsEnumerable("name"));  //true
var pros = Object.keys(per3);  //返回可枚举属性的名字数组
console.log("length" in pros);  //true
console.log(pros.propertyIsEnumerable("length"));  //false
```
属性name是自定义的，可枚举；属性length是Array.prototype的内建属性，不可枚举。
#### 8.属性类型
属性有两种类型：数据属性和访问器属性。二者均具有四个属性特征：

```
数据属性：[[Enumerable]]、[[Configurable]]、[[Value]]和[[Writable]]
访问器属性：[[Enumerable]]、[[Configurable]]、[[Get]]和[[Set]]
```
`[[Enumerable]] :布尔值，属性是否可枚举，自定义属性默认是true。 
[[Configurable]] :布尔值，属性是否可配置(可修改或可删除)，自定义属性默认是true。它是不可逆的，即设置成false后，再设置成true会报错。 
[[Value]]：保存属性的值。 
[[Writable]]：布尔值，属性是否可写，所有属性默认可写。 
[[Get]]：获取属性值。 
[[Set]]：设置属性值。 
`

ES 5提供了两个方法用于设置这些内部属性： 
Object.defineProperty(obj,pro,desc_map) 和 Object.defineProperties(obj,pro_map)。利用这两个方法为per3添加一个属性和创建一个新对象per4:

```
Object.defineProperty(per3,"sex",{
    value:"male",
    enumerable:false,
    configurable:false, //属性不能删除和修改，该值也不能被设置成true
});
console.log(per3.sex);  //'male'
console.log(per3.propertyIsEnumerable("sex"));  //false
delete per3.sex;    //不能删除
per3.sex = "female"; //不能修改
console.log(per3.sex);  //'male'
Object.defineProperty(per3,"sex",{
    configurable:true, //报错
});
per4 = {};
Object.defineProperties(per4,{
    name:{
        value:"dwqs",
        writable:true
    },
    blog:{
        value:"http://blog.92fenxiang.com"
    },
    Name:{
        get:function(){
            return this.name;
        },
        set:function(value){
            this.name = value;
        },
        enumerable:true,
        configurable:true
    }
});
console.log(per4.name); //dwqs
per4.Name = "Pomy";
console.log(per4.Name); //Pomy
```
`需要注意的是，通过这两种方式来定义新属性时，如果不指定特征值，则默认是false,也不能创建同时具有数据特征和访问器特征的属性。可以通过Object.getOwnPropertyDescriptor()方法来获取属性特征的描述，接受两个参数：对象和属性名。若属性存在，则返回属性描述对象。`

```
var desc = Object.getOwnPropertyDescriptor(per4,"name");
console.log(desc.enumerable); //false
console.log(desc.configurable); //false
console.log(desc.writable); //true
```
根据属性的属性类型，返回的属性描述对象包含其对应的四个属性特征
#### 9.禁止修改对象
`对象和属性一样具有指导其行为的内部特征。其中，[[Extensible]]是一个布尔值，指明改对象本身是否可以被修改（[[Extensible]]值为true）。创建的对象默认都是可以扩展的，可以随时添加新的属性。 
ES5提供了三种方式：`

###### * Object.preventExtensions(obj)：创建不可扩展的obj对象，可以利用Object.isExtensible(obj)来检测obj是否可以扩展。严格模式下给不扩展对象添加属性会报错，非严格模式下则添加失败。
###### * Object.seal(obj)：封印对象，此时obj的属性变成只读，不能添加、改变或删除属性（所有属性都不可配置），其[[Extensible]]值为false,[[Configurable]]值为false。可以利用Object.isSealed(obj)来检测obj是否被封印。
###### * Object.freeze(obj)：冻结对象，不能在冻结对象上添加或删除属性，不能改变属性类型，也不能写入任何数据类型。可以利用Object.isFrozen(obj)来检测obj是否被冻结。

`注意：冻结对象和封印对象均要在严格模式下使用。`

```
"use strict";
var per5 = {
    name:"Pomy"
};
console.log(Object.isExtensible(per5));   //true
console.log(Object.isSealed(per5));         //false
console.log(Object.isFrozen(per5));       //false
Object.freeze(per5);
console.log(Object.isExtensible(per5));   //false
console.log(Object.isSealed(per5));       //true
console.log(Object.isFrozen(per5));       //true
per5.name="dwqs";
console.log(per5.name);   //"Pomy"
per5.Hi = function(){
    console.log("Hi");
};
console.log("Hi" in per5);  //false
delete per5.name;
console.log(per5.name);  //"Pomy"
var desc = Object.getOwnPropertyDescriptor(per5,"name");
console.log(desc.configurable);  //false
console.log(desc.writable);  //false
```
`注意，禁止修改对象的三个方法只对对象的自有属性有效，对原型对象的属性无效，仍然可以在原型上添加或修改属性。`

```
function Person(name){
    this.name = name;
}
var person1 = new Person("Pomy");
var person2 = new Person("dwqs");
Object.freeze(person1);
Person.prototype.Hi = function(){
    console.log("Hi");
};
person1.Hi();  //"Hi";
person2.Hi();  //"Hi";
```
## JavaScript面向对象精要(二)
#### 构造函数和原型对象
`构造函数也是函数，用new创建对象时调用的函数，与普通函数的一个区别是，其首字母应该大写。但如果将构造函数当作普通函数调用（缺少new关键字），则应该注意this指向的问题`

```
var name = "Pomy";
function Per(){
    console.log("Hello "+this.name);
}
var per1 = new Per();  //"Hello undefined"
var per2 = Per();   //"Hello Pomy
```
`使用new时，会自动创建this对象，其类型为构造函数类型，指向对象实例；缺少new关键字，this指向全局对象。 
可以用instanceof来检测对象类型，同时每个对象在创建时都自动拥有一个constructor属性，指向其构造函数(字面量形式或Object构造函数创建的对象，指向Object，自定义构造函数创建的对象则指向它的构造函数)。`

```
console.log(per1 instanceof Per);  //true
console.log(per1.constructor === Per); //true
```
`每个对象实例都有一个内部属性：[[Prototype]]，其指向该对象的原型对象。构造函数本身也具有prototype属性指向原型对象。所有创建的对象都共享该原型对象的属性和方法。`

```
function Person(){}
Person.prototype.name="dwqs";
Person.prototype.age=20;
Person.prototype.sayName=function()
{
    alert(this.name);
};
var per1 = new Person();
per1.sayName();  //dwqs
var per2 = new Person();
per2.sayName();  //dwqs
alert(per1.sayName == per2.sayName);  //true
```
![WX20170105-103842@2x](media/14835469679812/WX20170105-103842@2x.png)
`所以，**实例中的指针仅指向原型，而不指向构造函数。**ES5提供了hasOwnProperty()和isPropertyOf()方法来反应原型对象和实例之间的关系`

```
alert(Person.prototype.isPrototypeOf(per2));  //true
per1.blog = "www.ido321.com";
alert(per1.hasOwnProperty("blog"));  //true
alert(Person.prototype.hasOwnProperty("blog"));  //false
alert(per1.hasOwnProperty("name"));  //false
alert(Person.prototype.hasOwnProperty("name"));  //true
```
`因为原型对象的constructor属性是指向构造函数本身，所以在重写原型时，需要注意constructor属性的指向问题。`

```
function Hello(name){
    this.name = name;
}
//重写原型
Hello.prototype = {
    sayHi:function(){
        console.log(this.name);
    }
};
var hi = new Hello("Pomy");
console.log(hi instanceof Hello);  //true
console.log(hi.constructor === Hello); //false
console.log(hi.constructor === Object); //true
```
`使用对象字面量形式改写原型对象改变了构造函数的属性，因此constructor指向Object，而不是Hello。如果constructor指向很重要，则需要在改写原型对象时手动重置其constructor属性`

```
Hello.prototype = {
    constructor:Hello,
    sayHi:function(){
        console.log(this.name);
    }
};
console.log(hi.constructor === Hello); //true
console.log(hi.constructor === Object); //false
```
`利用原型对象的特性，我们可以很方便的在JavaScript的内建原型对象上添加自定义方法`

```
Array.prototype.sum=function(){
    return this.reduce(function(prev,cur){
        return prev+cur;
    });
};
var num = [1,2,3,4,5,6];
var res = num.sum();
console.log(res);  //21
String.prototype.capit = function(){
    return this.charAt(0).toUpperCase()+this.substring(1);
};
var msg = "hello world";
console.log(msg.capit()); //"Hello World"
```
#### 继承
`利用[[Prototype]]特性，可以实现原型继承；对于字面量形式的对象，会隐式指定Object.prototype为其[[Prototype]]，也可以通过Object.create()显示指定，其接受两个参数：第一个是[[Prototype]]指向的对象(原型对象)，第二个是可选的属性描述符对象。`

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

```
function Rect(length,width){
    this.length = length;
    this.width = width;
}
Rect.prototype.getArea = function(){
    return this.width * this.length;
};
Rect.prototype.toString = function(){
    return "[Rect"+this.length+"*"+this.width+"]";
};
function Square(size){
    this.length = size;
    this.width = size;
}
//修改prototype属性
Square.prototype = new Rect();
Square.prototype.constructor = Square;
Square.prototype.toString = function(){
    return "[Square"+this.length+"*"+this.width+"]";
};
var rect = new Rect(5,10);
var square = new Square(6);
console.log(rect.getArea());  //50
console.log(square.getArea());  //36

```
如果要访问父类的toString()，可以这样做：

```
Square.prototype.toString = function(){
    var text = Rect.prototype.toString.call(this);
    return text.replace("Rect","Square");
}
```




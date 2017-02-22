#####方便大家查阅,本文已将发表在segmentfault社区,已上热门,谢谢大家支持.
[点我查阅](https://segmentfault.com/a/1190000008432611)

**你是否在面试中遇到过各种奇葩和比较细节的问题?**

```
[]==[]
//false
[]==![]
//true
{}==!{}
//false
{}==![]
//VM1896:1 Uncaught SyntaxError: Unexpected token ==
![]=={}
//false
[]==!{}
//true
undefined==null
//true
```
**看了这种题目,是不是想抽面试官几耳光呢?哈哈,是不是看了之后一脸懵逼,两脸茫然呢?心想这什么玩意**

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyivqddwfj20he0fvq98)

**其实这些都是纸老虎,知道原理和转换规则,理解明白这些很容易的,炒鸡容易的,真的一点都不难,我们要打到一切纸老虎,不信?**

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyi20xam2j20fa0bh426)
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyi2jg2t6j20go0b4t9j)


**我们就从[] == []和[] == ![]例子切入分析一下为什么输出的结果是true而不是其它的呢?**

## []==[]为什么是false?

有点js基础应该知道对象是引用类型,就会一眼看出来[] == []会输出false,因为左边的[]和右边的[]看起来长的一样,但是他们引用的地址并不同,这个是同一类型的比较,所以相对没那么麻烦,暂时不理解[] == []为false的童鞋这里就不细说,想要弄清楚可以通过这篇文章来了解JavaScript的内存空间详解.

[前端基础进阶（一）：内存空间详细图解](http://www.jianshu.com/p/996671d4dcc4)

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyiwuiu4cj20yg0izaba)

**变量对象与堆内存**

```
简单类型都放在栈（stack）里
对象类型都放在堆（heap）里
var a = 20;
var b = 'abc';
var c = true;
var d = { m: 20 }//地址假设为0x0012ff7c
var e = { m: 20 }//重新开辟一段内存空间假设为0x0012ff8f
console.log(e==d);//false
```
##### 那为什么引用值要放在堆中，而原始值要放在栈中，不都是在内存中吗，为什么不放在一起呢?那接下来，让我们来探索问题的答案！(扯远了)

`首先，我们来看一下代码：`

```
function Person(id,name,age){
    this.id = id;
    this.name = name;
    this.age = age;
}
var num = 10;
var bol = true;
var str = "abc";
var obj = new Object();
var arr = ['a','b','c'];
var person = new Person(100,"笨蛋的座右铭",25); 
```
然后我们来看一下内存分析图：

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyiyaxri0j20b706wmx5)

`变量num,bol,str为基本数据类型，它们的值，直接存放在栈中，obj,person,arr为复合数据类型，他们的引用变量存储在栈中，指向于存储在堆中的实际对象。
由上图可知，我们无法直接操纵堆中的数据，也就是说我们无法直接操纵对象，但我们可以通过栈中对对象的引用来操作对象，就像我们通过遥控机操作电视机一样，区别在于这个电视机本身并没有控制按钮。
`
##### 现在让我们来回答为什么引用值要放在堆中，而原始值要放在栈中的问题：
`记住一句话：能量是守衡的，无非是时间换空间，空间换时间的问题
堆比栈大，栈比堆的运算速度快,对象是一个复杂的结构，并且可以自由扩展，如：数组可以无限扩充，对象可以自由添加属性。将他们放在堆中是为了不影响栈的效率。而是通过引用的方式查找到堆中的实际对象再进行操作。相对于简单数据类型而言，简单数据类型就比较稳定，并且它只占据很小的内存。不将简单数据类型放在堆是因为通过引用到堆中查找实际对象是要花费时间的，而这个综合成本远大于直接从栈中取得实际值的成本。所以简单数据类型的值直接存放在栈中。`

搬运文章:[理解js内存分配](http://blog.sina.com.cn/s/blog_8ecde0fe0102vy6e.html)

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyiz360huj20go0b4aaq)

##### 知道的大神当然可以飘过,这里主要给计算机基础薄弱的童鞋补一下课.
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyj146yqpj20cy0aywgs)

## []==![]为什么是true?

**首先第一步:你要明白ECMAScript规范里面==的真正含义**
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcy8lmuwr1j21au0kswjj)
**GetValue 会获取一个子表达式的值（消除掉左值引用），在表达式 [] == ![] 中，[] 的结果就是一个空数组的引用(上文已经介绍到数组是引用类型)，而 ![] 就有意思了，它会按照 11.4.9 和 9.2 节的要求得到 false。**

首先我们了解一下运算符的优先级:
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcy9omb2ucj20v20vigpo)

刚看到这里是不是就咬牙切齿了,好戏还在后头呢,哈哈

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyhngm80hj20a00a03yp)

**!取反运算符的优先级会高于==,所以我们先看看!在ECAMScript是怎么定义的?**

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcy9rnlv51j20v40ao40a)

**所以![]最后会是一个Boolean类型的值(这点很关键,涉及到下面的匹配选择).**

**两者比较的行为在 11.9.3 节里，所以翻到 11.9.3：**
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcy8qs1kysj21dk11q48u)


**在这段算法里，[]是Object,![]是Boolean,两者的类型不同,y是Boolean类型,由此可知[] == ![]匹配的是条件 8，所以会递归地调用[] == ToNumber(Boolean)进行比较。**

`[]空数组转化成Boolean,那么这个结果到底是true还是false呢,这个当然不是你说了算,也不是我说了算,ECMAScript定义的规范说了算:我们来看看规范:`

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcy92pvsquj21a80hgn0y)

**[]是一个对象**,所以对应转换成*Boolean*对象的值为`true`;那么**![]**对应的*Boolean*值就是`false`
进而就成了比较**[] == ToNumber(false)**了


再来看看ECMAScipt规范中对于Number的转换
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcya3mnee7j216g0logpy)

由此可以得出此时的比较成了**[]==0**;

在此处因为 [] 是对象，0是数字Number,比较过程走分支 10(若Type(x)为Object且Type(y)为String或Number， 返回比较**ToPrimitive(x) == y**的结果。),可以对比上面那张图.

`ToPrimitive` 默认是调用 `toString` 方法的（依 8.2.8），于是 **ToPrimitice([]) 等于空字符串**。

再来看看*ECMAScript*标准怎么定义**ToPrimitice**方法的:

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcycf5tjn2j21fe0k0dm2)

是不是看了这个定义,还是一脸懵逼,ToPrimitive这尼玛什么玩意啊?这不是等于没说吗?
![](https://gss0.baidu.com/7Po3dSag_xI4khGko9WTAnF6hhy/zhidao/wh%3D600%2C800/sign=618965f65543fbf2c579ae25804ee6b8/6f061d950a7b0208dc217f1f64d9f2d3562cc860.jpg)

再来看看火狐MDN上面文档的介绍:
[JS::ToPrimitive]('https://developer.mozilla.org/zh-CN/docs/Mozilla/Projects/SpiderMonkey/JSAPI_Reference/JS::ToPrimitive)
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyegnfjpij210m11mqan)

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyj2c4dv4j205k05kt8l)

查了一下资料,上面要说的可以概括成:

```
ToPrimitive(obj,preferredType)

JS引擎内部转换为原始值ToPrimitive(obj,preferredType)函数接受两个参数，第一个obj为被转换的对象，第二个
preferredType为希望转换成的类型（默认为空，接受的值为Number或String）

在执行ToPrimitive(obj,preferredType)时如果第二个参数为空并且obj为Date的事例时，此时preferredType会
被设置为String，其他情况下preferredType都会被设置为Number如果preferredType为Number，ToPrimitive执
行过程如
下：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
3. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
4. 否则抛异常。

如果preferredType为String，将上面的第2步和第3步调换，即：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
3. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
4. 否则抛异常。

```
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyj3ahyznj206j06fdfu)

首先我们要明白**obj.valueOf()**和 **obj.toString()**还有原始值分别是什么意思,这是弄懂上面描述的前提之一:

**toString用来返回对象的字符串表示。**

```
var obj = {};
console.log(obj.toString());//[object Object]
    
var date = new Date();
console.log(date.toString());//Sun Feb 28 2016 13:40:36 GMT+0800 (中国标准时间)
```

**valueOf方法返回对象的原始值，可能是字符串、数值或bool值等，看具体的对象。**

```
var obj = {
  name: "obj"
};
console.log(obj.valueOf());//Object {name: "obj"}

var arr1 = [1];
console.log(arr1.valueOf());//[1]

var arr2 = [];
console.log(arr2.valueOf());//""空字符串


var date = new Date();
console.log(date.valueOf());//1456638436303
如代码所示，三个不同的对象实例调用valueOf返回不同的数据
```
**原始值指的是['Null','Undefined','String','Boolean','Number']五种基本数据类型之一(我猜的,查了一下
确实是这样的)**

弄清楚这些以后,举个简单的例子:

```
var a={};
ToPrimitive(a)

分析:a是对象类型但不是Date实例对象,所以preferredType默认是Number,先调用a.valueOf()不是原始值,继续来调
用a.toString()得到string字符串,此时为原始值,返回之.所以最后ToPrimitive(a)得到就是"[object Object]".

```

如果觉得描述还不好明白,一大堆描述晦涩又难懂,我们用代码说话:

```

const toPrimitive = (obj, preferredType='Number') => {
    let Utils = {
        typeOf: function(obj) {
            return Object.prototype.toString.call(obj).slice(8, -1);
        },
        isPrimitive: function(obj) {
            let types = ['Null', 'String', 'Boolean', 'Undefined', 'Number'];
            return types.indexOf(this.typeOf(obj)) !== -1;
        }
    };
   
    if (Utils.isPrimitive(obj)) {
        return obj;
    }
    
    preferredType = (preferredType === 'String' || Utils.typeOf(obj) === 'Date') ?
     'String' : 'Number';

    if (preferredType === 'Number') {
        if (Utils.isPrimitive(obj.valueOf())) {
            return obj.valueOf()
        };
        if (Utils.isPrimitive(obj.toString())) {
            return obj.toString()
        };
    } else {
        if (Utils.isPrimitive(obj.toString())) {
            return obj.toString()
        };
        if (Utils.isPrimitive(obj.valueOf())) {
            return obj.valueOf()
        };
    }
}

var a={};
ToPrimitive(a);//"[object Object]",与上面文字分析的一致
```
分析了这么多,刚才分析到哪里了,好像到了比较**ToPrimitive([]) == 0**现在我们知道**ToPrimitive([])=""**,也就是空字符串;

那么最后就变成了**""==0**这种状态,继续看和比较这张图
![](http://ww1.sinaimg.cn/large/a660cab2gy1fcy8qs1kysj21dk11q48u)

发现**typeof("")为string,0为number**,发现第5条满足规则,最后就成了**toNumber("")==0**的比较了,根据toNumber的转换规则:

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyhhn5tppj20vy09240w)

**所以toNumber("")=0,最后也就成了0 == 0的问题,于是[]==![]最后成了0 == 0的问题,答案显而易见为true,一波三折**

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyhmvo222j20c80c874n)

### 最后总结一下
==运算规则的图形化表示

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyjfmwc03j20xs0gyjv0)


```
前面说得很乱，根据我们得到的最终的图3，我们总结一下==运算的规则：

1. undefined == null，结果是true。且它俩与所有其他值比较的结果都是false。

2. String == Boolean，需要两个操作数同时转为Number。

3. String/Boolean == Number，需要String/Boolean转为Number。

4. Object == Primitive，需要Object转为Primitive(具体通过valueOf和toString方法)。

瞧见没有，一共只有4条规则！是不是很清晰、很简单。
```
### 主要参考文章和文献

[ECMAScript5.1规范中文版](http://yanhaijing.com/es5/#about)

[通过一张简单的图，让你彻底地、永久地搞懂JS的==运算](http://www.admin10000.com/document/9242.html)

[JavaScript中加号运算符的类型转换优先级是什么？](https://www.zhihu.com/question/21484710)

![](http://ww1.sinaimg.cn/large/a660cab2gy1fcyj463ngvj20c80b8mxt)

喜欢我总结的文章点个star
我的[github博客](https://github.com/jawil/blog)地址,总结的第一篇,不好之处和借鉴不得到支持还望见谅!

#js知识点总结

###1. 0与-0
```
0 ===-0，1/0 => Infinity, 1/-0 => -Infinity
```

###2.NaN
```
NaN !== NaN

0/0 //NaN
```
####Number.isNaN() 和 全局 isNaN()

Number.isNaN() 方法确定传递的值是否为NaN并且类型是Number。它是原始的全局isNaN()的更强大的版本。，只有在参数是真正的数字类型，且值为 NaN 的时候才会返回 true。
```
Number.isNaN = Number.isNaN || function(value) {
    return typeof value === "number" && isNaN(value);
}
```

全局的isNaN会先将value强制转换为Number类型，再进行判断。强制转换过程中如果无法转换成Number类型，那么就会直接转换成NaN。
```
isNaN = function(value) {
   Number.isNaN(Number(value));
}
```

 确定一个值是哪种基本类型可以使用 typeof 操作符，而确定一个值是哪种引用类型可以使用
instanceof 操作符。
###3.instanceof 和 isPrototypeOf
#####(1)首先了解下原型链相关知识:

每个函数都有一个prototype属性，这个属性是一个指针，他指向这个函数的原型对象。也就是每一个函数都有一个原型对象，这个对象可以通过函数的prototype属性访问到。

同时，这个原型对象中有一个constructor属性，这个属性也是一个指针，他指向我们创建的那个函数。所以原型和构造函数可以相互通过指针访问。

而每一个通过构造函数new出来的实例，都拥有 [[Prototype]]属性，这个属性也是一个指针，他指向他们的原型对象，实例通过这个指针可以访问到原型对象上的方法。虽然在脚本中没有标准的方式访问[[Prototype]] ，但 Firefox、Safari 和 Chrome 在每个对象上都支持一个属性__proto__ ；而在其他实现中，这个属性对脚本则是完全不可见的。

他们的关系如下图：
![1.png](https://github.com/huangtting/Blog/raw/master/images/1.png)

利用原型链实现继承就是将一个函数的默认原型对象替换成另一个函数的实例

借用高程的例子理解原型链:

```
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.property;
};
function SubType(){
	this.subproperty = false;
}
//继承了 SuperType
//SubType的原型对象被替换成了SuperType的一个实例
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
	return this.subproperty;
};
var instance = new SubType();
alert(instance.getSuperValue()); //true
```
如下图：

![2.png](https://github.com/huangtting/Blog/raw/master/images/2.png)

通过原型链进行搜索，就是通过实例的prototype属性不断向上寻找方法，直到原型链末端。

最后，所有引用类型默认都继承了Object ，而这个继承也是通过原型链实现的。所有函数的默认原型都是Object 的实例，因此默认原型都会包含一个内部指针，指向 Object.prototype 。

如下图:
![3.png](https://github.com/huangtting/Blog/raw/master/images/3.png)

#####(2)确定原型和实例的关系

第一种方式是使用 instanceof 操作符，这个操作符用来测试实例与原型链中出现过的构造函数，检测一个对象在其原型链中是否存在某个构造函数的 prototype 属性.

```
alert(instance instanceof Object); //true
alert(instance instanceof SuperType); //true
alert(instance instanceof SubType); //true
```

第二种方式是使用 isPrototypeOf() 方法。同样，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，因此 isPrototypeOf() 方法也会返回 true.

```
alert(Object.prototype.isPrototypeOf(instance)); //true
alert(SuperType.prototype.isPrototypeOf(instance)); //true
alert(SubType.prototype.isPrototypeOf(instance)); //true
```
#####原型链存在的问题

问题出在包含引用类型值的原型属性会被所有实例共享；而这也正是为什么要在构造函数中，而不是在原型对象中定义属性的原因。但是在通过原型来实现继承时，原型实际上会变成另一个类型的实例。于是，原先的实例属性也就顺理成章地变成了现在的原型属性了。

#####ES6的语法糖


###4.for in 和 hasOwnProperty

for in语句以任意顺序遍历一个对象的可枚举属性，包括原型链上的可枚举属性。

hasOwnProperty遍历对象本身的(不包括原型链)的可枚举以及不可枚举的属性。

通过for in 和hasOwnProperty配合可以判断一个属性是对象本身的还是来自于原型链，也可以得到对象本身的可数属性。

#####IE<9下的for in bug
underscore.js中处理了for in在IE<9下的bug。

IE < 9，重写的 `toString` 属性被认为不可枚举。根据这个特性来判断当前浏览器是否是IE<9,代码如下:

```
//检测浏览器是否是IE<9
var hasEnumBug = !{toString: null}.propertyIsEnumerable('toString');

//其他在IE<9下for in不能遍历到的属性，还有一个constructor没有包含在数组中，被单独提出单独处理了
var nonEnumerableProps = ['valueOf', 'isPrototypeOf', 'toString','propertyIsEnumerable', 'hasOwnProperty', 'toLocaleString'];
```

underscore.js通过遍历nonEnumerableProps，并检测每个属性是否包含在对象中，然后push进keys数组中。

详情可以看collectNonEnumProps函数。

###5.判断对象的类型

(1)通过Object.prototype.toString.call判断

isArray
```
if (!Array.isArray) {
  Array.isArray = function(arg) {
    return Object.prototype.toString.call(arg) === '[object Array]';
  };
}
```

isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError.
```
_.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name) {
        _['is' + name] = function(obj) {
            return toString.call(obj) === '[object ' + name + ']';
        };
});
```

对IE<9下，isArguments检测obj中是否有callee属性。因为IE<9，不提供对arguments的检测，所以对arguments调用Object.prototype.toString.call得到[object Object]。如下：
```
if (!_.isArguments(arguments)) {
        _.isArguments = function(obj) {
            return _.has(obj, 'callee');
        };
}
```

isBoolean
```
 _.isBoolean = function(obj) {
        return obj === true || obj === false || toString.call(obj) === '[object Boolean]';
    };
```

(2)通过typeof判断

typeof取值：
"undefined" ——如果这个值未定义；
"boolean" ——如果这个值是布尔值；
"string" ——如果这个值是字符串；
"number" ——如果这个值是数值；
"object" ——如果这个值是对象或 null ；
"function" ——如果这个值是函数。

有些时候， typeof 操作符会返回一些令人迷惑但技术上却正确的值。比如，调用 typeof null会返回 "object" ，因为特殊值 null 被认为是一个空的对象引用。Safari 5 及之前版本、Chrome 7 及之前版本在对正则表达式调用 typeof 操作符时会返回 "function" ，而其他浏览器在这种情况下会返回
"object"。

注意typeof null return 'object',这是一个遗留问题。但isObject不将null视作Object

isObject，需要注意的是JavaScript数组和函数是对象，字符串和数字不是
```
_.isObject = function(obj) {
        var type = typeof obj;
        //!!obj将obj转换成布尔类型，保证obj不是null
        // typeof null =>object
        return type === 'function' || type === 'object' && !!obj;
};
```

(3)其他

isArrayLike，类数组，underscore.js中认为有length属性，并且 length 属性值为 Number 类型的元素都是类数组。

包括数组、arguments、HTML Collection 以及 NodeList 等等
包括类似 {length: 10} 这样的对象
包括字符串等

```
var isArrayLike = function(collection) {
        // 返回参数 collection 的 length 属性值
        var length = getLength(collection);
        return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
};
```
isUndefined使用void 0，是因为void 0始终返回undefined，但是全局的undefined可能被重写。
```
 _.isUndefined = function(obj) {
        return obj === void 0;
    };
```

###6.bind
bind函数的polyfill比较复杂。

首先bind返回一个函数，该函数可以被new调用。要根据this的指向，通过instanceof判断是否是new调用，然后返回不一样的返回值。

以及bind时可以传入参数，这些参数会优先使用。

具体看代码吧。

###7.函数节流和去抖
#####函数节流
throttle。函数节流的作用是以一定的频率执行函数，控制函数的触发频率。

实现方法：
其一通过previous记录上一次执行的时间，判断是否已经wait足够久，如果是，则执行，并更新上次执行的时间戳，如此循环；
其二使用定时器，每次真正执行函数之前，如果已经存在定时器，则不执行。直到定时器触发，handler 被清除，然后执行，并重新设置定时器。

underscore.js中同时使用了这两个方法，为的是提供{leading:false}，{trailing:false}两种模式。
以scroll为例，滚动开始时会触发一次函数
滚动结束后也会触发一次，所以正常情况下，都会触发两次函数。
但是可以通过 {trailing: false} {leading: false}配置。

```
window.onscroll = _.throttle(log, 1000);
window.onscroll = _.throttle(log, 1000, {leading: false});
window.onscroll = _.throttle(log, 1000, {trailing:false});
```

#####函数去抖
debounce。函数去抖的作用是多次调用函数后，函数真正地执行是在最后一次函数调用后的wait毫秒。

实现方法：在多次连续调用的每一次都设置setTimeout，直到wait时间后，由setTimeout执行函数。






 
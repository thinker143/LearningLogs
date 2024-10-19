# 《JavaScript语言精粹》读书笔记

### 一、精华

1、JavaScript建立在一些非常优秀的想法（函数、弱类型、动态对象和富有表现力的对象字面量表示法）和非常糟糕的想法（基于全局变量的编程模型）之上。

2、JavaScript的函数主要是基于词法作用域的顶级对象。JavaScript是第一个成为主流Lambda语言。

3、JavaScript是一门弱类型的语言。

4、JavaScript有非常强大的对象字面量表示法。

5、JavaScript有一个无类型的对象系统，在这个系统中，对象可以直接从其他对象继承属性。。

### 二、语法

1、建议避免使用/**/注释，而用//注释代替它。

2，JavaScript标识符由一个字母开头，其后可以加上一个或多个字母，数字或下划线（还允许以下划线和美元符开头）。标识符被用于语句、变量、参数、属性名、运算符和标记。

3、JavaScript不允许使用保留字来命名变量或参数。

4、JavaScript只有一个数字类型，在内部被表示为64位的浮点数，和Java的double数字类型一样。

5、JavaScript的所有字符都是16位的，并没有字符类型。

6、没有代码块作用域，只有函数作用域；NaN == false；普通for循环和 for-in循环；如果一个函数没有指定返回表达式，那么返回值为undefined。

### 三、对象

1、对象是属性的容器。原型链是允许对象继承另一对象的属性，能够减少对象初始化的时间和内存消耗。

2、要检索对象有两个方法：

- object.key
- object["key"] 如果没有则返回undefined

3、更新

已存在的会被更新，不存在的会扩充

4、引用

对象通过引用来传递，永远不会被拷贝

5、原型

- 所有通过对象字面量创建的对象都会连接到Object.prototype这个标准的对象。
- 当对一个对象做出改变时，不会触及到该对象的原型。
- 原型只有在检索值的时候才会被用到，如果对象没有此属性，则从原型中寻找，以此类推。
- 原型关系时一种动态的关系，如果添加一个新的属性，则立刻可以对所有基于该原型的对象可见。
- 可以用hasOwnProperty()来检验是否为自身属性

6、删除

delete可以删除对象的属性

7、减少全局变量污染

通过创建对象创建全局变量可以减少全局污染。

```
MYAPP.stooge = {
    "first" : "JOE"
}
```

### 四、函数

1、在JavaScript中，函数就是对象。对象是key－value对的集合，并拥有一个连到原型的对象的隐藏连接。

- 对象字面量产生的对象连接到Object.prototype中
- 函数对象连接到Function.prototype中

2、函数字面量

如下：

```
var add = function() {

}
```

可以省略函数名，即为匿名函数。

3、调用

调用函数时，除了声音时定义的形式参数，每个函数海接收两个附加的参数： this，arguments(array－like，只有一个length属性)。 当实际参数arguments的个数与形式参数的个数不匹配的时候不会导致运行错误：

```
如果实际参数过多，超出的将会被忽略；
如果实际参数过少，缺失的将会被替代位undefined；
```

一共有四种调用模式：

- 方法调用模式

```
当方法时对象里的一个属性时：
myObject.increament()；
```

- 函数调用模式

```
add(3,4);
缺点是，调用时，函数内部的this是指向全局的，这是一个缺陷。
```

- 构造器调用模式

```
在一个函数前面带上new来调用。这种调用方法会将创建一个隐藏连接到该函数的prototype成员的新对象，
同时this将会绑定到新对象上。
这种属于构造函数，但是不推荐。
```

- call／apply调用模式

```
apply调用可以构建一个参数数组调用，并且可以绑定this
```

4、参数

当函数被调用时，会得到一个arguments数组。 通过它，函数可以访问所有它被调用时传递给它的参数列表。 这样，可以编写一个无须制定参数个数的函数。

```
var sum = function() {
    var i , sum = 0;
    for (i = 0; i < arguments.length; i++) {
        sum += arguments[i];
    }
    return sum;
};

document.writeln(sum(4,8,15,16,23,42)); -> 108
```

5、返回

没有返回值，则返回undefined.

6、异常

用try……catch来抛出一个exception对象，其中有两个属性：

- name: 异常类型
- message: 异常描述

```
try {
    xxx
} catch(e) {
    document.write(e.name + '' + e.message);
}
```

7、给类型增加方法

可以给基本类型：Object\number\array\string\boolean\Function...增加类型一边对所有对象可用。

```
Object.prototype.method = function () {};
```

8、递归

历遍所有的DOM节点,并对每个点进行一个function deal(){}的处理

```
var traversal = function (node, deal) {
    deal(node);
    node = node.firstChild;
    while (node) {
        traversal(node,deal);
        node = node.nextSibling;
    }
}
```

9、闭包

函数可以访问它呗创建时所处的上下文环境－>`闭包`；

```
var quo = function (status) {
    return {
        get_status:function() {
            return status;
        }
    }
}

var myQuo = quo("amazed");

此时myQuo获得了一个quo()返回的get_status方法的一个新对象，
但是这个时候myQuo.get_status()方法仍然可以访问quo对象中的status属性，
这个属性不是一个复制！！！而是就是他的本身！！！！
```

另外一种能说明`内部函数拥有外部函数的变量，比外部函数自己还有更长的生命周期`的情况： 设置一个setTimeOut()去延时调用内部函数，此时外部函数已经结束， 但是内部函数还能够访问道外部函数的变量。 由于这个原因，还可以用另外一个例子来说明： `内部函数拥有外部函数的变量，比外部函数自己还有更长的生命周期｀ 以及 `调用的变量不是复制，而就是它的本身！`

```
var add_the_handlers = function (nodes) {
    var i;
    for (i = 0; i < nodes.length; i++) {
        nodes[i].onclick = function () {
            alert(i);
        }
    }
};
这种情况下，输出的全是最后一个i，因为这里绑定是变量i，而不是构造时变量i的值。
```

解决方案：

```
var add_the_handler = function(nodes) {
    var i;
    for (i = 0; i < nodes.length; i++) {
        nodes[i].onclick = function(i) {
            return function(i) {
                alert(i);
            }
        }(i)
    }
}
```

10、回调

实现不连续事件处理。

```
func(request, function(){});
```

11、模块

模块是一个提供接口，却隐藏状态与实现的函数或者对象。 对于：运行完一个函数，然后需要保存一定状态，可以不放在全局变量中，而用闭包来实现。 其实就是对应着类c语言中的私有变量和私有函数,让数据更加安全。

```
String.method('deentity', function () {
    //私有变量
    var entity = {};

    //返回一个可以访问私有变量的函数
    return function() {
        ......
    }
//此处是立刻执行，立刻返回给deentity。
}())；

调用：deentity（）；
```

### 五、继承

1、原型

原型模式中，抛弃类的定义，基于原型的继承：一个新对象可以继承一个旧对象的属性。

```
if (typeof Object.beget !== 'function') {
    Object.beget = function(o) {
        var F = function() {};
        F.prototype = o;
        return new F();
    };
}
```

原型继承：

```
var women = {

}

var myWomen = Object.beget(women);
```

### 六、数组

1、JavaScript的数组允许混合类型。

```
var empty = [];
empty.length;
```

2、长度

每个数组都有一个length属性。但是这个length没有上界，数组可以无限扩展。 可以通过两种方式尾部加元素：

```
1. numbers[numbers.length] = 1;
2. numbers.push(1);
```

3、删除

```
arrayObject.splice(index,howmany,item1,.....,itemX)
index   必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
howmany 必需。要删除的项目数量。如果设置为 0，则不会删除项目。
item1, ..., itemX   可选。向数组添加的新项目。
```

4、枚举

- 因为数组其实是对象，可以用for in来遍历。但是for in无法保证属性的顺序。
- 还是用for循环好。
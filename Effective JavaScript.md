## 编写高质量JavaScript代码的68个有效方法



### 1. 了解你使用的JavaScript版本

ES5引入版本控制——严格模式（strict mode），启动严格模式的方式是在程序最开始加一个字符串字面量。

```
"use strict";

// 也可以在函数体开始处加入
function fn(){
  "use strict";
  // ...
}
```

字符串指令向后兼容，解释执行字符串字面量没有任何副作用，也就使得严格模式的代码可以运行在旧的JavaScript引擎上但不进行严格模式检查。比如下面这段代码严格模式下报错，不是严格模式下不会报错：

```
function fn(x){
  var arguments = [];
}
```

`"use strict"`指令只有在脚本或函数的顶部才生效，这是严格模式的陷阱。在开发中多个独立的文件要部署到产品环境时需要连接成一个单一的文件，如果一个file1运行于严格模式，file2不是运行于严格模式，如何正确的连接两个文件呢？

**解决方案：通过IIFE自执行的方式连接多个文件**，即：

```
(function(){
  // file1
  "use strict";
  // ...
})();
(function(){
  // file2
  // no strict-mode directive
  function fn(){
    var arguments = [];
    // ...
  }
})();
```

由于每个文件的内容都被放置在单独的作用域中，所以使用严格模式指令只影响本文件的内容，这种方式会导致文件内容不会在全局作用域内解释，也就是var、function声明的变量不会被视为全局变量。这与**模块化思想**很相似：模块系统通过自动地将每个模块的内容放置在单独的函数中的方式来管理文件和依赖，每个文件都是在局部作用域内，可以自行决定是否使用严格模式。




### 2. 理解JavaScript的浮点数

JavaScript中所有数字都是双精度浮点数。（－2^53 ~ 2^53）

位运算是按照整数的32位二进制运算的。

```
(8).toString(2); // "1000"
parseInt("1001", 2); // 9
```

⚠️浮点数运算是不精确的！不满足结合律。

```
0.1 + 0.2 // 0.30000000000000004
```



### 3. 当心隐式的强制转换

算术运算符`-, *, /, % `在计算之前都会将其参数转换为数字，而运算符`+`，既重载了数字相加，又重载了字符串连接操作，取决于参数类型。

```
2 + 3; // 5
"h" + "i"; // "hi"
"2" + 3; // "23"
```

**位运算符会将操作数转换为数字**，包括`~, &, ^, |, <<, >>, >>> `。这些强制转换会非常方便，我总结了几种转换为数字的方式：

```
// 转换为数字
+ a;
a - 0;

// 向下取整
~~ a;
a | 0;
a >> 0;
a << 0;
```

NaN是JavaScript中唯一不等于自身的值。检测方式：

```
function isRealNaN(x){
  return x !== x;
}
```

对象的隐式转换会调用自身的toString方法：

```
"test: " + Math; // "test: [object Math]"
Math.toString(); // "[object Math]"
```

但是，当对象同时包含toString和valueOf时，运算符`＋`的运行结果选择了转换为数字的valueOf：

```
var obj = {
	toString: function(){
		return "[object MyObject]";
	},
	valueOf: function(){
		return 10;
	}
}

"object: " + obj; // "object: 10"
```

上述约定也就意味着，具有valueOf方法的对象应该实现toString方法，返回valueOf方法产生的数字的字符串表示。

**JS中的假值：**（其他所有值都是真值）

```
false
0
-0
""
NaN
null
undefined
```



### 4. 原始类型优于封装对象

* JS原始类型：Boolean、Number、String、null、undefined

* 做相等比较时，原始类型的封装对象与原始类型行为不一样

* 获取和设置原始类型值的属性会隐式地创建封装对象


### 5. 避免对混合类型使用＝＝运算符

* 当参数类型不同时，＝＝运算符会进行隐式的强制转换


### 6. 了解分号插入的局限

* 在以`（，［，＋，－，／`字符开头的语句前绝不能省略分号，应该防御性地增加分号。不然会和前面省略了分号的语句解析成一条语句。

  ```
  ;(function(){
    // ...
  })()
  ```

* 在`return，throw，break，continue，++，—`这些关键字或运算符的参数之前绝对不能换行。关键字后的换行会强制自动的插入分号。

  ```
  return
  {};
  // 会被解析成
  return;
  {}
  ;
  ```

* for循环头部省略分号换行，解析时不会插入分号。空循环体的while也是。



### 7. 视字符串为16位的代码单元序列

这部分内容其实没太看懂。。。

* JavaScript字符串由16位的代码单元组成，而不是Unicode代码点组成。
* JS使用两个代码单元表示2^16及其以上的Unicode代码点。这两个代码单元被称为代理对。
* 代理对甩开了字符串元素计数，length、chatAt、chatCodeAt方法以及正则表达式模式（例如“.”）收到了影响，获取到的是代码单元。
* 使用第三方的库编写可识别代码点的字符串操作。
* 每当你使用一个含有字符串操作的库时，需要查阅文档，看如何处理代码点的整个范围。

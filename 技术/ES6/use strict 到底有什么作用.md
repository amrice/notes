![image-20210711223938112](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210711223945.png)

## 启用严格模式

使用use strict实际上是启动了严格模式(Strict mode)。我们可以将use strict语句放在js文件的开头第一行，也可以放在某个函数的函数体中的第一行，只是他们的作用域不一样而已。

## 跟普通模式(Snoopy Mode)有什么不同

严格模式实际上从ES5时代就已经存在了，当我们启用之后，它会强制JS处于一个“严格”的执行环境中，与“普通”模式相比，“严格”模式有以下三个不同的地方：

- 普通模式下的静默错误，在严格模式下将会直接抛出错误，很强势的提醒我们修正这个错误。
- 严格模式提高了JS的执行效率，如：它不再猜测不确定的数据类型，禁用with这种容易出错的语句等。
- 严格模式禁用了JS未来版本中可能用到的一些关键字，如：implements interface let package private yield 等。

## 严格模式带来的不同

### 1、给未声明的变量赋值会报ReferenceError

```jsx
var myVariable = 'Miau';
myVriable = 'Woof';
```

上面的代码，本来是想给myVariable重新赋值，但由于手误将myVariable写成了myVriable。如果是Snoopy mode不会有事，它会在window对象上创建一个新属性myVriable并赋值，但Strict Mode中会直接报错。

### 2、非法赋值会报TypeError

```jsx
var undefined = '5';
var NaN = 42;
```

上面的代码，给undefined，NaN赋值在Snoopy mode中不会出错，但Strict Mode中直接报TypeError。

```jsx
var obj = {};
Object.defineProperty(obj, 'a', { value: 42, writable: false });
obj1.a = 42;
```

还有对配置属性writable为false的属性赋值也会报同样的错误

### 3、同一个对象中定义相同的属性名会报SyntaxError

```jsx
var myObject = {
   x: '5',
   y: 'a string',
   x: 'another string'
}
console.log(myObject.x); // 'another string'
```

### 4、同一个函数中定义同名的参数会报SyntaxError

```jsx
function doTheThing (a, b, a, a) {
    // ...
}
```

上面的代码，Snoopy mode时后面的同名参数会覆盖前面的，而Strict mode中直接报语法错误。

### 5、函数的参数不会随arguments对象的改变而改变

```jsx
function f(a) {
  "use strict";
  a = 42;
  return [a, arguments[0]];
}
var pair = f(17);
console.assert(pair[0] === 42);
console.assert(pair[1] === 17);
```

严格模式下，函数的 arguments 对象会保存函数被调用时的原始参数。arguments[i] 的值不会随与之相应的参数的值的改变而变化，同名参数的值也不会随与之相应的 arguments[i] 的值的改变而变化。

### 6、不再支持arguments.callee--会报类型错误TypeError

```jsx
"use strict";
var f = function() { return arguments.callee; };
f(); // 抛出类型错误
```

上面的代码，Snoopy mode arguments.callee会指向当前正在执行函数

### 7、eval与arguments不能通过代码进行赋值和运算，否则报语法错误

```jsx
"use strict";
eval = 17;
arguments++;
++eval;
var obj = { set p(arguments) { } };
var eval;
try { } catch (arguments) { }
function x(eval) { }
function arguments() { }
var y = function eval() { };
var f = new Function("arguments", "'use strict'; return 17;");
```

### 8、eval函数引入的新变量不再会污染当前作用域

```jsx
function aFunction () {
    eval('var x = 2;');
}
```

在Snoopy mode下，上面的代码将会在aFunction中创建一个新变量x，而在Strict mode中，x不会出现在aFunction的作用域中，而是eval函数本身产生的作用域。

### 9、with语句被禁用

with语句太古老以至于本来就很少有人用，with语句的作用本来是方便对对象属性进行一次性操作，如：

```jsx
var obj = { x: 'Hello', y: 'Goodbye' };
with (obj) {
    console.log(x); // 'Hello'
}
```

with语句中obj将作为其接下来{}内所有操作的上下文。初一看这似乎没问题，直到遇到下面的情况：

```jsx
var obj = { x: 'Hello' };
var x = 'Non';
with (obj) {
    console.log(x);
}
```

此时js不知道x应该取obj中的x属性还是外面域中定义的x变量了，Strict mode中JS拒绝猜测！

### 10、this的指向不再是全局对象

```jsx
function test(){
    console.log(this);
}
test();
```

上面的代码，在Snoopy mode中会输出window，而Strict mode中会输出undefined

## 总结

上面列举的10点不同之处，感觉除了第一和第十可能在现代JS中遇到而引起错误，其他的情况一般人应该都不会遇到，因为其余八种情况，第一眼就能让人感觉到不对劲，就像有些公司的奇葩面试题，看上去太别扭。除了上面的10点之外还有其他的不同之处吗？我们再写写？。。。

## 参考资料

1. [https://exploringjs.com/es6/ch_one-javascript.html#sec_strict-mode-and-es6](https://exploringjs.com/es6/ch_one-javascript.html#sec_strict-mode-and-es6)
2. [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)
3. [https://javascript.info/strict-mode](https://javascript.info/strict-mode)
4. [https://stackoverflow.com/questions/1335851/what-does-use-strict-do-in-javascript-and-what-is-the-reasoning-behind-it](https://stackoverflow.com/questions/1335851/what-does-use-strict-do-in-javascript-and-what-is-the-reasoning-behind-it)
5. [https://johnresig.com/blog/ecmascript-5-strict-mode-json-and-more/](https://johnresig.com/blog/ecmascript-5-strict-mode-json-and-more/)
6. [https://medium.com/@hectorlorenzo/what-is-strict-mode-in-javascript-aa633039bf9f](https://medium.com/@hectorlorenzo/what-is-strict-mode-in-javascript-aa633039bf9f)
准确的说JS类是没有私有属性的，可能有人会说JS连类都不存在。。。相对于写Java/C++等强类型面向对象的语言的人来说确实是这样的。但JS有JS的规则，自从ES6支持class，extends关键字后，JS面向对象的功能变得简单而强大，这次看看如何在JS类中实现(伪实现)私有属性。

## 四种方法

应该说不止四种，这里只列举曾经出现的比较常用的四种方法

### 命名下划线

```jsx
// ES5Naming.js
class Point {

    constructor(x=0, y=0) {
        this._x = x;
        this._y = y;
    }

}

export default Point;
```

### 通过Symbol

```jsx
// ES6Symbol.js
const _x = Symbol('x');
const _y = Symbol('y');

class Point{

    constructor(x=0,y=0){
        this[_x] = x;
        this[_y] = y;
    }

}

export default Point;
```

### 通过WeakMap

```jsx
// ES6WeakMap
const _x = new WeakMap();
const _y = new WeakMap();

class Point {

    constructor(x=0, y=0) {
        _x.set(this, x);
        _y.set(this, y);
    }

}

export default Point;
```

### 通过#声明属性

```jsx
// ESFuture.js
class Point{

    #x;
    #y;

    constructor(x=0,y=0){
        this.#x = x;
        this.#y = y;
    }

}

export default Point;
```

## 各方法比较

第四种还是一种提案，目前处于stage2阶段，但Node.js 12以后与最新的谷歌浏览器已经支持，有兴趣的同学可以自己试试；

第三种是可以真正实现私有属性，但每个属性需要定义个WeakMap，且Babel目前对第四种实现的转译也是这种形式；

第二种用得少；

第一种方便，我一般用这种方法；

> Stage 0 - Strawman（展示阶段）
Stage 1 - Proposal（征求意见阶段）
Stage 2 - Draft（草案阶段）
Stage 3 - Candidate（候选人阶段）
Stage 4 - Finished（定案阶段）

## 测试结果

```jsx
// app.js
import {default as ES5NamingPoint} from './modules/private/ES5Naming';
import {default as ES6Symbol} from './modules/private/ES6Symbol';
import {default as ES6WeakMap} from './modules/private/ES6WeakMap';
import {default as ESFuture} from './modules/private/ESFuture';

// 命名下划线
let es5NamePt = new ES5NamingPoint(1,2);
console.log(es5NamePt); // Point { _x: 1, _y: 2 }
es5NamePt._x = 3; // 修改成功
console.log(es5NamePt); // Point { _x: 3, _y: 2 }

// Symbol - 每个属性定义个Symbol，但还是可以被访问和修改
let es6SymbolPt = new ES6Symbol(1,2);
console.log(es6SymbolPt); // Point { [Symbol(x)]: 1, [Symbol(y)]: 2 }
es6SymbolPt._x = 3; // 修改失败
console.log(es6SymbolPt); // Point { _x: 3, [Symbol(x)]: 1, [Symbol(y)]: 2 }

let symbols = Object.getOwnPropertySymbols(es6SymbolPt);
es6SymbolPt[symbols[0]] = 3; // 修改成功
console.log(es6SymbolPt); // Point { _x: 3, [Symbol(x)]: 3, [Symbol(y)]: 2 }

// WeakMap - 无法查看和修改Point的私有属性，每个属性得定义个WeakMap
let es6WeakMapPt = new ES6WeakMap(1,2);
console.log(es6WeakMapPt); // Point {}
es6WeakMapPt._x = 3; // 修改失败
console.log(es6WeakMapPt); // Point { _x: 3 } 增加了属性

// ES Future - 原生支持私有属性
let esFuturePt = new ESFuture(1,2);
console.log(esFuturePt); //Point {}
console.log(esFuturePt.#x); // SyntaxError: Undefined private field #x: must be declared in an enclosing class
```

最后附上babel转译第四种后的代码：

```jsx
"use strict";

function _instanceof(left, right) {
    if (right != null && typeof Symbol !== "undefined" && right[Symbol.hasInstance]) {
        return !!right[Symbol.hasInstance](left);
    } else {
        return left instanceof right;
    }
}

function _classCallCheck(instance, Constructor) {
    if (!_instanceof(instance, Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

function _classPrivateFieldSet(receiver, privateMap, value) {
    var descriptor = privateMap.get(receiver);
    if (!descriptor) {
        throw new TypeError("attempted to set private field on non-instance");
    }
    if (descriptor.set) {
        descriptor.set.call(receiver, value);
    } else {
        if (!descriptor.writable) {
            throw new TypeError("attempted to set read only private field");
        }
        descriptor.value = value;
    }
    return value;
}

var Point = function Point() {
    var x = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : 0;
    var y = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 0;

    _classCallCheck(this, Point);

    _x.set(this, {
        writable: true,
        value: void 0
    });

    _y.set(this, {
        writable: true,
        value: void 0
    });

    _classPrivateFieldSet(this, _x, x);

    _classPrivateFieldSet(this, _y, y);
};

var _x = new WeakMap();

var _y = new WeakMap();
```
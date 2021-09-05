[[_TOC_]]

都说单例模式是最简单的的设计模式，最适合入门，这里我只是抛砖引玉的简单介绍一下。

## 什么是单例模式

个人理解：某个对象，或者某个类的实例在内存中只允许有一个。我们可以通过某种方法多次获得这个实例，或者说多次初始化一个类的实例，但是单例模式会返回给你相同的实例，而不是每次一个新实例。

专业解释：**单例模式**，也叫**单子模式**，是一种常用的[软件设计模式](https://zh.wikipedia.org/wiki/软件设计模式)，属于创建型模式的一种。在应用这个模式时，单例对象的[类](https://zh.wikipedia.org/wiki/类_(计算机科学))必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局[对象](https://zh.wikipedia.org/wiki/对象)，这样有利于我们协调系统整体的行为。

更详细的介绍大家可以查看：https://zh.wikipedia.org/wiki/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F

那么单例模式究竟是如何实现只返回给你一个实例的？下面我们从ts类的实现角度来看看思路。



## 单例模式的实现思路

这里我们只看OOP的实现思路，JS中各种另类实现思路先不说。

### 一、避免直接调用new操作符

ts类中构造函数可以设置为私有，这样外面就无法直接调用new操作符来实例化了，如下：

```typescript
class Singleton {
  
  private constructor() {}
  
}

new Singleton() 
// 上面的new操作ts会报一下错误：
/**********************************
Constructor of class 'Singleton' is private and only accessible within the class declaration.
**********************************/
```

### 二、公开静态方法提供实例

上面我们屏蔽了外部调用new来实例化，那如何从外部得到实例？静态方法。

```typescript
class Singleton {
	
  static getInstance():Singleton {
    return new Single();
  }
  
	private constructor() {}
}
```

现在可以通过在外部调用Singleton.getInstance()来获取一个实例了，那如何保证每次都返回的是同一个实例？增加一个私有的静态属性。

```typescript
class Singleton {

  private _instance:Singleton;
  
  static getInstance():Singleton {
    if (!Singleton._instance) {
      Singleton._instance = new Singleton();
    }
    return Singleton._instance;
  }
  
	private constructor() {}
}
```

## 单例模式的3种实现

终于到实现部分了，下面看看常用的三种实现，它们各种都有自己的适用地方，这里不做好坏之分，只关注实现。关于三种实现的使用场景，请继续往后看。

### 实现一

这种可能算是最简单的实现，EventBus用过吗？vue项目中你可能已经用过。

```javascript
// 先建一个Singleton.js
import Vue from 'vue';

export default new Vue();

// 然后在其他文件中引用它，如a.js文件中，可以这样
import singleton from 'filepath/to/Singleton';
```

这种实现利用了ES6的模块加载机制来保证每次都只能获取同一个实例，从上面的实现思路看，它并没有提供new操作符，或者说没有禁用new操作符。关于ES6模块加载机制的详细资料可以查看：https://exploringjs.com/es6/ch_modules.html#sec_design-goals-es6-modules

有人会说，这种方法实现的EventBus依赖了vue。是的，继续看，后面会告诉你如何自己实现一个不依赖vue的EventBus。

### 实现二

这种实现，在n年前应该应用比较广，不过现在并没有消失，在一些第三方类库中经常看到。它主要应用在对已经定义好的类，或者对象进行处理。比如你已经有一个写好的类，想把它变为单例，那么这种方法适用你。

```typescript
interface SourceConstructor<T> {
    new (...args: Object[]): T;
}

export function Single<T>(ctor:SourceConstructor<T>): SourceConstructor<T> {
    let instance: T|null = null;
    let t: any = function(){
        if (!instance)
            instance = new ctor();
        return instance;
    };
    t.prototype = ctor.prototype;
    t.prototype.constructor = ctor;
    return t;
}
```

这种实现利用了闭包的特性，从实现思路来看，它将静态方法变成了闭包实现，即修改了原始的构造函数，通过闭包返回一个修改后的构造函数，通过新的构造函数来保证每次只能获取同一个实例。

上面的实现有个遗憾的地方就是返回的构造函数用了any类型，期待哪位同学改造一下，最好能用准确的类型。

对ts的泛型不了解？可以看看下面的链接：https://www.tslang.cn/docs/handbook/generics.html

### 实现三

这应该是ts正规的实现了，具体请参看上面的实现思路。

```typescript
class Singleton {

  private _instance:Singleton;
  
  static getInstance():Singleton {
    if (!Singleton._instance) {
      Singleton._instance = new Singleton();
    }
    return Singleton._instance;
  }
  
	private constructor() {}
}
```



## 应用示例

### 直接应用（实现二）

文件Single.ts - 主要实现

```typescript
interface SourceConstructor<T> {
    new (...args: Object[]): T;
}

export function Single<T>(ctor:SourceConstructor<T>): SourceConstructor<T> {
    let instance: T|null = null;
    let t: any = function(){
        if (!instance)
            instance = new ctor();
        return instance;
    };
    t.prototype = ctor.prototype;
    t.prototype.constructor = ctor;
    return t;
}
```

文件TestSingle.ts - 定义我们要使之成为单例的类

```typescript
import { Single } from "./Single";

class _TestSingle {
    name:string = 'testSingle';
}

export const TestSingle = Single(_TestSingle);
```

这里定义了_TestSingle类，然后用Single函数将将其包装后导出正在的TestSingle类，在外部调用：

文件client.ts

```typescript
import { TestSingle } from "./TestSingle";
// import { DecoratorSingle } from "./DecoratorSingle";

let testSingle1 = new TestSingle();
let testSingle2 = new TestSingle();
console.log(testSingle1.name); // testSingle
testSingle1.name = 'xxxx';
console.log(testSingle2.name); // xxxx
```



### Class Decorator（实现二）

这个需要提前开启tsc配置项experimentalDecorators，emitDecoratorMetadata

文件DecoratorSingle.ts

```typescript
import { Single } from "./Single";

@Single
class DecoratorSingle {
    name:string = 'DecoratorSingle';
}

export { DecoratorSingle };
```

测试文件：

```typescript
import { DecoratorSingle } from "./DecoratorSingle";

let decoratorSingle1 = new DecoratorSingle();
let decoratorSingle2 = new DecoratorSingle();
console.log(decoratorSingle1.name); // DecoratorSingle
decoratorSingle1.name = 'xxxx';
console.log(decoratorSingle2.name); // xxxx
```



### EventBus（实现三）

下面的实现只是经过了简单的单元测试，仅供学习使用，用于生产环境需谨慎！！！

我们知道EventBus提供了发送和接收事件的功能，且是单例。单例上面我们已经实现了，下面先看看如何实现发送和接收事件，然后再看看怎么将两者结合起来。

#### 定义事件的回调函数类型: IEventHandler.ts

```typescript
export type IEventHandler = (...rest: Object[]) => void;
```

#### 定义通用发送接收事件接口：IEventEmitter.ts

```typescript
import { IEventHandler } from "./IEventHandler";

export interface IEventEmitter {
    emit(evt:string, params?:Object):void
    on(etype:string, evtHandler:IEventHandler, params?:Object):number
    off(evtId:number):boolean
    clear():void
}
```

#### 发送接收事件的具体实现，需要实现IEventEmitter接口: EventEmitter.ts

```typescript
import { IEventEmitter } from "./IEventEmitter";
import { IEventHandler } from "./IEventHandler";

// 定义数组中元素类型
type EventItem = { eType: string, evtId: number, evtHandler: IEventHandler, rest: Object[] };
// 定义数组类型别名
type EventHandlers = Array<EventItem>;

export class EventEmitter implements IEventEmitter {

  	// 用map存放事件与回调函数的关系，也可以有别的选择，之前用{}.
    protected _evtMap = new Map<string, EventHandlers>();
  	// 每次添加事件与回调后的唯一id，方便销毁
    private static _evtId:number = 10;

  	// 发送事件，事件参数可以是不限，跟vue的$emit不同只能有一个参数。
    emit(eType:string, ...rest:Object[]):void {
        if (!eType) throw new Error(`xxxxxx`);
        let evtHandlers = this._evtMap.get(eType);
        if (!evtHandlers) return;
        evtHandlers.forEach(i => {
            const args = [...i.rest, ...rest]; // on方法添加的参数在前，emit方法的参数随后
            i.evtHandler.apply(null, args);
        });
    }

  	// 添加监听，添加监听时也带了参数
    public on(eType:string, evtHandler:IEventHandler, ...rest:Object[]): number {
        if (!eType||!evtHandler) throw new Error(`xxxxxx`);
        let evtHandlers = this._evtMap.get(eType);
        if (!evtHandlers) {
            evtHandlers = [];
            this._evtMap.set(eType, evtHandlers);
        }
        let evtId = ++EventEmitter._evtId;
        evtHandlers.push({ eType, evtId, evtHandler, rest });
        return evtId;
    }
		
  	// 删除监听
    off(evtId:number):boolean {
        let hasDel:boolean = false;
        for(let [,value] of this._evtMap) {
            let index = value.findIndex(i => i.evtId === evtId);
            if (index >= 0) {
                value.splice(index, 1);
                hasDel = true;
                break;
            }
        }
        return hasDel;
    }

  	// 清掉所有监听
    clear():void {
        this._evtMap = new Map<string, EventHandlers>();
    }

}
```

#### 将EventEmitter与Singleton结合起来：EventBus.ts

```typescript
import { EventEmitter } from "./EventEmitter";

export class EventBus extends EventEmitter {
    private static _instance: EventBus;

    private constructor() {super();}

    static getInstance(): EventBus {
        if (!EventBus._instance) {
            EventBus._instance = new EventBus();
        }
        return EventBus._instance;
    }
}
```

最后附上EventBus的单元测试用例

```typescript
import { EventBus } from "../src/event/EventBus";
import assert from 'assert';

describe('EventBus: ', ()=>{

    let evtBuses:EventBus[] = [];

    afterEach(()=>{
        evtBuses.forEach(i => {
            i.clear();
        });
        evtBuses = [];
    });

    it('ctor: 不同的实例是否相等?', () => {
        let evtBus1 = EventBus.getInstance();
        let evtBus2 = EventBus.getInstance();
        assert.strictEqual(evtBus1, evtBus2, '实例不相等');
        evtBuses.push(evtBus1, evtBus2);
    });

    it('on: 添加事件监听是否正常?', () => {
        let evtBus = EventBus.getInstance();
        let evtId = evtBus.on('xxx', (...rest) => {
            console.log(rest);
        });
        assert.ok(evtId, 'evtId不存在');
        evtBuses.push(evtBus);
    });

    it('emit: 能发送事件并触发回调?', (done) => {
        let evtBus = EventBus.getInstance();
        evtBus.on('xxx', a => {
            assert.strictEqual(a, 1, 'a != 1');
            done();
        });
        evtBus.emit('xxx', 1);
        evtBuses.push(evtBus);
    })

    it('off: 是否能正确删除事件监听?', () => {
        let result = {a:1};
        let evtBus = EventBus.getInstance();
        let evtId = evtBus.on('xxx', () => {
            result.a++;
        });
        evtBus.off(evtId);
        evtBus.emit('xxx');
        assert.strictEqual(result.a, 1, 'off失败')
        evtBuses.push(evtBus);
    });

    it('clear: 清除所有事件监听?', () => {
        let result = {a:1};
        let evtBus = EventBus.getInstance();
        evtBus.on('xxx', () => {
            result.a++;
        });
        evtBus.on('yyy', () => {
            result.a++;
        });
        evtBus.clear();
        evtBus.emit('xxx');
        evtBus.emit('yyy');
        assert.strictEqual(result.a, 1, 'clear失败');
        evtBuses.push(evtBus);
    });
});
```

## 代码

代码是在浏览器环境测试过的，如果你想直接跑起来，可能需要全局安装parcel与typescript。

```shell
npm install parcel-bundler typescript -g
```

然后执行下面的命令：

```shell
npm install && npm run dev
```

如果你想在node环境测试代码，直接执行：

```typescript
npm run run-in-node
```

单元测试可以执行以下命令：

```shell
npm run test
```


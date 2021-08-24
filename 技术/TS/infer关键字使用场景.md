从数组类型获取数组里面元素的类型

```tsx
type ElementOfArray<T> = T extends (infer U)[] ? U : T;

const eleBoolean:ElementOfArray<Array<boolean>> = true;
const eleString: ElementOfArray<string[]> = 'a';
```

从函数类型中获取函数的返回值类型

```tsx
type ReturnOfFun<T> = T extends (...rest: never[]) => infer U ? U : never;

const s1:ReturnOfFun<()=>string> = '1';
const s2:ReturnOfFun<()=>boolean[]> = [true];
const s3:ReturnOfFun<string> = 2;

// s3的T类型不是函数，所以会报错：
// Type 'number' is not assignable to type 'never'.(2322)
```

从Promise中获取最终返回的数据类型

```tsx
type TypeOfPromise<T> = T extends Promise<infer U> ? U : T;

const s1:TypeOfPromise<Promise<string>> = '1';
const s2:TypeOfPromise<Promise<Date[]>> = [new Date()];
const s3:TypeOfPromise<boolean> = true;

// s3的T类型不是Promise<T>类型，所以根据条件类型的结果获取的是传入的类型。
```

从函数类型中获取其参数的类型

```tsx
// 定义一个函数类型，函数有参数和返回值
type FunWithArgs<T extends {[k:string]: any}, R> = (props: T) => R;
// 定义一个从函数获取其参数类型的类型
type FunArgs<T extends FunWithArgs<any, any>> = T extends FunWithArgs<infer P, any> ? P : never;

// 使用示例
// 声明一个带参数的函数
const myFun:FunWithArgs<{x:boolean,y:number}, boolean[]> = props => {
    return [true];
}

// 从什么的函数中取得其参数的类型
const props: FunArgs<typeof myFun> = {x:true, y:1};
```

从Map/Object类型的键名获取其对应值得类型

```tsx
type User = {
  id: number,
  name: string
}
type Doc = { id: string }

type GetProperty<T, P extends keyof T> = T extends {[K in P]: infer V} ? V : never;

let s: GetProperty<User, 'id'> = 2;
let ss: GetProperty<Doc, 'id'> = '2';
```

一种简单处理方法：

```tsx
type GetProperty<T, P extends keyof T> = T[P];
```

从一种函数类型获取追加参数后的函数类型

```tsx
// 利用ts的类型工具Parameters，ReturnType
type AppendArguments<F extends (...args:any[]) => any, A> = (x: A, ...args: Parameters<F>) => ReturnType<F>;

// infer 的应用
type AppendArguments2<F, A> = F extends (...args: infer R) => infer T ? (x: A, ...args: R) => T : never;

function test(a:number, b:string): boolean {
  return a - +b > 0;
}

// 测试1
let appendFun:AppendArguments2<typeof test, string> = function(appended:string, a:number, b:string): boolean {
  return true;
}
// 测试2
let appendFun:AppendArguments1<typeof test, string> = function(appended:string, a:number, b:string): boolean {
  return true;
}
```
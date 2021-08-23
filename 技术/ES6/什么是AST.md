AST 就是 Abstract Syntax Tree的缩写，中文翻译叫：抽象语法树。它一般用来表示或者描述某种编程语言所写代码，比如如何标记表达式，变量，函数等。解释器或者编译器通过AST可以生成自己的一套代码。

下面的表达式 1+2，通过AST描述：

```jsx
1 + 2
```

```yaml
+ BinaryExpression
 - type: +
 - left_value: 
  LiteralExpr:
   value: 1
 - right_vaue:
  LiteralExpr:
   value: 2
```

如果是if语句，又是如何描述？

```jsx
if(2 > 6) {
    var d  = 90
    console.log(d)
}
```

```jsx
IfStatement
 - condition
  + BinaryExpression
   - type: >
   - left_value: 2
   - right_value: 6
 - body
  [
    - Assign
        - left: 'd';
        - right: 
            LiteralExpr:
            - value: 90
    - MethodCall:
         - instanceName: console
         - methodName: log
         - args: [
         ]
  ]
```

AST可以告诉编译器或者解释器，如何来解释或执行我们的代码，并且还可以利用AST来生成自己的经过优化的代码。

对于表达式 1+2，我们的大脑首先识别并记录的是这是一个加法运算，然后识别有左边的数字和右边的数字，最后将他们相加。编译器/解释器的工作过程跟我们的大脑相似。

我们定义一个二元运算的类来处理这种二元运算符相关的语句，如下：

```jsx
class Binary {
    constructor(left, operator, right) {
        this.left = left
        this.operator = operator
        this.right = right
    }
}
```

定义二元运算的Binary类后，上面的表达式1+2就可以表示为下面这样：

```jsx
New Binary(1, 'ADD', 2)
```

然后编译器在处理我们的代码时，就可能有下面的这个过程：

```jsx
const binExpr = new Binary(1, 'ADD', 2)
if(binExpr.operator == 'ADD') {
    return binExpr.left + binExpr.right
}
// 3 is returned
```

像数字，字符串，布尔类型等这些简单类型在AST中会表示为字面量，即直接使用他们所表示的值。通过一个Literal类来处理，如下：

```jsx
class Literal {
    constructor(value) {
        this.value = value
    }
}
```

比如，数字 1， 我们可以表示为：

```jsx
new Literal(1)
```

然后在解释器中执行时，就会出现这样的：

```jsx
const oneLit = new Literal(1);
console.log(oneLit.value);
```

之前的表达式1+2，就会变成这样：

```jsx
const oneLit = new Literal('1')
const twoLit = new Literal('2')
const binExpr = new Binary(oneLit, 'ADD', twoLit);
if (binExpr.operator === 'ADD') {
	return binExpr.left.value + binExpr.right.value;
}
```

if语句又是如何执行的呢？if语句都会包含一个条件和一个块(函数体)，所以定义if语句对应的类，应该有条件和块两部分

```jsx
class IfStmt {
	constructor(condition, body) {
		this.condition = condition;
		this.body = body;
	}
}
```

```jsx
if(9 > 7) {
    log('Yay!!')
}
```

现在看看上面的if语句如何执行

首先，9>7 是一个二元布尔表达式，所以根据上面Binary类，我们可以将其写成：

```jsx
const cond = new Binary(new Literal(9), 'GREATER', new Literal(7));
```

接着是调用一个log函数调用，函数调用也有对应的类来处理，调用函数我们需要知道函数名称，和参数，所以这个类至少包含名称和参数两个属性

```jsx
class FuncCall {
	constructor(name, args) {
		this.name = name;
		this.args = args;
	}
}
```

所以上面的log函数调用，我们可以写成下面这样的：

```jsx
const LogFuncCall = new FuncCall('log', []);
```

所以上面的if语句最终转换会是这样的

```jsx
const cond = new Binary(new Literal(9), "GREATER", new Literal(7));
const logFuncCall = new FuncCall('log', []);
// 函数体用数组，因为if条件后面可能需要执行多个语句
const ifStmt = new IfStmt(cond, [
    logFuncCall
])
```

解释器执行过程：

```jsx
const ifStmt = new IfStmt(cond, [ logFuncCall ]);

interpretIfStatment(ifStmt);

function inerpretIfStatment(ifStmt) {
	if (evalExpr(ifStmt.condition)) {
		for (const stmt of ifStmt.body) {
			evalStmt(stmt);
		}
	}
}
```

## AST的执行

从上文我们知道，各种语句，表达式，字面量等都对应有一个类来实现，比如IfStmt，Binary，Literal等，在这些类中分别定义了解释器要执行时所需的属性和条件等，比如Binary类中有operator，left，right。但是最终要执行这些语句，表达式等，得分别定义不同的执行方法，因为它们有各自不同的执行方法。现在的问题是我们将这些执行的方法也定义在这些类里吗？或许可以，大家可以去尝试一下。不过我们这里要介绍的是通过visitor模式来处理。

首先我们定义一个Visitor类，它会实现各个类中具体的操作函数，比如visitLiteral，visitBinary等，这些函数需要一个参数，即具体的语句，表达式类的实例

然后在每个类中定义一个visit方法，传入visitor实例，在visit方法中根据不同的语句表达式来执行visitor中定义的具体方法，这个具体方法需要接受类的实例，因为在执行的时候需要从类的实例中访问不同的属性和条件。

下面看看Visitor类

```jsx
class Visitor {
	visitBinary(binExpr) {
		switch(binExpr.operator){
			case 'ADD':
				return binExpr.left.visit(this) + binExpr.right.visit(this);
				break;
			case 'SUBTRACT':
				return binExpr.left.visit(this) - binExpr.right.visit(this);
				break;
			case 'MULTIPLY':
				break;
		}
	}
	visitLiteral(litExpr) {
		return litExpr.value;
	}
}
```

然后再在具体的语句类中增加visit方法

```jsx
class Literal {
	constructor(value){
		this.value = value;
	}
	visit(visitor) {
		return visitor.visitLiteral(this);
	}
}

class Binary {
	constructor(left, operator, right) {
		this.left = left;
		this.operator = operator;
		this.right = right;
	}
	visit(visitor) {
		return visitor.visitBinary(this);
	}
}
```

现在我们再来看看表达式 1+2 

```jsx
const oneLit = new Literal(1);
const twoLit = new Literal(2);
const binExpr = new Binary(oneLit, 'ADD', twoLit);
const visitor = new Visitor();

binExpr.visit(visitor); // 3
```

上面看上去一切都很自然，很简单。如果表达式变成 1+2+3，又该如何？

```jsx
const oneLit = new Literal(1);
const twoLit = new Literal(2);
const threeLit = new Literal(3);
const visitor = new Visitor();

const binExpr1 = new Binary(oneLit, 'ADD', twoLit);
const binExpr2 = new Binary(binExpr1, 'ADD', threeLit);

console.log(binExpr2.visit(visitor));
```

## 实现IF语句的执行

先给IfStmt类增加visit方法，如下：

```jsx
class IfStmt {
    constructor(condition, body) {
        this.condition = condition
        this.body = body
    }
    visit(visitor) {
        return visitor.visitIfStmt(this)
    }
}
```

然后再在Visitor类增加具体的实现：visitIfStmt方法

```jsx
class Visitor {
	
	...
	
	visitIfStmt(ifStmt){
		if(ifStmt.condition.visit(this)){
			for(const stmt of ifStmt.body){
				stmt.visit(this);
			}
		}
	}
}
```

## 实现函数调用与函数声明

JS中的函数声明一般是这样的

```jsx
function function_name(params) {}
```

对应函数声明我们也需要定义个专门的类来处理，如

```jsx
class FunctionDeclaration {
	constructor(name, body) {
		this.name = name;
		this.body = body;
	}

	visit(visitor) {
		return visitor.visitFunctionDeclaration(this);
	}
}
```

然后再在Visitor类中增加对应的实现方法

```jsx
class Visitor {
	
	...
	
	visitFunctionDeclaration(functionDeclaration){
		
	}
}
```

函数声明只是声明一个函数，而不是执行函数，这里我们在visitor类中需要将自定义函数(跟内建函数区别)的名称和函数体预先存起来，当在解释器执行时再调取出来执行，所以我们得实现一个存储函数声明的工具，如下：

```jsx
class FunStore {
	constructor() {
		this.map = new Map();
	}

	setFunc(name, body) {
		this.map.set(name, body);
	}

	getFunc(name) {
		return this.map.get(name);
	}
}
```

有了上面的存储函数声明的类，我们就可以实现Visitor中对函数声明的实现了，其实只需要调用FunStore类将函数声明存储起来，因为我们并不是执行这个函数，而是声明它

```jsx
class Visitor {
	
	...
	
	visitFunctionDeclaration(functionDeclaration){
		const {name, body} = functionDeclaration;
		funStore.setFunc(name, body);
	}
}
```

接下来我们来看看函数调用，又改如何实现？先给前面定义过的FuncCall类增加visit方法，如下：

```jsx
class FuncCall {
    constructor(name, args) {
        this.name = name
        this.args = args
    }
    visit(visitor) {
        return visitor.visitFuncCall(this)
    }
}
```

然后再在visitor类中增加对应的实现：visitFuncCall

```jsx
class Visitor {
	
	...
	
	visitFuncCall(funcCall){
		const funName = funcCall.name;
		const args = [];
		// 参数就是字面量或者表达式，所以此次需要调用他们对应的visit方法，先计算
		for(const exp of funcCall.args){
			args.push(exp.visit(this));
		}
		// 执行函数
		// 1. 先去FunStore中看是不是我们自己声明的函数
		// 2. 不是就执行3，是就取出函数体执行。
		// 3. 执行内建函数
		const funBody = funcStore.getFunc(funName) || funcCall.body;
		funBody.forEach(stmt => {
			stmt.visit(this);
		});
	}
}
```

下面我们看一个函数调用和函数声明的例子

```jsx
function addNumbers(a, b) {
	console.log(a + b);
}

function logNumbers() {
	console.log(5);
	console.log(6);
}
```

它会如何转换？

```jsx
const funcDec1 = new FuncDeclaration('logNumbers', [
	new FuncCall('console.log', [new Literal(5)]),
	new FuncCall('console.log', [new Literal(6)])
]);
const a = 1;
const b = 2;
const funcDec2 = new FuncDeclaration('addNumbers', [
	new FuncCall('console.log', [new Binary(new Literal(a), 'ADD', new Literal(b))])
]);
```

## 总结

上面我们实现了二元运算表达式，IF语句，函数声明和函数调用，以及他们对应的Visitor实现。但AST不仅仅只有这些定义和实现，还有下面的很多很多：

- Classes和Objects
- Method calls
- 封装和继承
- for...of 语句
- while语句
- for...in 语句
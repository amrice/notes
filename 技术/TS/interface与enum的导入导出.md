## interface的导入导出

当我们要通过默认方式导入接口时，可以按下面这样定义：

```tsx
// ITest.ts 接口定义
interface ITest {
		add(...content: string[]): void;
		getContent(): string;
}

export default ITest;

// 导入时可以这样：
import ITest from 'ITest';
```

当我们要通过非默认方式导入接口时，可以按下面这样定义：

```tsx
// ITest.ts 接口定义
export interface ITest {
		add(...content: string[]): void;
		getContent(): string;
}

// 导入时可以这样：
import { ITest } from 'ITest';
```

## enum的导入导出

当我们要通过默认方式导入enum类型时，可以按下面这样的定义：

```tsx
// 定义 UserType.ts
const enum UserType {
		Admin,
		Manager,
		Guest
}

export default UserType;

// 导入
import UserType from 'UserType';
```

当我们要通过非默认方式导入enum类型时，可以按下面这样定义：

```tsx
// 定义 UserType.ts
export const enum UserType {
		Admin,
		Manager,
		Guest
}

// 导入时可以这样：
import { UserType } from 'UserType';
```
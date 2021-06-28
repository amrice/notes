我们知道只需要在vite.config.js中设置不同的base，即可以针对不同的部署目录打出不同包。但是这个base是写死在vite的配置中的，也就是说没次要根据不同的环境去手动修改。有什么办法能动态配置这个base的值呢？

## 第一种尝试

一开始想到的是添加`.env.production_blog`与`.env.production_local`这样的环境变量，如下所示

```shell
# .env.production_blog
VITE_DEPLOY_DIR = /blog/
```

```shell
# .env.production_local
VITE_DEPLOY_DIR = /
```

然后再在vite.config.js中访问环境变量

```javascript
const blog_static_path = 'games/perfect-square/';
const deploy_dir = import.meta.env.VITE_DEPLOY_DIR;
// 根据环境变量组装base的值
const base = `${deploy_dir}${blog_static_path}`;

export default {
    base,
    build: {
        assetsInlineLimit: 0
    }
}
```

不过当执行`tsc && vite build --mode production_local`时报错了

```shel
const deploy_dir = import.meta.env.VITE_DEPLOY_DIR;
                          ^^^^

SyntaxError: Cannot use 'import.meta' outside a module
    at Object.compileFunction (node:vm:352:18)
    at wrapSafe (node:internal/modules/cjs/loader:1025:15)
    at Module._compile (node:internal/modules/cjs/loader:1059:27)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1124:10)
    at Module.load (node:internal/modules/cjs/loader:975:32)
    at Function.Module._load (node:internal/modules/cjs/loader:816:12)
    at Module.require (node:internal/modules/cjs/loader:999:19)
    at require (node:internal/modules/cjs/helpers:93:18)
    at loadConfigFromFile 
```

意思是说无法在业务模块外访问环境变量，查一下资料发现尤大有对这个问题做过说明

![image-20210628225038268](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210628225110.png)

简单说这就是一个鸡蛋和鸡的问题，反正就是不能这么干了。

## 第二种尝试

没办法再看看vite的文档，还有github项目，vite.config.js可以返回一个函数来处理这种动态的配置，而函数的参数正好有一个是mode。

```javascript
export default ({ command, mode }) => {
  if (command === 'serve') {
    return {
      // serve 独有配置
    }
  } else {
    return {
      // build 独有配置
    }
  }
}
```

所以我们可以直接删掉`.env.**`这些环境配置文件，然后修改`vite.config.js`

```javascript
// vite.config.js
// 打包要部署到博客时，加/blog/
// 打包用于本地开发时去掉/blog
const blog_static_path = 'games/perfect-square/';

export default ({ command, mode }) => {
    console.log(command, mode, '====');
    let base = '/';
    let static_path = '/';
    if (command === 'build') {
        if (mode === 'production_blog') {
            base = '/blog/';
            static_path = blog_static_path;
        } else {
            static_path = '';
        }
        base = `${base}${static_path}`;
    } else {
        base = '';
    }
    return {
        base,
        build: {
            assetsInlineLimit: 0
        }
    };
}
```

这样我们执行不同的命令就会根据不同的mode来配置不同的base了

```json
"scripts": {
    "dev": "vite",
    "//1": "build:local用于打包到博客本地调试",
    "build:local": "tsc && vite build --mode production_local",
    "//2": "build:blog用于打包要部署到博客的游戏代码",
    "build:blog": "tsc && vite build --mode production_blog",
    "serve": "vite preview"
}
```

但上面的做法实际上是与环境变量无关了，偏题了。如果一定要在配置文件中访问环境变量，那该怎么做？github上在尤大的回复后面有人给出了答案

## 第三种尝试

要访问环境变量，肯定得创建env文件，所以我们先跟第一种尝试一样创建两个env文件，

```shell
# .env.production_blog
VITE_DEPLOY_DIR = /blog/

# .env.production_local
VITE_DEPLOY_DIR = /
```

然后还是要用第二中尝试中用到的scripts命令来指定不同的mode

```json
"build:local": "tsc && vite build --mode production_local",
"build:blog": "tsc && vite build --mode production_blog",
```

不同mode值对应不同的env文件，编译的时候调用vite配置文件就会传入对应的mode值，然后我们修改vite的配置文件，通过加载env文件读取环境变量，然后组装我们的vite配置

```javascript
// vite.config.js

import { loadEnv } from "vite";

const blog_static_path = 'games/perfect-square/';

export default ({ command, mode }) => {
    const envObj = loadEnv(mode, process.cwd());
    console.log(envObj);

    let base = '/';
    if (command === 'build') {
        if (mode === 'production_blog') {
            base = `${envObj.VITE_DEPLOY_DIR}${blog_static_path}`
        }
    } else {
        base = '';
    }
    return {
        base,
        build: {
            assetsInlineLimit: 0
        }
    };
};
```

关键的是这一句`loadEnv(mode, process.cwd())`，loadEnv是从vite包中导出的，像是同步函数，返回的数据时一个对象，包含了对应env文件中定义的所有环境变量。

## 总结

第三种方法告诉我们一种手动加载env文件的方法，适用于你的环境变量比较多的时候。第二种方法适用于简单情形，因为所有判断逻辑都写在了vite的配置文件中。


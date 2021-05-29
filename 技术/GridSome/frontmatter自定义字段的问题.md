# frontmatter自定义字段的问题

## 问题

首先，运行环境是这样的：

```json
"@gridsome/source-filesystem": "^0.6.2"
"@gridsome/transformer-remark": "^0.6.4"
"gridsome": "^0.7.23"
```

在gridsome.config.js中也增加了插件的配置信息

```javascript
plugins: [
    {
      // Create posts from markdown files
      use: '@gridsome/source-filesystem',
      options: {
        typeName: 'Post',
        path: 'content/posts/**/*.md',
        quote: 'quote', // 名人名句
        enTitle: 'enTitle', // 显示在路径上的名称
        refs: {
          // Creates a GraphQL collection from 'tags' in front-matter and adds a reference.
          tags: {
            typeName: 'Tag',
            create: true
          }
        }
      }
    }
  ],
```

这里增加了自己定义的enTitle，quote两个字段，然后希望在Post中能读取这两个字段

```scheme
<page-query>
{
  allPost {
    edges {
      node {
        title
        excerpt
        date
        path
        quote
        published
        tags {
          id
          title
          path
        }
      }
    }
  }
}
</page-query>
```

然后居然编译失败，报了如下错误：

```she
Module build failed (from ./node_modules/gridsome/lib/plugins/vue-components/lib/loaders/page-query.js):
Error: Cannot query field "quote" on type "Post".

GraphQL request:17:9
16 |         path
17 |         quote
   |         ^
18 |         published
    at Object.module.exports (M:\**\**\gridsome_blog\node_modules\gridsome\lib\plugins\vue-components\lib\loaders\page-query.js:33:23)
```

大概意思是找不到quote字段。后来不读取quote字段就没问题了，但enTitle又报了……

## 处理

谷歌搜了一下，发现很多是cover_image这个字段报错，其实道理是一样的，根本原因是GraphQL的schema没有定义对于的字段。所以我们需要自己定义这个对于的schema。

在gridsome.server.js中增加对于的schema

```javascript
api.loadSource(({ addSchemaTypes }) => {
    addSchemaTypes(`
      type Post implements Node {
        id: ID!
        title: String
        quote: String
        cover_image: String
        published: Boolean
        enTitle: String
        date: String
        description: String
        tags: [Tag!]
      }
    `);
    addSchemaTypes(`
      type Tag implements Node {
        id: ID!
        title: String
      }
    `);
  });
```

## 中文路由

顺便说一下中文路由的问题，我们可以修改配置中的template

```javascript
templates: {
    Post: [
      {
        path: (node) => {
          return `/${node.date.split(' ')[0]}/${node.enTitle}`;
          // return `/${node.date.split(' ')[0]}/${node.fileInfo.name}`;
        }
      },
    ],
    // Post: '/:fileInfo__name',
    Tag: '/tag/:id',
}
```

上面的path函数本来可以用

```javascript
return `/${node.date.split(' ')[0]}/${node.fileInfo.name}`;
```

这样确实可以显示中文，但是当你打开Post后，刷新会出现404，gridsome内部对中文的处理还是存在问题，上面的做法只是修改了路由的显示而已，并未解决本质的问题。所以暂时我还是放弃了在路由上显示中文，自己增加了一个自定义字段enTitle来专门显示路由上的文字。

```javascript
return `/${node.date.split(' ')[0]}/${node.enTitle}`;
```



## 参考

>   https://gridsome.org/docs/schema-api/
>
>   https://graphql.org/learn/schema/
>
>   https://github.com/gridsome/gridsome/issues/1031


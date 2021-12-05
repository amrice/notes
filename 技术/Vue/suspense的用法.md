suspense组件提供了两个slot，分别是default和fallback，当suspense所有的异步子组件都完成后，会显示default中的内容，否则显示fallback中的内容。

所有异步子组件都完成是什么意思呢？就是组件中的异步操作要么正常resolve了，要么reject了。所以在异步子组件完成后有两种情况，一种是正常resolve，我们显示组件内容，一种是reject，我们可以显示错误信息。如果fallback用于loading提示，default用于成功，那么suspense是不是还缺了一个error的slot？

下面以vue3.2.16版本为例，将suspense组件包装一下，让它能显示loading，error，success的内容。

### 第一种做法，将error放到default里面，不过会报错

```vue
<template>
    <div>
        <suspense>
            <template #default>
                <div>
                    <slot name="error" v-if="$slots?.error"></slot>
                    <slot name="default" v-else></slot>
                </div>
            </template>
            <template #fallback>
                <slot name="fallback"></slot>
            </template>
        </suspense>
    </div>
</template>
```

这种做法是在default里面分出一种error，当pending的时候，显示fallback，当pending变为reject后，vue会报错

![image-20211205224020238](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20211205224026.png)

### 第二种做法，将error放到fallback中，正常

```vue
<template>
    <div>
        <suspense>
            <template #default>
                <div>
                    <slot name="default"></slot>
                </div>
            </template>
            <template #fallback>
                <slot name="error" v-if="$slots?.error"></slot>
                <slot name="fallback" v-else></slot>
            </template>
        </suspense>
    </div>
</template>
```

### 第三种做法，将error放到suspense组件外，正常

```vue
<template>
    <div>
        <slot name="error" v-if="$slots?.error"></slot>
        <suspense v-else>
            <template #default>
                <slot name="default"></slot>
            </template>
            <template #fallback>
                <slot name="fallback"></slot>
            </template>
        </suspense>
    </div>
</template>
```

上面不管是哪种方法，都是为了能加一个error分支，但第一种为何不行，还没理解到。。。

下面是使用的代码片段，加入我们封装的组件为MySuspense.vue，用于测试的异步子组件为AsyncComponent

```vue
<template>
    <MySuspense>
        <template v-if="error" #error>{{error}}</template>
        <template #default>
            <AsyncComponent></AsyncComponent>
        </template>
        <template #fallback>Loading...</template>
    </MySuspense>
</template>

<script setup>
import MySuspense from "./MySuspense.vue";
import AsyncComponent from "./AsyncComponent.vue";
import {ref, onErrorCaptured} from "vue";

const error = ref(null);

onErrorCaptured(err => {
    error.value = err;
});

</script>

<style scoped lang='scss'>

</style>
```

AsyncComponent.vue

异步子组件，这里还要再说一下，就是setup中有async/await或者组件通过import()函数导入的都算异步，其实异步子组件，并不是指组件的加载方式是异步，而是渲染是异步的，包括组件中的所有可能的异步操作，下面是一个例子

```vue
<template>
    <div>{{t}}xxx</div>
</template>

<script setup>

import { ref } from "vue";

const delay = async ms => {
    return new Promise((resolve, reject) => {
        if (ms > 10000) {
            setTimeout(() => reject('error'), ms);
        } else {
            setTimeout(() => resolve(1), ms);
        }
    });
}
// 方便测试，大于10s是reject，否则是resolve
const time = await delay(12000);
const t = ref(time);

</script>

```




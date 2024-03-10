---
title : 'Radix Vue'
date : 2024-02-25T14:05:36+08:00
draft : true
---

目的: 实现一个组件库，并学习其封装技术


#### vite-cli 初始化项目
```js
pnpm create vite
```

#### docs 初始化
```js
找一个nuxt的docs模板下载
links:https://docus.dev/ 
npx nuxi@latest init -t themes/docus
遇到问题:无法fetch，据说是ip没找到
添加hosts 185.199.110.133 raw.githubusercontent.com
在nuxt-studio导入自动配置(我一开始还看了半天commit，不知道哪来的这些文件。。。)
```

#### 配置monorepo
```js
// 参考视频：https://www.youtube.com/watch?v=HM03XGVlRXI
// 在pnpm定义workspace，pnpm-workspace.yaml
packages:
  # all packages in direct subdirs of packages/
  - 'packages/*'
  # all packages in subdirs of components/
  - 'components/**'
  # exclude packages that are inside test directories
  - '!**/test/**'

// 在playground中导入本地依赖
pnpm add radix-vue@workspace:*  //在后面加workspace
pnpm link radix-vue // 或者使用link
还有在package.json添加link确定路径的 //https://github.com/yarnpkg/rfcs/blob/master/implemented/0000-link-dependency-type.md
```

#### CheckBox组件
```js
// script setup是干嘛的？
答：可以在script中直接使用composition api以及js逻辑,并且与template分离

// html标签的role属性是干嘛的？
role是属于aria(accessible rich internet applications)用于加强浏览器的语义识别，主要是为了帮助盲人 //  links:https://www.youtube.com/watch?v=0hqhaije_8I

// provide和inject是干嘛的？
父传子的一种方法，并且可以在子组件中修改，但是不推荐直接修改，建议父组件再传一个方法下去 e.g:https://stackoverflow.com/questions/66962959/vue3-is-it-possible-to-change-the-data-given-by-provide-inject-from-childs-end

// setup script 的sfc怎么引用
issues:https://stackoverflow.com/questions/67669820/how-to-define-component-name-in-vue3-setup-tag  
Update from vue site "Since version 3.2.34, a single-file component using <script setup> will automatically infer its name option based on the filename, removing the need to manually declare the name even when used with <KeepAlive>

// export 重新导出,通过单个文件暴露其package提供的功能
我们可能会遇到两个问题：
export CheckboxRootVue from './CheckboxRoot.vue' //报错
export User from './user.js' 无效。这会导致一个语法错误。

要重新导出默认导出，我们必须明确写出 export {default as User}，就像上面的例子中那样。

export * from './user.js' 重新导出只导出了命名的导出，但是忽略了默认的导出。

如果我们想将命名的导出和默认的导出都重新导出，那么需要两条语句：
export * from './user.js'; // 重新导出命名的导出
export {default} from './user.js'; // 重新导出默认的导出

// tailwindcss [&]是干嘛的？
links:https://tailwindcss.com/docs/hover-focus-and-other-states#using-arbitrary-variants //可以写复杂的选择器

//tailwind失效？
只装了tailwindcss没有装postcss

// vue setup sfc导入路径报错
使用volar而不是vetur
// 如果防止hover事件冒泡，在checkbox 的indicator里面防止冒泡？
pointer-events:none
// vue3自定义组件中使用v-model
需要提供属性modalValue,emit事件"update:modelValue"
// <!-- CustomInput.vue -->
export default {
  props: ['modelValue'],
  emits: ['update:modelValue']
}
// App.vue 
<CustomInput v-model="searchText" />

// 为什么组件要两个div?
<template>
  {/* 第一个是为了居中 */}
  <div class="card-bg rounded-xl bg-neutral-200 flex items-center justify-center min-h-[300px]">
    {/* 第二个是为了隔离外部flex的影响 */}
    <div>
      <slot></slot>
    </div>
  </div>
</template>

//injection Key是干嘛的?
是provide 和inject的ts用法，设置key的同时可以设置value的类型
import { provide, inject } from 'vue'
import type { InjectionKey } from 'vue'

const key = Symbol() as InjectionKey<string>

provide(key, 'foo') // providing non-string value will result in error

const foo = inject(key) // type of foo: string | undefined

// withDefault 和defineProps组合的用法
export interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})

// 如果构建一个只读的ref?
// creates a readonly ref that calls the getter on .value access
toRef(() => props.foo)

//为什么组件要定义data-xxxx,因为样式可以根据属性来动态变更
<CheckboxRoot disabled
  class="data-[disabled]:bg-mauve9 bg-white shadow h-6 aspect-square rounded flex items-center justify-center focus-within-within:outline focus-within-within:outline-2 focus-within-within:outline-[#00000066]"
/>
//里面的disabled使用的是boolean判断
discussion:https://github.com/tailwindlabs/tailwindcss/discussions/12183

// slots default可以有多个？ 
// 在playground的上测试了一下，是可以有多个默认的. https://vuejs.org/guide/components/slots.html#dynamic-slot-names
const numberOfChildElements = (slots.default?.()[0].children?.length as number) ?? 0

// Primitive的作用是什么?
自定义修改组件的root，同时把定义好的属性传进去

// vue3 setup中的slots是什么？
slots是当父组件使用组件时，在标签内部传入的元素
  <Primitive>
    <div>1</div>
    <div>2</div>
  </Primitive>
// 在setup里面可以访问slots，并且可以配合component组件使用is属性以及v-bind实现子元素作为父组件并且继承父组件的props
    setup(props, { slots, attrs }) {
        const asChild = props.asChild
        console.log('[ slots ] >', slots.default?.())
        console.log('[ slots ] >', slots.footer?.())
        console.log({...attrs});
        
        if (asChild) {
            return () => h(Slot, attrs, slots.default?.())
        }
        else {
            return () => h('div', attrs, slots.default?.())
        }
    }
//log日志
(2) [
    {
    __v_isVNode : true,
    __v_skip : true,
    "type": "div",
    },
    {
    __v_isVNode : true,
    __v_skip : true,
    "type": "div",
    },
]
```
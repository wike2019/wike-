# vue3写作指南


## 前言

Vue 3 Core已经处于测试阶段两个多月了，喜欢新技术的我，尝试了vue3发现vue3对程序员的要求比以前高了很多，同时对ts的支持也提高了很多，本人经历了几个月的实践，总结了一些写vue3的技巧，和在写作过程中遇到的很多坑。



## 关于jsx



vue3 支持jsx写法，本来有一个编辑器项目，为了追求逼格，所以打算使用jsx来组织代码。随着项目的推进，发现jsx真的比模板语法差了很多，第一对代码的可维护性上没有html模板更加直观，同时写法难度很大，第二jsx 的render需要包含很多逻辑，对指令的支持非常不友好。推荐还是使用模板语法开发vue3组件。



## 关于ts 



vue3 对TS的支持明显好了很多，不像以前还需要装饰器等等东西，现在的代码完全函数式，对代码的压缩优化都有了很大的提高，所以推荐vue3使用ts做开发语言



##  组件代码组织



>  通过文件夹的方式区分不同组件

推荐的组织方式


```
├── src/    ----- 代码目录
│   ├── components/        ----- 组件目录 
│   │   └── Wkbtn/         ----- 单个组件 
|   │   │   └── index.vue  ----- 组件入口文件
|   │   │   └── btn.d.ts  -----  定义各种接口
|   │   │   └── usebtn.ts  ----- 组件代码的抽离文件（可选）
|   │   │   └── usestate.vue  ----- 组件代码抽离的数据文件（可选）
...其他文件夹
```



>  关于组件代码是否需要抽离出去，我觉得根据组件的复杂度来衡量，没有一定的标准。开发过程应该是先写在一个文件，然后根据组件的抽离难度，来决定是否抽离，虽然不抽离index.vue的setup函数的代码量会非常多，但抽离也不是那么容易的，尤其在配合其他第三方库时候抽离有时候难度非常大，所以是否抽离需要根据实际情况.
>
>  

## vue3代码写作技巧



###  props定义优化



场景我有一个props里面需要判断给定的值，例如只能是success，或者info


```javascript
export default defineComponent({
  props: {
    type: {
      type: String as PropType<'success' | 'info'>,
      default: 'info',
    },
  }
}）
```
PropType 还支持传各种类型，例如函数 

```javascript
export default defineComponent({
  props: {
  	Tooltip: {
 		 type: Function as PropType<(val: number) => number | string>
	}
 }
}）
```

PropType 还支持传各种类型，例如数组

```javascript
export default defineComponent({
  props: {
  	numdata: {
 		 type:Array as PropType<Array<number>>
   }
 }
}）
```


### 给组件声明事件



 ```javascript
export default defineComponent({
 emits: ['click'],
}）
 ```

除了系统默认事件，其他事件应该单独定义一个枚举文件。

 ```javascript
//constant.ts文件
export const UPDATE_MODEL_EVENT = 'update:modelValue'   

//组件文件
import {UPDATE_MODEL_EVENT} from "constant"
export default defineComponent({
 emits: [UPDATE_MODEL_EVENT],
}）
 ```



### 属性的传递



有时候做自定义组件的时候属性传递是很重要的，没必要每次一层层手动传递 有了 inheritAttrs: false 和 $attrs，你就可以手动决定这些 attribute 会被赋予哪个元。

```html
<template>
  <div class="autocomplete">
    <wk-input ref="inputRef"  v-bind="$attrs"></wk-input>
  </div>
</template>
```

```javascript
export default defineComponent({
  name: 'WkAutocomplete',
  inheritAttrs: false,
  props: {
  valueKey: {
    type: String,
    default: 'value',
  },
  modelValue: {
    type: [String, Number],
    default: '',
  },
  debounce: {
    type: Number,
    default: 300,
  },
  //..其他属性
  }
}）
```


### props相关



//在xx.d.ts 定义接口

```javascript
 interface IElBacktopProps {
  visibilityHeight: number
  target: string
  right: number
  bottom: number
}
```

```javascript
export default defineComponent({
 setup(props: IElBacktopProps,ctx){
   //其他代码
 }
}）
```

默认props的数据类型不是响应式的所以如果需要响应式需要做一层转换


```javascript
 const newProps=...toRefs(props)
```



### provide的使用



provide 不支持异步注入，所以你再onMounted等生命周期里注入子孙组件是得不到值的，所以不要试图异步注入。而且provide只支持有层级关系的组件，2个兄弟组件是没办法通过provide和,inject得到值的。
provide和inject的最常见作用是开发嵌套组件,子孙组件得到上级组件的注入属性，例如 el-form,el-form-item，el-button,el-button-group。



### vue3中路由的使用



vue3中路由不在绑定到this对象上，vue3不存在this,所以路由的写法有改变


```javascript
import { useRouter } from 'vue-router'

const router = useRouter()

router.replace({name:"home"})//替换
router.push({name:"home"})//追加
```



### 说说ref



-  ref 可以限定类型，这也是推荐的写法，限定类型的好处很多，所以有可能的情况下尽量限定类型。

-  ref 可以设置成refs的对象，这个和vue2很不同，推荐使用限定类型的方式定义，这样代码可维护性更高

-  ref 可以定义类型的时候设置可以为Nullable

-  ref 有时候会定义定时器的id, 

```javascript
 const CarouselItem=ref<CarouselItem[]>=[]  //第一种情况
 const itemView=ref<Nullable<HTMLElement>>(null) //第二种情况
 const background = ref<Nullable<string>>(null) //第三种情况
 const openTimer = ref<TimeoutHandle>(null) //第四种情况
```

通过这几种组合可以让你的代码维护性，可读性，健壮性提升一个档次



### 再谈h函数



```javascript
   import { h } from 'vue'
```
有时候我们的组件还需要支持命令式调用，这个时候h函数就非常有用，他可以命令式的创建各种组件。


```javascript
 h(
      Transition,
      {
        name: 'dialog-fade',
        onAfterEnter: this.afterEnter,
        onAfterLeave: this.afterLeave,
      },
      {
        default: () => overlay,
      },
    )

```



### vue3新组件teleport



用途: 可以控制HTML片段指定在某一父节点下呈现/渲染，而不必诉诸全局状态或将其拆分为两个组件，我觉得最大的功能就是入弹窗等组件，让他加入到body最后。或者有一些特殊的嵌套组件，指定加载在嵌套组件的末尾

```html
<teleport to="body">
   <div v-if="modalOpen" class="modal">
     <div>
       I'm a teleported modal! 
       (My parent is "body")
       <button @click="modalOpen = false">
          Close
        </button>
    </div>
  </div>
</teleport>
```



### 谈谈reactive



谈谈reactive定义与refs的区别，我觉得refs定义多为基础类型，reactive多为符合类型。

```javascript
 const data = reactive<{
    activeIndex: number
    containerWidth: number
    timer: null | ReturnType<typeof setInterval>
    hover: boolean
  }>({
    activeIndex: -1,
    containerWidth: 0,
    timer: null,
    hover: false,
  })
```



### 谈谈eventbus和组件实例



vue3 不再支持 on 和 off 所以使用 mitt 来替代 mitt是一个微型的库，实现了基本的 off on emit三个核心方法，所以又遇到需要使用eventbus的时候，使用这个库就可以了

vue3 组件不再有this,访问组件实例通getCurrentInstance方法

```javascript

import { getCurrentInstance } from 'vue'
setup(){
   const { ctx } = getCurrentInstance()
   console.log(ctx.$router.currentRoute.value)
}
```


## 关于我

网名WIKE 热爱前端，全栈工程师，文采不行请见谅，有什么建议可以邮箱联系我200569525@qq.com
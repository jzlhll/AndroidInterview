## Typescript学习

[TypeScript 教程 | 菜鸟教程](https://www.runoob.com/typescript/ts-tutorial.html)

### 简介

*TypeScript 是 JavaScript 的一个超集，支持 ECMAScript 6 标准*。

* 静态类型检查：编译时检查类型匹配，预防潜在错误。`number`、`string`、`boolean`、`array`、`tuple`、`enum` 等，此外也支持自定义类型。

> 1. ‌**`boolean`**‌
>    布尔类型，值为 `true` 或 `false` let  isActive: boolean = true;
>
> 2. ‌**`number`**‌
>    数字类型，包含整数、浮点数等let age: number = 25;
>
> 3. ‌**`string`**‌
>    字符串类型，支持文本数据 let name: string = "Alice";
>
> 4. ‌**`Array`**‌
>    数组类型，表示同类型元素的集合（可写作 `T[]` 或 `Array<T>`）
>
>    let list: number[] = [1, 2];
>
> 5. ‌**`tuple`**‌
>    元组类型，固定长度和类型的数组 let x: [string, number] = ["hello", 10];
>
> 6. ‌**`enum`**‌
>    枚举类型，定义命名常量集合 enum Color { Red, Green }
>
> 7. ‌**`any`**‌
>    任意类型，可动态赋值为任何类型 let dynamic: any = 4;
>
> 8. ‌**`void`**‌
>    空类型，常用于无返回值的函数 function log(): void { console.log("Done"); }
>
> 9. ‌**`null`**‌
>    表示空值 let n: null = null;
>
> 10. ‌**`undefined`**‌
>     表示未定义的值 let u: undefined = undefined;
>
> 11. ‌**`never`**‌
>     表示永不存在的值（如抛出异常的函数的返回类型）
>
>     function error(): never { throw new Error(); }
>
> 12. ‌**`object`**‌
>     非原始类型（除 `number`、`string` 等外的引用类型）13
>
>     let obj: object = { key: "value" };
>
> 13. ‌**`unknown`**‌
>     类型安全的 `any`，需类型检查后才能操作36
>
>     let uncertain: unknown = "Maybe a string";
>
> 14. ‌**`symbol`**‌
>     唯一不可变值，常用于对象属性键7
>
>     const sym: symbol = Symbol("key");



接口和自定义类型：interface, type复杂的数据结构。

```ts
interface Person {
  name: string;
  age: number;
  greet(): void;
}

class Student implements Person {
  constructor(public name: string, public age: number) {}

  greet() {
    console.log(`Hello, my name is ${this.name}`);
  }
}
```

类型别名：

```ts
type StringOrNumber = string | number;
let value: StringOrNumber = 42;
```

枚举：

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
let dir: Direction = Direction.Up;
```

类和模块支持：

​	构造函数，继承，public/private/protected。

模块和命名空间：

```ts
namespace CommonCode {
    interface Person{
        name: string
        age: number
        greet():void
    }

    abstract class AbsPerson{
        abstract name: string
        abstract age: number
        abstract greet():void
    }

    export class Student implements Person{
      xxxx
    }

    export class NewStudent implements AbsPerson{
        xxx
    }
}

export{CommonCode}
```

类型判断：

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(2));
  }
}
```

#### 其他

* 函数可选参数，默认参数，剩余参数

```ts
function buildName(firstName: string, lastName?: string) { //可选参数？
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}
function calculate_discount(price:number,rate:number = 0.50) { //默认参数
    var discount = price * rate; 
    console.log("计算结果: ",discount); 
} 
function buildName(firstName: string, ...restOfName: string[]) {//剩余参数
    return firstName + " " + restOfName.join(" ");
}
```

* const let var

  const常量不得后续修改；

  let块级访问，比如一个大括号就不得在括号外访问;

  var函数作用域或全局作用域，可能会被重复声明，最后一个有效。**避免使用var。**

  ```ts
  if (true) {
    let x = 10
    console.log(x) // 10
  }
  console.log(x) // 报错，x 未定义
  
  function foo() {
    var x = 1
    if (true) {
      var x = 2
    }
    console.log(x) // 2
  }
  ```

​	还有readonly和static。

* 泛型支持
* ??支持 类似kotlin的?:

## Vue3学习

https://www.cnblogs.com/wl-blog/p/15769906.html

https://www.bilibili.com/video/BV1zXcEeVEbu?spm_id_from=333.788.player.switch&vd_source=c2564a39cc59a81a52d9de6b7736dbfb&p=4

### vscode目录结构

index.html入口

script src main.ts引入src

div容器承载显示的vue



main.ts

createApp(App).mount('#app')

创建一个app，挂载在index.html的div app里。



App.vue

#### Vue中引入ts文件

在ts文件中，通过export {Student, NewStudent}来导出。

在vue中通过, import {Student, NewStudent} from "./Person.ts"导入。



### Vue标签

三个东西，template，script，style

*// 导出默认对象* 

const user = { name: '小明' };

export default user;

*// 导入默认导出时，可自定义名称（如myUser）*

import myUser from './myModule.js'; 

console.log(myUser.name); *// 输出：小明*

Vue2

以前拆分的data，methods，computed，watch的

### setup

vue3学习setup很重要。

* 返回值：可以是返回一个函数。返回渲染函数也是可以的。

* 了解methods,data, setup的关系。相互调用等。

	1. setup生命周期最早。setup没有this，其他有。
	1. methods，data就可以通过this来获取setup里面的变量，但是setup不能读取data，methods数据。
	1. 可以共存。但是vue2的可以读取vue3。

* 新建一个`<script setup lang="ts"> `来替代之前vue2的setup函数写法。可以不用写return了。

* 2个script，其中一个只用来定义name。

  ```vue
  <script lang="ts">
  export default {
    name: 'OtherPerson', //这是文件名字
    beforeCreate() {
      console.log("before create")
    },
  }
  </script>
  
  <script setup lang="ts">
      let name="allan"
      let age =18
      let tel = "12323124124"
      function changeName() {
          name = "zhangsan"
      }
      function changeAge() {
           age+=1
      }
      function showTel() {
          alert(tel)
      }
  </script>
  ```

  通过vueSetupExtend的插件，来配置name的变更。

  ```vue
  //vite.config.ts
  import vueSetupExtend from 'vite-plugin-vue-setup-extend'
  ...
    plugins: [
      vue(),
      vueSetupExtend(), //补充
    ],
  ....
  
  
  //Person.vue
  <script setup lang="ts" name="person9527">
      let name="allan"
      let age =18
      let tel = "12323124124"
      function changeName() {
          name = "zhangsan"
      }
      function changeAge() {
           age+=1
      }
      function showTel() {
          alert(tel)
      }
  </script>
  ```

### 响应式数据方式一 ref/reactive

类似LiveData....

* 基本类型的响应式数据

vue2，丢在data里面，return后的数据，就是响应式的。

vue3，响应数据两种：，ref和reactive。

 ```ts
 import { ref } from 'vue'
 let name= ref("allan")
 let age = ref(18)
 
 name.value = xx 来修改
 ```

调试里面看到的是refImpl，响应式基本数据。

* 对象类型的响应式数据 reactive

```ts
import { reactive } from 'vue'

let cat = reactive({"name":xxx, price:1000})

let games = reactive(
  {id:11111, name:"dfadf"},
  {id:222, name:"dfd"},
)
```

响应式对象、原对象。

调试里面可以看到变成了proxy，他就是响应式对象。

另外学到一个列表的展示方法：

```html
<li v-for="game in games" :key="g.id">{{game.name}}</li>
```

* 补充ref，其实可以做对象类型。

但是其实从console就能看出来，是通过

RefImp { value {proxy}}形式，包裹了reactive。而且调用上必须有value。现在的插件变了，官方也是在插件的设置中可以打开auto insert dot value.

value其实就是函数。

* ref vs reactive

ref定义基本类型；对象类型；

reactive定义 对象类型。

reactive的对象，**不能修改赋值，赋值就失去了响应式**。

只能通过Object.assign(car, {})来将新的对象的kv都赋值过去。

### toRefs和toRef

```javascript
import {reactive, toRefs} from 'vuew'
let person = reactive({name:"张三", age:18})

let {name, age} = toRefs(person) ////提取所有key用来做响应式对象

let newName = toRef(person， 'name') //提取其中一个用来做响应式对象
```

这样就可以得到name，age是一个ref。响应式。

而且，是只会修改对应的原始数据一份。



### Computed计算属性

> 类似kotlin的 val getter/getter

部分前端代码：

单向绑定：只能将变量绑定到页面上

```html
<input name="姓：" v-bind:value="xing"/>
```

双向绑定：变量绑定到界面上，页面的数据能同步到变量中

```html
<input name="姓：" v-model="xing"/>
```

computed

```vue
import {ref, reactive, computed} from 'vue'

//computed绑定
<h2>姓名:{{fullName}}</h2>
let fullName = computed(()=>{
  return xing.value + "_" + name.value
})

//类似函数绑定
<h2>姓名:{{fullName2()}}</h2>
function fullName2() {
  return xing.value + "-" + aa.value
}
```

函数绑定，如果有多个变量绑上同一个函数，会每一个函数都调用一次。而computed，在多个变量使用的情况下，只会跟随数据变动，触发一次。

* 上述的computed其实就是ref。

* 上述的computed定义就是只读的。不能修改。可以通过fullName.value查看。

```
     let fullName = computed({
        get() {
            return xing.value + "_" + aa.value
        },
        set(v) { //v随便命名
            console.log(`v ${v}`)
            //todo 自行解析v，来填充别的ref/reactive数据。
        }
     })
```

* 上面这个通过get(), set(v)的函数来定义computed，则变成了可读可写。

### Watch观察

> liveData Observer

观察数据的变化

只能观察以下4种数据：

* ref
* reactive
* 函数返回一个值（getter函数）
* 一个包含上述内容的数组。

#### 1. 监听ref的普通数据类型

```vue
<template>
     <div class="person">
        <h2>Num:{{num}}</h2>
     </div>
</template> 

<script setup lang="ts">
    import {ref, watch} from 'vue'
    let num = ref(0)

    watch(num, (newVal, oldVal) => {
        console.log(`newVal ${newVal} oldVal ${oldVal}`)
    })
</script>
```

```ts
const watchNumData = watch(num, (newVal, oldVal) => {
    console.log(`newVal ${newVal} oldVal ${oldVal}`)
})

watchNumData() //停止监听
```

注意添加的就是一个ref的对象。

对象()就是取消监听。

#### 2. 监听ref的对象类型

因为监听的对象的地址变化，如下代码将不会被回调：

```ts
let person = ref({name:"xx", age:18})

function changeName() {
 person.value.name = "xx"
  //
  person.value.age += 1
}

watch(person, (newVal, oldVal)=>{
  	//无法监听到
})
```

这个时候需要深度监听。

##### 深度监听

```ts
watch(person, (newVal, oldVal)=>{
  	//无法监听到
}, {deep:true, })
```

追加第三个参数对象，其中一项是deep:true则，能在改变person.value的某个属性的时候，触发回调，但是oldVal是不会变化的。除非你更新了整个person.value = {}

##### 立刻触发（粘性）

```ts
watch(person, (newVal, oldVal)=>{
  	//无法监听到
}, {immediate:true})
```

调用watch的瞬间，立刻触发一次。就像liveData一样。

#### 3. 监听reactive对象 

隐式地创建了深度监听。并且无法关闭深度监听。

```ts
let person =reactive({
  name:"xxx",
  age:18
})

watch(person, (value)=>{
	
})
```

#### 4. 监听reactive对象中的某个属性

##### 该属性是基本类型

```ts
watch(()=>person.name, (v)=>{})
```

这样就是watch能够监听的一种，getter函数来解决问题。函数式解决。

##### 该属性是对象类型

```ts
let person =reactie({
  xxx:
  xxx:
  obj: {
  	a:"aaa",
  	b:18
	  }
})
watch(()=> person.obj, (v)=>{}, {deep:true})
```

最佳开发实践，**监听reactive下的某个属性，都用getter函数。如果是对象类型，则加个deep。**

#### 5. 监听reactive多个属性

```ts
let person =reactie({
  xxx:
  xxx:
  obj: {
  	a:"aaa",
  	b:18
	  }
})
watch([()=> person.obj, ()=> person.name], (v)=>{}, {deep:true})
```

通过监听getter函数的数组来解决。

#### 6. watchEffect

自动监听使用到的任何ref或者reactive对象。

### ref标签

**在不同vue里面定义的id，最终整合到dom上会冲突。**

因此为了区分组件化的的vue，使用ref做标签来使用。

#### 1. 在html标签上-DOM节点

```vue
<template>
  <div>
    ...
    <h2 ref="nameRef" id="nameId">This is name</h2> //这里的id是不合适的，最后会被整合到html中可能重复。
    ...
  </div>
</template>

<script setup lang="ts">
    import { ref } from 'vue'
    let nameRef = ref() //通过ref()空的定义出对象映射到html的Dom元素。
  
    function onClick() {
      console.log(nameRef.value)
    }
</script>
```

通过定义一个空的ref()对象，来引入到标签上。得到的是DOM节点。

#### 2. 在组件标签上-实例对象

```vue
<!-- 主Vue -->
<template>
  <button @click="onClick">click In App</button>
  <br>
  <h2 ref="appTitle">App Title</h2>
  <Per info="ThisAPerson" ref="personPage"></Per>
</template>
<script lang="ts" setup>
  import {ref, reactive} from 'vue'
  import {type PersonBean} from './beans/Person'
  import Per from './components/Person.vue'
  let appTitle = ref()
  let personPage = ref()

  function onClick() {
      console.log("click on app ", appTitle.value, personPage.value) //就能够从personPage得到personList
  }
</script>

<!-- Person.vue 子-->
<template>
  <button >click In Person</button>
</template>

<script lang="ts" setup>
  import {reactive} from 'vue'
  import { type PersonBean } from '@/beans/Person'

  let personList = reactive<Array<PersonBean>>([
    {name:"a", age:12}
  ])
  defineExpose({personList})  //公开子vue的某个变量
</script>
```

如果想要公开子vue里面的成员。则需要defineExpose({  })。

#### reactive/ref支持泛型传入

通过泛型和单独定义接口的数据bean类，来约束代码类的合法性。

```ts
//beans/Person.ts
export interface PersonBean {
    name:string
    age:number
}

//x.vue
import {ref, reactive} from 'vue'
import {type PersonBean} from './beans/Person'

let person = ref<PersonBean>({name:"xxx", age:18})
let persons = ref<Array<PersonBean>>([
{name:"xxx", age:18},
{name:"dd", age:20},
])
let person2 = reactive<PersonBean>({name:"xxx", age:18})
let persons2 = reactive<Array<PersonBean>>([
{name:"xxx", age:18},
{named:"dd", age:20}, //这里名字写错就会有提示了。
])
```

### props

#### 自定义标签传递给子vue

```ts
<!-- 主vue中写入 -->
<template>
  ...
  <Per info="ThisAPerson" :myData="person"></Per> 
<!--重点：info就是一个自定义的标签，它传递了一个字符串，ThisAPerson-->
<!--重点：myData就是一个自定义的标签，它传递了person对象，通过冒号实现传递的取值而不是纯文字-->
</template>

<script lang="ts" setup>
  import {ref, reactive} from 'vue'
  import Per from './components/Person.vue'

  let person = ref<PersonBean>({name:"xxx", age:18})
  ...
</script>
  

<!-- 子vue中 -->
<template>
  <button >click In Person</button>
  <h3 >From App {{info}}</h3>                <!--重点这里-->  
  <h3 >From App My data is: {{myData}}</h3>  <!--重点这里-->
</template>

<script lang="ts" setup>
  defineProps(["info", "myData"])
</script>
```

可以自定义标签随意取名，比如这里的info，myData都是随便取的。然后在Person.vue中通过defineProps([])引入。

加一个冒号，则是表达式，会取出结果给传递过去。

```vue
<template>
  <button @click="onClick">click In App</button>
  <br>
  <Per info="ThisAPerson" :myData="person"></Per>
</template>

<script lang="ts" setup>
  import Per from './components/Person.vue'

  let person = ref<PersonBean>({name:"xxx", age:18})
 
  function onClick() {
    person.value.age += 1
  }

  watch(person, (newVal)=>{
    console.log("inApp on personChanged: ", newVal)
  }) //前面watch章节学到过，不开启deep:true就不会监听到变化

</script>

<style></style>


<template>
  <button >click In Person</button>
  <h3 >From App {{info}}</h3>
  <h3 >From App My data is: {{myData}}</h3>
</template>

<script lang="ts" setup>
  import {reactive, watch} from 'vue'
  import { type PersonBean } from '@/beans/Person'

  let personList = reactive<Array<PersonBean>>([
    {name:"a", age:12}
  ])

  let x = defineProps(["info", "myData"])
  console.log("x ", x)

  watch(x.myData, (newVal)=>{
    console.log("myData changed ", newVal)
  })

  defineExpose({personList})
</script>
```




HTML => HTMLParser => DOMTree + Attachment => RenderTree + layout => Painting => Display
 

# JS 基础知识点及常考面试题

## 原始（Primitive）类型

**涉及面试题：原始类型有哪几种？null 是对象嘛？**
6种：boolean、null、undefined、number、string、symbol
**原始类型**存储值，没有函数可以调用
'1'.toString() 的'1' 不是原始类型，而是强制转换成 String 类型(对象类型)
**number** 是浮点类型，比如 0.1 + 0.2 !== 0.3是为true
**string** 不可变，无论在 string 类型上调用何种方法，都不会对值有改变。
**typeof null** 会输出 object，JS 早期版本使用 32 位系统，为了性能考虑使用低位存储变量的类型信息，000 开头代表对象，然而 null 为全零，所以将它错误判断为 object

## 对象（Object）类型
**涉及面试题：对象类型和原始类型的不同之处？函数参数是对象会发生什么问题？**
原始类型存储值，对象类型存储**地址**（指针）
函数传参传递的是对象指针的副本

## typeof vs instanceof
**涉及面试题：typeof 是否能正确判断类型？instanceof 能正确判断对象的原理是什么？**
typeof 对于原始类型，除了 null 都可以显示正确类型
typeof 对于对象，除了函数都会显示 object
instanceof 可以判断一个对象的正确类型，判断原理是通过**原型链**来判断类型
``` JavaScript
const Person = function() {}
const p1 = new Person()
p1 instanceof Person // true

var str = 'hello world'
str instanceof String // false

var str1 = new String('hello world')
str1 instanceof String // true
```

## 类型转换
**该知识点常在笔试题中见到，熟悉了转换规则就不惧怕此类题目**
JS 中类型转换只有三种情况，分别是：
**转换为布尔值**
undefined， null， false， NaN， ''， 0， -0一律转为false
其他所有值都转为 true，包括对象
转换为数字
转换为字符串

**四则运算符**
如果一方为字符串，那么另一方也转换为字符串
如果一方不是字符串或者数字，那么会将它转换为数字或者字符串

``` JavaScript
1 + '1' // '11'
true + true // 2
4 + [1,2,3] // "41,2,3"
```

## this

**涉及面试题：如何正确判断 this？箭头函数的 this 是什么？**
``` JavaScript
function foo() {
  console.log(this.a)
}
var a = 1
foo()

const obj = {
  a: 2,
  foo: foo
}
obj.foo()

const c = new foo()
```

1.直接调用 foo ，this 一定是window
2.谁调用了函数，谁就是 this
3.对于 new 来说，this 绑定在 c 上面，不会被改变

**箭头函数**没有 this ，this 取决**包裹箭头函数的第一个普通函数的 this**
对箭头函数使用 bind 这类函数是无效的
bind等改变上下文的函数，this 取决于第一个参数，如果为空，就是 window
如果对一个函数进行多次 bind，this 由**第一次 bind 决定**
``` JavaScript
let a = {} 
let fn = function () { 
  console.log(this) 
} 
fn.bind().bind(a)() // => ?// fn.bind().bind(a) 等于 let fn2 = function fn1() {   return function() {     return fn.apply()   }.apply(a) } fn2()

```
**this 规则优先级**
1.new优先级最高
2.bind()
3.obj.foo() 
4.foo()

## == vs ===
**涉及面试题：== 和 === 有什么区别？**
**==对比的类型不一样的话，就进行类型转换**
1.首先判断类型是否相同。相同的话比大小
2.类型不相同进行**类型转换**
3.判断是否对比 null 和 undefined，是的话返回 true
4.判断是否为 string 和 number，是的话将字符串转换为 number
`1 == '1' //1 ==  1`
5.判断其中一方是否为 boolean，是的话就会把 boolean 转为 number 再进行判断
`'1' == true //  '1' ==  1  =>  1  ==  1`
6.判断一方是 object 且另一方为 string、number 或者 symbol，把 object 转为原始类型
`'1' == { name: 'yck' }         ↓ '1' == '[object Object]'`
**=== 判断两者类型和值是否相同**

## 闭包
**涉及面试题：什么是闭包？**
**闭包的定义**：函数 A 内有函数 B，函数 B 可以访问函数 A 的变量，函数 B 是闭包
**闭包的意义**：间接访问函数内部的变量
**相关面试题**：循环中使用闭包解决 `var` 定义函数的问题
第一种是使用闭包的方式

```
for (var i = 1; i <= 5; i++) {     
    function(j) {         
        setTimeout(()=>console.log(j), j*1000)     
    }(i) 
}
```

第二种使用 setTimeout 的第三个参数，这个参数会被当成 timer 函数的参数传入

```

for (var i = 1; i <= 5; i++) {   
    setTimeout(     
        function timer(j) {console.log(j)}, i * 1000, i
    ) 
}

```

第三种使用 let 定义 i 来解决问题，这个也是最为推荐的方式
```
for (let i = 1; i <= 5; i++) {   
    setTimeout(
        function timer() {console.log(i)}, 
        i * 1000) 
}
```

## 深浅拷贝
**涉及面试题：什么是浅拷贝？如何实现浅拷贝？什么是深拷贝？如何实现深拷贝？**
对象类型在赋值过程其实是复制了地址，如果改变了对象A, 指向同一地址的B也被改变的情况
通常在开发中不希望出现这样的问题，可以使用浅拷贝解决
**浅拷贝：**
1.Object.assign：只会拷贝所有属性到新对象，如果属性值是对象，只拷贝地址
```
let a = {age: 1}
let b = Object.assign({}, a) 
a.age = 2
console.log(b.age) // 1
```

2.通过展开运算符 ... 实现浅拷贝

**深拷贝**
通常通过 JSON.parse(JSON.stringify(object)) 解决
**存在的问题**：会忽略 undefined、 symbol、不能序列化函数、不能解决循环引用的对象
**自实现深拷贝**：推荐使用 **lodash** 的深拷贝函数

## 原型
**涉及面试题：如何理解原型？如何理解原型链？**
obj 有 `__proto__` 属性指向原型，不推荐使用（早期为了访问内部属性**prototype**而实现）
obj 通过 `__proto__` 找到原型对象
**constructor**有一个 **prototype** 属性，这个属性的值和 __proto__一样
原型对象的 **constructor 指向构造函数**，构造函数的 **prototype 属性指回原型对象**
`Proto.constructor = Constructor; Constructor.prototype = Proto`
不是所有函数都具有这个属性，Function.prototype.bind() 就没有这个属性
 
总结：
Object 是对象的爸爸，对象通过 `__proto__` 找到它
Function 是函数的爸爸，函数通过 `__proto__` 找到它
函数的 prototype 是一个对象
对象的 __proto__ 属性指向原型， __proto__ 将对象和原型连接起来组成了原型链


# 4.ES6 知识点及常考面试题
## var、let 及 const 区别
**涉及面试题：什么是提升？什么是暂时性死区？var、let 及 const 区别？**
var 存在提升，能在声明之前使用。var 在全局作用域下声明变量会导致变量挂载在 window 上
let、const 因为暂时性死区，不能在声明前使用
let 和 const 作用基本一致，但是后者声明的变量不能再次赋值

**提升（hoisting）**
**提升声明**：变量没有被声明，但却可以使用
**提升函数**：变量声明在函数之后，变量也会是函数，这说明函数提升优先于变量提升
```
console.log(a) // ƒ a() 
{}
function a() {}
var a = 1
```
var声明的变量会被提升到作用域的顶部
在全局作用域下使用 let 和 const 声明变量，变量并不会被挂载到 window 上
提升存在的根本原因是为了解决函数间互相调用

函数提升优先于变量提升，函数提升把函数挪到作用域顶部，变量提升把声明挪到作用域顶部

## 原型继承和 Class 继承
**涉及面试题：原型如何实现继承？Class 如何实现继承？Class 本质是什么？**
 JS 中并不存在类，class 只是语法糖，本质还是函数

**组合继承**
``` JavaScript
function Parent(value) { this.val = value } 
Parent.prototype.getValue = function() { console.log(this.val) }
function Child(value) { Parent.call(this, value) } 
Child.prototype = new Parent() 
const child = new Child(1) 
child.getValue() // 1 
child instanceof Parent // true
```
在子类的构造函数中通过 Parent.call(this) 继承父类的属性
改变子类的原型为 new Parent() 来继承父类的函数

优点：构造函数可以传参，不会与父类引用属性共享，复用父类的函数
缺点：继承父类函数时调用了父类构造函数，导致子类的原型上多了不需要的父类属性，存在内存的浪费

**寄生组合继承**
对组合继承进行优化，组合继承缺点在于继承父类函数时调用了构造函数，优化掉这点就行
function Parent(value) { this.val = value } Parent.prototype.getValue = function() { console.log(this.val) }function Child(value) { Parent.call(this, value) } Child.prototype = Object.create(Parent.prototype, {   constructor: {     value: Child,     enumerable: false,     writable: true,    configurable: true} })const child = new Child(1) child.getValue() // 1 child instanceof Parent // true

Object.create(proto, [properties])：创建新对象，使用现有对象提供新对象的proto
既解决了无用的父类属性问题，还能正确的找到子类的构造函数

**Class 继承**
class 实现继承的核心在于使用 extends 表明继承自哪个父类，并且在子类构造函数中必须调用 super，因为这段代码可以看成 Parent.call(this, value)。

##   模块化
**涉及面试题：为什么要使用模块化？都有哪几种方式可以实现模块化，各有什么特点？**
1.解决命名冲突
2.提供复用性
3.提高代码可维护性

**立即执行函数**
早期使用立即执行函数实现模块化，通过函数作用域解决命名冲突、污染全局作用域的问题

**CommonJS**
CommonJS 最早是 Node 在使用，在 Webpack 也能见到它
``` JavaScript
// a.js
module.exports = { a: 1 }  // or  exports.a = 1
// b.js
var module = require('./a.js')
module.a  // -> log 1
```
**ES Module**
ES Module 是原生实现的模块化方案，与 CommonJS 有以下几个区别
CommonJS 动态导入， require(${path}/xx.js)
CommonJS 同步，因为用于服务端，文件导入快，同步导入卡住主线程影响不大
ES Module异步导入，因为用于浏览器，需要下载文件，同步导入会对渲染有影响

CommonJS 在导出时都是值拷贝，导入的值不会改变，如果想更新值，必须重新导入一次
ES Module 采用实时绑定，导入导出都指向同一个内存地址
ES Module 会编译成 require/exports 来执行
``` JavaScript
// 引入模块 API
import XXX from './a.js'
import { XXX } from './a.js'// 导出模块 API
export function a() {}
export default function() {}
```
## Proxy
**涉及面试题：Proxy 可以实现什么功能？**
Proxy 是 ES6 新增的功能，用来自定义对象中的操作
let p = new Proxy(target, handler)
target 代表添加代理的对象，handler 自定义对象中的操作，比如 set 或 get 函数
通过 Proxy 来实现一个数据响应式
``` JavaScript
let onWatch = (obj, setBind, getLogger) => {   
  let handler = {     
    get(target, property, receiver) {       
      getLogger(target, property)       
      return Reflect.get(target, property, receiver)     
    },     
    set(target, property, value, receiver) {       
      setBind(value, property)       
      return Reflect.set(target, property, value)     
    }   
  }   
  return new Proxy(obj, handler) 
}
let obj = { a: 1 } 
let p = onWatch(obj, (v, property) => {     
  console.log(`监听到属性${property}改变为${v}`)   
  },   
  (target, property) => {     
    console.log(`'${property}' = ${target[property]}`)   
  }
) 
p.a = 2 // 监听到属性a改变 p.a // 'a' = 2
```
Vue3.0 用 Proxy 替换原本的API
Proxy 无需递归为每个属性添加代理，一次即可完成操作，性能更好
原本实现有一些数据更新不能监听到，但是 Proxy 可以监听到任何方式的数据改变
唯一缺陷可能是浏览器的兼容性不好

# map, filter, reduce
**涉及面试题：map, filter, reduce 各自有什么作用？**
### map
作用是**生成新数组，遍历原数组**，将每个元素拿出来做变换然后放入新数组
[1, 2, 3].map(v => v + 1) // -> [2, 3, 4]
map 的回调函数接受三个参数，分别  是当前索引元素，索引，原数组
['1','2','3'].map(parseInt)
第一轮遍历 parseInt('1', 0) -> 1
第二轮遍历 parseInt('2', 1) -> NaN
第三轮遍历 parseInt('3', 2) -> **NaN**

### filter
作用是**删除数组中不需要的元素，生成新数组时将返回值为 true 的元素放入**
``` JavaScript
let array = [1, 2, 4, 6]
let newArray = array.filter(item => item !== 6)
console.log(newArray) // [1, 2, 4]
```
filter 的回调函数也接受三个参数，用处也相同

### reduce
将数组中的元素通过回调函数最终转换为一个值
**demo1：实现将函数的元素全部相加得到一个值**
``` JavaScript
const arr = [1, 2, 3]
let total = 0
for (let i = 0; i < arr.length; i++) {   
  total += arr[i] 
}
console.log(total) //6 
```
使用reduce 
``` JavaScript
const arr = [1, 2, 3]
const sum = arr.reduce((acc, current) => acc + current, 0)
console.log(sum)
```
reduce 接受两个参数，**回调函数**和**初始值**
分解reduce 的过程
首先初始值为 0，该值会在执行第一次回调函数时作为第一个参数传入
回调函数接受四个参数，分别为累计值、当前元素、当前索引、原数组，后三者想必大家都可以明白作用，这里着重分析第一个参数
在一次执行回调函数时，当前值和初始值相加得出结果 1，该结果会在第二次执行回调函数时当做第一个参数传入
所以在第二次执行回调函数时，相加的值就分别是 1 和 2，以此类推，循环结束后得到结果 


**demo2: 通过 reduce 来实现 map 函数**
``` JavaScript
const arr = [1, 2, 3]
const mapArray = arr.map(value => value * 2)
const reduceArray = arr.reduce((acc, current) => {  
  acc.push(current * 2)   
  return acc 
  }, []
) 
console.log(mapArray, reduceArray) // [2, 4, 6]
```


# 5.JS 异步编程及常考面试题
异步编程是 JS 中至关重要的内容
## 并发（concurrency）和并行（ parallelism）区别
**涉及面试题：并发与并行的区别？**
并发是宏观概念：任务 A 和任务 B在一段时间内通过任务间切换完成
并行是微观概念： CPU 存在两个核心，同时完成任务 A、B

## 回调函数（Callback）
涉及面试题：什么是回调函数？回调函数有什么缺点？如何解决回调地狱问题？
回调函数致命的弱点，容易写出回调地狱（Callback hell）
回调地狱的根本问题就是：
嵌套函数存在耦合性，一旦改动就会牵一发而动全身
嵌套函数一多，就很难处理错误
回调函数的缺点：不能使用 try catch 捕获错误，不能直接 return

## Generator
涉及面试题：你理解的 Generator 是什么？
首先 Generator 函数调用和普通函数不同，它会返回一个迭代器
执行next（） 传入的参数等于上一个 yield 的返回值，
如果不传参，yield 返回 undefined

## Promise
涉及面试题：Promise 的特点是什么，分别有什么优缺点？什么是 Promise 链？Promise 构造函数执行和 then 函数执行有什么区别？
三种状态：等待中（pending）；完成了 （resolved）；拒绝了（rejected）
链式调用：每次调用 then 之后返回一个全新的 Promise
如果在 then 中 使用return， return 的值会被 Promise.resolve() 包装
优点：解决了回调地狱
缺点：无法取消 Promise，错误需要通过回调函数捕获

Promise.prototype.then():
定义在原型对象Promise.prototype上
作用是为 Promise 实例添加状态改变时的回调函数
第一个参数是resolved的回调函数，第二个参数（可选）是rejected的回调函数


## async 及 await：异步终极解决方案
涉及面试题：async 及 await 的特点，它们的优点和缺点分别是什么？await 原理是什么？
函数加上 async ，就会返回一个 Promise；await 只能配套 async 使用
返回值使用 Promise.resolve() 包裹，和 then 中处理返回值一样
优势：处理 then 调用链时能清晰写出代码，毕竟写一大堆 then 也很恶心，优雅解决回调地狱
缺点： await 将异步代码改造成同步代码，多个异步代码没有依赖性使用了await 会降低性能
let a = 0let b = async () => {   a = a + await 10   console.log('2', a) // -> '2' 10} b() a++console.log('1', a) // -> '1' 1
函数 b 先执行，执行到 await时，因为后来的表达式不返回 Promise ，就会包装成 Promise.reslove(返回值)，然后执行函数外的同步代码，然后执行异步代码
因为 await 内部实现了 generator ，generator 保留堆栈中东西，所以a = 0 保存下来
这时候 a = 0 + 10
await原理：  generator 加上 Promise 的语法糖，且内部实现了自动执行 generator

## 常用定时器函数
涉及面试题：setTimeout、setInterval、requestAnimationFrame 各有什么特点？
setInterval作用和 setTimeout 一致，只是该函数是每隔一段时间执行一次回调函数
通常不建议 setInterval，因为不能保证在预期时间执行任务，并且存在执行累积的问题
循环定时器的需求可以通过 requestAnimationFrame 实现

## 6.手写 Promise

``` JavaScript
let timeout = 2000
let p = new Promise((resolve, reject) => {
    setTimeout(() => {
        console.log("网络请求");
        let res = "ok"
        resolve(res)
        reject(res)
    }, timeout);
})

p.then((res) => {
    console.log(res);
}).catch((e) => {
    console.log("请求失败");
    console.log(e);
})


function MyPromise (fn) {
    const that = this
    state
    value
    resolveCallbacks

    function resolve (value) {
        value
        state
        resolveCallbacks.map(cb => cb(value))
    }

    try {
        fn(resolve)
    } catch () {

    }
}

MyPromise.prototype.then((onFullFilled, onRejected) => {
    if (state === PENDING) {
        resolveCallbacks.push(onFullFilled)
    }
})

```

# 7.Event Loop
## 进程与线程
涉及面试题：进程与线程区别？JS 单线程带来的好处？
CPU 工作时间片的一个描述
进程：描述了 CPU 在运行指令及加载和保存上下文所需的时间，代表一个程序
线程：进程中更小的单位，描述了执行一段指令所需的时间
浏览器中，一个 Tab 页是一个进程，一个进程中有多个线程，比如渲染线程、JS 引擎线程、HTTP 请求线程等。发起一个请求就是创建一个线程，请求结束后线程销毁

JS 运行的时候可能会阻止 UI 渲染，说明了两个线程是互斥
因为 JS 可以修改 DOM， JS 执行时 UI 线程还在工作会导致不能安全的渲染 UI
单线程的好处：可以节省内存，节约上下文切换时间，没有锁的问题


## 执行栈
涉及面试题：什么是执行栈？
执行栈：存储函数调用的栈结构，遵循先进后出的原则。

## 浏览器中的 Event Loop
涉及面试题：异步代码执行顺序？解释什么是 Event Loop ？
执行 JS 代码就是往执行栈中放入函数
遇到异步代码时会被挂起并在需要执行的时候加入到 Task（有多种 Task） 队列中
一旦执行栈为空，Event Loop 就从 Task 队列拿出需要执行的代码并放入执行栈中执行
所以本质上JS 的异步还是同步行为
不同的任务源会被分配到不同的 Task 队列中，任务源分为 微任务和 宏任务
ES6 规范中microtask 称为 jobs，macrotask 称为 task
待续。。。

8.JS 进阶知识点及常考面试题
## 手写 call、apply 及 bind 函数
涉及面试题：call、apply 及 bind 函数内部实现是怎么样的？
考虑几点：
1.不传入第一个参数，上下文默认为 window
2.改变 this 指向，让新的对象可以执行该函数，并能接受参数
Function.prototype.myCall = function(context) {    if (typeof this !== 'function') {        throw new TypeError('Error')    }    context = context || window    context.fn = this    const args = [...arguments].slice(1)    const result = context.fn(...args)     delete context.fn    return result }
待续。。。


## new
涉及面试题：new 的原理是什么？通过 new 的方式创建对象和通过字面量创建有什么区别？
调用 new 的过程中会发生四件事情：
新生成一个对象
链接到原型
绑定 this
返回新对象

## 手动实现new
function create() {   // 1.create empty obj   let obj = {}    // 2.get Constructor   let Con = [].shift.call(arguments)  // arguments: array[func]   // 3.set prototype of empty obj obj.__proto__ = Con.prototype// 4.bind "this" & call Constructor   let result = Con.apply(obj, arguments)// 5.determine wherther the value of return is Object   return result instanceof Object ? result : obj }

对于创建对象来说，更推荐使用字面量的方式创建对象（无论性能上还是可读性）
因为使用 new Object() 创建对象需要通过作用域链一层层找到 Object，但使用字面量的方式就没这个问题

## instanceof 的原理
instanceof 可以正确判断对象类型
内部机制是通过判断对象原型链中是不是能找到类型的prototype
``` JavaScript
function myInstanceof(left, right) {   
    let prototype = right.prototype   
    left = left.__proto__   
    while (true) {     
        if (left === null || left === undefined)       
            return false     
        if (prototype === left)       
            return trueleft = left.__proto__   
    } }
```
## 为什么 0.1 + 0.2 != 0.3
涉及面试题：为什么 0.1 + 0.2 != 0.3？如何解决这个问题？
因为0.1 在二进制中无限循环，JS采用的浮点数标准会裁剪掉这种数字，造成精度丢失
**解决方法**
parseFloat((0.1 + 0.2).toFixed(10)) === 0.3 // true


## 垃圾回收机制  Garbage collection
涉及面试题：V8 下的垃圾回收机制是怎么样的？
V8 实现了准确式 GC，GC 算法采用了分代式GC机制
因此，V8 将内存（堆）分为新生代和老生代两部分

**新生代算法**
对象存活短
新生代空间分为 From 和 To ，一个使用，另一个空闲
新对象被放入 From 
当 From 被占满时，GC启动，检查 From 存活的对象，并复制到 To 中，失活对象销毁
当复制完成后将 From 和 To 互换，GC结束

**老生代算法**
存活时间长且数量多，使用了两个算法，分别是标记清除算法和标记压缩算法
什么情况下对象会出现在老生代空间中：
新生代的对象已经经历过一次 Scavenge 算法
To 的对象占比大小超过 25 %

## 10.JS 思考题
思考题一：JS 分为哪两大类型？都有什么各自的特点？你该如何判断正确的类型？
1.对于原始类型来说，可以指出 null 和 number 存在的问题。
对于对象类型来说，可以从垃圾回收的角度去切入，也可以说一下对象类型存在深浅拷贝的问题
2.对于判断类型来说，可以去对比 typeof 和 instanceof 的区别
可以指出 instanceof 判断类型不是完全准确。

思考题二：你理解的原型是什么？
起码说出原型小节中的总结内容，然后指出一些小点，比如不是所有函数都有 prototype 属性，然后引申出原型链的概念，提出如何使用原型实现继承，继而可以引申出 ES6 中的 class 实现继承

思考题三：bind、call 和 apply 各自有什么区别？
首先说出三者的不同，如果自己实现过其中的函数，可以说出自己的思路。
然后聊一聊 this 的内容，有几种规则判断 this 到底是什么，this 规则会涉及到 new，那么最后可以说下自己对于 new 的理解

思考题四：ES6 中有使用过什么？
思路引导：
比如说说 class，那么 class 又可以拉回到原型的问题
可以说说 promise，那么线就被拉到了异步的内容
可以说说 proxy，那么如果你使用过 Vue 这个框架，就可以谈谈响应式原理的内容
同样可以说说 let 这些声明变量的语法，可以谈及与 var 的不同，说到提升的内容


思考题五：JS 是如何运行的？
可以先说 JS 是单线程运行，说说你理解的线程和进程的区别。
然后讲到执行栈，接下来的内容就是涉及 Eventloop 了，微任务和宏任务的区别，哪些是微任务，哪些又是宏任务，还可以谈及浏览器和 Node 的 Eventloop 的不同，最后聊一聊 JS 中的垃圾回收


# 11.浏览器基础知识点及常考面试题
## 事件机制
涉及面试题：事件的触发过程是怎么样的？知道什么是事件代理嘛？
事件是文档和浏览器窗口中发生的特定的交互瞬间
IE的事件流是冒泡流(event bubbling)：事开始时由具体元素接收，逐级向上传播到不具体的节点
Netscape是捕获流(event capturing)：不具体节点向下传播至具体元素

事件触发有三个阶段：
1.window 往事件触发处传播，遇到注册的捕获事件会触发
2.传播到事件触发处时触发注册的事件
3.从事件触发处往 window 传播，遇到注册的冒泡事件会触发

特例，如果给 body 的子节点同时注册冒泡和捕获事件，事件触发会按照注册的顺序执行

注册事件
通常使用 addEventListener 注册事件，第三个参数可以是布尔值或对象
对于布尔值 useCapture 参数，默认值为 false ，useCapture 决定事件是捕获还是冒泡
对于对象参数，可以使用以下几个属性
capture：布尔值，和 useCapture 作用一样
once：布尔值，值为 true 表示该回调只会调用一次，调用后移除监听
passive：布尔值，表示永远不会调用 preventDefault
如果希望事件只触发在目标上，可以使用 stopPropagation 阻止事件的进一步传播
stopPropagation 可以用来阻止事件冒泡和捕获
stopImmediatePropagation 同样也能阻止事件，还能阻止该事件目标执行别的注册事件

**事件代理：**
例如：li的事件注册在ul上
如果节点中的子节点是动态生成的，那么子节点的事件应该注册在父节点上
事件代理相较于直接注册的优点：节省内存、不需要注销事件


## 跨域
涉及面试题：什么是跨域？为什么浏览器要使用同源策略？有几种方式解决跨域问题？了解预检请求嘛

因为浏览器出于安全考虑，有同源策略
**同源策略**：如果**协议、域名或者端口**有一个不同就是跨域，Ajax 请求会失败
具体的安全考虑： 防止 CSRF 攻击（Cross-site request forgery，利用用户的登录态发起恶意请求）
没有同源策略，A 网站可以被任意其他来源的 Ajax 访问到内容
如果 A 网站存在登录态，对方可以通过 Ajax 获得你的任何信息
当然跨域并不能完全阻止 CSRF
跨域的请求其实已经发出去了
请求必然是发出去了，但是浏览器拦截了响应。
通过表单的方式可以发起跨域请求，为什么 Ajax 就不会?
因为跨域是为了阻止用户读取到另一个域名下的内容，Ajax 可以获取响应，浏览器认为这不安全，所以拦截了响应。但是表单并不会获取新的内容，所以可以发起跨域请求。
同时也说明了跨域并不能完全阻止 CSRF，因为请求毕竟是发出去了

解决跨域问题的方式
JSONP
利用 <script> 标签没有跨域限制的漏洞，指向需要访问的地址并提供回调函数来接收数据
JSONP 使用简单且兼容性不错，但是只限于 get 请求
function jsonp(url, jsonpCallback, success) {   let script = document.createElement('script')   script.src = url   script.async = true   script.type = 'text/javascript'   window[jsonpCallback] = function(data) {     success && success(data)   }   document.body.appendChild(script) } jsonp('http://xxx', 'callback', function(value) {   console.log(value) })

CORS
服务端设置 Access-Control-Allow-Origin 开启 CORS
该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源。
通过这种方式解决跨域问题的话，会在发送请求时出现两种情况，分别为简单请求和复杂请求

简单请求
以 Ajax 为例，当满足以下条件时，会触发简单请求
13.	使用下列方法之一：
	GET
	HEAD
	POST
14.	Content-Type 的值仅限于下列三者之一：
	text/plain
	multipart/form-data
	application/x-www-form-urlencoded
请求中的任意 XMLHttpRequestUpload 对象均没有注册任何事件监听器
XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问

复杂请求
首先发起预检请求，该请求是 option 方法，通过该请求知道服务端是否允许跨域请求
对于预检请求，Node 设置 CORS会遇到一个坑
以 express 框架举例：
app.use((req, res, next) => {   res.header('Access-Control-Allow-Origin', '*')   res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS')   res.header(     'Access-Control-Allow-Headers',     'Origin, X-Requested-With, Content-Type, Accept, Authorization, Access-Control-Allow-Credentials')   next() })

该请求会验证你的 Authorization 字段，没有的话就会报错。
当前端发起了复杂请求后，你会发现就算你代码是正确的，返回结果也永远是报错的。因为预检请求也会进入回调中，也会触发 next 方法，因为预检请求并不包含 Authorization 字段，所以服务端会报错。
想解决这个问题很简单，只需要在回调中过滤 option 方法即可
res.statusCode = 204res.setHeader('Content-Length', '0') res.end()

document.domain
该方式只能用于二级域名相同的情况下，比如 a.test.com 和 b.test.com 适用于该方式。
只需要给页面添加 document.domain = 'test.com' 表示二级域名都相同就可以实现跨域

postMessage
这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息
// 发送消息端window.parent.postMessage('message', 'http://test.com')// 接收消息端var mc = new MessageChannel() mc.addEventListener('message', event => {   var origin = event.origin || event.originalEvent.origin   if (origin === 'http://test.com') {     console.log('验证通过')   } })

存储
涉及面试题：有几种方式实现存储功能，分别有什么优缺点？什么是 Service Worker？

cookie，localStorage，sessionStorage，indexDB
cookie 已经不建议用于存储
如果没有大量存储需求，可以使用 localStorage 和 sessionStorage 
对于不怎么改变的数据尽量使用 localStorage ，否则用 sessionStorage 
cookie安全性
value：保存登录态的话要加密
http-only：不能通过js访问，减少xss(Cross Site Scripting) 跨站脚本攻击
xss：使用js程序盗用cookie
secure：智能在HTTPS协议的请求中携带
same-site：规定浏览器不能在跨域请求中携带Cookie，减少CSRF攻击
Service Worker
Service Worker 是运行在浏览器背后的独立线程，一般用来实现缓存功能
传输协议必须为 HTTPS，因为 Service Worker 涉及请求拦截，所以保障安全
Service Worker 实现缓存的三个步骤：
1.注册 Service Worker
2.监听到 install 事件以后缓存文件
3.下次访问的时候通过拦截请求的方式查询是否存在缓存，存在则读取缓存，否则请求数据

源码待续。。。


12.浏览器缓存机制
注意：该知识点属于性能优化领域，并且整一章节都是一个面试题。
缓存是性能优化中简单高效的一种方式，可以显著减少网络传输所带来的损耗
对于数据请求来说，可以分为发起网络请求、后端处理、浏览器响应三个步骤。
浏览器缓存帮助我们在第一和第三步骤中优化性能。
比如说直接使用缓存而不发起请求，或者发起了请求但后端存储的数据和前端一致，那么就没有必要再将数据回传回来，这样就减少了响应数据。

缓存位置
从缓存位置来说分为四种，并且有优先级，当依次查找缓存且都没有命中的时候才请求网络
15.	Service Worker
16.	Memory Cache
17.	Disk Cache
18.	Push Cache
19.	网络请求

Service Worker
Service Worker 的缓存与其他缓存机制不同，它可以自由控制缓存哪些文件、如何匹配缓存、如何读取缓存，并且缓存是持续性的。
Service Worker 没有命中缓存的时候，调用 fetch 函数获取数据
如果没有在 Service Worker 命中缓存的话，会根据缓存查找优先级去查找数据
但是不管从 Memory Cache 还是从网络请求中获取数据，浏览器都显示从 Service Worker 中获取的内容


Memory Cache
读取内存中的数据比磁盘快，可是持续性短，会随着进程的释放而释放。 一旦关闭 Tab 页面，内存的缓存也就被释放了。

Disk Cache
读取速度慢点，但是比之 Memory Cache 胜在容量和存储时效性上
在所有浏览器缓存中，Disk Cache 覆盖面最大
它会根据 HTTP Header 的字段判断哪些资源需要缓存，哪些资源可以不请求直接使用，哪些资源已经过期需要重新请求。并且即使在跨站点的情况下，相同地址的资源一旦被硬盘缓存下来，就不会再次去请求数据

Push Cache
Push Cache 是 HTTP/2 中的内容，当以上三种缓存都没有命中时，它才会被使用。并且缓存时间也很短暂，只在会话（Session）中存在，一旦会话结束就被释放。

所有的资源都能被推送，但是 Edge 和 Safari 浏览器兼容性不好
可以推送 no-cache 和 no-store 的资源
一旦连接被关闭，Push Cache 就被释放
多个页面可以使用相同的 HTTP/2 连接，也就是使用同样的缓存
Push Cache 的缓存只能被使用一次
浏览器可以拒绝接受已经存在的资源推送
你可以给其他域名推送资源

缓存策略
通常缓存策略分为两种：强缓存和协商缓存，缓存策略通过设置 HTTP Header 实现

强缓存
强缓存通过设置两种 HTTP Header 实现：Expires 和 Cache-Control 
强缓存表示在缓存期间不需要请求，state code 为 200
Expires
Expires: Wed, 22 Oct 2018 08:41:00 GMT
Expires 是 HTTP/1 的产物，表示资源会在 22 Oct 2018 08:41:00 后过期，需要再次请求。
Expires 受限于本地时间，如果修改了本地时间，可能会造成缓存失效

Cache-control
Cache-control: max-age=30
Cache-Control 出现于 HTTP/1.1，优先级高于 Expires 
该属性值表示资源会在 30 秒后过期，需要再次请求
Cache-Control 可以在请求头或者响应头中设置，并且可以组合使用多种指令

协商缓存
如果缓存过期了，就需要发起请求验证资源是否更新
协商缓存通过设置两种 HTTP Header 实现：Last-Modified 和 ETag 
当浏览器请求验证资源时，如果资源没改变，服务端返回 304 状态码，并更新缓存有效期

Last-Modified 和 If-Modified-Since
Last-Modified 表示本地文件最后修改日期，If-Modified-Since 会将 Last-Modified 的值发送给服务器，询问服务器在该日期后资源是否有更新，有更新的话就会将新的资源发送回来，否则返回 304 状态码。
Last-Modified 的弊端：
如果本地打开缓存文件，即使没有修改， Last-Modified还是会被修改，服务端不能命中缓存导致发送相同的资源
 Last-Modified 以秒计时，如果在不可感知的时间内修改文件，服务端会认为资源还是命中了，不会返回正确的资源

因为以上弊端，在 HTTP / 1.1 出现了 ETag 
ETag 和 If-None-Match
ETag 类似于文件指纹，If-None-Match 会将当前 ETag 发送给服务器，询问该资源 ETag 是否变动，有变动的话就将新的资源发送回来。并且 ETag 优先级比 Last-Modified 高。

如果缓存策略没设置，浏览器会怎么处理？
浏览器会采用一个启发式的算法，（响应头的 Date  -  Last-Modified 的 10% ）作为缓存时间

实际场景应用缓存策略
频繁变动的资源
首先使用 Cache-Control: no-cache 使浏览器每次都请求服务器，然后配合 ETag 或者 Last-Modified 来验证资源是否有效。这样的做法虽然不能节省请求数量，但是能显著减少响应数据大小。

代码文件
HTML 文件一般不缓存或者缓存时间很短
现在使用工具来打包代码，可以对文件名进行哈希处理，代码修改后才会生成新的文件名
给代码文件设置缓存有效期一年 Cache-Control: max-age=31536000
只有当 HTML 文件引入的文件名发生改变才会下载最新的代码文件，否则使用缓存


13.浏览器渲染原理
注意：该章节都是一个面试题。
渲染引擎：Firefox Gecko， Chrome & Safari WebKit

浏览器接收到 HTML 文件并转换为 DOM 树

字节数据 => 字符串 =(词法分析，标记化)=> 标记(Token) => Node => DOM Tree

将 CSS 文件转换为 CSSOM 树
字节数据 => 字符串 =(词法分析，标记化)=> 标记(Token) => Node => 	CSSOM  

为什么操作 DOM 慢
因为 DOM 是属于渲染引擎的东西，而 JS 是 JS 引擎的东西。
JS 操作 DOM 涉及两个线程间的通信，势必带来性能损耗。
操作 DOM 次数多就等同于一直在进行线程间的通信，并且操作 DOM 会带来重绘回流
所以导致性能问题

经典面试题：插入几万个 DOM，如何实现页面不卡顿？
首先肯定不能一次性全部插入，这样肯定会卡顿，所以重点应该是如何分批次部分渲染 DOM
1.requestAnimationFrame循环的插入 DOM
2.虚拟滚动（virtualized scroller）
原理：只渲染可视区域内容，非可见区域完全不渲染，用户滚动实时替换渲染的内容

什么情况阻塞渲染
渲染的前提是生成渲染树，所以 HTML 和 CSS 肯定会阻塞渲染
1.如果想渲染快，应该降低一开始需要渲染的文件大小，并且扁平层级，优化选择器
2.当浏览器解析 script 标签时，会暂停构建 DOM。如果想首屏渲染得快，就不应该在首屏加载 JS 文件，这也是建议将 script 放在 body 标签底部的原因
除了放在底部，还可以给 script 标签添加 defer 或者 async 属性
defer 属性表示JS 文件会并行下载，但是会在 HTML 解析完成后顺序执行
async 属性表示 JS 文件下载和解析不会阻塞渲染

重绘（Repaint）和回流（Reflow）
重绘和回流会在设置节点样式时频繁出现，同时影响性能
	重绘：节点需要更改外观而不影响布局，比如改变 color 
	回流：布局或者几何属性需要改变
回流必定重绘，重绘不一定回流
回流所需的成本比重绘高的多，改变子节点可能导致父节点回流
以下动作可能会导致性能问题：
改变 window 大小
改变字体
添加或删除样式
文字改变
定位或者浮动
盒模型
很多人不知道，重绘和回流和 Eventloop 有关
20.	Eventloop 执行完 Microtasks 后，会判断 document 是否更新，浏览器是 60Hz 的刷新率，每 16.6ms 才会更新一次
21.	然后判断是否有 resize 或者 scroll 事件，有的话触发事件，所以  resize 和 scroll 事件也是至少 16ms 才触发一次，并自带节流功能
22.	判断是否触发了 media query
23.	更新动画并且发送事件
24.	判断是否有全屏操作事件
25.	执行 requestAnimationFrame 回调
26.	执行 IntersectionObserver 回调，该方法用于判断元素是否可见，可以用于懒加载
27.	更新界面
28.	以上是一帧中可能会做的事情。如果在一帧中有空闲时间，就会执行 requestIdleCallback 

既然我们已经知道了重绘和回流会影响性能，那么接下来我们将会来学习如何减少重绘和回流的次数。
减少重绘和回流
	使用 transform 替代 top
<div class="test"></div><style>   .test {     position: absolute;     top: 10px;     width: 100px;     height: 100px;     background: red;   }</style><script>   setTimeout(() => {     // 引起回流     document.querySelector('.test').style.top = '100px'   }, 1000)</script>
	使用 visibility 替换 display: none ，因为前者只引起重绘，后者引发回流（改变了布局）
	不要把节点属性值放在循环里当成循环里的变量
for(let i = 0; i < 1000; i++) {     // 获取 offsetTop 会导致回流，因为需要去获取正确的值     console.log(document.querySelector('.test').style.offsetTop) }
	不要使用 table 布局，可能造成整个 table 的重新布局
	动画实现的速度选择，速度越快，回流越多，可以选择使用 requestAnimationFrame
	CSS 选择符从右往左匹配查找，避免节点层级过多
	将频繁重绘或者回流的节点设置为图层，图层能阻止该节点的渲染行为影响别的节点。比如对于 video 标签，浏览器会自动将该节点变为图层
设置节点为图层的方式可以通过以下几个属性，生成新图层
	will-change
	video、iframe 标签

思考题：在不考虑缓存和优化网络协议的前提下，考虑可以通过哪些方式来最快渲染页面，也就是常说的关键渲染路径，这部分也是性能优化中的一块内容。

怎么测量有没有加快渲染速度呢
 
发生 DOMContentLoaded 事件后会生成渲染树，生成渲染树就可以进行渲染了，这一过程更大程度上和硬件有关系了

提示如何加速：
29.	从文件大小考虑
30.	从 script 标签使用上来考虑
31.	从 CSS、HTML 的代码书写上来考虑
32.	从需要下载的内容是否需要在首屏使用上来考虑


14.安全防范知识点
## XSS
涉及面试题：什么是 XSS 攻击？如何防范 XSS 攻击？什么是 CSP？
XSS：Cross Site Scripting 攻击者将代码注入网页中执行
XSS 分为两类：持久型和非持久型
持久型：代码被服务端写入数据库，这种攻击危害性很大，导致大量用户都受到攻击
举个例子，对于评论功能就得防范持久型 XSS 
非持久型：通过修改 URL 参数加入攻击代码，诱导用户访问链接从而攻击

**转义字符**
首先，对于用户的输入应该是永远不信任的。
最普遍的做法就是转义输入输出的内容，对于引号、尖括号、斜杠进行转义


**白名单过滤**
也可以通过黑名单过滤，但考虑到需要过滤的标签和标签属性太多，更推荐白名单
使用 js-xss实现

**CSP**
CSP 本质是建立白名单，开发者告诉浏览器哪些外部资源可以执行
只需要配置规则，如何拦截是由浏览器实现，从而减少 XSS 攻击
开启 CSP：
33.	设置 HTTP Header 的 Content-Security-Policy
34.	设置 meta 标签的方式 <meta http-equiv="Content-Security-Policy">

## CSRF Cross-site request forgery 跨站请求伪造
涉及面试题：什么是 CSRF 攻击？如何防范 CSRF 攻击？
原理：攻击者构造后端请求地址，诱导用户点击，如果用户处于登录态，后端会进行相应的逻辑
举个例子，网站有一个通过 GET 提交用户评论的接口，攻击者可以在钓鱼网站中加入图片，图片地址就是评论接口
**防范 CSRF 的几种规则**：
Get 请求不对数据修改
不让第三方网站访问到用户 Cookie
阻止第三方网站请求接口
请求时附带验证信息，比如验证码或者 Token

**SameSite**
对 Cookie 设置 SameSite 属性，表示 Cookie 不随着跨域请求发送

**验证 Referer**
通过验证 Referer 判断该请求是否为第三方网站发起
**Token**
服务器下发一个随机 Token，每次发起请求将 Token 携带上，服务器验证 Token

**点击劫持**
涉及面试题：什么是点击劫持？如何防范点击劫持？
点击劫持是一种视觉欺骗的攻击手段
攻击者将通过 iframe 嵌入自己的网页，并将 iframe 设置为透明，在页面中透出按钮诱导点击

**防御方法：**
X-FRAME-OPTIONS：一个 HTTP 响应头，防御用 iframe 嵌套的点击劫持攻击
对于远古浏览器来说，只有通过 JS 的方式来防御点击劫持

## 中间人攻击
涉及面试题：什么是中间人攻击？如何防范中间人攻击？
**定义**：攻击方同时与服务端和客户端建立连接，获得双方的通信信息，还能修改通信信息
不建议使用公共 Wi-Fi，因为很可能就会发生中间人攻击
**防御中间人攻击**：增加安全通道来传输信息，例如HTTPS








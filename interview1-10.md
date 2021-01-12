HTML => HTMLParser => DOMTree + Attachment => RenderTree + layout => Painting => Display
 
# 1.JS 基础知识点及常考面试题
### 原始（Primitive）类型

**涉及面试题：原始类型有哪几种？null 是对象嘛？**
6种：boolean、null、undefined、number、string、symbol
**原始类型**存储值，没有函数可以调用
'1'.toString() 的'1' 不是原始类型，而是强制转换成 String 类型(对象类型)
**number** 是浮点类型，比如 0.1 + 0.2 !== 0.3是为true
**string** 不可变，无论在 string 类型上调用何种方法，都不会对值有改变。
**typeof null** 会输出 object，错误判断

### 对象（Object）类型
**涉及面试题：对象类型和原始类型的不同之处？函数参数是对象会发生什么问题？**
原始类型存储值，对象类型存储**地址**（指针）
函数传参传递的是对象指针的副本

### typeof vs instanceof
**涉及面试题：typeof 是否能正确判断类型？instanceof 能正确判断对象的原理是什么？**
typeof 对于原始类型，除了 null 都可以显示正确类型
typeof 对于对象，除了函数都会显示 object
instanceof 可以判断对象的正确类型（通过**原型链**）
``` JavaScript
const Person = function() {}
const p1 = new Person()
p1 instanceof Person // true

var str = 'hello world'
str instanceof String // false

var str1 = new String('hello world')
str1 instanceof String // true
```

### 类型转换
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

### this
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

1.直接调用 foo ，this 是window
2.谁调用了函数，谁就是 this
3.对于 new 来说，this 绑定在 c 上面，不会被改变

**箭头函数**没有 this ，this 取决**包裹箭头函数的第一个普通函数的 this**，bind 这类函数是无效的
bind等改变上下文的函数，this 取决于第一个参数，如果为空，就是 window
如果对一个函数多次 bind，this 由**第一次 bind 决定**
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

### == vs ===
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

### 闭包
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

### 深浅拷贝
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

### 原型
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
### var、let 及 const 区别
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

### 原型继承和 Class 继承
**涉及面试题：原型如何实现继承？Class 如何实现继承？Class 本质是什么？**
 JS 中并不存在类，class 只是语法糖，本质还是函数

**组合继承**
function Parent(value) { this.val = value } Parent.prototype.getValue = function() { console.log(this.val) }function Child(value) { Parent.call(this, value) } Child.prototype = new Parent() const child = new Child(1) child.getValue() // 1 child instanceof Parent // true
在子类的构造函数中通过 Parent.call(this) 继承父类的属性，然后改变子类的原型为 new Parent() 来继承父类的函数
优点：构造函数可以传参，不会与父类引用属性共享，复用父类的函数
缺点：继承父类函数时调用了父类构造函数，导致子类的原型上多了不需要的父类属性，存在内存的浪费

**寄生组合继承**
对组合继承进行优化，组合继承缺点在于继承父类函数时调用了构造函数，优化掉这点就行
function Parent(value) { this.val = value } Parent.prototype.getValue = function() { console.log(this.val) }function Child(value) { Parent.call(this, value) } Child.prototype = Object.create(Parent.prototype, {   constructor: {     value: Child,     enumerable: false,     writable: true,    configurable: true} })const child = new Child(1) child.getValue() // 1 child instanceof Parent // true

Object.create(proto, [properties])：创建新对象，使用现有对象提供新对象的proto
既解决了无用的父类属性问题，还能正确的找到子类的构造函数

**Class 继承**
class 实现继承的核心在于使用 extends 表明继承自哪个父类，并且在子类构造函数中必须调用 super，因为这段代码可以看成 Parent.call(this, value)。

###   模块化
**涉及面试题：为什么要使用模块化？都有哪几种方式可以实现模块化，各有什么特点？**
1.解决命名冲突
2.提供复用性
3.提高代码可维护性

立即执行函数
早期使用立即执行函数实现模块化，通过函数作用域解决命名冲突、污染全局作用域的问题
(function(globalVariable){    globalVariable.test = function() {}    // ... 声明各种变量、函数都不会污染全局作用域})(globalVariable)

CommonJS
CommonJS 最早是 Node 在使用，在 Webpack 也能见到它
// a.jsmodule.exports = { a: 1 }// or  exports.a = 1// b.jsvar module = require('./a.js')module.a // -> log 1

ES Module
ES Module 是原生实现的模块化方案，与 CommonJS 有以下几个区别
CommonJS 动态导入， require(${path}/xx.js)，后者暂不支持，但已有提案
CommonJS 同步导入，因为用于服务端，文件都在本地，同步导入卡住主线程影响不大
ES Module异步导入，因为用于浏览器，需要下载文件，同步导入会对渲染有影响
CommonJS 在导出时都是值拷贝，导入的值不会改变，如果想更新值，必须重新导入一次
ES Module 采用实时绑定的方式，导入导出的值都指向同一个内存地址
ES Module 会编译成 require/exports 来执行
// 引入模块 APIimport XXX from './a.js'import { XXX } from './a.js'// 导出模块 APIexport function a() {}export default function() {}

### Proxy
**涉及面试题：Proxy 可以实现什么功能？**
Proxy 是 ES6 新增的功能，用来自定义对象中的操作
let p = new Proxy(target, handler)
target 代表添加代理的对象，handler 自定义对象中的操作，比如 set 或 get 函数
通过 Proxy 来实现一个数据响应式
let onWatch = (obj, setBind, getLogger) => {   let handler = {     get(target, property, receiver) {       getLogger(target, property)       return Reflect.get(target, property, receiver)     },     set(target, property, value, receiver) {       setBind(value, property)       return Reflect.set(target, property, value)     }   }   return new Proxy(obj, handler) }let obj = { a: 1 } let p = onWatch(   obj,   (v, property) => {     console.log(`监听到属性${property}改变为${v}`)   },   (target, property) => {     console.log(`'${property}' = ${target[property]}`)   } ) p.a = 2 // 监听到属性a改变 p.a // 'a' = 2


Vue3.0 用 Proxy 替换原本的API 原因在于 Proxy 无需一层层递归为每个属性添加代理，一次即可完成操作，性能更好，并且原本的实现有一些数据更新不能监听到，但是 Proxy 可以完美监听到任何方式的数据改变，唯一缺陷可能是浏览器的兼容性不好

### map, filter, reduce
**涉及面试题：map, filter, reduce 各自有什么作用？**
map 作用是生成新数组，遍历原数组，将每个元素拿出来做变换然后放入新数组
[1, 2, 3].map(v => v + 1) // -> [2, 3, 4]
map 的回调函数接受三个参数，分别  是当前索引元素，索引，原数组
['1','2','3'].map(parseInt)
第一轮遍历 parseInt('1', 0) -> 1
第二轮遍历 parseInt('2', 1) -> NaN
第三轮遍历 parseInt('3', 2) -> **NaN**
filter 的作用是删除数组中不需要的元素，生成新数组时将返回值为 true 的元素放入
let array = [1, 2, 4, 6]let newArray = array.filter(item => item !== 6)console.log(newArray) // [1, 2, 4]
filter 的回调函数也接受三个参数，用处也相同
reduce 可以将数组中的元素通过回调函数最终转换为一个值
实现将函数的元素全部相加得到一个值
const arr = [1, 2, 3]let total = 0for (let i = 0; i < arr.length; i++) {   total += arr[i] }console.log(total) //6 
使用 reduce 
const arr = [1, 2, 3]const sum = arr.reduce((acc, current) => acc + current, 0)console.log(sum)
reduce 接受两个参数，回调函数和初始值
分解reduce 的过程
首先初始值为 0，该值会在执行第一次回调函数时作为第一个参数传入
回调函数接受四个参数，分别为累计值、当前元素、当前索引、原数组，后三者想必大家都可以明白作用，这里着重分析第一个参数
在一次执行回调函数时，当前值和初始值相加得出结果 1，该结果会在第二次执行回调函数时当做第一个参数传入
所以在第二次执行回调函数时，相加的值就分别是 1 和 2，以此类推，循环结束后得到结果 



想必通过以上的解析大家应该明白 reduce 是如何通过回调函数将所有元素最终转换为一个值的，当然 reduce 还可以实现很多功能，接下来我们就通过 reduce 来实现 map 函数
const arr = [1, 2, 3]const mapArray = arr.map(value => value * 2)const reduceArray = arr.reduce((acc, current) => {   acc.push(current * 2)   return acc }, []) console.log(mapArray, reduceArray) // [2, 4, 6]



# 5.JS 异步编程及常考面试题
异步编程是 JS 中至关重要的内容
### 并发（concurrency）和并行（ parallelism）区别
**涉及面试题：并发与并行的区别？**
并发是宏观概念：任务 A 和任务 B在一段时间内通过任务间切换完成
并行是微观概念： CPU 存在两个核心，同时完成任务 A、B

### 回调函数（Callback）
涉及面试题：什么是回调函数？回调函数有什么缺点？如何解决回调地狱问题？
回调函数致命的弱点，容易写出回调地狱（Callback hell）
回调地狱的根本问题就是：
7.	嵌套函数存在耦合性，一旦改动就会牵一发而动全身
8.	嵌套函数一多，就很难处理错误
回调函数的缺点：不能使用 try catch 捕获错误，不能直接 return

### Generator
涉及面试题：你理解的 Generator 是什么？
首先 Generator 函数调用和普通函数不同，它会返回一个迭代器
执行next（） 传入的参数等于上一个 yield 的返回值，
如果不传参，yield 返回 undefined

### Promise
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


### async 及 await：异步终极解决方案
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

### 常用定时器函数
涉及面试题：setTimeout、setInterval、requestAnimationFrame 各有什么特点？
setInterval作用和 setTimeout 一致，只是该函数是每隔一段时间执行一次回调函数
通常不建议 setInterval，因为不能保证在预期时间执行任务，并且存在执行累积的问题
循环定时器的需求可以通过 requestAnimationFrame 实现

# 6.手写 Promise

``` JavaScript
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'
function MyPromise(fn) {
    const that = this
    that.state = PENDING  // 一开始 Promise 的状态应该是 pending
    that.value = null  // 保存 resolve 或者 reject 中传入的值
    that.resolvedCallbacks = []  // 保存 then 中的回调
    that.rejectedCallbacks = []  // 保存 then 中的回调

    function resolve(value) {
        if (that.state === PENDING) {
            that.state = RESOLVED
            that.value = value
            that.resolvedCallbacks.map(cb => cb(that.value)) //遍历回调数组并执行 
        }
    }
    function reject(value) {
        if (that.state === PENDING) {
            that.state = REJECTED
            that.value = value
            that.rejectedCallbacks.map(cb => cb(that.value))
        }
    }
    try {
      fn(resolve, reject)
    } catch (e) {
      reject(e)
    }
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
    const that = this
    // 判断两个参数是否为函数类型
    // 当参数不是函数时，创建一个函数赋值给对应参数，实现了透传
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
    onRejected = 
        typeof onRejected === 'function' ? onRejected : r => { throw r }
    
    // 如果状态是等待态的话，就往回调函数中 push 函数
    if (that.state === PENDING) {
        that.resolvedCallbacks.push(onFulfilled)
        that.rejectedCallbacks.push(onRejected)
    }
    
    // 当状态不是等待态时，就去执行相对应的函数
    if (that.state === RESOLVED) {
        onFulfilled(that.value)
    }
    if (that.state === REJECTED) {
        onRejected(that.value)
    }
}
```


# 7.Event Loop

### 进程与线程
涉及面试题：进程与线程区别？JS 单线程带来的好处？
**CPU 工作时间片的一个描述**
进程：描述了 CPU 在运行指令及加载和保存上下文所需的时间，代表一个程序
线程：进程中更小的单位，描述了执行一段指令所需的时间
浏览器中，一个 Tab 页是一个进程，一个进程中有多个线程，比如渲染线程、JS 引擎线程、HTTP 请求线程等。发起一个请求就是创建一个线程，请求结束后线程销毁

JS 运行的时候可能会阻止 UI 渲染，说明了两个线程是互斥
因为 JS 可以修改 DOM， JS 执行时 UI 线程还在工作会导致不能安全的渲染 UI
单线程的好处：可以节省内存，节约上下文切换时间，没有锁的问题


### 执行栈
涉及面试题：什么是执行栈？
执行栈：存储函数调用的栈结构，遵循先进后出的原则。

### 浏览器中的 Event Loop
涉及面试题：异步代码执行顺序？解释一下什么是 Event Loop ？
执行 JS 代码就是往执行栈中放入函数，遇到异步代码时会被挂起并在需要执行的时候加入到队列中
一旦执行栈为空，Event Loop 就从 Task 队列拿出需要执行的代码并放入执行栈中执行
所以本质上JS 的异步还是同步行为
不同的任务源会被分配到不同的 Task 队列中，任务源分为 微任务和 宏任务
ES6 规范中microtask 称为 jobs，macrotask 称为 task

``` JavaScript
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end')
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')
// script start => async2 end => Promise => script end 
// => promise1 => promise2 => async1 end => setTimeout
```
上述代码的 async 和 await 的执行顺序。
调用 async1 时，马上输出 async2 end，并返回一个 Promise
接下来在遇到 await的时候就让出线程开始执行 async1 外的代码
所以可以把 await 看成是让出线程的标志。

然后当同步代码执行完毕以后，执行异步代码，又会回到 await 的位置执行返回的 Promise 的 resolve 函数，这又会把 resolve 丢到微任务队列中
接下来去执行 then 中的回调
当两个 then 中的回调执行完毕以后，又会回到 await 的位置处理返回值
这时候可以看成是 Promise.resolve(返回值).then()
然后 await 后的代码全部被包裹进了 then 的回调中
所以 console.log('async1 end') 会优先执行于 setTimeout

如果你觉得上面这段解释还是有点绕，那么我把 async 的这两个函数改造成你一定能理解的代码

new Promise((resolve, reject) => {
  console.log('async2 end')
  // Promise.resolve() 将代码插入微任务队列尾部
  // resolve 再次插入微任务队列尾部
  resolve(Promise.resolve())
}).then(() => {
  console.log('async1 end')
})

也就是说，如果 await 后面跟着 Promise 的话，async1 end 需要等待三个 tick 才能执行到。那么其实这个性能相对来说还是略慢的，所以 V8 团队借鉴了 Node 8 中的一个 Bug，在引擎底层将三次 tick 减少到了二次 tick。但是这种做法其实是违法了规范的，当然规范也是可以更改的，这是 V8 团队的一个 PR，目前已被同意这种做法。

#### Event Loop 执行顺序如下所示：
**同步(宏),检查异步,执行微任务，下一轮loop执行异步(宏)**
执行同步代码，这属于宏任务
执行完同步代码后，执行栈为空，查询是否有异步代码需要执行
执行微任务
执行完微任务后，如有必要会渲染页面
然后开始下一轮 Event Loop，执行宏任务中的异步代码，也就是 setTimeout 中的回调函数

以上代码虽然 setTimeout 写在 Promise 之前，但是因为 Promise 属于微任务而 setTimeout 属于宏任务，所以会有以上的打印。

微任务包括 process.nextTick ，promise ，MutationObserver。
宏任务包括 script ， setTimeout ，setInterval ，setImmediate ，I/O ，UI rendering

这里很多人会有个误区，认为微任务快于宏任务，其实是错误的。因为宏任务包括了 script，浏览器会先执行一个宏任务，接下来有异步代码的话才会先执行微任务。

### Node 中的 Event Loop


# 8.JS 进阶知识点及常考面试题
### 手写 call、apply 及 bind 函数
**涉及面试题：call、apply 及 bind 函数内部实现是怎么样的？**
**用法**：都是用于改变函数体内this的指向
**bind**: 返回新函数, this指向第一个参数obj，后面的参数传入原函数
`var testObj = test.bind(obj, ...);`
使用new 操作符调用绑定函数时，该参数无效
**apply和call的调用返回函数执行结果**
**apply**：
`fun.apply(thisArg, [argsArray])`
**call**：
`fun.call(thisArg, arg1, arg2, ...)`

**总结**
如果传递的参数不多，使用`fn.call(thisObj, arg1, arg2 ...)`
如果传递的参数很多，调用`fn.apply(thisObj, [arg1, arg2 ...])`
如果新函数长期绑定某个函数给某个对象使用，则使用
`const newFn = fn.bind(thisObj);`
`newFn(arg1, arg2...)`

#### call
不传入第一个参数，那么上下文默认为 window
改变了 this 指向，让新的对象可以执行该函数，并能接受参数
``` JavaScript
Function.prototype.myCall = function(context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  const args = [...arguments].slice(1)
  const result = context.fn(...args)
  delete context.fn
  return result
}
```
首先 context 为可选参数，如果不传的话默认上下文为 window
接下来给 context 创建 fn 属性，并将值设置为需要调用的函数
因为 call 可以传入多个参数作为调用函数的参数，所以需要将参数剥离出来
然后调用函数并将对象上的函数删除
以上就是实现 call 的思路，apply 的实现也类似，区别在于对参数的处理，所以就不一一分析思路了

#### apply
``` JavaScript
Function.prototype.myApply = function(context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  let result
  // 处理参数和 call 有区别
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

#### bind
bind 的实现对比其他两个函数略微地复杂了一点，因为 bind 需要返回一个函数，需要判断一些边界问题
``` JavaScript
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  const _this = this
  const args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}

```
以下是对实现的分析：
前几步和之前的实现差不多，就不赘述了
bind 返回了一个函数，对于函数来说有两种方式调用，一种是直接调用，一种是通过 new 的方式，我们先来说直接调用的方式
对于直接调用来说，这里选择了 apply 的方式实现，但是对于参数需要注意以下情况：因为 bind 可以实现类似这样的代码 f.bind(obj, 1)(2)，所以我们需要将两边的参数拼接起来，于是就有了这样的实现 args.concat(...arguments)
最后来说通过 new 的方式，在之前的章节中我们学习过如何判断 this，对于 new 的情况来说，不会被任何方式改变 this，所以对于这种情况我们需要忽略传入的 this

###  new
涉及面试题：new 的原理是什么？通过 new 的方式创建对象和通过字面量创建有什么区别？
调用 new 的过程：生成对象、链接到原型、绑定 this、返回对象
``` JavaScript
function create() {   
    // 1.create empty obj   
    let obj = {}    
    // 2.get Constructor   
    let Con = [].shift.call(arguments)  // arguments: array[func]   
    // 3.set prototype of empty obj 
    obj.__proto__ = Con.prototype
    // 4.bind "this" & call Constructor   
    let result = Con.apply(obj, arguments)
    // 5.determine wherther the value of return is Object   
    return result instanceof Object ? result : obj }
```
对于创建对象来说，更推荐使用字面量的方式创建对象（无论性能上还是可读性）
因为使用 new Object() 创建对象需要通过作用域链一层层找到 Object，但使用字面量的方式就没这个问题

###  instanceof 的原理
instanceof 可以判断对象类型，内部机制是通过判断对象原型链中是不是能找到类型的prototype
``` JavaScript
function myInstanceof(left, right) {   
    let prototype = right.prototype   
    left = left.__proto__   
    while (true) {     
        if (left === null || left === undefined)       
            return false     
        if (prototype === left)       
            return trueleft = left.__proto__   
    } 
}
```

###  为什么 0.1 + 0.2 != 0.3
涉及面试题：为什么 0.1 + 0.2 != 0.3？如何解决这个问题？
因为0.1 在二进制中无限循环，JS采用的浮点数标准会裁剪掉这种数字，造成**精度丢失**
**解决方法**：parseFloat((0.1 + 0.2).toFixed(10)) === 0.3 // true


### 垃圾回收机制  Garbage collection
涉及面试题：V8 下的垃圾回收机制是怎么样的？
V8 实现了准确式 GC，GC 算法采用了分代式GC机制，因此，V8 将内存（堆）分为新生代和老生代两部分

**新生代算法**
特点是对象存活短，新生代的空间分为 From 和 To ，一个使用，另一个空闲
新对象被放入 From，当 From 被占满时，GC启动，检查 From 存活的对象，并复制到 To 中，失活对象销毁
当复制完成后将 From 和 To 互换，GC结束

**老生代算法**
存活时间长且数量多，使用了两个算法，分别是标记清除算法和标记压缩算法
什么情况下对象会出现在老生代空间中：
新生代的对象已经经历过一次 Scavenge 算法
To 的对象占比大小超过 25 %

# 10.JS 思考题
### 思考题一：JS 分为哪两大类型？都有什么各自的特点？你该如何判断正确的类型？
1.对于原始类型来说，可以指出 null 和 number 存在的问题。
对于对象类型来说，可以从垃圾回收的角度去切入，也可以说一下对象类型存在深浅拷贝的问题

2.对于判断类型来说，可以去对比 typeof 和 instanceof 的区别
可以指出 instanceof 判断类型不是完全准确。

### 思考题二：你理解的原型是什么？
起码说出原型小节中的总结内容，然后指出一些小点，比如不是所有函数都有 prototype 属性
然后引申出原型链的概念，提出如何使用原型实现继承，继而可以引申出 ES6 中的 class 实现继承

### 思考题三：bind、call 和 apply 各自有什么区别？
首先说出三者的不同，如果自己实现过其中的函数，可以说出自己的思路
然后聊一聊 this 的内容，有几种规则判断 this 到底是什么，this 规则会涉及到 new，那么最后可以说下自己对于 new 的理解

### 思考题四：ES6 中有使用过什么？
思路引导：
比如说说 class，那么 class 又可以拉回到原型的问题
可以说说 promise，那么线就被拉到了异步的内容
可以说说 proxy，那么如果你使用过 Vue 这个框架，就可以谈谈响应式原理的内容
同样可以说说 let 这些声明变量的语法，可以谈及与 var 的不同，说到提升的内容

### 思考题五：JS 是如何运行的？
可以先说 JS 是单线程运行，说说你理解的线程和进程的区别。
然后讲到执行栈，接下来的内容就是涉及 Eventloop 了，微任务和宏任务的区别，哪些是微任务，哪些又是宏任务，还可以谈及浏览器和 Node 的 Eventloop 的不同，最后聊一聊 JS 中的垃圾回收



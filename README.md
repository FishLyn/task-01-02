# Part1 模块二作业

## 简答题

<b>1、描述引用计数的工作原理和优缺点</b>

答：工作原理，在程序开始执行的时候，会设置引用数。当数据被引用的时候，引用数会增加，取消引用的时候，引用数减少，当引用数为0时，会被 GC 垃圾回收给清除

​		优点：当发现垃圾的时候，会立即被回收；当内存即将被占满的时候，才会启动垃圾回收，能有效减少程序的停顿

​		缺点：不能回收循环引用的对象；回收花费的时间长

```js
// 详细说明循环引用
function data () {
    const obj1 = {}
    const obj2 = {}
    obj1.name = obj2
    obj2.name = obj1
    
    return 'success'
}
data()
// 函数执行完之后，函数内部定义的变量都应该被回收，但是如果根据引用计数来回收的话，在回收obj1的时候会发现此时obj1被obj2引用了，因此不会回收obj1
```



<b>2、描述标记整理算法的工作流程</b>

答：第一步：和标记清除一致，都会找到所有的活动对象并标记

​		第二步：将标记的所有活动对象的位置整理到一块，将整个内存分为活动对象区和非活动对象区

​		第三步：清除所有非活动对象区的数据



<b>3、描述 V8 中新生代存储区垃圾回收的流程</b>

答：新生代存储区又分为相同大小的两个区域，使用空间 From 和 空闲空间 To

​		当 From 空间存储的数据达到一定量的时候，会先用标记整理整理好内存空间；

​		然后将所有活动对象通过复制算法拷贝到 To 空间；

​		最后清空 From 空间的所有数据，并 将 From 空间 和 To 空间进行交换，也就是使用空间变成空闲状态，空闲空间变成使用状态

​		PS：数据在拷贝过程中会发生晋升，新生代的数据会被移动到老生代区



<b>4、描述增量标记算法在何时使用，及工作原理</b>

答：增量标记算法会在程序执行的过程中穿插执行，当增量标记算法执行的时候，程序会暂停运行，等待这一轮的算法执行完之后才会继续执行

​		增量标记算法不会一次性将所有工作做完，会在程序执行的过程中，一步一步，多次执行标记工作，等程序快执行完的时候，才一次性将所有未被标记的数据清除



## 代码题1

基于以下代码完成下面的四个练习

```js
const fp = require('lodash/fp')

// 数据
// horsepower 马力，dollar_value 价格，in_stock 库存
const cars = [
    {name: "Ferrari FF", horsepower: 660, dollar_value: 700000, in_stock: true},
    {name: "Spyker C12 Zagato", horsepower: 650, dollar_value: 648000, in_stock: false},
    {name: "Jaguar XKR-S", horsepower: 550, dollar_value: 132000, in_stock: false},
    {name: "Audi R8", horsepower: 525, dollar_value: 114200, in_stock: false},
    {name: "Aston Martin One-77", horsepower: 750, dollar_value: 1850000, in_stock: true},
    {name: "Pagani Huayra", horsepower: 700, dollar_value: 1300000, in_stock: false},
]
```



<b>1、使用函数组合 fp.flowRight() 重新实现下面这个函数</b>

```js
let isLastInStock = function (cars) {
    // 获取最后一条数据
    let last_car = fp.last(cars)
    // 获取最后一条数据的 in_stock 属性值
    return fp.prop('in_stock', last_car)
}
```

答：分析：我们需要实现两个功能，1、获取数组中最后一个数据；2、获取数组中最后一条数据的 in_stock 属性值

​		因此，可以创建两个函数，fn1(arr) 返回数组的最后一条数据，fn2(obj) 传入上个函数返回的对象，并返回对象的 in_stock 属性值，所以可以用函数组合的方式来实现

```js
// 创建获取数组最后一条数据的函数
function getLast (arr) {
    return fp.last(arr)
}
// 创建获取对象 in_stock 属性值的函数
function getInStock (obj) {
    return fp.prop('in_stock', obj)
}

const res = fp.flowRight(getInStock, getLast)
console.log(res(cars)) // false
```



<b>2、使用 fp.flowRight()，fp.prop()，fp.first() 获取第一个 car 的 name</b>

答：分析：获取数组第一条数据对象的属性值，可以先用 fp.first() 获取第一条数据，再用 fp.prop() 获取属性值，最后再用 fp.flowRight() 将之前两个函数组合起来

```js
// 获取数组第一条数据
function getFirst (arr) {
    return fp.first(arr)
}
// 获取对象的属性值
function getInStock (obj) {
    return fp.prop('name', obj)
}

const res = fp.flowRight(getInStock, getFirst)
console.log(res(cars)) // Ferrari FF
```



<b>3、使用帮助函数 _average 重构 averageDollarValue，使用函数组合的方式实现</b>

```js
let _average = function (xs) {
    return fp.reduce(fp.add, 0, xs) / xs.length
} // <- 无须改动

let averageDollarValue = function (cars) {
    let dollar_values = fp.map(function (car) {
        return car.dollar_value
    }, cars)
    return _average(dollar_values)
}
```

答：分析：从代码执行能看出这个是求所有汽车的平均价格的需求

​		1、遍历数组，并将所有汽车的价格存储到数组中

​		2、使用函数 _average 计算上述数组中所有数据的平均值

```js
// 创建计算数组内所有数据的平均值的函数
let _average = function (xs) {
    return fp.reduce(fp.add, 0, xs) / xs.length
} // <- 无须改动
// 创建遍历数组并返回所有价格的数组
function dollarValues (arr) {
    return fp.map(item => {
        return item.dollar_value
    }, arr)
}
const res = fp.flowRight(_average, dollarValues)
console.log(res(cars)) // 790700
```



<b>4、使用 flowRight 写一个 sanitizeNames() 函数，返回一个下划线连接的小写字符串，把数组中的 name 转换为这种形式：例如：sanitizeNames(["Hello World"]) => ["hello_world"]</b>

```js
let _underscore = fp.replace(/\W+/g, '_') // <-- 无须改动，并在 sanitizeNames 中使用它
```

答：分析：第一步将字符串的所有空格替换为下划线，第二步，将所有字符改成小写

```js
let _underscore = fp.replace(/\W+/g, '_') // <-- 无须改动，并在 sanitizeNames 中使用它

function sanitizeNames (arr) {
    return fp.map(fp.flowRight(fp.toLower, _underscore), arr)
}
console.log(sanitizeNames(["Hello World"])) // ["hello_world"]
```



## 代码题2

基于下面提供的代码，完成后续的四个练习

```js
// support.js
class Container {
    static of (value) {
        return new Container(value)
    }
    
    constructor (value) {
        this._value = value
    }
    
    map (fn) {
        return Container.of(fn(this._value))
    }
}
class Maybe {
    static of (x) {
        return new Maybe(x)
    }
    isNothing () {
        return this._value === null || this._value === undefined
    }
    constructor (x) {
        this._value = x
    }
    map (fn) {
        return this.isNothing() ? this : Maybe.of(fn(this._value))
    }
}

module.exports = {
    Maybe,
    Container
}
```



<b>1、使用 fp.add(x, y) 和 fp.map(f, x) 创建一个能让 functor 里的值增加的函数 ex1</b>

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let maybe = Maybe.of([5, 6, 11])
let ex1 = // ...你需要实现的位置
```

答：分析：现有一个数组，需要给数组中值都增加 y，因此有两个变量，可以采用柯里化的方式，将传入数组参数封装成函数并返回，调用的时候再传入需要增加的数值

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let maybe = Maybe.of([5, 6, 11])
let ex1 = y => arr => fp.map(item => fp.add(item, y), arr)
console.log(maybe.map(ex1(2))) // Maybe { _value: [ 7, 8, 13 ] }
```



<b>2、实现一个函数 ex2，能够使用 fp.first 获取列表的第一个元素</b>

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let xs = Container.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do'])
let ex2 = // ... 你需要实现的位置
```

答：分析：Container.of 已经生成了一个 Container实例对象，所以可以直接用法 xs._value获取传入的数组的值，再用 fp.first() 就能获取数组的第一个值了

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let xs = Container.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do'])
let ex2 = arr => fp.first(xs._value)
console.log(ex2()) // do
```



<b>3、实现一个函数 ex3，使用 safeProp 和 fp.first 找到 user 的名字的首字母</b>

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let safeProp = fp.curry(function (x, o) {
    return Maybe.of(o[x])
})
let user = { id: 2, name: 'Albert' }
let ex3 = // ... 你需要实现的位置
```

答：分析：需要传入两个参数，对象和属性名，所以仍然用柯里化将对象先封装一层，调用的时候再传入属性名

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let safeProp = fp.curry(function (x, o) {
    return Maybe.of(o[x])
})
let user = { id: 2, name: 'Albert' }
let ex3 = obj => prototype => fp.first(safeProp(prototype, obj)._value)
const userObj = ex3(user)
console.log(userObj('name')) // A
```



<b>4、使用 Maybe 重写 ex4，不要有 if 语句</b>

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let ex4 = function (n) {
    if (n) { return parseInt(n) }
}
```

答：分析：ex4 用的是 if 来判断 n 的值是否为true，我们可以使用 Maybe 函子来判断是否是空值

```js
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let ex4 = n =>  Maybe.of(n).map(parseInt)._value

console.log(ex4(10.2)) // 10
console.log(ex4(null)) // null
console.log(ex4(undefined)) // undefined
```














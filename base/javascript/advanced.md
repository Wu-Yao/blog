# 进阶

## async/await

> 什么是 async/await

一句话，它就是 Generator 函数的语法糖，ES7 提出的 async 函数，终于让 JavaScript 对于异步操作有了终极解决方案，async 函数是 Generator 函数的语法糖，使用 关键字 async 来表示，在函数内部使用 await 来表示异步，想较于 Generator，async 函数的改进在于下面四点：

- 更好的语义:async 和 await 相较于 \* 和 yield 更加语义化, async 是异步的简写， async function 声明一个异步函数，await 可以理解为 async wait（异步等待），用于等待一个异步方法的执行；
- 返回值是 Promise。async 函数返回值是 Promise 对象，比 Generator 函数返回的 Iterator 对象方便，可以直接使用 then() 方法进行调用；

> 语法

<b>async</b> 函数返回一个 Promise 对象

<b>async</b> 函数内部 <b>return</b> 语句返回的值，会成为 <b>then</b> 方法回调函数的参数。

```
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"


```

上面代码中，函数 <b>f</b> 内部 <b>return</b> 命令返回的值，会被 <b>then</b> 方法回调函数接收到。

<b>async</b> 函数内部抛出错误，会导致返回的 Promise 对象变为 <b>reject</b> 状态。抛出的错误对象会被 <b>catch</b> 方法回调函数接收到。

```
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log('resolve', v),
  e => console.log('reject', e)
)
//reject Error: 出错了
```

> Promise 对象的状态变化

<b>async</b> 函数返回的 Promise 对象，必须等到内部所有 <b>await</b> 命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到 <b>return</b> 语句或者抛出错误。也就是说，只有 <b>async</b> 函数内部的异步操作执行完，才会执行 <b>then</b> 方法指定的回调函数。

```
const delay = timeout => new Promise(resolve=> setTimeout(resolve, timeout));
async function f(){
    await delay(1000);
    await delay(1000);
    await delay(1000);
    return 'done';
}

f().then(v => console.log(v)); // 等待3s后才输出 'done'
```

> await 命令

正常情况下，<b>await</b> 命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。

```
async function f() {
  // 等同于
  // return 123;
  return await 123;
}

f().then(v => console.log(v))
// 123
```

> 错误处理

任何一个 <b>await</b> 语句后面的 Promise 对象变为 <b>reject</b> 状态，那么整个 <b>async</b> 函数都会中断执行。

```
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
```

上面代码中，第二个 <b>await</b> 语句是不会执行的，因为第一个 <b>await</b> 语句状态变成了 <b>reject</b>

有时，我们希望即使前一个异步操作失败，也不要中断后面的异步操作。这时可以将第一个 await 放在 <b>try...catch</b> 结构里面，这样不管这个异步操作是否成功，第二个 <b>await</b> 都会执行

```
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world
```

> 深入理解

理解 <b>async</b> 函数需要先理解 <b>Generator</b> 函数，因为 <b>async</b> 函数是 <b>Generator</b> 函数的语法糖。

<b>Generator</b> 函数有多种理解角度。语法上，首先可以把它理解成， <b>Generator</b> 函数是一个状态机，封装了多个内部状态。

执行 <b>Generator</b> 函数会返回一个遍历器对象，也就是说， <b>Generator</b> 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 <b>Generator</b> 函数内部的每一个状态。

形式上， <b>Generator</b> 函数是一个普通函数，但是有两个特征。一是，<b>function</b> 关键字与函数名之间有一个星号；二是，函数体内部使用 <b>yield</b> 表达式，定义不同的内部状态（yield 在英语里的意思就是“产出”）。

```
function *gen() {
    yield 1;
    yield 2;
    return 3;
}

const it = gen();

console.log(it.next()); // { value: 1, done: false }
console.log(it.next()); // { value: 2, done: false }
console.log(it.next()); // { value: 3, done: false }
console.log(it.next()); // { value: undefined, done: true }
console.log(it.next()); // { value: undefined, done: true }

```

<b>Generator</b> 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 <b>Generator</b> 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象;

下一步，必须调用遍历器对象的 <b>next</b> 方法，使得指针移向下一个状态。也就是说，每次调用 <b>next</b> 方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个 <b>yield</b> 表达式（或 <b>return</b> 语句）为止

在执行 <b>next</b> 方法后，顺序执行了 <b>yield</b> 的返回值。返回值有 value 和 done 两个状态。value 为返回值，可以是任意类型。done 的状态为 false 和 true，true 即为执行完毕。在执行完毕后再次调用返回{value: undefined, done: true} 注意:在遇到 return 的时候，所有剩下的 yield 不再执行，直接返回{ value: undefined, done: true }

> for...of 循环

for...of 循环可以自动遍历 Generator 函数运行时生成的 Iterator 对象，且此时不再需要调用 next 方法。

```
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

> 手动实现一个 async/await

```

function co(gen) {

    // 初始化迭代器
    let it = gen();
    return new Promise(function (resolve, reject) {
        // 递归调用next，确保yeild后面的内容依次全部执行
        !function next(lastVal) {
            let { value, done } = it.next(lastVal);
            if (done) {
                resolve(value);
            } else {
            // 每次yield 的值，会作为next函数的参数
            // 所以上面的例子 let template = yield readFile('./template.txt') ，template就是yeild表达式返回的值
                value.then(next, reason => reject(reason));
            }
        }();
    });
}

// 所需要执行的Generator函数，内部的数据在执行完成一步的promise之后，再调用下一步
var func = function* (){
  var f1 = yield new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(num+1)
        }, 1000)
    })
  var f2 = yield new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(num+1)
        }, 1000)
    })
  console.log(f2) ;
};

co(func);
```

## Promise 原理解析

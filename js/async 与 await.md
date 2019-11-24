`文章摘自： https://segmentfault.com/a/1190000007535316 `

## 1. async与await

### 1.1 async与await是什么

> 1. async 用于申明一个 function 是异步的
>
> 2. await 用于等待一个异步方法执行完成 

### 1.2 async起什么左右

>1.   async 函数（包含函数语句、函数表达式、Lambda表达式）会**返回一个 Promise** 对象，如果在函数中 `return` 一个直接量，async 会把这个直接量通过 `Promise.resolve()` 封装成 Promise 对象。 

```javascript
async function testAsync() {
    return "hello async";
}
const result = testAsync();
console.log(result);

c:\var\test> node --harmony_async_await .
Promise { 'hello async' }
```

> 2. async 函数如果没有返回值，则会返回 `Promise.resolve(undefined)`。 
> 3.  Promise的特点 - 无等待，所以在没有 `await`的情况下执行 async 函数，他会立即执行，返回一个 Promise 对象，并且不会阻塞后面的语句，这和普通函数返回 Promise 对象的函数并无二致

### 1.3 await 到底在等待什么

> 1. await 等待的是一个表达式，这个表达式的计算结果是 Promise 对象或者其他值，并没有特殊的限定。
> 2. async 函数返回一个 Promise 对象，所以 await 可以用于等待一个 async 函数的返回值 
> 3. await 是可以接收普通函数调用或者直接量的。所以下面示例也是完全正确的。

```javascript
function getSomething() {
    return "something";
}

async function testAsync() {
    return Promise.resolve("hello async");
}

async function test() {
    const v1 = await getSomething();
    const v2 = await testAsync();
    console.log(v1, v2);
}

test();
```

### 1.4 await 等到要等的 然后呢

> 1. `await`是个运算符，用于组成表达式，await 表达式的运算结果取决于它等待的东西
> 2. 如果等待的**不是**一个 Promise 对象，那 await 表达式的运算结果解释它等到的东西
> 3. 如果等待的**是**一个 Promise 对象，await 会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。
> 4. 但是当 `await 在 async 中`的时候，async 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

### 1.5 async/await 帮我们干了什么

>  async 会将其后的函数（函数表达式或 Lambda）的返回值封装成一个 Promise 对象，而 await 会等待这个 Promise 完成，并将其 resolve 的结果返回出来 

### 1.6 示例

1.  用 `setTimeout` 模拟耗时的异步操作，先来看看不用 async/await 会怎么写 

   ```javascript
   function takeLongTime() {
       return new Promise(resolve => {
           setTimeout(() => resolve("long_time_value"), 1000);
       });
   }
   
   takeLongTime().then(v => {
       console.log("got", v);
   });
   
   // 改用 async/await 
   
   function takeLongTime() {
       return new Promise(resolve => {
           setTimeout(() => resolve("long_time_value"), 1000);
       });
   }
   
   async function test() {
       const v = await takeLongTime();
       console.log(v);
   }
   ```

   > `takeLongTime()` 没有申明为 `async`。实际上，`takeLongTime()` 本身就是返回的 Promise 对象，加不加 `async` 结果都一样，如果没明白，请回过头再去看看上面的“async 起什么作用” 

### 1.7 async/await 的优势在于处理 `then `链

 假设一个业务，分多个步骤完成，每个步骤都是**异步**的，而且**依赖于上一个步骤**的结果。我们仍然用 `setTimeout` 来模拟异步操作： 

```javascript
/**
 * 传入参数 n，表示这个函数执行的时间（毫秒）
 * 执行的结果是 n + 200，这个值将用于下一步骤
 */
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}

// 使用Promise 方式来实现这三个步骤的处理
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}
doIt();

// 使用async/await
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}
doIt();

// c:\var\test>node --harmony_async_await .
// step1 with 300
// step2 with 800 = 300 + 500
// step3 with 1800 = 300 + 500 + 1000
// result is 2000
// doIt: 2907.387ms
```

 输出结果 `result` 是 `step3()` 的参数 `700 + 200` = `900`。`doIt()` 顺序执行了三个步骤，一共用了 `300 + 500 + 700 = 1500` 毫秒，和 `console.time()/console.timeEnd()` 计算的结果一致。 


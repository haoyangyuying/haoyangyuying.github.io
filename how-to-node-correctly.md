## How to learn node correctly

### 异步流程控制
- Node.js是为异步而生的，让Node.js从DIRT（数据敏感实时应用）扩展到更多的场景

#### 1) 异步流程控制学习重点
- Promise是必须会的， 推荐使用Async函数+Promise组合
- Node.jsSDK里面的callback写法是必须会的
- Node.js学习重点，中流砥柱是Promise, 终极解决方案是Async/Await

#### 2）Api写法：Error-first Callback 和 EventEmitter
- Error-first Callback 定义错误优先的回调写法只需要注意2条规则即可：
  - 回调函数的第一个参数返回的error对象，如果error发生了，它会作为第一个err参数返回，如果没有，一般做法是返回null。

  ```js
  function(err, res) {
    // process the error and result
  }
  ```
  - 回调函数的第二个参数返回的是任何成功响应的结果数据。如果结果正常，没有error发生，err会被设置为null，并在第二个参数就出返回成功结果数据
- EventEmitter
  - 事件模块是 Node.js 内置的对观察者模式“发布/订阅”（publish/subscribe）的实现，通过EventEmitter属性，提供了一个构造函数。该构造函数的实例具有 on 方法，可以用来监听指定事件，并触发回调函数。任意对象都可以发布指定事件，被 EventEmitter 实例的 on 方法监听到。

  ```js
    var EventEmitter = require('events')
    var util = require('util')

    var MyEmitter = function() {

    }

    util.inherits(MyEmitter, EventEmitter)

    const MyEmitter = new MyEmitter();

    myEmitter.on('event', (a, b) => {
      console.log(a, b, this);
    });

    myEmitter.emit('event', 'a', 'b');
  ```
- 如何更好的查看Node.js文档
  - Node.js异步原理，核心在于Node.js SDK中API调用，然后交由EventLoop去执行
  - 用Dash查看离线文档

#### 3）中流砥柱：Promise
- promise是对回调地狱的思考或者改良方案，错误优先的回调方式，promise也是由commonjs社区提出来的
- promise意味着还没有完成的操作，但是在未来会完成。与promise最主要的交互方式是将函数传入它的then方法从而获得Promise最终的值或者Promise最终拒绝
  - 递归，每个异步操作返回的都是promise对象
  - 状态机，三种状态的转换，只在promise对象内部可以控制，外部不能改变状态
  - 全局异常处理
- 定义

```js
var promise = new Promise(function(resolve, reject) {
  if (/* everything turned out fine*/) {
    resolve("stuff worked!")
  }
  else {
    reject((Error("It broke"));
  }
})

```

- 调用

```js
promise.then(function(text){
  console.log(text)
  return Promise.reject(new Error('error occurred'))
}).catch(function(err){
  console.log(err)
})
```

- Promise 的标准化, 主要交互方式是通过then函数，如果Promise成功执行resolve了，就会传给最近的then函数，如果出错reject，那就交给catch来捕获异常

#### 4）终极解决方案：Async/Await
- Async/Await是异步操作的终极解决方案
  - 通过await Student.getAllAsync();来获取所有的students信息。
  - 通过await ctx.render渲染页面
  - 由于是同步代码，使用try/catch做的异常处理

```js
exports.list = async (ctx, next) => {
  try {
    let students = await Student.getAllAsync();

    await ctx.render('students/index', {
      students: students
    })
  } catch (err) {
    return ctx.api_error(err);
  }
}
```

- 正常写法

```js
const pkgConf = require('pkg-conf');

async function main(){
    const config = await pkgConf('unicorn');

    console.log(config.rainbow);
    //=> true
}

main();
```

- await + Promise

```js
const Promise = require('bluebird');
const fs = Promise.promisifyAll(require("fs"));

async function main() {
  const contents = await fs.readFileAsync('myfile.js', 'utf8')
  console.log(contents);
}

main();
```

- await + co + generator

```js
const co = require('co');
const Promise = require('bluebird');
const fs = Promise.promisifyAll(require("fs"));

async function main() {
  const contents = co(function* () {
    var result = yield fs.readFileAsync("myfile.js", "utf8")
    return result;
  })

  console.log(contents);
}
main();
```

- 要点
  - co的返回值是promise，所以await可以直接接co。
  - co的参数是genrator
  - 在generator里可以使用yield，而yield后面接的有5种可能，故而把这些可以yield接的方式成为yieldable，即可以yield接的。
    - Promises
    - Thunks (functions)
    - array (parallel execution)
    - objects (parallel execution)
    - Generators 和 GeneratorFunctions
- 由上面3中基本用法可以推出Async函数要点如下：
  - Async函数语义上非常好
  - Async不需要执行器，它本身具备执行能力，不像Generator需要co模块
  - Async函数的异常处理采用try/catch和Promise的错误处理，非常强大
  - Await接Promise，Promise自身就足够应对所有流程了，包括async函数没有纯并行处理机制，也可以采用Promise里的all和race来补齐
  - Await释放Promise的组合能力，外加co和Promise的then，几乎没有不支持的场景
- 综上所述
  - Async函数是趋势，如果Chrome 52. v8 5.1已经支持Async函数 ( https://github.com/nodejs/CTC/issues/7 )了，Node.js支持还会远么？
  - Async和Generator函数里都支持promise，所以promise是必须会的。
  - Generator和yield异常强大，不过不会成为主流，所以学会基本用法和promise就好了，没必要所有的都必须会。
  - co作为Generator执行器是不错的，它更好的是当做Promise 包装器，通过Generator支持yieldable，最后返回Promise，是不是有点无耻？
- 彩蛋
  - 技术总监的时候看技术能力和态度，CTO考虑团队文化，人品和能否在公司长流
  - 基本的Node.js几个特性，比如事件驱动、非阻塞I/O、Stream等
  - 异步流程控制相关，Promise是必问的
  - 掌握1种以上Web框架，比如Express、Koa、Thinkjs、Restfy、Hapi等，会问遇到过哪些问题、以及前端优化等常识
  - 数据库相关，尤其是SQL、缓存、Mongodb等
  - 对于常见Node.js模块、工具的使用，观察一个人是否爱学习、折腾
  - 是否熟悉linux，是否独立部署过服务器，有+分
  - js语法和es6、es7，延伸CoffeeScript、TypeScript等，看看你是否关注新技术，有+分
  - 对前端是否了解，有+分
  - 是否参与过或写过开源项目，技术博客、有+分

## REFERENCES
- [i5ting-how-to-learn-node-correctly](https://i5ting.github.io/How-to-learn-node-correctly/#10306)

[back](./)

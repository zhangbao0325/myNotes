# 38｜异步处理：Future是什么？它和async/await是什么关系？

![image-20240131110813374](/Users/mochen/Library/Application Support/typora-user-images/image-20240131110813374.png)

其实 Rust 的 Future 跟 JavaScript 的 Promise 非常类似。如果你熟悉 JavaScript，应该熟悉 Promise 的概念，02也简单讲过，它代表了在未来的某个时刻才能得到的结果的值，

Promise 一般存在三个状态；

- 初始状态，Promise 还未运行；
- 等待（pending）状态，Promise 已运行，但还未结束；
- 结束状态，Promise 成功解析出一个值，或者执行失败。

==只不过 JavaScript 的 Promise 和线程类似，一旦创建就开始执行，对 Promise await 只是为了“等待”并获取解析出来的值；====而 Rust 的 Future，只有在主动 await 后才开始执行。==

讲到这里估计你也看出来了，谈 Future 的时候，我们总会谈到 async/await。一般而言，async 定义了一个可以并发执行的任务，而 await 则触发这个任务并发执行。大多数语言，包括 Rust，async/await 都是一个语法糖（syntactic sugar），它们使用状态机将 Promise/Future 这样的结构包装起来进行处理。
# deno_core源码

## core

```js
  function opAsync(name, ...args) {    
    const id = nextPromiseId++;
    try {
      const maybeResult = asyncOps[name](id, ...new SafeArrayIterator(args));

      if (maybeResult !== undefined) {
        movePromise(id);
        return unwrapOpResultNewPromise(id, maybeResult, opAsync);
      }
    } catch (err) {
      movePromise(id);
      if (!ReflectHas(asyncOps, name)) {
        return PromiseReject(new TypeError(`${name} is not a registered op`));
      }
      ErrorCaptureStackTrace(err, opAsync);
      return PromiseReject(err);
    }
    let promise = PromisePrototypeThen(
      setPromise(id),
      unwrapOpError(eventLoopTick),
    );
    promise = handleOpCallTracing(name, id, promise);
    promise[promiseIdSymbol] = id;
    return promise;
  }
```

1. `function opAsync(name, ...args) { ... }`：定义了一个名为 `opAsync` 的函数，接受一个名称 `name` 和一系列参数 `args`。
2. `const id = nextPromiseId++;`：生成一个唯一的异步操作标识符 `id`，并递增 `nextPromiseId`。
3. `const maybeResult = asyncOps[name](id, ...new SafeArrayIterator(args));`：调用名为 `name` 的异步操作，并传入 `id` 和参数列表。`asyncOps` 是一个包含异步操作的对象，根据传入的 `name` 找到对应的异步操作函数，并调用它。`SafeArrayIterator` 是一个安全的迭代器，用于处理参数列表。
4. `if (maybeResult !== undefined) { ... }`：如果异步操作返回的结果不为 `undefined`，则执行以下代码。这意味着异步操作已经同步完成，并且直接返回了结果。
5. `movePromise(id);`：将具有标识符 `id` 的异步操作移动到已完成的 Promise 列表中。
6. `return unwrapOpResultNewPromise(id, maybeResult, opAsync);`：调用 `unwrapOpResultNewPromise` 函数，将异步操作的结果包装成一个新的 Promise，并返回。
7. `catch (err) { ... }`：如果异步操作抛出了异常，执行以下代码。
8. `movePromise(id);`：将具有标识符 `id` 的异步操作移动到已完成的 Promise 列表中。
9. `if (!ReflectHas(asyncOps, name)) { ... }`：检查 `asyncOps` 中是否存在名为 `name` 的异步操作。如果不存在，则返回一个拒绝的 Promise，表示未注册的异步操作。
10. `ErrorCaptureStackTrace(err, opAsync);`：捕获异常的堆栈信息，以便进行调试。
11. `return PromiseReject(err);`：返回一个拒绝的 Promise，将捕获到的异常作为拒因。
12. `let promise = PromisePrototypeThen( ... );`：创建一个新的 Promise，通过 `PromisePrototypeThen` 方法将其与异步操作的结果或错误进行关联。
13. `promise = handleOpCallTracing(name, id, promise);`：进行异步操作调用的跟踪处理，可能是记录日志、性能统计等操作。
14. `promise[promiseIdSymbol] = id;`：将 Promise 对象的 `promiseIdSymbol` 属性设置为 `id`，用于标识该 Promise。
15. `return promise;`：返回处理好的 Promise 对象。

总体来说，这段代码实现了一个异步操作的处理函数，它会调用异步操作并返回一个 Promise 对象，用于异步操作的结果或错误处理。

`PromisePrototypeThen` 是 JavaScript 中 Promise 对象的原型方法 `then` 的另一种调用方式。它的作用是创建一个新的 Promise 对象，并指定当原始 Promise 对象状态改变时，要执行的回调函数。

具体来说，`PromisePrototypeThen` 接受两个参数：`onFulfilled` 和 `onRejected`。这两个参数分别是成功状态和失败状态的回调函数。

在给定的代码中，`PromisePrototypeThen` 的作用是创建一个新的 Promise 对象，将 `setPromise(id)` 的结果作为成功状态的回调函数，将 `unwrapOpError(eventLoopTick)` 的结果作为失败状态的回调函数。这样就可以在异步操作完成后，根据成功或失败的情况执行相应的操作。

总之，`PromisePrototypeThen` 是用于创建一个新的 Promise 对象，并指定成功和失败状态的回调函数，以便处理异步操作的结果。




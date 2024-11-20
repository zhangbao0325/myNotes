![image-20231116161122407](/Users/mochen/Library/Application Support/typora-user-images/image-20231116161122407.png)





![image-20231116195951449](/Users/mochen/Library/Application Support/typora-user-images/image-20231116195951449.png)













Future Concepts

​	Async functions are called like any other Rust function. However, calling these functions does not result in the function body executing. Instead, calling an `async fn` returns a value representing the operation. This is conceptually analogous to a zero-argument closure. To actually run the operation, you should use the `.await` operator on the return value.

=> 调用async函数，返回的只是一个代表的操作，类似一个类似闭包函数，并不会真正执行，只有调用await时，才会真正执行函数。

```rust
async fn say_world() {
    println!("world");
}

#[tokio::main]
async fn main() {
    // Calling `say_world()` does not execute the body of `say_world()`.
    let op = say_world();

    // This println! comes first
    println!("hello");

    // Calling `.await` on `op` starts executing `say_world`.
    op.await;
}
```

outputs:

```text
hello
world
```

Tokio 

The `#[tokio::main]` function is a macro. It transforms the `async fn main()` into a synchronous `fn main()` that initializes a runtime instance and executes the async main function.

For example, the following:

```rust
#[tokio::main]
async fn main() {
    println!("hello");
}
```

gets transformed into:

```rust
fn main() {
    let mut rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        println!("hello");
    })
}
```

The details of the Tokio runtime will be covered later.







![image-20231115001612453](/Users/mochen/Library/Application Support/typora-user-images/image-20231115001612453.png)











![image-20231115002502011](/Users/mochen/Library/Application Support/typora-user-images/image-20231115002502011.png)
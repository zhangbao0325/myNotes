# Copy

## 相关类型 

​	官方文档列出了实现copy trait的全部类型： https://doc.rust-lang.org/std/marker/trait.Copy.html#

### implment type 

- bool
- char
- i8
- u64
- i64
- usize
- fn()
- *const String
- *mut String
- &[Vec<u8>]
- &String

### not implement type 

- str
- String
- [u8]
- Vec<u8>
- &mut String

### 特殊类型取决于内部数据是否实现了copy trait

- Option<T>

  - copy: Option<4>
  - not copy:  Option<String>

- [a,b] :  

  - copy:  [u8,4]
  - not copy:  [Vec<u8>, 4] 

- (a,b): 

  - copy : (&str, &str)
  - not copy :  (String, u32)

  

```rust
fn is_copy<T: Copy>() {}

fn types_impl_copy_trait() {
    is_copy::<bool>();
    is_copy::<char>();

    // all iXX and uXX, usize/isize, fXX implement Copy trait
    is_copy::<i8>();
    is_copy::<u64>();
    is_copy::<i64>();
    is_copy::<usize>();

    // function (actually a pointer) is Copy
    is_copy::<fn()>();

    // raw pointer is Copy
    is_copy::<*const String>();
    is_copy::<*mut String>();

    // immutable reference is Copy
    is_copy::<&[Vec<u8>]>();
    is_copy::<&String>();

    // array/tuple with values which is Copy is Copy
    is_copy::<[u8; 4]>();
    is_copy::<(&str, &str)>();
    is_copy::<Option<i32>>();
}

// 下面的编译器都会报错
fn types_not_impl_copy_trait() {
    // unsized or dynamic sized type is not Copy
    is_copy::<str>();
    is_copy::<[u8]>();
    is_copy::<Vec<u8>>();
    is_copy::<String>();

    // mutable reference is not Copy
    is_copy::<&mut String>();

    // array / tuple with values that not Copy is not Copy
    is_copy::<[Vec<u8>; 4]>();
    is_copy::<(String, u32)>();
    is_copy::<(str, u32)>();
    
    is_copy::<Option<String>>();
}

fn main() {
    types_impl_copy_trait();
    types_not_impl_copy_trait();
}
```

# AsRef

Q: rust里的as_ref()有什么作用，应该如何使用，写一些示例说明下

A: 在Rust中，`as_ref`方法是`AsRef` trait的一部分。这个trait用于进行类型转换，它允许将一个类型转换成一个引用类型。`as_ref`方法通常用于转换所有权类型（如`String`, `Vec<T>`, etc.）到它们对应的借用类型（如`&str`, `&[T]`, etc.），同时不改变原始数据的所有权。

这是一个特别有用的方法，当你需要传递一个引用到一个期望引用的函数，同时不想转移数据的所有权时。它也经常用于在使用`Option`或`Result`类型时，将它们的内容转换为引用，以便于进行处理。

下面是`as_ref`方法的一些示例：

### 示例1：将`String`转换为`&str`

```rust
1fn print_str(s: &str) {
2    println!("{}", s);
3}
4
5fn main() {
6    let my_string = String::from("Hello, Rust!");
7    print_str(my_string.as_ref()); // 将String转换为&str并传递给函数
8}
```

### 示例2：在`Option`中使用`as_ref`

```rust
1fn main() {
2    let text = Some(String::from("Hello, Rust!"));
3    
4    // 使用as_ref来获取Option内部String的引用
5    match text.as_ref() {
6        Some(t) => println!("{}", t),
7        None => println!("No text found"),
8    }
9
10    // text仍然拥有数据的所有权，所以我们可以继续使用它
11    println!("text is still owned: {:?}", text);
12}
```

### 示例3：在`Result`中使用`as_ref`

```rust
1fn main() {
2    let result: Result<String, &str> = Ok(String::from("Hello, Rust!"));
3
4    // 使用as_ref来获取Result中Ok变体的引用
5    match result.as_ref() {
6        Ok(t) => println!("{}", t),
7        Err(e) => println!("Error: {}", e),
8    }
9
10    // result仍然拥有数据的所有权，所以我们可以继续使用它
11    println!("result is still owned: {:?}", result);
12}
```

​	在上面的例子中，我们看到`as_ref`方法如何在不同的场景下使用。它使得从所有权类型到借用类型的转换更加简洁和直观，并且保持了原始数据的所有权不变。这在编写处理引用和需要保留所有权的函数时特别有用。

```rust
// as_ref和Borrow的区别 ?

// as_ref 是转引用函数, 将具有所有权对象转换成引用对象,
// 不改变被转换对象的基础上产生一个引用对象.

// as_ref 并不是所有类型都默认支持, 很多时候都需要自己去声明.
// as_ref 是AsRef trait 的公共接口方法.
// 只有那些实现了 as_ref 公共接口方法的类型才能使用as_ref.
// 目前: Option, Box, Result 这三种类型默认提供支持as_ref.

// as_ref 和 Borrow 的区别是:
// 基础数据类型引用:
//     Borrow 可以直接在 int, &str, String, vec, [], struct, enum 这种类型上直接指定&来引用.
//     as_ref 则不行, 它需要声明泛型T: AsRef<int>, T: AsRef<str>, T: AsRef<struct name> 来支持.
// 嵌套数据类型引用: Some(&int) , Box(&int) ,
//     Borrow 必须在定义结构时声明 Some<&int> , Box<&int> 才是引用.
//     as_ref 则直接可以在这些嵌套结构上使用as_ref.
// 引用的引用
//     Borrow 引用的引用的表现形式是:   &str -> &&str
//     as_ref 引用的引用的表现形式是:   &str -> &str


fn borrow_example() {
    let s = 1;
    let x = &s;           // 直接引用
    println!("s:{}; x: {}", s, x);
}

fn borrow_nest_example() {

    fn hello(x: Option<&i32>) -> Option<&i32> {      // 指定Some<&i32>
        match x {
            Some(_item) => x,
            None => None
        }
    }

    let s = 1234;
    let z = hello(Some(&s));  // 传入之前要先把引用声明好.
    println!("s: {};  z: {:?}", s, z);
}


#[allow(dead_code)]
#[allow(unused_variables)]
fn borrow_reference_to_reference() {
    let a: &str = "aaaa";
    let b: &&str = &a;
    println!("a:{}; b:{}", a, b);
}


fn as_ref_example() {

    // as_ref 在这种场景的使用上不如 borrow,
    // 这是因为这种写法要求把所有权转移进来.
    // 又不能把&str 返回回去, 因为生命周期会冲突.
    // 所以as_ref不建议在这种场景下使用.

    fn hello<T: AsRef<str>>(x: T) {
        let xx = x.as_ref();
        println!("xx: {}", xx);
    }
    let s = String::from("hello");
    hello(s);
	  // println!("s: {}", s);  这一行就报错了，因为所有权已经转移进去了
  
}


fn as_ref_nest_example() {
    // as_ref 非常适合这种场景, 简单快捷.

    fn hello(x: Option<i32>) -> Option<i32> {
        x.as_ref();       // Option<i32>  to  Option<&i32> 很方便后续代码的编写.
        x
    }

    let s = 1234;
    let z = hello(Some(s));
    println!("s: {}; z: {:?}", s, z);
}


fn as_ref_reference_to_reference() {
    #[allow(dead_code)]
    #[allow(unused_variables)]
    fn hello<T: AsRef<str>>(x: T) {
        let y: &str = x.as_ref();
        let z: &str = y.as_ref();   // 引用上再引用, 永远只有一层引用.
    }

    let s = "hello";
    hello(s);
}


fn main() {
    borrow_example();
    borrow_nest_example();
    borrow_reference_to_reference();
    as_ref_example();
    as_ref_nest_example();
    as_ref_reference_to_reference();
}


==> 
s:1; x:1
s:1234; z:Some(1234)
a:aaaa; b:aaaa
s:1234; z:Some(1234)
y:aaaa; z:"aaaa"
```



# From && Into

- 参考文档
  - https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html

TryFrom && TryInto

类似于 [`From` 和 `Into`](https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html)，[`TryFrom`](https://rustwiki.org/zh-CN/std/convert/trait.TryFrom.html) 和 [`TryInto`](https://rustwiki.org/zh-CN/std/convert/trait.TryInto.html) 是类型转换的通用 trait。不同于 `From`/`Into` 的是，`TryFrom` 和 `TryInto` trait 用于易出错的转换，也正因如此，其返回值是 [`Result`](https://rustwiki.org/zh-CN/std/result/enum.Result.html) 型。

- 参考文档：
  - https://doc.rust-lang.org/std/convert/trait.TryFrom.html
  - https://rustwiki.org/zh-CN/rust-by-example/conversion/try_from_try_into.html
- 参考示例1

```RUST
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug, PartialEq)]
struct EvenNumber(i32);

impl TryFrom<i32> for EvenNumber {
    type Error = ();

    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        } else {
            Err(())
        }
    }
}

fn main() {
    // TryFrom

    assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
    assert_eq!(EvenNumber::try_from(5), Err(()));

    // TryInto

    let result: Result<EvenNumber, ()> = 8i32.try_into();
    assert_eq!(result, Ok(EvenNumber(8)));
    let result: Result<EvenNumber, ()> = 5i32.try_into();
    assert_eq!(result, Err(()));
}

```



- 参考示例2：

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    address: String,
}

impl From<&Person> for String {
    fn from(person: &Person) -> Self {
        // format!("name: {}, age: {}, address: {}", person.name, person.age, person.address)
        format!("{},{},{}", person.name, person.age, person.address)
    }
}

impl TryFrom<&str> for Person { 
    type Error = anyhow::Error;
    fn try_from(value: &str) -> Result<Self, Self::Error> {
        let parts: Vec<&str> = value.split(",").collect();
        Ok(Person {
            name: parts[0].to_string(),
            age: parts[1].parse::<u32>()?,
            address: parts[2].to_string(),
        })
    }
}



fn testFrom() {
    use std::borrow::Borrow;
    let person = Person {
        name: "tom".to_string(),
        age: 18,
        address: "china".to_string(),
    };
    
    let s: String = person.borrow().into();
    println!("{:?}", s);

    let p: Person = s.as_str().try_into().unwrap();
    println!("{:?}", p);
}
```


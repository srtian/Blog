对于如何管理计算机内存，不同的语言都有着自己的实现方式，但大体上分为两种：

- 自带垃圾回收，在程序运行期间不断的去寻找用不上的内存
- 需要自己去分配和释放内存

而Rust 则和以上两种方式不一样。它是通过所有权系统来管理内存。编译器在编译时会根据一系列的规则来进行检查，在运行时，所有权系统的任何功能也不会减慢程序

# 4.1 什么是所有权

## 1、stack and heap 
栈和堆都是代码运行时可供使用的内存，但它们的结构不同。

栈以放入值得顺心存储值，并以相反的顺序取出值，也就是我们常说的先进后出。栈中的所有数据都必须占用已知且固定的大小。在编译时，大小未知或者大小可能会变化的数据，要改为存储在堆上。

而堆时缺乏组织的，当向堆中放入数据时，我们需要请求一定大小的空间。然后操作系统在堆得某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的指针。这个过程叫做在堆上分配内存。

入栈比分配内存要快，因为入栈时，操作系统无需要为存储新数据去搜索内存空间，其位置总在栈顶。


## 2、所有权规则：

- Rust 中的每一个值都有一个被称为其所有者的变量
- 值在任一时刻有且只有一个所有者
- 当所有者（变量）离开作用域时，这个值将会被丢弃


## 3、变量作用域
和其他语言差不多


## 4、String
Rust中的 String 和 字面量的 char 不一样，它是可变的， Rust 有第二个字符串类型，`String`。这个类型被分配到堆上，所以能够存储在编译时未知大小的文本。可以使用 `from` 函数基于字符串字面值来创建 `String`
```rust

#![allow(unused)]
fn main() {
	let mut s = String::from("hello");
	s.push_str(", world!"); // push_str() 在字符串后追加字面值
	println!("{}", s); // 将打印 `hello, world!`
}

```
而导致这两者的区别，主要是在于对内存的处理上


## 5、内存和分配
字符串字面量，我们在其编译的时候就知道其内容了，所以文本被直接硬编码进最终的可执行文件中，这使得字符串字面量快速且高效。不过这得益于字符串字面量的不变性。

而对于 String 来讲，为了支持可变，可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容。这意味着：

- 必须在运行时向操作系统请求内存（ `String::form `  ）
- 需要一个当我们处理完String时将内存返回给操作系统的方法(即垃圾回收)

rust的垃圾回收和其他语言不太一样，当一个变量的所有者离开作用域时，即被释放，这样说起来很简单。但在一些较为复杂的场景，代码的运行将可能不可预测：

#### 变量与数据的交互方式：移动
```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);

error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```
这里和JavaScript的不太一样的是，首先这里的拷贝从表现来讲，有点像浅拷贝，但Rust会使第一个变量无效，这个操作被称为 移动，因此当  `let s2 = s1`时，s1 的内存就会被释放掉，只存在于s2 。这样就解决了，像JavaScript里面经常出现的，由于浅拷贝而引发的变量变更而导致的种种问题。

#### 变量与数据的交互方式：克隆
有意思的是，在对于整型的数据来说，它的表现也不一样，下面的代码就是可以运行的：
```rust

#![allow(unused)]
fn main() {
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
}
```
这是由于像整型这样在编译时已知大小的类型被存储在栈上，所有拷贝的速度很快。而拥有这个性质的数据大致有这些：

- 所有的整数类型，u32
- bool
- 浮点类型
- 字符类型 char
- 元组（需要其类型也是可copy的）


#### 所有权和函数
将值传递给函数，在语义上与变量赋值相似，都会出现以上移动或者是克隆的情况
```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域
    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效
    let x = 5;                      // x 进入作用域
    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x
} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作
fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放
fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

#### 返回值和作用域
返回值同样的也会转移所有权：
```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}

```


# 4.2 引用和借用
上面提到了，在函数使用变量的时候，会出现的变量的所有权变更的问题。但在实际开发中，一直需要关注这种所有权变更的问题，实际也是颇费心智的（至少对于写惯了JS的我来讲，的确是有些不习惯）

而在Rust中，除了上述的规则，还提供了引用的方式来在函数中使用变量：
```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}
fn calculate_length(s: &String) -> usize {
    s.len()
    s.push_str(", hhhh") // 这里会报错
}
```
`&s1` 语法让我们创建一个指向值 s1 的引用，但并不拥有它。变量s 有效的作用域和函数参数是一样的，不过当引用离开作用域后，并不会丢弃使用它指向的数据。另外由于其是在函数内部，并不拥有所引用变量的所有权，因此，我们不能去更改它


## 可变引用
我们也可以使用 mut 来产生可变引用，这样我们家就可以编辑该变量了。
```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}

```
不过可变引用有一个很大的限制：在特定作用域中，只能有一个可变引用。

这样限制的好处是Rust可以在编译时避免数据竞争。数据竞争类似于竞态条件，可由这三个行为造成：

- 两个或多个指针同时访问同一数据
- 至少有一个指针被用来写入数据
- 没有同步数据访问的机制


## 垂直引用
在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个 **悬垂指针**（_dangling pointer_），所谓悬垂指针是其指向的内存可能已经被分配给其它持有者，在Rust中，垂直指针是被禁止的：
```rust
fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！
```
总的来讲，关于引用主要有两点：

- 在任意给定时间，**要么** 只能有一个可变引用，**要么** 只能有多个不可变引用。
- 引用必须总是有效的。


# 4.3 slice
slice也是一个没有所有权的数据类型。slice 允许我们引用集合中一段连续的元素序列，而不是引用整个集合。
```rust

#![allow(unused)]
fn main() {
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
	}
}
```

## 字符串 slice
```rust

#![allow(unused)]
fn main() {
	let s = String::from("hello world");

	let hello = &s[0..5];
	let world = &s[6..11];
}

```
我们可以像如上的方式在指定的 range 创建一个 slice，其中 第一个数字是slice 的第一个位置，而第二数字，则是slice最后一个位置的后一个值。在其内部，slice的数据结构存储了slice的开始位置和长度，长度对应于 后一个值 减去前一个值，<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1613963964691-9ddd9075-4ef8-4789-95db-faaf9f2697f0.png#align=left&display=inline&height=960&name=image.png&originHeight=960&originWidth=1064&size=91982&status=done&style=none&width=1064)

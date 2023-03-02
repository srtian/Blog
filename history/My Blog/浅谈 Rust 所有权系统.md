
# 一、什么是 stack 和 heap
stack 和 heap 都用于变量的内存存储。对于大多数的编程人员来讲，都无需去关心内存是如何分配到 heap 和 stack 中的（实际上，对于 stack   和 heap 的区别，也很多人并不是很清楚）。譬如在JavaScript中，大多数人知道一个结论：基本数据类型放在 stack 中，而引用数据类型放在 heap 里面。但一旦问到，为何需要这样进行划分的时候，少有人可以说上一二。

那它们到底有什么区别呢？ 首先它们虽然都是可供代码使用的内存，但结构是不同的。 stack是一个线性的数据结构，以放入值得顺序存储值并以相反的顺序取出值，也就是我们常说的：后进先出。其数据的输入和取出，则通常被称为 进栈、出栈。由于 `stack`是线性的数据结构，所以 stack 中的所有数据都是必须占用**已知且固定**的大小。

而 heap 则是一个非线性的数据结构，是缺乏组织的。因此对于一些在初始时，大小未知或大小可能发生变化的数据，则可以放在堆中。当我们想要在堆中存储一些数据时，我们通常其实是请求一块大小合适的空间，然后操作系统在堆中搜索一个足够大的空间以匹配我们所请求的内存量，并将其标记为已用，并返回一个表示该位置地址的 **指针， **而这个指针也通常被存在 stack 中。这个过程也就是 **堆内存分配**，也常被称为 **内存分配。**
> 需要注意的是，将变量推入 stack 中，其实并不能被认为是内存分配，因为它本质上，只是按顺序压入 stack中，比不需要去进行显性的分配

由于它们数据结构不同、存储方式不同。也决定了，将值压入 stack 要比在 heap 上进行内存分配要来的快。因为入 stack 时，操作系统无需为新数据搜索内存空间，位置固定于 stack 顶部，只需压入即可。而堆内存分配则需要先找到一块足够存放数据的内存空间，然后才能将变量放入，生成指针（很多时候，还需要将指针压入 stack 中保存）。同理，访问 stack 的变量也比访问 heap 得数据要快。


# 二、什么是所有权
搞清楚栈和堆得区别后，我们大致就可以清楚我们通常说的 GC 其实主要关注的就是 heap 上的内存的回收。而我们今天要说的的所有权，其实也是主要管理堆数据，例如：哪部分代码正在使用 heap上的哪些数据，最大限度的减少堆上的重复数据，清理堆上不再使用的数据确保不会耗尽空间。

所有的编程语言，都有着属于自己的管理计算机内存的方式。大体上可分为两个大的流派：

1. 语言自带垃圾回收机制，可以在程序运行时不断的去寻找不再使用的内存。比如JavaScript、Go等语言，就自带垃圾回收机制
2. 语言没有自带垃圾回收，需要开发者亲自进行内存的分配和释放，比较典型的就是如 C、 C++。

而Rust则没有走上面两条道路，而是通过所有权系统来管理内存，编译器在编译时会根据一系列的规则去对代码进行进行检查，确定变量的回收时机，因此，当程序运行时，所有权系统不会减慢程序。

而所有权系统具体有以下三点最重要的规则：

- Rust 中的每一个值都有一个被称为其 owner 的变量
```rust
let a = 5   // a 是 5 的 owner
```

- 每个值在任何一刻都只能有一个 owner
- 当 owner 离开作用域时，这个值的内存将被回收

# 三、变量的作用域
Rust 根据作用域管理指针，在作用域中申请内存，离开作用域则会释放作用域。Rust 中的作用域也非常简单, Rust 是词法作用域，以大括号为边界，一个大括号对应着一个作用域：
```rust
fn main() {
    let content = String::from("Srtian");
    println!("{}",content);
}
```
比如如上的代码，在 `String::from` 处为 `content` 申请了内存，而在离开大括号后，content也就离开了作用域后被释放掉。

# 四、所有权的具体表现
上面几部分以及差不多将Rust的所有权系统简单的减少了一遍，接下来就让我们来看看，所有权系统到底是如何作用域 Rust 的内存管理的。

## 4.1、所有权的移动
在Rust中，对于已知大小的值，将其进行复制到另一个值会很容易：
```rust
fn main() {
    let a =  "5" ; 
    let b = a ; //将值a复制到b 
    println!（"{}", a)  // 5 
    println!("{}", b)  // 5 
} 
```
因此 a 存储在  `stack` 中，所以我们可以对其直接进行复制。但对于放在 `heap` 中的数据，我们就不能这么简单的进行复制了（放在 `heap` 中的数据，也就是被所有权系统所管理的数据）：
```rust
fn main() { 
	let s1 = String::from("hello");
	let s2 = s1; // 将s1复制到s2
    println!("{}", s1)  // 这里会报错，因为s1在这里已经被释放了
    println!("{}", s2)  // hello
}
```
 当我们运行上面代码时，会出现报错。这是因为，当我们复制存储在 `heap` 中的值时，Rust 为了防止诸如：二次释放这样的错误，它在处理这种场景时，会直接认为 s1 不再有效。这样 Rust 就无需再在 s1 离开作用域时再需要清理它。

熟悉诸如JavaScript等语言的朋友，应该对浅拷贝和深拷贝很熟悉，其实上述的这个操作有点浅拷贝的意思，它只会拷贝指针、长度和容量，而不会直接拷贝数据。但 Rust 同时也会让第一个变量直接无效，因此也不能粗暴的将其理解为浅拷贝。
> 还有个需要注意的： Rust 永远不会自动创建数据的"深拷贝"。因此，所有自动的复制，都可以认为对于运行时的性能影响较小

而当我们确实需要深拷贝去拷贝 heap 上的数据时，我们可以使用 `clone` 方法来对值进行深拷贝：
```rust

#![allow(unused)]
fn main() {
	let s1 = String::from("hello");
	let s2 = s1.clone();
	println!("s1 = {}, s2 = {}", s1, s2);
}
```

## 4.2、所有权的借用
所有权的移动或者变量的深拷贝并不能满足工程师们日常的开发需求，比如：
```rust
fn main(){
    let contents = String::from("hello srtian");
    some_process(contents);
    println!("{}",contents); // error
}
fn some_process(word:String) {
    println!("some_process {}",word);
}
```
在上面的代码中， some_process(contents) contents变量会『移动』给some_process的参数word，contents变量就不能再次使用了。而且对于每次内存的重新分配，许多资源在时间和空间的开销都太昂贵了，但在日常开发中，类似的需求还是很多的。因此在这种情况下， Rust 提供了借用的选项。

所有权的借用也非常简单，我们只需在借用的变量前，加&字符即可：
```rust
struct Person {
    age: u8
}

fn main() {
    let jake = Person { age: 18 };
    let srtian = &jake;

    println!("jake: {:?}
srtian: {:?}", jake, srtian);
}
```
在上述代码中，尽管没有 clone。但上面的代码仍然会编译并输出。同样，如果是不可复制的值被借用，可以将其作为参数传递给函数，也就是解决我们上面所说的那个问题：
```rust
fn sum(vector: &Vec<i32>) -> i32 {
    let mut sum = 0;
    for item in vector {
        sum = sum + item
    }
    sum
}

fn main() {
    let v = vec![1,2,3];
    let v_ref = &v;
    let s = sum(v_ref);
    println!("sum of {:?}: {}", v_ref, s); // 不会报错
}
```
不过，需要注意的是，对于借用来的变量，我们是不能对其进行更改的。这其实也符合我们日常生活的基本常识，借来的东西，我们都需要原样进行返回。不过，Rust 也提供了方法来对借用来的变量进行更改：
```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}

```
我们需要将 s 修改为 mut。然后传入的参数以及接受的参数，都需要显式的表明是 mut 的。不过需要注意的是，在特定的作用域中，特定的数据只能有一个可变引用。这个限制允许可变性的存在，不过是以一种受限的方式允许的。这样做的好处在于 Rust 可以在编译时就避免 数据竞争。数据竞争类似于竞态条件，它可由三种行为造成：

1. 两个或更多指针同时访问统一数据
2. 至少有一个指针被用来写入数据
3. 没有同步数据访问的机制

有时候，我们会希望返回借来的值。比如我们想要返回字符串中较长的一个，我们可以写出如下的代码：
```rust
fn longest(x: &str, y: &str) -> &str {
    if x.bytes().len() > y.bytes().len() {
        x
    } else {
        y
    }
}

fn main() {
    let jake = "jake";
    let srtian = "srtian";

    println!("{}", longest(jake, srtian));
}
```
以上的代码不能成功编译，会报错：
```rust
fn longest(x: &str, y: &str) -> &str {
                                ^ expected lifetime parameter
 
 = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
```
这就有关乎变量的生命周期了，生命周期是借用变量的有效范围。Rust 强大的编译器让我们在大多数情况下，无需显式的编写它们，而是通过推断去实现。但在一些需要生命周期参与的场景下，还是需要我们手动的去添加申明周期函数。譬如，我们想要解决上面的错误，就需要进行生命周期的手动声明：
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.bytes().len() > y.bytes().len() {
        x
    } else {
        y
    }

fn main() {
    let jake = "jake";
    let srtian = "srtian";

    println!("{}", longest(jake, srtian));
}
```
如此我们就能将借用的变量进行返回来。



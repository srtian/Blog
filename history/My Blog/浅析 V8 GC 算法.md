
# 前言
通常来讲，所有动态创建的对象都会被非配到堆内存中，但由于堆内存本身存在大小限制，因此当我们创建的对象的大小到达一定的数量时，就会出现内存不足而导致程序报错的问题；这时，我们就需要通过将那些被程序所不能访问到的对象所占用的内存进行释放，从而获得新的可用空间。对于内存的管理，现在市面上大致有以下三种方案：

- 以C、C++为代表的手动内存管理，需要程序员自己对内存进行申请、释放以及分配
- 以 Go、Python 为代表的高级语言，通过各种 垃圾回收（GC）实现内存管理
- Rust 通过所有权机制以及生命周期，来实现内存管理<br />而正如上述第二种方案所述，V8 进行内存管理的方案就是使用 GC 去实现；而很多初学 JavaScript 的同学在看 MDN 关于 [内存管理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#garbage_collection) 的章节后，大致就会了解到，V8 是使用标记-清除算法（Mark-Sweeping GC ）实现的垃圾回收。但如果对V8的垃圾回收做更为深入的了解后，又会发现，Mark-Sweeping GC 并不能涵盖 V8 垃圾回收的主要思想，还需要引入诸如：新生代、老生代、From\To 等等新的概念，就像这样：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/296173/1655104701521-ac86e32a-dcc9-47a9-b253-72d8ba466369.png#clientId=u537e9e48-652d-4&from=paste&height=306&id=ud1772bc4&name=image.png&originHeight=611&originWidth=1304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112147&status=done&style=none&taskId=u8b20260b-5ad1-4548-9abe-2a623d58c4f&title=&width=652)<br />这样的困惑在于：大多数的文章都是在告诉大家 V8 怎么做（How），但并没有去解释：这些东西是什么（What）以及为何要这样 (Why)。因此，本文将尝试从 标记清除算法开始去分析：标记清除算法本身是什么以及存在什么问题，以及为了解决这些问题又需要引入哪些算法。希望能通过这篇文章，让大家能对 V8 组成的大概架构有所了解，更希望能够通过V8 的设计思路，能对一些经典算法的应用以及设计思想有所了解，从而应用到自己的开发中去。


# 一、标记清除（Mark Sweep GC）
标记清除算法可以说是应用最为广泛的 GC 算法之一，其核心思想在于：首先对堆内存进行遍历进行标记，从而区分活动对象与非活动对象（这里需要补充的是活动对象与非活动对象，我们通常讲能被程序所访问到的对象称之为活动对象，反之不能被访问的则是非活动对象；且当一个活动对象转化为非活动对象后，这个过程将不可逆。），然后再进行一次遍历，将标记为非活动对象的内存进行回收，放入空闲链表中，最后在空闲链表中根据一定的信息进行分配内存；伪代码如下：
```rust
fn markSweep() {
    mark() // mark all live objects, start from the roots
    sweep()
}

fn mark() {
    for(root in $roots) {
        markObj(*root)
    }
}

fn markObj(obj) {
    if(obj.mark === FALSE) {
        obj.mark = TRUE
        for(Child in Children(obj)) {
            markObj(*child)
        }
    }
}

fn sweep() {
   for(root in $roots) {
       if(root.mark === TRUE) {
       // If it is found to be a live object
       // reset it to FALSE and wait until the next GC
          root.mark = FALSE
       } else {
           freeList.next = root
       }
   }
}
```

需要注意的是，我们在这里进行对象遍历时，通常使用的是深度优先搜索；这是因为较之广度优先，两者完成搜索所需的步数并不会有所区别，只取决于需要遍历的对象的数量；但在内存使用方面，深度优先所需的内存量是比广度优先要少的。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/296173/1655104721919-45b5fc59-b6d5-4171-90f2-4237b8a17c16.png#clientId=u537e9e48-652d-4&from=paste&height=535&id=u4aeb16ed&name=image.png&originHeight=1070&originWidth=1304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=665990&status=done&style=none&taskId=u7e760cfd-91d1-4009-b326-37a4952e443&title=&width=652)
> Difference between BFS and DFS - GeeksforGeeks

在完成非活动对象的释放后，我们还需要对已回收的垃圾进行再利用；由于我们在清除阶段就已经将垃圾对象链接到空闲链表了，所以进行分配时，我们只需要根据所需要的内存空间去空闲链表中遍历查找合适大小的分块即可。伪代码如下：
```rust
fn allocation(size) {
    let chunk = pickupChunk(size, $freeList)
    if(chunk !== NULL) {
        return chunk
    } else {
        allocationFail()
    }
}



fn pickupChunk(size, freeList) {
    while(freeList) {
        if(freeList.size >= size) {
            return freeList.chunk
        } else {
            freeList = freeList.next
        }
    }
}
```

这里我们使用的策略是 First-fit，即发现大于等于 size 的 chunk 时，就立即返回该 chunk，除此之外还有 Best-fit 以及 Worst-fit。其中 Best-fit 是遍历空闲链表，返回大于等于 size 的最小 chunk。而 Worst-fit 则会找到空闲链表中的最大的 chunk，然后将其分割成 mutator 申请的大小和分割后剩余的大小，其目的是将分割后剩余 chunk 最大化，但由于这种方式很容易生成大量小的 chunk，所以一般不推介使用。其流程如下：

### 1.1、小结
上述我们已经对 Mark-Sweeping GC 有了一个简单的了解了，这里我们可以先来讨论一下，Mark-Sweeping GC 的优劣：<br />优点：

- 实现简单：其主要就是遍历所有对象，对其进行标记，从而区分活动对象与非活动对象，然后再对非活动对象的内存进行回收，放入空闲链表中，后续分配只要遍历空闲链表即可。<br />缺点：
- 碎片化：在标记-清除算法的使用过程中会逐渐产生被细化的分块，不久后就会导致无数的小分块散布在堆的各处。我们称这种状况为碎片化（fragmentation）。而过于碎片化，则会导致内存使用效率不高，以及分配速度较慢（需要遍历的对象数量变多了）的问题。
- 速度不佳：首先我们在进行标记以及清除，都需要去对堆内存进行遍历，其次在进行内存分配时，在最糟糕的情况下，也需要去遍历整个空闲链表。

也正是由于 Mark-Sweeping GC 存在以上一些问题，所以实际 V8 在实现内存回收上并没有单纯的使用 Mark-Sweeping GC 来进行内存回收，而是采取了一些非常巧妙的算法来帮助我们提升 GC 的效率。由于我们在标记清除的过程中，需要对整个堆内存去进行遍历，这无疑会增大我们遍历的成本，所以在这种情况下，我们就可以运用经典的算法思想：缩小范围，来提升我们算法的运行效率，这也就是我们下一部分的主角：分代垃圾回收（Generational GC）的核心作用。

> 需要注意的是，具体的标记-清除算法实现，大都不是进行简单的标记以及清除，而是使用了 三色标记-清除，在原有的基础上增加了一种标记（灰色）来实现并行回收、提升效率，在这里就不做过多的介绍了，其核心思想并没有太大的区别，感兴趣的同学可以自行了解一下～



# 二、分代垃圾回收（Generational GC）

分代垃圾回收主要通过引入“年纪”的概念，通过年纪来区分对象，从而优先回收容易成为垃圾的对象，从而提升垃圾回收效率。它基于以下几个考虑因素：

- 管理堆内存的一部分要比管理整个堆内存要快。
- 较新的对象具有较短的生存时间，而较旧的对象具有较长的生存时间。
- 较新的对象往往相互关联，并由应用程序几乎同时进行访问。

基于以上的考虑，分代垃圾回收会将对象分类为几代，然后针对不同的代使用不同的 GC 算法。通常我们会将刚生成的对象称之为新生代对象，而到了一定年纪（存活过一定时间）的对象成为老生代对象。

在进行了根据年纪的划分后，我们就能根据实际情况，来对内存进行因地制宜的管理了：

- 对于新生代对象来说，由于大部分新生代对象存活的生命周期都不会太长，针对于它们的 GC 可以在很短的时间内结束，因此我们对于新生代GC（minor GC 即小规模 GC）可以单独去进行处理；而对于那些在 Minor GC 中，能够进行存活的对象，我们就可以将上升为老生代对象，我们可以将整个上升的过程称之为 晋升（promotion）。
- 而对于老生代对象的 GC，我们又将其称之为：老生代GC(major GC)；且因为 老生代对象 很难成为垃圾，因此我们就可以在频率上下文章：对于 Minor GC，因为其本身所管理的对象就很容易成为垃圾；因此我们可以增加调用 MinorGC 的频率，而对于 Major GC，因此我们就可以降低其频率。

而在V8 的实现中，Major GC 主要使用的也正是 Mark-Sweep GC，这里设计的目的就在于：通过对对象进行分类，减少标记-清除所需遍历对象的数量（只需要遍历老生代对象）以及频率，从而减少标记清除所需要的时间。

# 三、GC复制算法（Copying GC）
上文我们提到，对于新生代对象的内存，由于其特性，因此 GC 触发的频率会比较频繁，所以在这种场景下，我们就需要一种足够高效的算法来帮助我们来管理内存了。GC复制算法（Copying GC）就属于这种算法。

GC复制算法是 Marvin L. Minsky 在 1963 年研究出来的算法，距今已有半个世纪，但其核心思想仍然广泛的用于各个方面；简单来讲，就是将内存空间分为相等的两块区域，一块区域是对象空间（From），另一块区域则是空闲区域；当我们需要对新生代对象进行回收时，其实就是将对象区域的所有活动对象转移到空闲区域，而在转移完成后，对象区域剩余的则就可以视为垃圾回收掉，最后再将 From 和 To 进行反转，这样原有的空闲区域就变成新的对象区域可以继续使用了。具体步骤大致如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/296173/1655105220189-00c98549-0868-4d51-9462-862b4098e32b.png#clientId=u537e9e48-652d-4&from=paste&height=428&id=u0ec7858a&name=image.png&originHeight=856&originWidth=1096&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130076&status=done&style=none&taskId=u83d1f441-6085-4d20-bc0d-9781cbc42f1&title=&width=548)
> 需要注意的是，第二阶段的 Garbage 并不是连续的，这里我只是为了方便画图以及理解，所以放在一块儿，实际情况下非活动对象是散落在 From 中的。另一方面，这里没有表现出我们在分代垃圾回收所提到的“晋升”（为了减少不必要的信息的干扰）。


而在分配阶段，Copying GC 也和 Mark-Sweeping GC 存在不同，前面我们有介绍，Mark-Sweeping GC 在分配内存时，大致使用的方式是：通过需要的内存大小，去遍历空闲链表，在找到满足条件的 chunk 后，进行返回。这样做的方式存在的问题在于：在最坏的情况下，我们可能需要遍历完整个空闲链表才能找到合适的空间。而 Copying GC 则不存在这种问题，如上图所示，当我们完成复制已经清理工作后，内存所摆放的位置将会变得有序，而空闲的空间也是连续的，因此我们只需要直接根据所申请的内存大小分配即可。因此较之 Mark-Sweeping GC 需要遍历空闲链表才能完成内存分配，Copying GC 的分配要简单的多，不需要进行遍历，可以理解为：O(n) 到 O(1) 的提升。<br />以上便是 Copying GC 的大致流程了，我们再来小结一下，Copying GC 的一些优缺点：<br />优点在于：

- 吞吐量优秀：Mark-Sweeping GC 消耗的吞吐量是搜索活动对象（Mark）所花费的时间和搜索整体堆（Sweep）所花费的时间之和。而 Copying GC 则只对活动对象进行 Copy，所以跟一般的Mark-Sweeping GC 相比，它所需的时间会较少。也就是说，其吞吐量优秀。
- 分配速度快。
- 不会发生碎片化。

缺点则是：

- 堆内存使用率不高：需要将内存一分为二，实际使用的内存只有一半，因此内存的使用效率不高。

综上所述，Copying GC 在效率上较之 Mark-Sweeping GC 会高上不少，但在内存使用率上还是存在较大缺陷，因此在V8的实践上，也并没有将其作为主要内存回收方式，而是主要用于新生代内存的管理，而且在内存分配上，新生代的内存空间也不大（空间越小，节省的空间越大），因此在 V8 中还隐含着一个条件：如果一个对象足够大时，会直接晋升到老生代中。而之前我们提到，老生代中使用的是 Mark-Sweeping GC，虽然我们使用了分代垃圾回收来减少 Mark-Sweeping GC 所需管理的对象的数量以及进行 GC 的频率，但仍然有个问题还没有解决，那就是碎片化的问题。要解决这个问题就可以引出我们的下一位主角：GC标记-压缩算法 了。

# 三、GC标记-压缩算法（Mark Compact GC）
Mark Compact GC 其实是 Mark-Sweeping GC 和 Copying GC 的融合体。顾名思义，Mark-Compact 也是由 Mark 和 Compact 两个阶段组成的，这里的 Mark 阶段其实和我们之前所说的，遍历堆内存进行标记是一样的，不一样的是，然后，我们在堆上移动活动对象，目的是压缩堆，以便所有活动对象都位于堆的开头，而在这些对象之后的所有内存都可以分配。因此 Mark-Compact 解决了碎片问题，通常以内存开销和更长的运行时间为代价。而具体的 Compact 算法有很多种，比如：

- Edward’s Two-finger compactions
- Two-Finger
- Lisp 2 collector
- ImmixGC
- 。。。

在这里简单的介绍一些 Lisp 2 collector，先来介绍一些它的特性：

- 它是基于滑动收集器的算法（这里的滑动，指的是实现“压缩”的方式，除此之外有复制等方式）。
- 在每个活动对象头中添加一个 **forwarding** 字段。
- 这里的开销也是此类算法的主要缺陷之一。
- 可用于不同大小的对象（有些算法只能处理相同大小的算法，比如：Two-Finger）。

接下来，就让我们用伪代码来介绍一下这种算法吧。首先在调用上，正如上面所说的，它和 Mark-Sweeping 并无太大区别，分为两个阶段：
```rust
fn markCompact() {
    mark() // mark all live objects, start from the roots
    compact() // compact the heap
}
```
其中 mack 阶段之前并无什么区别，因此在这里就不再赘述了，而 compact 阶段则需要三次遍历堆内存，因此可以分为三部分：
```rust
fn compact() {
    setForwardingPtr() // compute the forwarding address
    adjustPtr() // update the roots and references
    moveObj() // move the objects
}
```
在第一部中，我们会搜索整个堆内存，并给活动对象加上 forwarding 指针（初始情况下，我们的 forwarding 指针为 nul），大致步骤如下：
```rust
fn setForwardingPtr() {
    let scan = $heapStart  // a ptr to search for objects in the heap
    let newAdress = $heapStart // a ptr to the target location
    while(scan < $heapEnd) {
        // set the pte to newAddress
        // newAddress moves according to the object length
        if(scan.mark == TRUE) {
            scan.forwarding = newAddress
            newAddress += scan.size
        }
        scan += scan.size
    }
}
```
这里我们需要使用 forwarding 指针来记录空间的原因在于：Copying GC 会划分出 From 空间以及 To 空间，因此我们在移动的对象时，不用去关心对象覆盖的问题，而在 Mark-Compact 中，我们是在同一个空间实现对象的压缩的，因此可能会存在移动前对象会被覆盖的问题，因此在进行移动前，我们需要事先将各对象的指针全部更新到预计要移动的地址。这样当我们要进行移动操作时，只要顺序的去移动活动对象即可。因此接着我们就需要更新指针：
```rust
fn adjustPtr() {
    // rewrite the pointer
    for(root in $roots) {
        *root = *root.forwarding
    }
    scan = $heapStart
    // Rewrite the pointers of live obj
    while(scan < $heapEnd) {
        if(scan.mark == TRUE) {
            for(child in children(scan)) {
                *child = *child.forwarding
            }
            scan += scan.size
        }
    }
}
```
经过上述的步骤，我们就进行了指针的移动，最后我们就可以搜索整个堆内存，将活动对象移动到 forwarding 指针所指向的地址，从而实现活动对象的“压缩”：
```rust
fn moveObj() {
    let scan = $heapStart
    while(scan < $heapEnd) {
        if(scan.mark == TRUE) {
            let newAddress = scan.forwarding
            move(newAddress, scan) // move the object itself there
            newAddress.mark = FALSE // unmark it, to prepare for the next invocation of compact
        }
        scan += scan.size
    }
}
```

### 小结
优点：

- 堆内存利用效率高：Mark-Sweeping GC 相较于 Copying GC 来说，堆的利用率高上很多；而较之 Mark-Sweep GC，则不会产生碎片化，因此对于 Mark-Sweep GC 在内存利用率上也有一定优势。

缺点：

- 压缩效率慢：在我们上述的例子中，compact 阶段需要遍历3次堆内存，这无疑在效率上大打折扣（虽然也有一些 compact 算法只需要进行两次堆内存的遍历即可实现压缩，但也都或多或少的存在一些问题）

# 总结
自此，我们就大体的粗缆了V8 GC 的算法部分的重要组成，并对这些算法进行了一定的介绍，相信看到这里你也会对其设计的思路有了一个大体的理解，对于这些算法，我的理解是：如果没有遇到相关问题时，只需要理解其算法思想以及解决一些问题的思路即可，这才是这些算法的精华所在，譬如：

-  如果一个算法执行效率过慢如何解决？ 
   - 根据分代垃圾回收的思想，我们可以对数据进行初步分类，再根据分类的一些特征去进行适宜的操作
-  对于需要频繁操作或者调用的数据，可以采取以空间换时间的方法，来提升效率（其实这个思想用到的地方有很多，比如JS 的 JIT(just-in-time)，CDN缓存资源的一些设计，用 map 去替换数组提升查的效率等等） 
-  等等 

因此还是希望能从中吸取一些对自己开发能有用的东西吧，最后如果有同学对垃圾回收算法有兴趣的同学，推荐一本关于垃圾回收的书籍，相信看完后会对市面上常见的 GC 算法都有一些了解：[《垃圾回收的算法与实践》](https://book.douban.com/subject/26821357/)

参考资料

-  [《垃圾回收的算法与实践》](https://book.douban.com/subject/26821357/) 
-  [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management) 
-  [https://v8.dev/blog/trash-talk](https://v8.dev/blog/trash-talk) 

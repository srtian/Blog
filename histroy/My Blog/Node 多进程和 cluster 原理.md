
# 一、概述
由于node.js 主线程是单线程的，因此当我们的 node 程序运行时，通常只启动了一个进程，只能在一个 CPU 中进行运算，无法运用服务器中的多核 CPU。这显然是对于多核CPU服务器性能的浪费，因此我们需要寻求一些解决方案，来充分利用服务器的多核CPU，从而提升我们程序的运行效率。而对于这种情况，我们通常的做法就是 多进程分发策略：即主进程接收所有的请求，通过一定的负载均衡策略分发到不同的子进程中。而这一方案，其实也有两种不同的实现方式：

1. 主进程监听一个端口，子进程不监听端口，主进程分发请求到子进程中。这样做的好处在于，通常只需要占用一个端口，通信相对简单，转发策略也更为灵活。缺点则是实现相对会比较复杂，对主进程的稳定性也有更高的要求。
2. 主进程和子进程分别监听不同的端口，通过主进程分发请求到子进程。即创建一个主进程，以及若干个子进程。由主进程监听客户端连接请求，并根据特定的策略，转发给子进程。这样做的优势在于：实现相对简单，各实例相对独立，这对服务稳定性有好处。而缺点则是：增加端口的占用，且进程之间通信会比较麻烦。

而node 的 cluster 模块就是第一个方案的实现。cluster 模块是对child_process 模块的进一步封装，专用于解决单进程NodeJS Web服务器无法充分利用多核CPU的问题。

cluster 具体使用起来很简单，node官方文档也将的非常仔细，在此也就不多做赘述了：
> [http://nodejs.cn/api/cluster.html#cluster_cluster](http://nodejs.cn/api/cluster.html#cluster_cluster)

其原理官方文档也有所提及：其工作进程是由 child_process.fork() 方法创建，因此它们可以使用 IPC 和 父进程通信，从而使其各进程交替处理连接服务。在 node 的主从模型中，master 主管监听端口，以及将对应任务分发给 worker 子进程，起着一个中枢的作用。按照我们通常的理解，如果根据使用各 worker 进程的负载情况来挑选woker来执行对应的任务，效率应该会比直接循环发放要来的高，但 node 文档中提到这种声明方式会受到操作系统的调度机制影响，使其分发变得不稳定，因此 node 也就将 循环法 作为了默认的分发策略。
> 需要注意的是，node 官方文档中使用 worker 来表示主进程fork出的子进程，这其实会让不少前端开发者会将其与浏览器环境中的 worker 多线程相混淆，但他们其实不是一个东西。


# 二、线程和进程
上面提到，cluster 是解决单进程的问题。但大家估计也听说过JavaScript是单线程的语言（实际上，我在面试一些候选人的时候，问到 node 的多进程实现，他们也会反问我JavaScript是单线程的，关多进程什么事情）。因此为了方便后续的讨论，我们再次还需要再说明一下，线程和进程的区别。

首先，进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程是资源分配的基本单位，而线程是独立调度的基本单位，一个进程中可以有多个线程，它们可以共享进程资源。它们的主要区别如下：

- 拥有资源：进程是资源分配的基本单位，而线程则不拥有资源，只能去访问其隶属于的进程的资源
- 调度：线程中是独立调度的基本单位
- 系统开销：线程的切换，只需保存和设置少许的寄存器内容，开销很小。而进程的切换，则涉及当前执行CPU环境的保存和新调度进程CPU环境的设置
- 通信方面：线程间可以通过直接读取同一进程中的数据进行通信，但是进程通信需要借助 IPC。

# 三、Cluster原理
对于 cluster 模块，我们主要需要关注两部分的内容：

1. cluster 是如何做到多个进程监听同一端口的
2. node 是如何实现负载均衡请求分发的

## 3.1、多进程监听同一端口
在 cluster 模式中，存在 master 和 worker 的概念。我在上面也提到了，这个 worker 并不是我们在浏览器中的worker。在这里，master 指的是主进程，而 worker 指的是子进程。它们的创建也非常简单：<br />                       ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618838760382-aef9b538-5d9a-4369-a03c-9c30914ef595.png#clientId=u7f471342-e022-4&from=paste&height=366&id=u2dbe4144&name=image.png&originHeight=732&originWidth=1070&originalType=binary&ratio=1&size=146177&status=done&style=none&taskId=uc17db156-8851-4a2e-8bf1-ebdca56983d&width=535)<br />在上述代码中，第一次 require 的 cluster 对象就模式是一个 master。对应的源码也非常简单，本质上是通过进程环境变量设置来进程判断，这是node的主进程在进行子进程管理时的标识，当调用`cluster.fork()` 时，会生成一个子进程时会以一个自增ID的形式生成这个环境变量。如果没有设置就是 master 进程，反之即为 worker：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618838731846-8471dae5-7759-445e-838f-602b37f1d7d6.png#clientId=u7f471342-e022-4&from=paste&height=258&id=u42e5a9bb&name=image.png&originHeight=516&originWidth=1598&originalType=binary&ratio=1&size=125649&status=done&style=none&taskId=u3dd6f8a3-a633-4440-be32-7e00e2d7ad4&width=799)<br />[                             https://github.com/nodejs/node/blob/master/lib/cluster.js](https://github.com/nodejs/node/blob/master/lib/cluster.js)<br />因此我们第一次调用 cluster 模块就是 master 进程，后续的则都为 worker。另外主进程和子进程 require 文件也不同：

- 主进程：internal/cluster/primary
- 子进程：internal/cluster/child

### 主进程模块
先让我们来瞅瞅 master 进程的创建过程，由于代码其实还挺多，因此就不全部粘贴出来了，可自行去浏览：
> [https://github.com/nodejs/node/blob/7397c7e4a303b1ebad84892872717c0092852921/lib/internal/cluster/primary.js](https://github.com/nodejs/node/blob/7397c7e4a303b1ebad84892872717c0092852921/lib/internal/cluster/primary.js)

可以看到，当我们执行 `cluster.fork` 时，一开始会调用 `setupPrimary` 方法，创建主进程。因为这个方法是通过 `cluster.fork` 调用，因此会调用多次，但该模块有个全局变量 `initialized` 用于区分是否为首次创建，如果是首次则进行创建，如果不是则跳过。代码如下：<br />               ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618838827871-b8222a8e-ebe8-4fc2-a0da-20e421a77d9e.png#clientId=u7f471342-e022-4&from=paste&height=258&id=u861d27c6&name=image.png&originHeight=516&originWidth=1296&originalType=binary&ratio=1&size=107800&status=done&style=none&taskId=u2763db2a-d6c9-4ab8-93dd-169a68089a6&width=648)<br />而  `cluster.fork` 方法，其实也很简单，具体代码如下：<br />              ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618839042529-ec225098-fd5d-4a9f-8258-aeb7b6827dd0.png#clientId=u7f471342-e022-4&from=paste&height=384&id=ued1fad08&name=image.png&originHeight=768&originWidth=1260&originalType=binary&ratio=1&size=170944&status=done&style=none&taskId=ud948b475-2948-4c62-8976-9d9842cff8c&width=630)<br />首先是进程进程的初始化，也就是创建 master。然后就是进行id的递增，再创建 worker 子进程。而在 `createWorkerProcess` 方法中，实际是使用 `child_process` 来创建子进程的。

需要注意的是，在初始化代码时，我们调用了两次 `cluster.fork` 方法，因此会创建两个子进程，在创建后又会调用我们项目根目录下的 `cluster.js` 启动一个新实例，但此时 `cluster.isMaster` 为 false，因此会 `reuqire`到子进程的方法。

且由于是 worker 进程，因此代码会`require('./app.js')`模块，在该模块中会监听具体的端口：<br />                   ![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618841656732-a347e092-9c55-4b61-bbd3-25f6cf3d3c64.png#clientId=u7f471342-e022-4&from=paste&height=294&id=u83915fcc&name=image.png&originHeight=588&originWidth=1210&originalType=binary&ratio=1&size=124836&status=done&style=none&taskId=u0fc9ae2c-ceba-495b-8ace-0c9090b1a57&width=605)<br />这里的 `server.listen` 会调用 `net` 模块中的 `listenInCluster` 方法，该方法中有一个关键信息：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618841835848-d3623e88-8076-4d1f-a922-e88f2fb5547f.png#clientId=u7f471342-e022-4&from=paste&height=492&id=ue6c3029b&name=image.png&originHeight=984&originWidth=1582&originalType=binary&ratio=1&size=215723&status=done&style=none&taskId=uc5febabc-b6a5-4975-bbe6-b198610034e&width=791)<br />上面代码中，首先判断是否为**主进程**，如果是就是**真实的监听端口启动服务**，而如果非主进程则调用 `internal/cluster/child`  中的`cluster._getServer`方法。

然后就会通过一个`send`方法，如果监听到 `listening` 就发送一个 `messgae` 给主线程，而主线程同样的也有一个 `listening` 时间，监听到该事件后将子进程通过 `EventEmitter` 绑定到主进程上，这样就完成了主子进程的关联绑定，并且只监听了一个端口。而主子进程之间的通信方式，就是我们常说的 IPC 通信。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618842084917-26fa359d-d461-42b7-8faf-2417927ece06.png#clientId=u7f471342-e022-4&from=paste&height=312&id=u6b17498a&name=image.png&originHeight=624&originWidth=1396&originalType=binary&ratio=1&size=147078&status=done&style=none&taskId=u3d390777-fb40-4b14-8485-12d438cfaa2&width=698)

总结起来如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/296173/1618924368949-8d5487aa-39b8-4644-821d-3e0e1cb86ad5.png#clientId=u00e4b072-6445-4&from=paste&height=1670&id=ueff375cb&name=image.png&originHeight=1670&originWidth=1460&originalType=binary&ratio=1&size=197021&status=done&style=none&taskId=u463a2351-8add-4733-86d3-db952c1de12&width=1460)<br />（图转自：[https://github.com/chyingp/nodejs-learning-guide/blob/master/%E6%A8%A1%E5%9D%97/cluster.md](https://github.com/chyingp/nodejs-learning-guide/blob/master/%E6%A8%A1%E5%9D%97/cluster.md)）

## 3.2、负载均衡原理
cluster 模块进行负载均衡处理，主要涉及两个模块：

- round robin handle.js  此模块是针对于非 Windows 平台应用的模式，主要的做法是轮询处理，也就是轮询调度分发给空闲的子进程，处理完成后回到 worker 空闲池中。需要注意的是如果已经完成绑定了子进程，就会复用该子进程，如果没有就会重新进行判断。
- shared handle.js 此模块针对 Windows 平台应用的模式，通过将文件描述符、端口等信息传递给子进程，子进程通过复写掉 `cluster._getServer` 方法，从而在 `server.listen` 中保证只有主进程监听端口，主子进程通过 IPC 进程通信，其次主进程根据平台或者协议不同，应用不同模块来进行分发请求给子进程处理。


# 参考资料

1. [https://github.com/chyingp/nodejs-learning-guide/blob/master/%E6%A8%A1%E5%9D%97/cluster.md](https://github.com/chyingp/nodejs-learning-guide/blob/master/%E6%A8%A1%E5%9D%97/cluster.md)
2. [http://nodejs.cn/api/cluster.html](http://nodejs.cn/api/cluster.html)
3. [https://www.cnblogs.com/dashnowords/p/10958457.html](https://www.cnblogs.com/dashnowords/p/10958457.html)
4. [https://juejin.cn/post/6844903764093042695](https://juejin.cn/post/6844903764093042695)

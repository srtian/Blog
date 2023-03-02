
# 前言

这篇文章主要是讨论一下 Serverless 的发展历程以及在前端的一些落地场景，以及我在看伯克利的这篇论文：[《Cloud Programming Simplified: A Berkeley View onServerless Computing》](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf) 的一些记录。具体想要详细的了解 Serverless 也建议直接去阅读这篇论文，相信会有不少收获的。


# 一、 什么是Serverless

讨论一个技术的具体使用场景，首先需要将其做一个定义，确定我们说要讨论的范围。确定讨论的范围，归根结底，首先需要了解 Serverless 所产生的背景以及其具体以什么方式解决了什么样的问题。


## 1.1 Serverless的背景以及历史

在云计算已经普及的今天， Serverless 已经是一个在各大技术论坛或者演讲上，出现频率非常高的技术名词了，在前端也不例外，几乎每个大的前端技术演讲会议，都会或多或少的提到它，并扯扯它和前端的一些交集或者结合的落地场景。

但溯本回源，Serverless 的产生，归根结底是要解决什么问题呢？要回答这个问题，我们不得不去回顾云计算的发展历程。在云计算刚刚兴起的时候，市场上主流的两种云服务方案分别是亚马逊推出的 EC2 和谷歌推出的 App Engine (GAE)。这两种方案分别代表了两种思路：

- EC2 选择了提供底层基础，它的实例使用起来和一台物理服务器十分类似，没有任何的额外功能，你可以在上面运行任何类型、任何语言的服务；
- GAE 则选择了提供高层抽象，包括令人印象深刻的自动缩扩容等能力，但同时对用户能够运行的代码做出了限制 —— 要想获得这些特性，就必须使用google提供的储存和计算服务，遵循相应的规范。

最终市场选择了AWS的 EC2，这主要是因为开发者们更倾向于使用和自己本地开发环境相同的环境来运行服务，这样做，开发好的代码基本不需要什么改动就可以轻易部署到云实例上去。但这种模式虽然极大的给予了开发者自由度，但也意味着几乎所有的运维操作都交由开发者自己去解决，**因此大部分的云服务的使用者在使用云服务的同时，不得不承担复杂的运维成本以及较低的硬件使用率。**Serverless的产生就是为了解决这些问题而产生的。<br />              ![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420346-a2281eb5-256e-4772-9948-f75899359077.png#align=left&display=inline&height=120&originHeight=120&originWidth=619&size=0&status=done&style=none&width=619)




而回顾Serverless 这一名词的诞生，需要我们将时间线拉回到2012年，这是 Serverless 这一技术名词第一次出现在大家视野中的时间点。Ken 在他的文章：[Why The Future Of Software And Apps Is Serverless](https://readwrite.com/2012/10/15/why-the-future-of-software-and-apps-is-serverless/?__cf_chl_jschl_tk__=2e846b39e94ae54ff7af11c6e12cb23f8f25fff2-1607050809-0-AeqVODmUcZBg0qZXOwSFRle_BUy9MciSv0JnWmm3Hiqwj2-jHoGnAd3tb9GI8WGww5wC-kPL2RcPcFxrTmFLM-QwUsy_sAoBzJQvnAd11W-zLVnwQUloDUsQeUzNGO2neSMmiOB_4XA9rsRy9DE4qgHGygZNjzSb2QkSA98u_ugxxccMRFFUnB0qxmdhqZJqXMzAVdQq57i9Y38dFbBqut4mfF378CGZDOZDlS1QXb_kdHB1Tcagke3c35Lhw6kNQU7OsMrEEddGEfntErkpHrMQCdEaIR8RJLsYxhI65MKw2GQuKXLNRaMlgQBWUJD9nN_fQ90SLMZJA0sTzk6bhoSNKkQokDchXc9eEzSmzK25xmynuK9oAdQ5r4D1oic601OCwrW1A64_1lGYxNA_wdaDtifGDxDztBlzsNCj2rmRyx7qAZlvPGa4nLjSFc1BtQ) 中提出了 Serverless 之一名词，开始让 Serverless 进入大家的视野。引述这篇文章的一段话，算是对 Serverless 早期定义的解释：

> 
## Thinking Serverless
> The phrase “serverless” doesn’t mean servers are no longer involved. It simply means that developers no longer have to think that much about them. Computing resources get used as services without having to manage around physical capacities or limits.


总的来说，这个时间点的 Serverless，更多的是关于对于计算机底层运维方面的抽象的讨论，也算是去思考如何解决 **复杂的运维成本** 这一问题。


而真正让 Serverless 名声大噪的是 Amazon 在 2015 年发布的 [AWS Lambda](https://aws.amazon.com/lambda/) ，提出了 **Cloud Function **的概念，让 Serverless 提高的一个全新的高度，它不仅仅通过抽象底层运维能力，来为云开发者提供运维能力的支持以及抽象，并且提供快速缩扩容以及按调用收费的机制，提升了资源利用率，降低使用者的成本。这也是第一个真正意义上我们今天所说的 Faas 平台，正是从那一年开始， Serverless 开始成为国际上炙手可热的名词，出现在各大云计算的会议之上。


而到了2017年，国内的 Paas 以及 laas 平台，也推出了自己的函数计算平台，加入到了 Serverless 的推广以及建设当中。


## 1.2 Serverless 的技术组成

明确了Serverless的产生背景，当今社区上对Serverless的定义其实还是比较模糊的，但总的来讲，翻阅一些较为权威的资料，大体上还是较为相同的，譬如号称 Serverless 白皮书的：[《Cloud Programming Simplified: A Berkeley View onServerless Computing》](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf)中关于 Serverless 的定义是：

> Put simply, serverless computing  =  FaaS + BaaS，In our definition, for a service to be considered serverless, it must scaleautomatically with no need for explicit provisioning, and be billed based on usage.


提取几个关键字：serverless = FaaS + Baas，且必须能够实现**自动缩扩容**和**按使用量计费**。另外在[《Serverless Architectures》](https://martinfowler.com/articles/serverless.html)，也是将 Serverless 视为 FaaS 和 BaaS 的结合：<br />![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420359-1a7ee5f1-7dee-4149-974e-57f72f976876.png#align=left&display=inline&height=716&originHeight=716&originWidth=3140&size=0&status=done&style=none&width=3140)<br />因此在这里，我们在此讨论的 Serverless 也就按照 Serverless = FaaS + BaaS 的定义了（但其实我个人更倾向于将Serverless视为一种降低开发门槛，提升开发效率的架构模式） ：<br />![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420339-9f4633da-87bb-4642-892b-ad06da227022.png#align=left&display=inline&height=396&originHeight=396&originWidth=762&size=0&status=done&style=none&width=762)

其中 FaaS（Functions as a Service）直译过来就是：函数即服务。FaaS 是无服务器计算的一种形式，当前使用最广泛的是 AWS 的 Lambada 函数计算平台。FaaS 本质上是一种事件驱动的由消息触发的服务，FaaS 供应商一般会集成各种同步和异步的事件源，通过订阅这些事件源，可以突发或者定期的触发函数运行。


而这里的函数，则是提供了比微服务更微细小的程序单元。比如，我们可以将微服务按照某个用户特定的一系列CRUD操作进行拆分。而在FaaS下，用户的每个操作，比如创建这一操作，就对应着我们在函数计算平台上的一个函数，只要通过触发器触发它，就可以执行操作事件。下面这个图就很形象的体现函数计算的特点：<br />![](https://cdn.nlark.com/yuque/0/2020/gif/296173/1609222420356-0994bff1-0304-457f-a6f1-597941071c2e.gif#align=left&display=inline&height=457&originHeight=457&originWidth=1024&size=0&status=done&style=none&width=1024)<br />（图来自：[https://developer.aliyun.com/article/574222](https://developer.aliyun.com/article/574222)）<br />而 BaaS（Backend-as-a-Service）后端即服务，它是基于 API 的第三方服务，用于实现应用程序中的后端功能核心功能，包含常用的数据库、对象存储、消息队列、日志服务等等。

下面这个表格列举了一下传统的Serverful(也就是云计算)和 Serverless 的区别：

|  | <br />特性 | AWS Serverless 云计算 | AWS Serverful 云计算 |
| --- | :--- | :--- | :--- |
| 开发者 | 何时运行程序 | 由用户根据事件自行选择 | 除非明确停止，否则会一直运行。 |
|  | 编程语言 | JavaScript、Python、Java、Go等有限的语言 | 任何语言 |
|  | 程序状态 | 保存在存储（无状态） | 任何地方（有状态或无状态） |
|  | 最大内存大小 | 0.125~3GiB（用户自行选择） | 0.5~1952GiB（用户自行选择） |
|  | 最大本地存储 | 0.5GiB | 0~3600 GiB （用户自行选择） |
|  | 最长运行时间 | 900秒 | 随意 |
|  | 最小计费单元 | 0.1秒 | 60秒 |
|  | 每计费单元价格 | $0.0000002 | $0.0000867 - $0.4080000 |
|  | 操作系统和库 | 云供应商选择 | 用户自行选择 |
| 系统管理员 | 服务器实例 | 云供应商选择 | 用户自行选择 |
|  | 扩展 | 云供应商负责提供 | 用户自己负责 |
|  | 部署 | 云供应商负责提供 | 用户自己负责 |
|  | 容错 | 云供应商负责提供 | 用户自己负责 |
|  | 监控 | 云供应商负责提供 | 用户自己负责 |
|  | 日志 | 云供应商负责提供 | 用户自己负责 |


转自：[https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf)


总的来说，Serverless 相较于 serverful，有以下三个方面的巨大改变：

1. 弱化了存储与计算之间的联系。将服务的存储以及计算分开部署以及计费，服务的存储变成独立的服务，而计算则变得无状态化，从而变得更有利于调度和缩扩容。
2. 代码的执行不再需要手动分配资源，只需提供一份代码，其他的资源的调度以及分配都交由Serverless平台去完成
3. 按使用量计费。 serverless按照服务的使用量进行计费，而不是像传统的serverful服务那样，按照使用的资源（ECS实例、VM的规格等）计费。


# 二、 Serverless和前端结合的落地场景

从实用角度出发，那Serverless从前端开发工程师的角度来讲，可以让我们更专注于业务开发，一些常见的服务端问题，我们都可以交给 Serverless 来解决，比如：

- Serverless 不需要关心内存泄露等问题, 因为它的云函数服务是使用完即销毁
- Serverless 不需要我们自己搭建服务端环境, 也不需要我们自己去预估流程的峰值，以及关心资源的利用率、容灾等问题，因为它自身可以根据流量快速扩所容，并按真实使用量计费
- Serverless 有完善的配套服务, 如云数据库, 云消息队列, 云存储等, 充分利用这些服务，可以极大的扩宽我们能力边界，做我们之前没时间或者没有能力去做的事情

下面是伯克利的论文中，所列举出来的关于2018年 Serverless 的具体使用场景的分布：<br />![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420363-ca0bc095-1655-4e6f-bcbe-aa40008bc59e.png#align=left&display=inline&height=610&originHeight=610&originWidth=1700&size=0&status=done&style=none&width=1700)


## 2.1 小程序云开发

按照上图中，Serverless使用场景中，占比最高的就是Web和API 服务，这方面比较典型的开发场景就是 小程序的云开发了。


在传统的小程序开发流程中，我们需要前端工程师对小程序端进行开发，而后端工程师进行服务端的开发。如果开发的团队规模较小，可能还需要前端工程师去将服务端的开发也完成了，但由于小程序的后端开发其实和其他的后端应用本质上是一样的，需要关心应用的负载均衡、容灾、监控等一些运维操作，但这些知识又触及到了大部分前端工程师的知识盲点，往往需要很多时间去了解和学习，完成的产品也往往不尽人意。


而在基于 Serverless 的小程序云开发的模式下，就可以做到让开发者只关心业务需求的实现，由一个前端工程师参与开发，在不具备完善的运维知识的情况下，使用云开发平台将后端功能封装而成的 BaaS 就可以完成整个应用的开发。以下是微信小程序云开发所提供的几项基础能力支持：

| 能力 | 作用 | 说明 |
| :--- | :--- | :--- |
| 云函数 | 无需自建服务器 | 在云端运行的代码，微信私有协议天然鉴权，开发者只需编写自身业务逻辑代码 |
| 数据库 | 无需自建数据库 | 一个既可在小程序前端操作，也能在云函数中读写的 JSON 数据库 |
| 存储 | 无需自建存储和 CDN | 在小程序前端直接上传/下载云端文件，在云开发控制台可视化管理 |
| 云调用 | 原生微信服务集成 | 基于云函数免鉴权使用小程序开放接口的能力，包括服务端调用、获取开放数据等能力 |
| [微信支付](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/wechatpay.html) | 免鉴权原生使用微信支付 | 免签名计算、免 access_token 使用微信支付能力 |


（详情可看：[微信官方文档-小程序-云开发](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html)）

具体实践案例：[miniprogram-foodmap](https://github.com/cloudkits/miniprogram-foodmap)


## 2.2. 数据编排，从 BFF 到 SFF

BFF对于大多数的前端工程师已经不再陌生了，它的产生主要是基于：对不同的设备可能需要使用不同的后端 API，也可能对数据格式以及数据量有不同的要求。因此BFF所做的工作通常就是将后端的数据以及接口进行编排，适配成前端所需要的数据格式，提供给前端进行使用。具体的模型如下：<br />            ![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420346-a243867b-83de-45e1-b430-cc084a2c60a3.png#align=left&display=inline&height=419&originHeight=419&originWidth=628&size=0&status=done&style=none&width=628)<br />不过虽然这种模式虽然解决了接口协调的问题，但也带来了一些新的问题：

- 如果针对不同的设备都需要开发一个BFF应用，这无疑回面临一些重复开发的成本，
- BFF 层通常是由善于处理高网络 I/O 的Node 应用负责的，而传统的服务端运维 Node 应用还是较重的，需要我们去购买虚拟机或者将其托管到 PaaS 平台，但基于微服务高可用的诉求，就会导致服务器资源的浪费。
- 前端之前完全不用去考虑并发的情况，只需关系页面的渲染，而在加入BFF后，高并发的压力也集中到了 BFF 上。

而 Serverless 则可以帮我们很好的解决这些问题。由于 BFF 只是做无状态的数据编排，因此它天然就是适合 FaaS 这种按需分配，弹性扩容，用完即毁的模型进行替换。我们可以使用一个个函数来实现对各个接口的聚合或者裁剪，前端向 BFF 发起请求，就相当于是 FaaS 的一个 HTTP 触发器，触发一个函数的执行，由这个函数来针对具体的业务逻辑来发起请求获取数据，再对数据进行聚合以及裁剪，最后将数据返回给前端。<br />                  ![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420357-7d75c80a-0417-462b-920a-c1815e765645.png#align=left&display=inline&height=497&originHeight=497&originWidth=570&size=0&status=done&style=none&width=570)<br />这样做的好处就是一方面可以节省资源，降低成本，不用去一直维持Node服务的虚拟机的开销，另一方面，将运维的压力也从BFF转移到来FaaS 服务，前端无需关心BFF的维护以及并发等场景。此外，我们还可以在FaaS平台中去充分利用云服务提供商所提供的其他功能，从而实现服务编排，增强我们在SFF层的能力。


# 三、 Serverless 的未来与展望


## 3.1  对 Serverless 的“火”保持理性

虽然现在 Serverless 这一概念已经被市场上的相关利益者吹的很火，仿佛已经马上就可以对传统的开发方式进行革命，但作为普通的开发者，我们需要保持相对理性，才能更为客观的去了解一门技术。


首先，Serverless 是真的已经火热到家喻户晓，成为一个大家都应该去了解的技术，去拥抱的开发模式了么？<br />下面是 Google Trends 对于三个名词的搜索热度的排名，可以看到与前端强相关的两个技术名词 graphql 和 BFF 都比 Serverless 的搜索热度高出不少。![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420351-9d8ad812-36d8-47df-956a-904514f78727.png#align=left&display=inline&height=1272&originHeight=1272&originWidth=2596&size=0&status=done&style=none&width=2596)<br />其次，作为一个出自美国的技术，同时AWS 的 [Lambda](https://aws.amazon.com/lambda/) 无论在技术成熟度已经服务水平都比国内走在前面的情况下，按 Google Trends 区域搜索热度划分，也很有意思：其中中国以100遥遥领先于其他国家，第二名的新加坡才17的热度。

因此，至少从这个数据来看，Serverless 在国内的关注度远高于国外的（当然这也与国内的实际项目发展需求有关，serverless天然的具有一定的落定场景）


## 3.2 当前 Serverless 的局限性

那 Serverless 在经过这几年的发展，为什么没有真正的火起来呢，首先就是这项技术本身所存在的一些问题，下列是伯克利的论文中列举出的 Serverless 现今仍然存在的四个不足，或者说是阻碍它快速发展的因素：

![](https://cdn.nlark.com/yuque/0/2020/jpeg/296173/1609222420371-d265405f-9e81-42f1-8250-0c46e946f5f7.jpeg#align=left&display=inline&height=556&originHeight=556&originWidth=1382&size=0&status=done&style=none&width=1382)而正式由于存在以上一些原因，导致复杂的企业级的业务系统无法基于现在这么简单的Faas来实现，只有一些业务场景较为简单的应用才是它现在的落地场景，而一个架构思想想要成为主流，得到快速的发展，必须要应用在企业主要流程的业务系统之中，只有这样，才能体现它在企业中所带来的巨大价值与收益。因此如何结合Serverless 的思想，落地于企业的核心业务场景中去，展现其真正价值，为企业带来降本增效的收益，并沉淀出强大的 Serverless 开发框架以及最佳实践，才能将 Serverless 推向云时代的主流架构这一宝座。


## 3.3 Serverless 的发展展望

这里直接引用伯克利对与 Serverless 计算在未来十年的发展展望以及趋势的预测：<br />![](https://cdn.nlark.com/yuque/0/2020/png/296173/1609222420424-9060ab13-8ea7-4cf1-b324-140a11070b65.png#align=left&display=inline&height=1044&originHeight=1044&originWidth=1746&size=0&status=done&style=none&width=1746)<br />其中对我个人印象最为深刻的是最后一条：

> Serverless computing will become the default computing paradigm of the Cloud Era, largely replacing serverful computing and thereby bringing closure to the Client-Server Era.


简单来说，他们认为 Serverless Computing 将会成为云时代的默认范例大面积的替换传统的云计算，并革命掉客户端-服务器端的时代。我对这个展望个人是比较认同的，因为从宏观角度来看，技术的发展必定是一个不断降低门槛的过程，抽象底层逻辑，提升开发效率的过程。而 Serverless 架构的核心思想，按照 AWS 的 CTO 的说法, Serverless 作为一个架构模式，要做的到的是：

> "Everyone wants just to focus on business logic."


而这也真符合商业发展对降本增效的诉求，因此从这个角度出发，Serverless架构的落地以及推广，未来可期。


参考链接

- [《Cloud Programming Simplified: A Berkeley View onServerless Computing》](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf)
- [Serverless For Frontend 前世今生](https://zhuanlan.zhihu.com/p/77095720)
- [从IaaS到FaaS—— Serverless架构的前世今生](https://aws.amazon.com/cn/blogs/china/iaas-faas-serverless/)
- [当我们在聊Serverless时你应该知道这些](https://developer.aliyun.com/article/574222)
- [探索 Serverless 中的前端开发模式(多场景)](https://mp.weixin.qq.com/s/ZJBWGxkQezCIl2MFnHIsbQ)
- [Serverless Architectures](https://martinfowler.com/articles/serverless.html)

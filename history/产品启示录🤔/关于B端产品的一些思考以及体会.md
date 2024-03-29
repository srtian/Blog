这半年来，由于某些原因，我在开发之余，也承担了平台前端产品需求的挖掘，设计和梳理的工作，也做成了诸如公开数据集模块，模型自动化等需求的前端交互设计或者是总体的业务流程设计，有些还挺不错，受到了一些好评。做着做着就由感而发，也对B端的产品设计有了一些自己的思考，因此总结一下这半年来，在这方面的体会以及心路历程。


## 一、AI 产品的不确定性
相较于传统的B端产品，AI平台最大的特点就是不确定，这个不确定表现在很多方面。譬如：

- 业界没有明显的标杆性的产品可做参考，且就算部分友商也有类似的项目，也大多是内部使用，并没有开源。因此很多东西需要自己去探索，去踩坑。只有坑踩多了，才会有更明确的方向感，才会逐步去靠近所要解决的核心问题
- 另外，在做 AI 平台的时候，可以很明显的感受得到，有时候需求方对于自己想要的东西也是不确定的，有时候嘴上说着我们就需要简单的功能，这样会比较方便。但随着沟通的不断深入，需求就会不断的被挖出来（这点我体会很深，毕竟我可是疯狂给自己加需求小王子，手动狗头）
- 等等

这些不确定性，都让我们在产品的探索过程中，不得不去不断试错，从错误和失败中吸取教训，并逐步完善自己对于产品设计的方法论。比如：

- 做事情要聚焦，做平台也是，开发不是去堆砌功能，而是要发现问题，解决问题；在完成功能的开发后，要跟进这个功能的使用情况，及时收集种子用户的使用意见，并快速作出相应的改进。
- 做一个功能不能拍脑袋决定，不能觉得有用就去开发。这样成了还好，不成的话即浪费了自己的开发成本，又徒增平台的复杂性，得不偿失。合理的做法是，要想明白自己做的功能是给谁做的，解决了他什么问题。然后将这功能的原型准备好，找对应的同学好好聊聊，看他们是否真的觉得这个问题是他们的痛点，然后再决定是不是要做。

因此，总的来讲，对于像AI平台，这种不能充分吸取其他产品的设计经验、充满着不确定性的产品，首先要做的就是保持理性，保持克制，尽量减少试错成本，不断试错，快速迭代。（个人经验是如果方便的话，如果一个需求有对应的提出人，可以在完成一个简单的原型后就找到对应的需求提出人，确定基础功能的实现是符合他的预期的）

## 二、易用性

B端产品的一大特点就是普遍业务复杂度高，而AI平台在此基础上同时还有较高的学习成本。按照俞军老师的用户价值公式：
> 用户价值=新体验-旧体验-切换成本

因此如何增强用户价值，在做好新体验的同时，切换成本的缩小至关重要，这也是对于B端产品来说，易用性很重要的原因，因为它不仅可以作用于新体验，更可以在切换成本的缩小上提供巨大助力。

这里的成本，对于AI平台，我个人理解可以分为几个方面：

- 学习成本：名词较多，业务场景复杂，上手成本较高，需要耗费不少精力以及时间去学习。
- 资源迁移成本：AI平台做的很多事情，有时候算法工程师都有着自己的一些实现方式，只不过这些方式不利于统一管理或者是记录。譬如数据管理模块，模型管理模块这些模块，在进行迁移的时候，需要算法工程师将一些在开发集的资源迁移到平台这边来，而这些工作量也主要集中的数据的迁移，以及格式的对齐上。
- 心智耗费成本：这个成本也是由于平台本身业务以及功能的复杂所导致的。譬如说：AI平台，在多个模块相互关联的情况下，往往有时候发起一个任务，需要多个模块的资源进行协调，而这个时候用户有时候就不得不去记忆不同模块资源的对应ID，这其实是很让人心累（尤其是在命令行使用时）。

而减少这些成本，我们则可以从这几个方面入手（主要参考的是尼尔森十大可用性原则）：

- 人性化帮助
- 容错处理
- 防错原则
- 撤销重做原则
- 状态可见

（具体后面再写个介绍一下这一块）


## 三、情感化
在B端，很多时候都会忽略情感化的事情，但据我这几个月的实践来看，情感化其实在B端也是非常重要的一环。<br />对于B端我们可以从马斯洛五层出发，将实现的层次分为三个：

- 可用性
- 稳定性
- 情感化

一个好的情感化设计可以极大的提升用户体验，拉近平台与用户的距离。

对于情感化设计，我个人有以下一些心得：

- 设计合理，保持克制：相关的情感化设计，要和全平台的相关设计逻辑保持一致，且由于B端产品的主要目的其实是为了提升效率，因此我们的情感化设计也不能有损于我们的主要目标，这也需要我们对于这些情感化的添加要保持克制
- 明确用户，投其所好：对于自己要服务的用户，我们也需要明确，并对设计符合他们审美，以及使用习惯

如何去做：

- 产品引导
- 文字 or 邮件关怀
- 图形


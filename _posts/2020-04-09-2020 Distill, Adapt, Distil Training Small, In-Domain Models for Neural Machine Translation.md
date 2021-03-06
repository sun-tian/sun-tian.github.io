---
layout:     post           # 使用的布局（不需要改）
title:      2020 Distill, Adapt, Distill Training Small, In-Domain Models for Neural Machine Translation           # 标题 
subtitle:   Distill, Adapt, Distill #副标题
date:       2020-04-09             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/post-bg-coffee.jpeg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - nlp
    - paper
    - 机器翻译
    - 领域适应



---

# 2020 Distill, Adapt, Distill: Training Small, In-Domain Models for Neural Machine Translation

# Abstract

探讨了在领域适应设置中，利用序列级知识提取训练小型、记忆效率高的机器翻译模型的最佳实践。We explore best practices for training small, memory efficient machine translation models with sequence-level knowledge distillation in the domain adaptation setting.

在三对语言中 (Russian-English, German-English, Chinese- English)，每对语言有三个域(patents, subtitles, news, TED talks)，实验表明蒸馏distilling提取两次以获得最佳性能：

<u>一次使用通用域数据general-domain data，另一次使用适用教师的域内数据domain data with an adapted teacher。</u>

# 1 Introduction

​		机器翻译系统依赖于大量的数据来推断从一-种语言到另一种语言的翻译规则。这在一些重要的小众领域提出了挑战，如.专利和医学文献翻译，因为聘请专家生成合适的培训数据的成本很高。一种经济有效的替代方法是领域改编，它利用了大量来自不那公困难和容易获得的领域的并行文档，比如电影字幕和新闻文章。
​		领域适应在实践中效果良好。但是，这些大型数据集(我们称之为通用域数据集)会带来一些可伸缩性问题。大数据集需要大模型;神经机器翻译系统可能需要几天或几周的时间来训练。有些机型需要千兆字节的磁盘空间，因此部署到边缘计算设备上颇具挑战性。在推理期间，它们还可能需要过多的计算，这使得它们在生产环境中扩展时速度很慢，成本也很高。(戈登,2019)
​		为了缓解这些问题，知识蒸馏(又称教师-学生)[Hinton等人。用于将模型压缩成可管理的形式。但是，尽管知识蒸馏是实践中最常用的模型压缩形式，但它也是最不容易理解的一种形式。
​		在这项工作中，我们进行了大规模的实证分析，试图发现最佳实践时，使用知识蒸馏结合领域适应。在几种常识性配置中，我们发现知识提取的两个阶段具有最佳性能:一个阶段使用通用域数据，另一个阶段使用域内数据有一个适应的老师。我们在多种语言对(俄语-英语、德语-英语、汉语-英语)、领域(专利、字幕、新闻、TED演讲)和学生规模 上进行实验。

# 2 Background

## Domain Adaptation

​		通过在更容易访问的通用领域中利用大量数据，域适应有助于克服小众领域中缺乏高质量的培训数据的问题。领域适应通常是通过持续的训练来完成的[Luong and Manning,2015;Zoph etalo，2016]， 包括两个步骤:
1. 模型被随机初始化和训练，直到在通用域数据上收敛。
2. 使用步骤1得到的参数初始化新模型，并对其进行训练,直到收敛到域内数据集。

​		我们可以将域自适应视为从通用域数据集中提取有用的归纳偏差，将其编码并作为良好的权值初始化传递给域内模型。还有其他从通用域数据集中提取归纳偏差的方法(包括混合微调[Chu等人])。和成本加权[Chen等人]，持续训练是最常见的，也是本文的重点。

## Knowledge Distillation

​		知识蒸馏是一种通过利用计算更复杂的“教师”网络的概率分布来改进参数不足的“学生”模型的性能的方法。Kim和Rush[2016]提出了将知识蒸馏扩展到机器翻译的两种方法:单词级知识蒸馏和顺序级知识蒸馏。

更一般的序列级知识蒸馏包括三个步骤：
1. 一个大型的教师网络被随机初始化和训练，直到数据收敛。
2. 训练数据的源端使用教师来解码，以产生“提炼”的目标数据。
3. 一个较小的学生模型被随机初始化和训练，直到收敛到提取的源-目标对(丢弃数据中的原始目标序列)。

​        知识蒸馏的目标是训练学生模型模仿教师在译文上的概率分布。由于教师和学生是在相同的数据集上训练的，理论上他们应该能够学习相同的分布。然而，在实践中，把老师作为额外的训练信号可以提高学生的考试成绩。1对这一现象的解释包括黑暗知识[Furlanello等人2018]， 模式约简[Zhou等人]和正规化[戈登和杜赫，2019 年;盾等。,但没有明确的证据。序列级知识蒸馏在两个行业中都得到了广泛的应用[Xia等人]和研究，是本文的第二个重点。

# 3 Distilling and Adapting

​		领域适应和知识蒸馏如何相互作用，当应用在组合之前并不清楚。具体来说，我们的研究问题是:
- 经过提炼的模型更容易还是更难适应新领域?
- 知识蒸馏应该用于域内数据吗?如果是，应该如何训练教师?

​        为了回答这些问题，我们对图1中分配的9种可能的配置进行了实验。为了便于参考，我们将主要通过它们的配置编号来引用小型的域内模型，并鼓励读者参考图1。每个配置都有两个相关属性。
**Distilling In-Domain Data** 如何使用知识蒸馏对域内数据进行预处理?有些模型没有经过预处理(配置1、4和7),而另一”些人则使用教师对域内培训数据进行预处理。该教师可能是仅在域内数据(配置2、5和8)上培训的基线，或者可以在通用域数据上培训，然后通过继续培训(配置3、6和9)来适应域内数据。

**Initialzation** 如何初始化模型?-一个模型可以被随机初始化(配置1、2和3)，也可以从一个在通用域数据上训练的模型中进行调整。这个通用域模型可以是直接在通用域数据_上训练的基线(配置4、5和6)，也可以是在通用域老师的输出上训练的学生模型(配置7、8、9)。

![image-20200509143617844](https://tva1.sinaimg.cn/large/007S8ZIlly1gem6u9umamj30gi09mt9p.jpg)

**如何使用知识蒸馏对域内数据进行预处理？**

1，4，7，模型没有经过预处理

2，5，8，这个老师仅在通用数据上训练

3，6，9，先在通用域上训练然后通过持续训练来适应域内数据

**如何初始化模型？**

1，2，3，随机初始化

4，5，6，通用域数据上的训练模型

7，8，9，在通用域老师的输出上训练的学生模型

# 4 Experiments

## 4.1 Data

![image-20200509143706243](https://tva1.sinaimg.cn/large/007S8ZIlly1gem6v1bj96j30a0039mxg.jpg)

- 训练3种语言对的模型，每种是1个通用域数据集和2个域内数据集
- **通用域数据** OpenSubtitles2018（电影字幕），WMT2017（新闻字幕、议会程序和网络爬虫数据）
- **域内数据** WIPO提供的COPPA-V2（专利摘要），TED演讲（公开的演讲文本）
- **预处理** 所有数据集都用Moses tokenized，通用域数据是BPE vocabulary 30000 tokens. 这模拟了对单个通用域模型进行训练，然后根据遇到的新域对其进行调整，在域内数据上对BPE进行再训练以生成不同的词汇表将迫使我们重新构建模型，从而使适应变得不可能。
- **评估** 通用域的评估数据集包括newstest2016和OpenSubtitles2018的最后2500行，WIPO是保留的3000行，TED演讲保留了大约2000行
- 模型的评估是通过解码适当的开发集来执行的，波束搜索大小为10，并与使用multi-bleu的参考来进行比较

## 4.2 Architectures and Training

![image-20200509143751605](https://tva1.sinaimg.cn/large/007S8ZIlly1gem6vu7ygyj30ai04waap.jpg)

- 所有的模型都是Transformer，老师teacher使用大型超参数设置进行训练，中小学生Medium, Small, and Tiny students进行配置和语言域的设置。
- 对模型进行30万次更新，100个epoch的训练，直到模型在10个检查点（提前停止）上没有改进
- 持续的训练 学生会从一些组合蒸馏combination of the distilled and 未蒸馏的参考数据集un-distilled reference dataset的持续训练上受益
- 作者建议，只要开发精度有所提高，要用这个分数来代替没有继续训练的分数

# 5 Recommendations

## 5.1 Adapt Teachers

​		在本节中,我们将比较没有老师配置1、只训练领域内数据的老师配置2)和改编自一般领域的老师(配置3)的域内模型培训情况。表3列出了这两名教师在每个语言对和域中的表瑰。结果表明,除了德英两种语言的知识产权组织外,适应能力大大提高了所有领域内教师的绩效。5
​		表4显示了在各种设置下训练学生模型之前,使用这些教师提取域内数据的结果。我们看到,在几乎每一个案例中,使用适应性强的老师都能得到最好的或接近最好的结果。这在定程度上是意料之中的,因为发展分数越高的模型越容易成为更好的教师[张大坤,2018]。虽然对学生来说,知识蒸馏通常被视为“简化”数据,但在这种情况下,我们怀疑经过调整的教师关于通用域的知识是通过域内数据蒸馏传递给学生的。

## 5.2 Adapt the Best Student

​		我们还直接在通用域数据上训练小模型,并使它们适应域内数据。可能的配置是随机初始化(配置1),初始化从一个在通用域数据上训练的基线模型(配置4),或者初始化从个从通用域老师那里提取的学生模型(配置7)。表5显示了这些模型在通用域数据集上的性能,表6显示了它们在域内数据集上的性能。
​		适应性教师在域内数据集(0-2个BLEU)上获得了有限的收益,而在通用域数据上直接训练小模型则获得了更大的收益(5-10个BLEU)。我们认为这是因为有大量的数据需要充分揭示教师对译文的概率分布[ Fang et al]。,2019]。虽然一个适应的教师可能包含许多采自一般领域的信息但仅通过翻译更小的领域内数据集是无法向学生表达这些知识的。为了充分利用通用域数据,必须在通用域数据上直接训练小模型。6
​		我们还观察到,中等规模的模型并没有小到可以从~般领域的知识精馏中获益,因此它们的一般领域得分并没有因为精馏而提高。这些精简的中型模型(配置刀)在域内数据上的表现世比它们的基线对应部分(配置4稍差。实际上,图2显示域内性能与通用域性能大致线性相关,而不管在调整之前是否应用了精馏。
​		这意味着精馏不会影响模型的适应性,因此应该对具有最佳通用域性能的模型进行调整,而不管是否应用了精馏。在不使用精馏的情况下,采用精馏模型可以比采用基线模型提高最多1个蓝单位的性能。

## 5.3 Distill, Adapt, Distill

​		最后,我们测试这两种改进小的域内模型的方法是否正交。我们可能会假设,在通用域数据上直接训练小模型完全消除了调整教师或使用域内教师的需要。为了测试这一点,我们还训练了适应的学生模型
一个基线老师(配置8)和一个适应的老师(配置
​		表7显示,第二次使用领域内的数据对已适应的教师进行提取可以进一步提高已提取的模型的性能,而使用未适应的领域内教师有时会损害性能。
​		这些结果为我们提供了一个通用的方法来训练小的、域内的模型,使用知识蒸馏和域适应的组合
1. 提取通用域数据以提高通用域学生的性能。
2. 将步骤1中的最佳模型调整为域内数据。(2-10个蓝比没有适配好
3. 调整教师和提取领域再次。(0-2比没有或不适应的
老师更好)
		按照这个过程,将得到如图1所示的配置6或9。实际上,配置9执行得最好或接近最好(在0以内)。1BLEU在几乎所有情况下,如表9所示。对于那些没有经过常规蒸馏改进的中型模型,配置6的性能最好。
		在德语-英语WPo上培训的模型是一个例外,从通用领域进行调整并不能提高性能。这与表3的结果一致,表3显示适应性并不能提高教师。我们怀疑这是因为德语英语的W|Po数据集是所有域内数据集中最大的,因此没有必要进行调整。未来的工作也可能受益于数据集之间域相似性的量化[Brit等人]。,这将指导在此类情况下使用域适应。

## 5.4 Training Times

​		在这项工作中训练的模型总共需要10个月的单gpu计算时间。表10按模型大小和数据集对其进行了分解。
​		虽然两次提取可能会获得最佳性能,但它也增加了所需的计算时间。配置9不是训练单一的域内模型,而是需要训练一个通用域的老师和一个通用域的学生,然后对两者进行调整。这可以将训练模型所需的计算量增加24倍大量的计算还花费在使用教师模型解码通用域数据进行序列级知识提取上,这可能需要24天的GPU时间(使用10的波束大小和10的批处理大小)。这可以通过并行使用多个gpu任意加速,但是未来的工作可能会探索如何以种更便宜的方式提取教师。

# 6 Related Work

​		我们的工作是少数几个专门专注于培训小型、参数不足的领域内模型的工作之一。但是，也有类似的工作，它们不能直接进行比较，而是使用知识提炼来适应新的领域。
**Knowledge Adaptation知识适应** 利用知识蒸馏将知识从多个标记的源域转移到未标记的目标域。这与我们的设置形成了对比，我们的设置为通用域和域内数据都提供了标签。罗德等。[2017]引入了“知识适应”的概念，利用多层感知器为未标记的域内数据提供情绪分析标签。类似的工作包括迭代双域自适应[Zeng等人]。和领域转换网络[Wanget al. ，2019]。,2019]。 这些想法并不局限于机器翻译;孟等人最近的研究。" [2020]利用知识蒸馏训练域内语音识别系统，orbesi - arteaga 等人。[2019] 在磁共振成像扫描的分割上做了类似的工作。
**Compressing Pre-trained Language Models** 通过NMT中的持续训练来压缩预训练语言模型的域适配与预训练语言模型的思想以及对不同任务的微调密切相关，这些任务可能来自于不同于预训练数据的数据分布。由于语言模型在训练和评估时非常繁琐，因此更多地关注于知识提取的压缩方面。山等。[2019]， Sun 等。[2019],刘et al。[2019]独立证明了知识蒸馏可以在不影响下游任务的情况下压缩预先训练好的模型。唐等。[2019] 表明任务特定的信息可以从一个大的转换器中提取出来，变成一个更小的双向RNN。这些方法可以合理地推广到NMT的领
域自适应。

# 7 Conclusion

​		在这项工作中，我们进行了大规模的实证调查，以确定最佳实践时，使用序列级知识蒸馏和领域适应的组合。我们发现，从通用域调整模型可以使它们成为更好的老师，而使用通用域数据进行提取不会影响模型的适应性。这导致我们建议提取两次以获得最佳结果:一次是在通用领域，以尽可能提高学生的表现，另一次是使用适应的领域内教师。结果是稳健的多种语言对，学生规模，域内设置。

# References

1. [Hintonet al., 2015] Geoffrey Hinton, Oriol Vinyals, and Jeff Dean. **Distilling the knowledge in a neural network**. March 2015.（知识蒸馏技术的提出文章）
2. [Luong and Manning, 2015] Minh-Thang Luong and Christopher D Manning. **Stanford neural machine translation systems for spoken language domains**. In Proceedings of the International Workshop on Spoken Language Translation, pages 76 -79, 2015.（领域适应技术提出的文章）
3. [Zophet al., 2016] BarretZoph, Deniz Yuret, Jonathan May,
  and Kevin Knight. **Transfer learning for Iow-Resource neural machine translation**. In Proceedings of the 2016 Conference on Empirical Methods in Natural Language Processing, pages 1568- -1575, Stroudsburg, PA, USA, 2016. Association for Computational Linguistics.（迁移学习技术在NMT的提出文章）
4. [Chuet al., 2017] Chenhui Chu, Raj Dabre, and Sadao Kurohashi. **An empirical comparison of domain adaptation methods for neural machine translation**. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (Volume 2: Short Papers), pages 385- -391, Stroudsburg, PA, USA, 2017. Association for Computational Linguistics.（从通用域数据集中提取归纳偏差的方法：混合微调方法）
5. [Chenet al, 2017] Boxing Chen, Colin Cherry, George Foster, and Samuel I .arkin. **Cost weighting for neural machine translation domain adaptation**. In Proceedings of the First Workshop on Neural Machine Translation, pages 40- 46, Stroudsburg, PA, USA, 2017. Association for Computational Linguistics.（从通用域数据集中提取归纳偏差的方法：成本加权方法）
6. [Kim and Rush, 2016] Yoon Kim and Alexander M Rush. **Sequence-Level knowledge distillation**. June 2016.（提出将知识蒸馏应用到机器翻译上）


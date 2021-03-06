---
layout:     post           # 使用的布局（不需要改）
title:      2019 Cross-lingual Language Model Pretraining             # 标题 
subtitle:   2019 Cross-lingual Language Model Pretraining   #副标题
date:       2020-06-12             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/post-bg-coffee.jpeg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - nlp
    - paper
    - 机器翻译
    - 预训练模型


---

# 2019 Cross-lingual Language Model Pretraining

![image-20200612180113436](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpntvn0ipj30l004at9a.jpg)

[Github-[XLM]](https://github.com/facebookresearch/XLM)

# Abstract

​		最近的研究表明生成式预训练对于英语自然语言理解的效率。在这项工作中，我们将这种方法扩展到多种语言，并展示了跨语言预培训的有效性。我们提出了两种学习跨语言模型（XLM）cross-lingual的方法：一种仅依靠单语言数据的无监督方法，另一种利用并行数据和新的跨语言模型目标的监督方法。我们获得了跨语言分类，无监督和有监督的机器翻译的最新结果。在XNLI上，我们的方法以4.9％的绝对精度提高了技术水平。在无人监督的机器翻译中，我们在WMT’16的德语-英语中获得了34.3 BLEU，比以前的水平提高了9 BLEU。在有监督的机器翻译中，我们在WMT’16罗马尼亚英语上获得了38.5 BLEU的最新技术水平，比以前的最佳方法高出4 BLEU。我们的代码和预先训练的模型将公开提供。

# 1 Introduction

​		句子编码器的生成式预训练（Radford等人，2018; Howard and Ruder，2018; Devlin等人，2018）导致对许多自然语言理解基准的大力改进（Wang等人，2018）。在这种情况下，在大型无监督文本语料库上学习了Transformer（Vaswani等，2017）语言模型，然后对自然语言理解（NLU）任务（例如分类（Socheret等，2013）或自然语言）进行微调。语言推论（Bowman等，2015； Williams等，2017）。尽管对学习通用句子表达的兴趣激增，但该领域的研究基本上是单语的，并且主要围绕英语基准测试（Conneau和Kiela，2018年; Wang等人，2018年）。学习和评估多种语言中的跨语言句子表示的最新发展（Conneau et al。，2018b）旨在减轻以英语为中心的偏见，并建议可以构建通用的跨语言编码器，将任何句子编码为共享的嵌入空间。

​		在这项工作中，我们证明了在多种跨语言理解（XLU）基准上进行跨语言语言模型预训练的有效性。准确地说，我们做出了以下贡献：

1. 我们介绍了一种使用跨语言模型来学习跨语言表示形式的无监督新方法，并研究了两个单语言的预培训目标。

2. 我们引入了一个新的监督学习目标，当可以使用并行数据时，可以改善跨语言的预培训。

3. 在跨语言分类，无监督机器翻译和有监督机器翻译方面，我们的表现明显优于现有技术。

4. 我们证明，跨语言语言模型可以对资源匮乏的语言的复杂性提供显着的改进。

5. 我们将公开提供我们的代码和预先训练的模型。

# 2. Related Work

​		我们的工作建立在Radford等人的基础上。 （2018）;霍华德和鲁德（2018）; Devlin等。 （2018）研究了用于预训练Transformer编码器的语言建模。他们的方法极大地改善了GLUE基准中的几个分类任务（Wang等，2018）。 Ramachandran等。 （2016年）表明，语言建模预训练还可以显着改善机器翻译任务，即使对于像英德这样的高资源语言对，也存在大量并行数据。与我们的工作同时，在BERT存储库1上展示了使用跨语言建模方法进行跨语言分类的结果。我们将这些结果与第5节中的方法进行比较。

​		从文字嵌入对齐和Mikolov等人的工作开始，对齐文本表示的分布已有很长的传统。 （2013a）利用小型词典来对齐来自不同语言的单词表示。一系列后续研究表明，可以使用跨语言表示来提高单语言表示的质量（Faruqui和Dyer，2014年），正交变换足以对齐这些单词分布（Xing等人，2015年），并且所有这些技术都可以应用于任意数量的语言（Ammar等，2016）。经过这一工作，跨语言监督的需求进一步降低（Smith等，2017），直到完全消除（Conneau等，2018a）。在这项工作中，我们通过对齐句子的分布并减少了对并行数据的需求，使这些想法更进一步。

​		对齐多种语言的句子表示形式的工作量很大。通过使用并行数据，Hermann和Blunsom（2014）； Conneau等。 （2018b）; Eriguchi等。 （2018）研究了零射跨语言句子分类。但是，最近最成功的跨语言编码器方法可能是Johnson等人的方法之一。 （2017）用于多语言机器翻译。他们表明，通过使用单个共享的LSTM编码器和解码器，可以使用单个序列到序列模型对多种语言对执行机器翻译。他们的多语言模型在资源匮乏的语言对上胜过最新技术，并实现了零镜头翻译。遵循这种方法，Artetxe和Schwenk（2018）表明所得的编码器可用于产生跨语言的句子嵌入。他们的方法利用了超过2亿个并行句子。他们通过学习基于固定句子表示的分类器，从而在XNLI跨语言分类基准（Conneau等，2018b）上获得了最新的技术水平。尽管这些方法需要大量并行数据，但最近在无监督机器翻译中的工作表明，句子表示可以完全无监督的方式对齐（Lample等人，2018a; Artetxe等人，2018）。例如，Lample等人。 （2018b）在WMT’16德语-英语中获得了25.2 BLEU，而没有使用平行句子。与这项工作类似，我们证明了我们可以以完全无监督的方式对齐句子的分布，并且我们的跨语言模型可以用于多种自然语言理解任务，包括机器翻译。

​		与我们最相似的作品可能是Wada和Iwata（2018）的作品，作者在其中用来自不同语言的句子训练了LSTM（Hochreiter and Schmidhuber，1997）语言模型。它们共享LSTM参数，但是使用不同的查找表来表示每种语言中的单词。他们专注于对齐单词表示，并表明他们的方法在单词翻译任务中效果很好。

# 3 Cross-lingual language models

​		在本节中，我们介绍了在整个工作过程中考虑的三个语言建模目标。 其中两个仅需要单语数据（无监督），而第三个则需要并行句子（有监督）。 我们考虑N种语言。 除非另有说明，否则我们假设我们有N个单语语料库{C i} i = 1 ... N，我们用n i表示C i中的句子数。

## 3.1 Shared sub-word vocabulary

​		在我们所有的实验中，我们使用通过字节对编码（BPE）创建的相同共享词汇来处理所有语言（Sennrich等，2015）。 如Lample等人所示。 （2018a），这大大改善了在共享相同字母或锚定标记（例如数字（Smith等人，2017））或专有名词的语言之间嵌入空间的对齐方式。 我们从单语语料库中随机抽取的句子的串联中学习了BPE拆分。 根据概率为{q i} i = 1 ... N的多项式分布对句子进行采样，其中：

![image-20200612180524617](https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@master/assets/picgoimg/20200720171601.jpg)

​		我们认为α= 0.5。 使用这种分布进行采样会增加与低资源语言相关联的令牌的数量，并减轻对高资源语言的偏见。 特别地，这防止了资源匮乏语言的单词在字符级别被分割。

## 3.2 Causal Language Modeling(CLM)

​		我们的因果语言建模（CLM）任务包括一个经过训练的变压器语言模型，该模型经过训练以给定句子P（w t | w 1，...，w t-1，θ）中给定先前单词的单词的概率。 虽然递归神经网络在语言建模基准方面获得了最先进的性能（Mikolov等人，2010; Jozefowicz等人，2016），但Transformer模型也非常具有竞争力（Dai等人，2019）。

​		在LSTM语言模型的情况下，通过向LSTM提供先前迭代的最后一个隐藏状态来执行时间反向传播（Werbos，1990）（BPTT）。 就变形金刚而言，先前的隐藏状态可以传递给当前批次（Al-Rfou等人，2018），以为批次中的第一个单词提供上下文。 但是，该技术无法扩展到跨语言设置，因此为了简单起见，我们仅在每批中保留第一个单词而没有上下文。		

## 3.3 Masked Language Modeling(MLM)

​		我们还考虑了Devlin等人的掩盖语言建模（MLM）目标。 （2018），也称为Cloze任务（Taylor，1953年）。 继Devlin等。 （2018），我们从文本流中随机抽取15％的BPE令牌，在80％的时间中将其替换为[MASK]令牌，在10％的时间内将其替换为随机令牌，而在10％的时间内将它们保持不变 时间。 我们的方法与Devlin等人的传销之间的差异。 （2018）包括使用任意数量的句子（以256个令牌截断）的文本流而不是句子对。 为了解决稀有令牌和频繁令牌（例如标点符号或停用词）之间的不平衡问题，我们还使用类似于Mikolov等人的方法对频繁输出进行了二次采样。 （2013b）：根据多项式分布对文本流中的令牌进行采样，其权重与其反转频率的平方根成正比。 我们的传销目标如图1所示。

## 3.4 Translation Language Modeling(TLM)

​		CLM和MLM目标均不受监督，仅需要单语数据。 但是，这些目标在可用时不能用于利用并行数据。 我们引入了新的翻译语言建模（TLM）目标，以改善跨语言的预培训。 我们的TLM目标是MLM的扩展，在这里我们不考虑单语文本流，而是连接如图1所示的并行句子。我们在源句子和目标句子中都随机屏蔽了单词。 为了预测英语句子中掩盖的单词，模型可以关注周围的英语单词或法语翻译，以鼓励模型将英语和法语表示对齐。 特别是，如果英语水平不足以推断出被掩盖的英语单词，则该模型可以利用法语环境。 为了便于对齐，我们还重置了目标句子的位置。

## 3.5 Cross-lingual Language Models

​		在这项工作中，我们考虑使用CLM，MLM或与TLM结合使用的MLM进行跨语言语言模型预训练。 对于CLM和MLM目标，我们使用64个连续句子流（由256个标记组成）的批次训练模型。 在每次迭代中，一批由来自相同语言的句子组成，这些句子是从上述分布{q i} i = 1 ... N中抽样的，其中α= 0.7。 当将TLM与MLM结合使用时，我们会在这两个目标之间交替使用，并以类似的方式对语言对进行采样。

# 4 Cross-lingual language model pretraining

​		在本节中，我们说明如何使用跨语言模型来获得：

- 更好的初始化句子编码器，实现零镜头跨语言分类
- 更好地初始化有监督和无监督的神经机器翻译系统
- 资源匮乏的语言的语言模型
- 无监督的跨语言单词嵌入

## 4.1 Cross-lingual classification

​		我们预先训练的XLM模型提供通用的跨语言文本表示形式。类似于在英语分类任务上的单语言模型优化（Radford等，2018； Devlin等，2018），我们在跨语言分类基准上对XLM进行了微调。我们使用跨语言自然语言推理（XNLI）数据集来评估我们的方法。准确地讲，我们在预训练的变压器的第一个隐藏状态之上添加了一个线性分类器，并微调了英语NLI训练数据集上的所有参数。然后，我们评估模型在15种XNLI语言中做出正确NLI预测的能力。继Conneau等。 （2018b），我们还包括训练和测试集的机器翻译基线。我们在表1中报告结果。

## 4.2无监督机器翻译

​		预训练是无监督神经机器翻译（UNMT）的关键要素（Lample等，2018a; Artetxe等，2018）。 Lample等。 （2018b）表明，用于初始化查询表的预训练交叉语言单词嵌入的质量对无监督机器翻译模型的性能有重大影响。我们建议通过对整个编码器进行预训练并取消对具有跨语言模型的编码器，以引导UNMT的迭代过程。我们探索了各种初始化方案，并评估了它们对几种标准机器翻译基准的影响，包括WMT'14英法，WMT'16英德和WMT'16英罗马尼亚。结果列于表2。

## 4.3监督机器翻译

​		我们还研究了跨语言建模的预训练对有监督的机器翻译的影响，并扩展了Ramachandran等人的方法。 （2016）改为多语言NMT（Johnson等，2017）。我们评估了CLM和MLM预培训对WMT’16罗马尼亚英语的影响，并在表3中列出了结果。

## 4.4低资源语言建模

​		对于资源匮乏的语言，通常最好以相似但资源较多的语言来利用数据，特别是当它们共享大量词汇时。例如，在Wikipedia上用尼泊尔语撰写的句子大约有10万，而在印地语中大约是句子的6倍。这两种语言在100k子词单位的共享BPE词汇表中，也具有超过80％的共同令牌。我们在表4中提供了尼泊尔语语言模型与使用尼泊尔语训练但丰富了北印度语和英语数据组合的跨语言模型之间的困惑比较。

## 4.5无监督的跨语言词嵌入

​		Conneau等。 （2018a）展示了如何通过将单语单词嵌入空间与对抗训练（MUSE）对齐来执行无监督单词翻译。 Lample等。 （2018a）显示，使用两种语言之间的共享词汇，然后将fastText（Bojanowski等人，2017）应用于其单语语料库的串联，也可以为共享语言的语言直接提供高质量的跨语言词嵌入（Concat）普通字母。在这项工作中，我们还使用了共享词汇，但是通过跨语言模型（XLM）的查找表获得了词嵌入。在第5节中，我们在三种不同的指标上比较了这三种方法：余弦相似度，L2距离和跨语言单词相似度。

# 5 实验与结果

​		在本节中，我们将通过经验证明跨语言模型的预训练对几个基准的强大影响，并将我们的方法与现有技术进行比较。

## 5.1培训细节

​		在所有实验中，我们使用的Transformer架构具有1024个隐藏单元，8个头部，GELU激活（Hendrycks和Gimpel，2016年），辍学率为0.1和学习的位置嵌入。我们使用Adam优化器（Kingma和Ba，2014），线性预热（Vaswani等人，2017）和10％到5.10 -4的学习速率训练模型。

对于CLM和MLM目标，我们使用256个令牌的流和大小为64的迷你批次。与Devlin等人不同。 （2018），如3.2节所述，小批量中的序列可以包含两个以上的连续句子。对于TLM目标，我们采样了4000个令牌的小批量，这些令牌由具有相似长度的句子组成。我们使用语言的平均困惑度作为培训的停止标准。对于机器翻译，我们仅使用6层，并创建2000个令牌的小批量。

在XNLI上进行微调时，我们使用大小为8或16的迷你批处理，并裁剪句子

长度为256个字。我们使用80k BPE拆分和95k的词汇表，并在XNLI语言的Wikipedia上训练12层模型。我们使用5.10 -4到2.10 -4的值对Adam优化器的学习率进行采样，并使用20000个随机样本的较小评估时期。我们使用变压器最后一层的第一个隐藏状态作为随机初始化的最终线性分类器的输入，并对所有参数进行微调。在我们的实验中，在最后一层使用最大池或均值池并没有比使用第一个隐藏状态更好。

我们在PyTorch中实现了所有模型（Paszke et al。，2017），并在64个Volta GPU上进行了语言建模任务，在8个GPU上进行了MT任务训练。我们使用ﬂ oat16操作来加快训练速度并减少模型的内存使用量。

## 5.2数据预处理

我们使用WikiExtractor 2从Wikipedia转储中提取原始句子，并将其用作CLM和MLM目标的单语数据。对于TLM目标，我们仅使用涉及Conneau等人的涉及英语的并行数据。 （2018b）。准确地说，我们对法语，西班牙语，俄语，阿拉伯语和中文使用MultiUN（Ziemski等，2016），对印地语使用IIT孟买语料库（Anoop等，2018）。我们从OPUS 3网站Tiedemann（2012）中提取以下语料库：用于德语，希腊语和保加利亚语的EUbookshop语料库，用于土耳其语，越南语和泰语的OpenSubtitles 2018，用于乌尔都语和斯瓦希里语的Tanzil以及用于斯瓦希里语的GlobalVoices。对于中文，日文和泰文，我们使用Chang等人的标记器。 （2008），Kytea 4标记器和PyThaiNLP 5标记器。对于所有其他语言，我们使用Moses提供的标记器（Koehn等，2007），必要时使用默认的英语标记器。我们使用fastBPE 6学习BPE代码并将单词分为子单词单元。按照第3.1节中介绍的方法，从所有语言中抽取的句子串联学习BPE代码。

## 5.3结果与分析

在本节中，我们演示了跨语言模型预训练的有效性。在跨语言分类，无监督和有监督的机器翻译方面，我们的方法明显优于现有技术。

跨语言分类在表1中，我们评估了两种类型的预训练跨语言编码器：一种仅在单语言语料库上使用MLM目标的无监督跨语言模型；以及监督的跨语言模型，该模型使用额外的并行数据将MLM和TLM损失结合在一起。继Conneau等。 （2018b），我们包括两个机器翻译基准：TRANSLATETRAIN，其中将英语MultiNLI培训集机器翻译成每种XNLI语言;以及TRANSLATE-TEST，其中将XNLI的每个开发和测试集翻译成英语。我们报告了Conneau等人的XNLI基线。 （2018b），Devlin等人的多语言BERT方法。 （2018）和Artetxe和Schwenk（2018）的最新作品。

我们完全不受监督的MLM方法为零镜头跨语言分类树立了新的技术水平，并且明显优于Artetxe和Schwenk（2018）的监督方法，后者使用了2.23亿个并行句子。准确地说，传销的平均准确度（Δ）为71.5％，而准确度为70.2％。通过TLM目标（MLM + TLM）利用并行数据，我们可以显着提高性能达到3.6％的准确度，甚至将现有技术水平进一步提高到75.1％。在斯瓦希里语和乌尔都语资源匮乏的语言上，我们分别比以前的先进技术高6.2％和6.3％。除了MLM外，使用TLM还可以提高英语的准确性83.2％至85％的准确性，优于Artetxe和Schwenk（2018）和Devlin等人。 （2018）通过准确度分别为11.1％和3.6％。

在每种XNLI语言（TRANSLATE-TRAIN）的训练集上进行微调时，我们的监督模型比零击方法优胜1.6％，达到了平均水平，平均准确性为76.7％。该结果尤其证明了我们方法的一致性，并表明可以在具有强大性能的任何语言上对XLM进行微调。与多语言BERT相似（Devlin等人，2018），我们观察到TRANSLATE-TRAIN的平均准确度比TRANSLATE-TEST的高出2.5％，此外，我们的零射方法比TRANSLATE-TEST的高出0.9％。

无监督机器翻译对于无监督机器翻译任务，我们考虑3种语言对：英语-法语，英语-德语和英语-罗马尼亚语。我们的设置与Lample等人的设置相同。 （2018b），除了初始化步骤外，我们使用跨语言建模来预训练完整模型，而不是仅使用查找表。

对于编码器和解码器，我们考虑不同的可能的初始化：CLM预训练，MLM预训练或随机初始化，这导致9种不同的设置。然后，我们遵循Lample等。 （2018b）并通过降噪自动编码损失和在线反向翻译损失来训练模型。结果报告在表2中。我们将我们的方法与Lample等人的方法进行了比较。 （2018b）。对于每种语言对，我们都观察到与现有技术水平相比有显着改进。我们重新实现了Lample等人的NMT方法。 （2018b）（EMB），并获得了比其论文中报道的更好的结果。我们希望这是由于我们的多GPU实施使用了大得多的批处理。在德语-英语中，如果仅考虑神经无监督方法，我们的最佳模型的性能要比以前的无监督方法高9.1个BLEU和13.3个BLEU。与仅对查找表（EMB）进行预训练相比，对使用MLM进行编码器和解码器进行预训练可以使德语德语上的一致性提高多达7个BLEU。我们还观察到，传销的目标预训练始终胜过传CLM的预训练，在EnglishFrench上从30.4降至33.4 BLEU，在RomanianEnglish上从28.0降至31.8。这些结果与Devlin等人的结果一致。 （2018）观察到更好的

38.5

表3：监督MT的结果。 BLEU在WMT’16罗马尼亚英语中得分。 Sennrich等人先前的最新技术。 （2016年）同时使用反向翻译和集成模型。 ro↔en对应于双向训练的模型。

与CLM相比，在针对MLM目标进行培训时，可以消除NLU任务。我们还观察到编码器是预训练的最重要元素：与对编码器和解码器进行预训练相比，仅对解码器进行预训练会导致性能显着下降，而仅对编码器进行预训练只会对最终结果产生很小的影响BLEU得分。

资源匮乏的语言模型在表4中，我们研究了跨语言语言模型对提高尼泊尔语言模型的困惑性的影响。为此，我们在Wikipedia上训练了尼泊尔语言模型，以及来自英语或北印度语的其他数据。尼泊尔语和英语是遥远的语言，尼泊尔语和北印度语是相似的，因为它们共享相同的梵文和梵语祖先。使用英语数据时，我们将尼泊尔语言模型的困惑度降低了17.1点，从仅尼泊尔语言模型的157.2降低为使用英语时的140.1。使用印地语的其他数据，我们将困惑度降低了41.6％。最后，通过利用英语和北印度语的数据，我们将尼泊尔语的困惑程度降低到109.3以上。跨语言建模所带来的困惑，可以部分由跨语言共享的n-gram锚点来解释，例如在Wikipedia文章中。因此，跨语言模型可以通过这些锚点来传递由印地语或英语单语语料库提供的其他上下文，以改善尼泊尔语言模型。

无监督的跨语言单词嵌入MUSE，Concat和XLM（MLM）方法提供了具有不同属性的无监督的跨语言单词嵌入空间。在表5中，我们研究了使用相同单词词汇的这三种方法，并根据MUSE词典计算了余弦相似度和单词翻译对之间的L2距离。我们还通过Camacho-Collados等人的SemEval’17跨语言单词相似度任务来评估余弦相似度度量的质量。 （2017）。我们发现XLM在跨语言单词相似度方面胜过MUSE和Concat，达到了0.69的Pearson相关性。有趣的是，在XLM跨语言单词嵌入空间中的单词翻译对也比MUSE或Concat更紧密。具体来说，MUSE的余弦相似度和L2距离分别为0.38和5.13，而XLM的相同度量标准分别为0.55和2.64。请注意，XLM嵌入的特殊性是与句子编码器一起训练，这可能会增强这种紧密性，而MUSE和Concat是基于fastText词嵌入的。

表5：无监督的跨语言词嵌入余弦相似度和源词及其翻译之间的L2距离。 Camacho-Collados等人在SemEval’17跨语言单词相似性任务中的Pearson相关性。 （2017）。

# 6 结论

在这项工作中，我们第一次展示了跨语言模型（XLM）预培训的强大影响。我们调查了仅需单语语料库的两个无监督训练目标：因果语言建模（CLM）和屏蔽语言建模（MLM）。我们证明CLM和MLM方法都提供了可用于预训练模型的强大的跨语言功能。在无人监督的机器翻译中，我们表明MLM预训练非常有效。我们在WMT’16的德语英语上达到了34.3 BLEU的最新水平，比以前的最佳方法高出9 BLEU。同样，我们在有监督的机器翻译方面获得了很大的改进。我们在WMT’16罗马尼亚英语达到了38.5 BLEU的新状态，相当于提高了4 BLEU积分。我们还证明了跨语言语言模型可以用来改善尼泊尔语言模型的困惑，并且它提供了无监督的跨语言词嵌入。在不使用单个并行句子的情况下，在XNLI跨语言分类基准上进行微调的跨语言模型已经平均比以前的监督技术先进了1.3％。我们工作的关键贡献是翻译语言建模（TLM）目标，该目标通过利用并行数据来改善跨语言模型的预培训。 TLM通过使用一批并行语句而不是连续语句自然地扩展了BERT MLM方法。除了使用MLM之外，我们还通过使用TLM获得了显着的收益，并且我们证明了这种受监督的方法平均以4.9％的精度击败了XNLI上的现有技术。我们的代码和预先训练的模型将公开提供。

# References

Rami Al-Rfou, Dokook Choe, Noah Constant, Mandy Guo, and Llion Jones. 2018. Character-level language modeling with deeper self-attention. arXiv preprint arXiv:1808.04444.

Waleed Ammar, George Mulcaire, Yulia Tsvetkov, Guillaume Lample, Chris Dyer, and Noah A Smith. 2016. Massively multilingual word embeddings. arXiv preprint arXiv:1602.01925.

Kunchukuttan Anoop, Mehta Pratik, and Bhattacharyya Pushpak. 2018. The iit bombay englishhindi parallel corpus. In LREC.

Mikel Artetxe, Gorka Labaka, Eneko Agirre, and Kyunghyun Cho. 2018. Unsupervised neural machine translation. In International Conference on Learning Representations (ICLR).

Mikel Artetxe and Holger Schwenk. 2018. Massively multilingual sentence embeddings for zeroshot cross-lingual transfer and beyond. arXiv preprint arXiv:1812.10464.

Piotr Bojanowski, Edouard Grave, Armand Joulin, and Tomas Mikolov. 2017. Enriching word vectors with subword information. Transactions of the Association for Computational Linguistics, 5:135–146.

Samuel R. Bowman, Gabor Angeli, Christopher Potts, and Christopher D. Manning. 2015. A large annotated corpus for learning natural language inference. In EMNLP.

Jose Camacho-Collados, Mohammad Taher Pilehvar, Nigel Collier, and Roberto Navigli. 2017. Semeval2017 task 2: Multilingual and cross-lingual semantic word similarity. In Proceedings of the 11th International Workshop on Semantic Evaluation (SemEval2017), pages 15–26.

Pi-Chuan Chang, Michel Galley, and Christopher D Manning. 2008. Optimizing chinese word segmentation for machine translation performance. In Proceedings of the third workshop on statistical machine translation, pages 224–232.

Alexis Conneau and Douwe Kiela. 2018. Senteval: An evaluation toolkit for universal sentence representations. LREC.

Alexis Conneau, Guillaume Lample, Marc’Aurelio Ranzato, Ludovic Denoyer, and Herv Jegou. 2018a. Word translation without parallel data. In ICLR.

Alexis Conneau, Ruty Rinott, Guillaume Lample, Adina Williams, Samuel R. Bowman, Holger Schwenk, and Veselin Stoyanov. 2018b. Xnli: Evaluating cross-lingual sentence representations. In Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing. Association for Computational Linguistics.

Zihang Dai, Zhilin Yang, Yiming Yang, William W. Cohen, Jaime Carbonell, Quoc V. Le, and Ruslan Salakhutdinov. 2019. Transformer-XL: Language modeling with longer-term dependency.

Jacob Devlin, Ming-Wei Chang, Kenton Lee, and Kristina Toutanova. 2018. Bert: Pre-training of deep bidirectional transformers for language understanding. arXiv preprint arXiv:1810.04805.

Akiko Eriguchi, Melvin Johnson, Orhan Firat, Hideto Kazawa, and Wolfgang Macherey. 2018. Zeroshot cross-lingual classiﬁcation using multilingual neural machine translation. arXiv preprint arXiv:1809.04686.

Manaal Faruqui and Chris Dyer. 2014. Improving vector space word representations using multilingual correlation. Proceedings of EACL.

Dan Hendrycks and Kevin Gimpel. 2016. Bridging nonlinearities and stochastic regularizers with gaussian error linear units. arXiv preprint arXiv:1606.08415.

Karl Moritz Hermann and Phil Blunsom. 2014. Multilingual models for compositional distributed semantics. arXiv preprint arXiv:1404.4641.

Sepp Hochreiter and J¨urgen Schmidhuber. 1997.

Long short-term memory. Neural computation, 9(8):1735–1780.

Jeremy Howard and Sebastian Ruder. 2018. Universal language model ﬁne-tuning for text classiﬁcation. In Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), volume 1, pages 328–339.

Melvin Johnson, Mike Schuster, Quoc V Le, Maxim Krikun, Yonghui Wu, Zhifeng Chen, Nikhil Thorat, Fernanda Vi´egas, Martin Wattenberg, Greg Corrado, et al. 2017. Googles multilingual neural machine translation system: Enabling zero-shot translation. Transactions of the Association for Computational Linguistics, 5:339–351.

Rafal Jozefowicz, Oriol Vinyals, Mike Schuster, Noam Shazeer, and Yonghui Wu. 2016. Exploring the limits of language modeling. arXiv preprint arXiv:1602.02410.

Diederik P Kingma and Jimmy Ba. 2014. Adam: A method for stochastic optimization. arXiv preprint arXiv:1412.6980.

Philipp Koehn, Hieu Hoang, Alexandra Birch, Chris Callison-Burch, Marcello Federico, Nicola Bertoldi, Brooke Cowan, Wade Shen, Christine Moran, Richard Zens, et al. 2007. Moses: Open source toolkit for statistical machine translation. In Proceedings of the 45th annual meeting of the ACL on interactive poster and demonstration sessions, pages 177–180. Association for Computational Linguistics.

Guillaume Lample, Alexis Conneau, Ludovic Denoyer, and Marc’Aurelio Ranzato. 2018a. Unsupervised machine translation using monolingual corpora only. In ICLR.

Guillaume Lample, Myle Ott, Alexis Conneau, Ludovic Denoyer, and Marc’Aurelio Ranzato. 2018b. Phrase-based & neural unsupervised machine translation. In EMNLP.

Tom´aˇs Mikolov, Martin Karaﬁ´at, Luk´aˇs Burget, Jan Cernock`y, and Sanjeev Khudanpur. 2010. Recurrent neural network based language model. In Eleventh Annual Conference of the International Speech Communication Association.

Tomas Mikolov, Quoc V Le, and Ilya Sutskever. 2013a.

Exploiting similarities among languages for machine translation. arXiv preprint arXiv:1309.4168.

Tomas Mikolov, Ilya Sutskever, Kai Chen, Greg S Corrado, and Jeff Dean. 2013b. Distributed representations of words and phrases and their compositionality. In Advances in neural information processing systems, pages 3111–3119.

Adam Paszke, Sam Gross, Soumith Chintala, Gregory Chanan, Edward Yang, Zachary DeVito, Zeming Lin, Alban Desmaison, Luca Antiga, and Adam Lerer. 2017. Automatic differentiation in pytorch. NIPS 2017 Autodiff Workshop.

Alec Radford, Karthik Narasimhan, Tim Salimans, and Ilya Sutskever. 2018. Improving language understanding by generative pre-training. URL https://s3-us-west-2.amazonaws.com/openaiassets/research-covers/languageunsupervised/language understanding paper.pdf.

Prajit Ramachandran, Peter J Liu, and Quoc V Le. 2016. Unsupervised pretraining for sequence to sequence learning. arXiv preprint arXiv:1611.02683.

Rico Sennrich, Barry Haddow, and Alexandra Birch. 2015. Neural machine translation of rare words with subword units. In Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics, pages 1715–1725.

Rico Sennrich, Barry Haddow, and Alexandra Birch. 2016. Edinburgh neural machine translation systems for wmt 16. arXiv preprint arXiv:1606.02891.

Samuel L Smith, David HP Turban, Steven Hamblin, and Nils Y Hammerla. 2017. Ofﬂine bilingual word vectors, orthogonal transformations and the inverted softmax. International Conference on Learning Representations.

Richard Socher, Alex Perelygin, Jean Wu, Jason Chuang, Christopher D Manning, Andrew Ng, and Christopher Potts. 2013. Recursive deep models for semantic compositionality over a sentiment treebank. In Proceedings of the 2013 conference on empirical methods in natural language processing, pages 1631–1642.

Wilson L Taylor. 1953. cloze procedure: A new tool for measuring readability. Journalism Bulletin, 30(4):415–433.

Jrg Tiedemann. 2012. Parallel data, tools and interfaces in opus. In LREC, Istanbul, Turkey. European Language Resources Association (ELRA).

Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, and Illia Polosukhin. 2017. Attention is all you need. In Advances in Neural Information Processing Systems, pages 6000–6010.

Takashi Wada and Tomoharu Iwata. 2018. Unsupervised cross-lingual word embedding by multilingual neural language models. arXiv preprint arXiv:1809.02306.

Alex Wang, Amapreet Singh, Julian Michael, Felix Hill, Omer Levy, and Samuel R Bowman. 2018. Glue: A multi-task benchmark and analysis platform for natural language understanding. arXiv preprint arXiv:1804.07461.

Paul J Werbos. 1990. Backpropagation through time:

what it does and how to do it. Proceedings of the IEEE, 78(10):1550–1560.

Adina Williams, Nikita Nangia, and Samuel R. Bowman. 2017. A broad-coverage challenge corpus for sentence understanding through inference. In NAACL.

Chao Xing, Dong Wang, Chao Liu, and Yiye Lin. 2015. Normalized word embedding and orthogonal transform for bilingual word translation. Proceedings of NAACL.

Michal Ziemski, Marcin Junczys-Dowmunt, and Bruno Pouliquen. 2016. The united nations parallel corpus v1. 0. In LREC.
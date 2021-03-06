---
layout:     post           # 使用的布局（不需要改）
title:      2020 Unsupervised Domain Adaptation for Neural Machine Translation with Iterative Back Translation           # 标题 
subtitle:   DAFE for MT #副标题
date:       2020-06-12             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/post-bg-coffee.jpeg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - nlp
    - paper
    - 机器翻译
    - 领域适应


---

# 2020 Unsupervised Domain Adaptation for Neural Machine Translation with Iterative Back Translation

![image-20200613092021795](https://tva1.sinaimg.cn/large/007S8ZIlly1gfqee8hhf6j30ry0520tr.jpg)

# Abstract

​		最先进的神经机器翻译（NMT）系统需要大量数据，并且在几乎没有监督数据的域上表现不佳。由于数据收集昂贵且在许多情况下不可行，因此需要无监督的域自适应方法。我们**对域内单语数据应用迭代反向翻译（IBT）训练方案，该方案反复使用基于Transformer的NMT模型在yy的一个翻译方向上创建域内伪并行句子对，然后使用它们进行训练另一个方向的模型**。在没有监督数据的情况下，对德语到英语翻译任务的三个域进行了评估，仅此简单的技术（没有任何域外并行数据）就已经可以超越以前的所有域自适应方法，比最强大的先前方法高出+9.48 BLEU方法，并且在不适用的基准上最多可增加+27.77 BLEU。此外，在德语到英语和罗马尼亚语到英语语言对的监督下，如果有可用的带域外数据，我们可以进一步增强性能，在最强的基线上获得+19.31 BLEU的改进，以及+47.69 BLEU的增量反对不适应的模式。

# 1 Introduction

​		当前的神经机器翻译（NMT）系统非常耗语料。尽管一些具有大量并行翻译数据的域（即高资源域）可以看到NMT的好处，但具有较小并行语料库的其他域（即低资源域，例如法律，医学和技术）仍然会遭受模型性能不佳。一种解决方案是在这样的域中收集相同类型的大型并行数据，这既昂贵又费时，在某些情况下甚至是不切实际的。
​		一种万能解决方案是开发有效的方法，将NMT模型推广到新领域，即领域适应。例如，我们可以通过训练NMT模型来对来自另一个高资源领域（如国会听证本）的平行翻译进行培训，从而将英语医疗文件翻译成德语。 **NMT领域适应的大多数先前工作都集中在有监督的背景下，那里仍然有少量并行的领域内数据**[Luong和Manning，2015年； 2012年。 Freitag和Al-Onaizan，2016年； [Chu and Wang，2018]。**但是，在某些领域中收集翻译对需要熟练的人类作家，并且并不是每个方都能负担得起或等待监督的数据。因此，我们的工作重点是无监督域自适应，其中没有域内并行数据可用**。<u>现有方法可分为两类：以数据为中心的方法和以模型为中心的方法。以数据为中心的方法依赖于伪并行语料库，该伪并行语料库通过将域内目标语料库复制到源端（称为Copy [Currey et al。，2017]）或将域内目标方句子与通过训练有素的NMT模型（称为反向翻译）来翻译他们的对应词[Sennrich等，2015]。以模型为中心的方法主要集中于对域外并行数据上的翻译任务和目标侧单语言域内数据上的语言建模任务的多任务学习[Domhan and Hieber，2017; Dou等人，2019]。</u>

​		尽管有各种先前的方法，但域内单语数据的丰富价值仍未得到充分挖掘。仅尝试了语言建模和一次性反向翻译。在过去两年中无监督NMT和文本样式转换的巨大成功的启发下，我们着重介绍了一种**迭代反向翻译（IBT）方案**，<u>以在我们的方法中更好地利用域内文本[Lample等人，2018; Artetxe等，2017; Jin等，2019]</u>。**该方案反复生成从源语言到目标语言的翻译，并训练翻译模型以将生成的数据映射回原始的源句子，然后我们执行相同的生成和训练过程，但方向相反。在训练中重复此过程，直到收敛为止**。<u>在没有任何并行或域外数据的情况下，我们发现仅此方法就已经超过了以前依赖于其他域数据的所有最新领域适应方法，证明了域内语料库可以更直接地受益到任务比域外配对数据</u>。值得注意的是，**我们提出的方法是以模型为中心的，因此独立于以数据为中心的方法，与之结合，我们可以获得进一步的显着改进**。此外，我们发现，通过屏蔽语言建模，在大规模无监督语料库上对NMT模型的编码器和解码器进行预训练[Lample和Conneau，2019]可以使我们的方法和其他流行的无监督域自适应方法都受益。

​		我们通过基于Transformer的NMT系统[Vaswani et al。，2017]在两种不同的数据设置和两种语言对下评估我们的方法。对于模型在两个特定域之间适应的第一个设置，我们的香草模型在四个基线模型中最强的基础上实现了+9.48 BLEU评分的改进，在未适应的基线上实现了+27.77 BLEU的改进。对于第二种设置，如果提供了通用域数据（WMT），则我们的模型与通过反向翻译进行的数据增强相结合，与以前的最佳方法相比，可实现+19.31 BLEU的改进，而与未经适应的模型相比，可实现+47.69 BLEU的改进。

# 2 Methods

​		在本节中，我们首先说明IBT的体系结构，然后制定总体培训策略。

## 2.1 Model Architecture

​	为了适应我们提出的框架，我们采用了Transformer [Vaswani et al。，2017]，该编码器具有用于序列到序列转换模型的编码器-解码器结构，如图1所示。按照[Lample and Conneau， 2019]，我们通过元素逐级求和操作将语言嵌入添加到标准令牌和位置嵌入中。这种语言嵌入可以告知编码器和解码器正在处理哪种语言。例如，从德语翻译为英语时，我们将编码器的语言嵌入设置为德语（通过查找表），而将解码器的语言嵌入设置为英语。对于相反的翻译方向（即从英语到德语），我们只需要反转编码器和解码器的**语言嵌入设置**，而无需更改模型架构。这样，可以使用相同的模型体系结构同时翻译任何语言对。

​		以下各段介绍了我们模型的三个关键属性：

​		**共享子词词汇表  **   在我们的实验中，我们使用通过字节对编码（BPE）创建的相同共享词汇表来处理所有语言[Sennrich等，2016]。这不仅使我们能够在具有相同模型的任何语言对之间进行翻译，而且改善了共享相同字母或锚定标记（例如数字）的语言之间嵌入空间的对齐方式。 BPE拆分是根据从单语语料库中随机抽取的句子的串联来学习的。

​		**共享的潜在表示形式 **    在源语言和目标语言之间共享所有编码器参数（包括我们执行联合标记化以来的嵌入矩阵），以便编码器可以将任何源语言的输入映射到共享的潜在表示空间，然后将其翻译为目标语言由解码器提供。此外，我们在两种语言之间共享解码器参数，以将语言嵌入作为语言标识符来减小参数大小。我们还在翻译和语言建模任务之间共享编码器和解码器，这确保了通过降噪自动编码器目标实现的语言建模的好处，可以很好地从噪声源转换为翻译，并最终帮助NMT模型更有效地翻译。

​		**初始化**        编码器和解码器均通过预训练的参数进行初始化，这些参数是通过在语言对中的两种语言的大规模单语语料库上对masked语言模型进行预训练而获得的[Lample and Conneau，2019]。这种初始化不仅可以加速模型收敛，而且可以提高自适应性能，这将在5.1节中进行讨论。

## 2.2 Training Objectives

​		在无人监督的领域适应设置中，我们假设访问域外并行训练语料（X out，Y out）和域内单语数据X in和Y in。 我们有以下三个培训目标：

​		**语言建模**      在NMT中，语言建模通常是通过对自动编码器进行降噪来实现的，

![image-20200611183052494](https://tva1.sinaimg.cn/large/007S8ZIlly1gfojcbjagrj30fo022gls.jpg)

​		其中C是单词损坏函数，其中某些单词被随机删除，删减和交换； P s→s和P t→t为lm分别在源侧和目标侧上运行的编码器和解码器的组成； θ表示包括编码器和解码器的模型参数。

​		**迭代反向翻译**       我们有两个NMT模型，分别是从源语言转换为目标语言的模型s2t（·），以及反之亦然的模型t2s（·）（它们由一种模型体系结构实现）。 在每次迭代中，我们将sourcey从每个源语言句子x∈X转换为目标语言句子模型s2t（x）。 同样，我们将每个目标句子y∈Y转换为源语言模型t2s（y）中的对应句子。 然后（x，模型s2t（x））和（模型t2s（y），y）对可以用作合成并行数据，以通过最小化以下损失来训练两个NMT模型：

![image-20200611183121595](https://tva1.sinaimg.cn/large/007S8ZIlly1gfoj2yw9xaj30fq01y3yr.jpg)

​		这种伪监督训练范式称为迭代反向翻译。 需要注意的是，在最小化此目标函数时，我们不会通过用于生成翻译的模型反向传播。

​		**有监督的机器翻译**        当给定并行数据，表示为（X para，Y para）时，我们还可以最小化有监督的翻译损失：

![image-20200611183145013](https://tva1.sinaimg.cn/large/007S8ZIlly1gfoj3dgzsej30fq018jrf.jpg)

​		监管数据（X out，Y out），也可以是通过对域外数据进行训练的NMT模型进行反向翻译的合成对。

## 2.3 Training Strategy

​		如算法1所示，在每次迭代中，我们随机绘制一批数据以分别最小化上述三个损耗方程式1、2和3。迭代将继续进行，直到验证集的BLEU分数在一定时期内没有增加为止。

![image-20200612100254936](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpa07uyhej30g209yac1.jpg)

# 3 Experiments

## 3.1 Data Setup

​		我们在两种不同的数据设置下验证了我们的模型。 对于第一个设置，我们使用德国英语OPUS语料库1[Tiedemann，2012年]中的法律(law)，医疗(MED)和信息技术(IT)数据集进行训练，并测试我们的模型在每对域上的域适应能力。 第二个设置是将在通用域WMT数据集上训练的模型适应TED[Duh，2018]，LAW和MED数据集。 此设置使用两种语言对:德语-英语（de-en）和罗马尼亚语-英语（roen）。 de-en和ro-en的通用域WMT数据集分别来自WMT-14 2和WMT-16 3任务。 列车组的数据统计如表1所示。 WMT-14的验证和测试集的大小都是3K，所有其他域都是2K。

![image-20200612101025143](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpa80vwr7j30fq0a4wg5.jpg)

​		请注意，要为每个域构建一个不对齐的单语语料库，我们将原始的平行语料库随机打包，并将该语料库分为两个相同的平行句子数。我们分别使用前半部分和后半部分的目标句子和源句子。因此，单语语料库中没有平行句子。

## 3.2 Baselines

​		我们将我们的模型与以下所述的四个最强基准进行比较。

​		**COPY**     将域内数据的目标侧语句复制到源侧，然后与域外数据组合为训练数据，以训练NMT模型[Currey等，2017]。

​		**BACK**       首先使用域外数据训练目标源NMT模型，然后通过翻译目标域内单语句子将其用于生成伪域并行数据以进行模型训练[Senrich等，2015]。

​		**DALI**        首先执行DALI词典归纳，以提取域内词典，然后通过对单语言域内目标句子进行单词到单词的反向翻译来构建伪并行域内语料库，该语言被用于微调预先训练的域外NMT模型[Hu等，2019]。

​		**DAFE**       对域外并行数据的翻译模型和域内目标侧单语数据的语言模型执行多任务学习，同时将域和任务嵌入学习器插入基于转换器的模型中[Dou等， 2019]

​		值得注意的是，COPY，BACK和DALI是以数据为中心的方法，而DAFE和我们的方法是以模型为中心的。

## 3.3 Settings

​		变压器模型中的编码器和解码器的体系结构遵循常规做法，使用6层，8头，尺寸为1024。对于单词破坏功能，单词掉落和消隐采用概率为0.1且出现单词换行的均匀分布用3个令牌的窗口实现。 Adam优化器的学习率为0.0001。

# 4 Experiments

## 4.1 Main Results

​		**在特定域之间进行调整**      我们的主要结果显示在表2中，左六列显示在特定域之间进行模型适应的设置。从该表中，我们可以看到未适应的基线模型（1）U NADAPTED的性能非常差，验证了先前的说法，即当前NMT模型不能很好地推广到新域中的数据。相比之下，复制方法（2）C OPY和反向翻译方法（3）BACK可以显着提高自适应性能，而BACK显示出更好的性能。要注意的是，尽管在大多数情况下，BACK甚至比其他两个基线（DALI和DAFE）都要好得多，但是它是较早提出的并且要简单得多。

![image-20200612101814440](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpag60kt9j30wu0dcaep.jpg)

​		我们在表2第（6）行中显示的方法IBT取得了比所有基准更高的性能，在最强基准上的绝对增幅为+0.91至+9.48 BLEU分数，在非自适应基准上的绝对增幅为+19.91至+27.77 BLEU分数。值得注意的是，我们的IBT方法仅需要域内单语数据，但仍大大优于那些依赖于并行域外数据的基线模型，这表明以前的方法并未耗尽域内数据中包含的潜力。

​		**从通用域改编为特定域**        在第二个数据设置中，表2的右6列显示了将通用域语料库（WMT）训练的NMT模型改编为特定域（TED，Law和MED）的结果两种语言对：从德语到英语，从罗马尼亚语到英语。在此设置下，与先前的设置相比，C OPY和BACK都具有更好的性能，这表明在域内并行数据较少的情况下，域外数据通常会有所帮助。我们的IBT方法在很大程度上超越了U NADAPTED，但在某些情况下并没有超过基线BACK和DAFE。这两种设置共同构成了两种方法之间的全面比较：一种侧重于内部（我们）探索域内单语数据，另一种目的是从域外并行数据中提取信息并将其传输至域数据（先前的方法）。在域外资源较少或与域内数据相距较远的情况下，我们的方法可以执行得更好，而在其他情况下，我们的方法的表现会稍差一些。

​		**将IBT与域外或反向转换数据相结合**        我们的方法以模型为中心，并且可以与任何以数据为中心的方法相结合，例如添加域外数据或反向转换数据。在表2的第（7）行中，在IBT训练期间，我们还将监督翻译训练任务插入域外数据中，这可以带来约+1到+4的BLEU改进（MED除外） ro-en语言对中的目标域）。此外，代替直接使用域外数据，我们可以对通过对域外数据进行训练的NMT模型合成的域内数据的反向翻译对实施监督翻译训练，从而获得更好的性能，如表2的第（8）行所示。尽管第（7）行和第（8）行中用于训练的数据都相同，但是IBT + Back的性能仍然优于，这表明对域内数据进行模型训练是总是比域外数据更好的选择。总体而言，与最佳基准模型相比，我们的最佳设置可以收获+19.31 BLEU的改进，与未适应模型相比，我们的+46.69 BLEU的改进。我们还尝试将域外并行数据与反向翻译数据组合在一起以进行有监督的翻译训练，但发现它的性能比当前设置差。

## 4.2 Ablation Study

​		我们进行了消融研究，以检查模型中关键组件的贡献，如表3所示。我们首先从LAW域将IBT + BACK模型应用于MED和IT，其验证集BLEU分数报告于表3中的第（1）行在此表的（1）和第（2）行中，我们可以看到，如果未使用预训练的参数初始化模型，则性能将遭受10到20个BLEU分数的严重降低。此外，两种语言建模（LM）都与等式中的一样。 （1）和迭代反向翻译（IBT），如等式（2）是模型的组成部分，尤其是IBT，如第（3）行所示，（4）。最有趣的是，除了删除迭代的反向翻译部分之外，如果我们进一步删除源侧句子的语言建模（如第（6）行所示），则适应的性能比仅删除迭代的性能要好得多回译部分。这表明当没有反向翻译训练时，源端的语言建模会产生负面影响，这实际上与直觉一致，在这种情况下，解码器只需要学习如何在目标端输出流利的语言即可。

![image-20200612102047536](/Users/suntian/Library/Application Support/typora-user-images/image-20200612102047536.png)

# 5 Discussion

## 5.1 Pre-Training is Always Helpful

​		通过实验，我们发现用预训练参数初始化翻译模型可以同时有益于我们的方法和基线，如表4所示。我们在两种设置下比较了三种基线（U NADAPTED，C OPY，BACK）：和 当我们从LAW领域适应MED和IT时，无需进行预培训。 对于所有三种适应方法，预训练始终可以带来很大的改进。 这表明，良好的模型初始化对于无监督域自适应问题至关重要，可以避免缺乏监督数据。

![image-20200612102138456](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpajpb6e1j30fm07kwft.jpg)

## 5.2 Do More In-Domain Monolingual Data Help?

​		我们的方法的优点之一是，如果单语数据变大，它将继续获得更高的性能。为了验证这一说法，我们收集了额外的单语数据并分析了在表5中添加这些额外数据之前和之后的性能差异。我们考虑了额外的单语数据的两个来源：域外和域内数据。具体来说，我们研究了de-en和ro-en语言对从WMT数据到TED的适应性。域外数据源可以是WMT数据本身，而对于域内数据，我们收集了TED演讲的附加数据集。在抓取所有TED演讲网页4直至2020年1月初之后，我们以三种语言（英语，德语和罗马尼亚语）提取了成绩单，并保留了该成绩单的唯一TED演讲身份。 5请注意，对于任何语言对，我们都确保TED演讲的笔录仅在源侧或目标侧出现一次，以避免出现平行句子。当将额外的单语数据与原始域内数据组合在一起时，我们总是通过复制对域内数据进行上采样，以使其具有与额外数据相同的大小。

​		通过比较表5中的行（1）和（2），我们发现，对于我们的方法，域外和原始域内单语言数据的组合可能比单独域内数据更好。第（3）行显示，域内单语数据的添加可以带来更大的改善，这与直觉相一致，即域内数据应始终对适应更有用。我们还将域内数据添加到最佳设置的IBT + BACK中，并获得了更高的BLEU分数，非常接近监督的翻译性能，如表5中的第（6）和（7）行所示。证明了我们方法的巨大潜力：我们始终可以寻求收集更多易于访问的域外或域内单语数据，以不断提高无监督自适应性能。

![image-20200612102209642](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpak8xw1pj30fu0c6di0.jpg)

## 5.3 How Does Our Model Perform on Semi-supervised Adaptation?

​		尽管这项工作的目标是非监督域自适应，但我们也很想知道我们的方法在域内并行句子对数量有限的情况下对半监督域自适应的影响。我们首先通过在IT域的并行语料库上预训练NMT模型，然后对将域外IT数据与子域连接在一起的数据进行微调，来测试标准的半监督设置S EMI。采样域内法律数据。我们对20K，80K，160K和320K真实域内句子进行了二次采样。请注意，在级联期间，如果子采样的域内数据少于域外数据，我们将重复域内数据，直到达到与域外数据相同的大小为止。为了进行比较，我们还使用此组合数据来训练模型中的监督损耗L sup（表示为IBT + S EMI）。为了更清楚地比较图2a，我们还绘制了IBT和受监督MT的未受影响的性能。正如我们所看到的，S EMI的性能随着域内并行数据的数据大小的增加而猛增，而IBT + S EMI仅在有限的程度上明显增加。这表明我们的方法在半监督域自适应任务中的局限性。另一方面，在资源不足的情况下，域内数据少于160K，我们的IBT + S EMI模型是最佳选择，因为它在很大程度上胜于常规的半监督训练。

![image-20200612102255635](https://tva1.sinaimg.cn/large/007S8ZIlly1gfpal1l8uqj30gc0caac2.jpg)

## 5.4 How do Different Sizes of Out-of-Domain Data Affect?

​		在本节中，我们要研究以下问题：对不同的非自适应NMT模型进行的微调如何影响适应性能？为此，我们对不同大小的域外并行数据进行了二次采样，对它们进行了NMT模型训练（U NADAPTED），并通过三种方式对其进行了微调：BACK，IBT和IBT + BACK。具体来说，我们从通用域（WMT14）到德语-英语对的IT域进行了调整，使用了所有域内单语数据，并对0.1、0.5、1和4百万域外并行数据进行了子采样。如图2b所示，IBT不使用任何域外数据，因此它保持不变，而所有其他设置都表明，随着域外数据数量的增加，性能得到了改善，并且当数量增加时，性能略有波动域外数据超过100万。值得注意的是，IBT + BACK始终在性能上始终领先于所有其他产品，并且其性能也以比其他产品更高的速度增长，这表明我们的方法更好地利用了域外监督数据。

# 6 Related Work

​		NMT之前的大多数域适应工作都集中在有监督的设置上，在该设置中，存在大量的域外并行数据和少量的域内并行数据。顺序微调[Luong and Manning，2015; Freitag和Al-Onaizan，2016年]方法首先在域外数据上训练NMT模型，然后对域内数据进行微调。 Britz等。建议联合培训翻译任务和领域区分任务以减轻领域转移。 Kobus等。使用域令牌和域嵌入来强制NMT模型考虑域信息。 Joty等。为那些组合更多域内数据的域外数据分配更高的权重，以消除不必要的噪声。我们提出的方法专注于解决没有域内并行句子的适应问题，这是一个更严格的设置。

​		NMT的无监督域适应可以分为两个线程：以数据为中心和以模型为中心。以数据为中心的方法主要提出选择或生成与领域相关的伪并行数据以进行模型训练。在数据选择方面，Duh等人。使用语言模型对域外数据进行排名，并选择排名最高的并行句子作为合成数据。更具代表性的方法是基于反向翻译的方法（Senrich等，2015）和基于复制的方法（Currey等，2017），这些方法很简单，但已被广泛证明是有效的。另一方面，尚未充分探索基于模型的方法。这一系列方法通过引入额外的学习目标，例如自动编码目标[Cheng等，2016]和语言建模目标[Ramachandran等，2017; Dou等人，2019]相反，我们建议利用在线迭代反向翻译，以无监督的方式更好地利用域内数据。

# 7 Conclusion

​		在本文中，我们确定了迭代反向翻译训练方案可以极大地改善NMT的无监督域自适应。 在三个资源匮乏的领域中，这种原始方法显示出比以前的四个模型中最强的模型提高了+9.48 BLEU得分，与未经适应的基准相比提高了+27.77 BLEU。 通过进一步与流行的数据增强方法结合并利用来自两种语言对的通用领域的监督数据，我们的模型显示出比最强的基线模型高出+19.31 BLEU的主要改进，而与未经适应的模型相比则有+47.69的改进。

# References

[Artetxe et al., 2017 ] Mikel Artetxe, Gorka Labaka, Eneko Agirre, and Kyunghyun Cho. Unsupervised neural machine translation. arXiv preprint arXiv:1710.11041, 2017.

[ Britz et al., 2017 ] Denny Britz, Quoc Le, and Reid Pryzant. Effective domain mixing for neural machine translation. In Proceedings of the Second Conference on Machine Translation, pages 118–126, Copenhagen, Denmark, September 2017. Association for Computational Linguistics.

[Cheng et al., 2016 ] Yong Cheng, Wei Xu, Zhongjun He, Wei He, Hua Wu, Maosong Sun, and Yang Liu. Semisupervised learning for neural machine translation. In Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 1965–1974, Berlin, Germany, August 2016. Association for Computational Linguistics.

[ Chu and Wang, 2018 ] Chenhui Chu and Rui Wang. A survey of domain adaptation for neural machine translation. arXiv preprint arXiv:1806.00258, 2018.

[Currey et al., 2017 ] Anna Currey, Antonio Valerio Miceli Barone, and Kenneth Heaﬁeld. Copied monolingual data improves low-resource neural machine translation. In WMT, 2017.

[Domhan and Hieber, 2017 ] Tobias Domhan and Felix Hieber. Using target-side monolingual data for neural machine translation through multi-task learning. In EMNLP, 2017.

[Dou et al., 2019 ] Zi-Yi Dou, Junjie Hu, Antonios Anastasopoulos, and Graham Neubig. Unsupervised domain adaptation for neural machine translation with domainaware feature embeddings. In EMNLP/IJCNLP, 2019.

[Duh et al., 2013 ] Kevin Duh, Graham Neubig, Katsuhito Sudoh, and Hajime Tsukada. Adaptation data selection using neural language models: Experiments in machine translation. In Proceedings of the 51st Annual Meeting of the Association for Computational Linguistics (Volume 2: Short Papers), pages 678–683, Soﬁa, Bulgaria, August 2013. Association for Computational Linguistics.

[Duh, 2018 ] Kevin Duh. The multitarget ted talks task.

http://www.cs.jhu.edu/ ∼ kevinduh/a/multitarget-tedtalks/, 2018.

[Freitag and Al-Onaizan, 2016 ] Markus Freitag and Yaser Al-Onaizan. Fast domain adaptation for neural machine translation. arXiv preprint arXiv:1612.06897, 2016.

[Hu et al., 2019 ] Junjie Hu, Mengzhou Xia, Graham Neubig, and Jaime Carbonell. Domain adaptation of neural machine translation by lexicon induction. In Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics, pages 2989–3001, Florence, Italy, July 2019. Association for Computational Linguistics.

[Jin et al., 2019 ] Zhijing Jin, Di Jin, Jonas Mueller, Nicholas Matthews, and Enrico Santus. Imat: Unsupervised text attribute transfer via iterative matching and translation. In Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing, EMNLP-IJCNLP 2019, Hong Kong, China, November 37, 2019, pages 3095–3107, 2019.

[ Joty et al., 2015 ] Shaﬁq Joty, Hassan Sajjad, Nadir Durrani, Kamla Al-Mannai, Ahmed Abdelali, and Stephan Vogel. How to avoid unwanted pregnancies: Domain adaptation using neural network models. In Proceedings of the 2015 Conference on Empirical Methods in Natural Language Processing, pages 1259–1270, Lisbon, Portugal, September 2015. Association for Computational Linguistics.

[ Kobus et al., 2016 ] Catherine Kobus, Josep Maria Crego, and Jean Senellart. Domain control for neural machine translation. In RANLP, 2016.

[ Lample and Conneau, 2019 ] Guillaume Lample and Alexis Conneau. Cross-lingual language model pretraining. ArXiv, abs/1901.07291, 2019.

[ Lample et al., 2018 ] Guillaume Lample, Myle Ott, Alexis Conneau, Ludovic Denoyer, and Marc’Aurelio Ranzato. Phrase-based & neural unsupervised machine translation. In EMNLP, 2018.

[ Luong and Manning, 2015 ] Minh-Thang Luong and Christopher D Manning. Stanford neural machine translation systems for spoken language domains. In Proceedings of the International Workshop on Spoken Language Translation, pages 76–79, 2015. 

[ Ramachandran et al., 2017 ] Prajit Ramachandran, Peter Liu, and Quoc Le. Unsupervised pretraining for sequence to sequence learning. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing, pages 383–391, Copenhagen, Denmark, September 2017. Association for Computational Linguistics.

[ Sennrich et al., 2015 ] Rico Sennrich, Barry Haddow, and Alexandra Birch. Improving neural machine translation models with monolingual data. arXiv preprint arXiv:1511.06709, 2015.

[ Sennrich et al., 2016 ] Rico Sennrich, Barry Haddow, and Alexandra Birch. Neural machine translation of rare words with subword units. In Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 1715–1725, Berlin, Germany, August 2016. Association for Computational Linguistics.

[ Tiedemann, 2012 ] J¨org Tiedemann. Parallel data, tools and interfaces in opus. In Nicoletta Calzolari (Conference Chair), Khalid Choukri, Thierry Declerck, Mehmet Ugur Dogan, Bente Maegaard, Joseph Mariani, Jan Odijk, and Stelios Piperidis, editors, Proceedings of the Eight International Conference on Language Resources and Evaluation (LREC’12), Istanbul, Turkey, may 2012. European Language Resources Association (ELRA).

[ Vaswani et al., 2017 ] Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, and Illia Polosukhin. Attention is all you need. In Advances in neural information processing systems, pages 5998–6008, 2017.

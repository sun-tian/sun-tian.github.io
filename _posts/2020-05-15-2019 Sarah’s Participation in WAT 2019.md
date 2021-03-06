---
layout:     post           # 使用的布局（不需要改）
title:      2019 Sarah s Participation in WAT 2019           # 标题 
subtitle:   Sarah s Participation in WAT 2019 #副标题
date:       2020-05-15             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/post-bg-coffee.jpeg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - nlp
    - paper
    - 机器翻译
    - 专利

---

# 2019 Sarah’s Participation in WAT 2019

# Abstract

​		本文介绍了我们的MT系统参与WAT 2019的情况。 我们参与了(i)Patent，(ii)Timely Disclosure，(iii)Newswire和(iv)Mixed-domain tasks。 <u>我们的主要关注点是探索类似的Transformer模型在各种任务上的表现如何</u>。 **我们观察到，对于具有较小数据集的任务，我们最好的模型设置是具有较少数量的注意头的较浅模型**。 我们调查了NMT中经常出现在生产设置中的实际问题，例如应对多语言和简化部署中的前后处理管道。

# 1 Introduction

​		本文描述了我们的机器翻译系统参与第六届亚洲翻译研讨会(WAT-2019)的翻译任务（Nakazawa et al.，2019）。 我们参与了(i)Patent，(ii)Timely Disclosure，(iii)Newswire和(iv)Mixed-domain tasks。 我们在一个受约束的设置下训练我们的系统，这意味着除了共享任务管理器提供的资源之外，没有使用任何额外的资源。 我们基于变压器架构构建了所有MT系统（Vaswani等人，2017）。 我们针对每项任务的主要任务总结如下:

-   专利任务:我们为六个翻译方向搭建了多个翻译系统。 我们还探索了一种多语言方法，并将其与单向模型进行了比较We built several translation systems for six translation directions. We also explored a multilingual approach and compared it with the unidirectional models.。
-   及时披露任务:我们尝试了一种简化的数据处理方法，使模型直接在原始文本上进行训练，而不需要特定语言的前/后处理。
-   Newswire任务:我们探索了在相对较小的数据集上调整变压器模型的超参数，发现紧凑模型能够实现具有竞争力的性能。
-   混合领域任务:我们探索了缅甸英语的低资源翻译方法。

# 2 JPO专利任务

## 2.1任务说明

​		对于专利翻译任务，我们使用了JPO专利语料库(JPC)4.3版，它是由日本专利协会(JPO)构建的。 与以前的WAT任务类似（Nakazawa等人，2015，2016，2017，2018），该任务包括中日，韩日和英日的专利描述翻译。 每个语言对的训练集由1M个平行句子组成。 我们在没有任何外部资源的情况下使用了组织者提供的技术培训，验证和测试分割。 我们为每种语言对训练了单独的单向模型。 此外，我们还探索了多语言NMT方法For the patent translation task, we used the JPO Patent Corpus (JPC) version 4.3, which is constructed by the Japan Patent Ofﬁce (JPO). Similar to previous WAT tasks (Nakazawa et al., 2015, 2016, 2017, 2018), the task includes patent description translations for Chinese-Japanese, Korean-Japanese, and English-Japanese. Each language pair’s training set consists of 1M parallel sentences. We used the ofﬁcial training, validation, and test split provided by the organizer without any external resources. We trained individual unidirectional models for each language pair. Additionally, we explored multilingual NMT approaches for this task.。

## 2.2数据处理

​		我们使用SentencePiece（Kudo和Richardson，2018）来训练基于字节对编码(BPE)的子字单元。 我们使用以下工具对数据进行了预令牌化We used SentencePiece (Kudo and Richardson, 2018) for training subword units based on bytepair encoding (BPE). We pre-tokenized the data using the following tools:

​		•Juman版本7.01 1用于日语，

​		•[Stanford Word Segmenter](https://nlp.stanford.edu/software/segmenter.shtml) version 2014-0616 for Chinese，

​		•韩国语Mecab-ko 3，以及

​		•Moses tokenizer for English。

​		源句和目标句被合并以训练一个联合词汇。 我们将词汇表大小设置为100,000，并从词汇表中移除出现次数少于10次的子词，遵循组织者发布的基线NMT系统类似的预处理步骤 [4](http://lotus.kuee.kyoto-u.ac.jp/WAT/WAT2019/baseline/ dataPreparationBPE.html)。Source and target sentences are merged for training a joint vocabulary. We set the vocabulary size to 100,000 and removed subwords that occur less than 10 times from the vocabulary, following similar pre-processing steps for the baseline NMT system released by the organizer.

## 2.3 模型

​		我们的NMT模型基于Fairseq toolkit（Ott et al.，2019）中的Transformer（Vaswani et al.，2017）实现。 表1总结了用于我们实验的参数的详细信息。 编码器的输入嵌入，解码器的输入和输出嵌入层连接在一起（Press and Wolf，2017年），在不影响性能的情况下节省了大量参数。 使用Adam（Kingma和Ba，2015年）对模型进行优化，使用β 1=0.9，β 2=0.98和=1e-8。 我们使用了与（Ott et al.，2018）相同的学习速率时间表，并在4个Nvidia V100 GPU上运行实验，从而在Fairseq(-FP16)中实现混合精度训练。 选择验证集上性能最好的模型对测试集进行解码。 我们用不同的随机种子训练4个独立的模型来执行集成译码。Our NMT model is based on the Transformer (Vaswani et al., 2017) implementation in the Fairseq toolkit (Ott et al., 2019). The details of the parameters used for our experiments are summarized in Table 1. Encoder’s input embedding, decoder’s input and output embedding layers were tied together (Press and Wolf, 2017), which saves signiﬁcant amounts of parameters without impacting performance. The model was optimized with Adam (Kingma and Ba, 2015) using β 1 = 0.9, β 2 = 0.98, and  = 1e-8. We used the same learning rate schedule as (Ott et al., 2018) and run the experiments on 4 Nvidia V100 GPUs, enabling mixed-precision training in Fairseq (--fp16). The best performing model on the validation set was chosen for decoding the test set. We trained 4 independent models with different random seeds to perform ensemble decoding.

![image-20200515161538513](https://tva1.sinaimg.cn/large/007S8ZIlly1get7fe4u2ej30aw0cemyp.jpg)

# 2.4 结果

​		表2显示了专利任务的模型性能。 为了简单起见，我们只报告了JPCN测试集上的结果，该测试集是JPCN{1，2，3}和ZH-JA的表达式模式任务(JPCEP)的联合。 对于每个测试集的详细分类，我们希望请读者参阅概述论文。 由于在撰写本文时还没有人的评估结果，因此我们仅以BLEU评分的形式给出结果。 显然，集成解码明显优于单模解码。 在受限制的设置下，我们的最佳投稿将在WAT排行榜5中获得zh-ja，jazh和JA-EN的第一名。 有趣的是，我们的模型在ja-ko翻译上表现不佳，在ja-ko翻译上的表现落后于组织者基于序列到序列LSTM的基线系统。 更仔细的调查可以帮助我们理解训练管道中的哪个组件（例如，数据处理或标记化）可能导致这种差异。Table 2 shows our model performance for the patent task. For brevity, we only reported the results on the JPCN test set, which is a union of JPCN{1,2,3}, and the Expression Pattern task (JPCEP) for zh-ja. For the detailed breakdown for each test set, we would like to refer readers to the overview paper. Since human evaluation result is not available as the time of this writing, we only present the results in terms of BLEU scores. It is clear that ensemble decoding signiﬁcantly outperforms single model decoding. Under the constrained settings, our best submissions obtain the ﬁrst place in the WAT leaderboard 5 for zh-ja, ja-zh, and ja-en. Interestingly, our model did not perform well on ja-ko translation, where the performance was behind the organizer’s baseline system which is based on a sequence-to-sequence LSTM. More careful investigation could help us understand which component in our training pipeline (e.g., data processing or tokenization) could possibly cause this difference.

![image-20200516090340221](https://tva1.sinaimg.cn/large/007S8ZIlly1geu0kafh67j30f20n043q.jpg)

# 2.5 Multilingual Experiments

​		鉴于这项任务涉及多个语言对，我们在提交期后进一步实验了多语言NMT方法。 我们遵循了（Johnson et al.，2017）中的方法，即在每个源语句中添加一个用于指示目标翻译语言的人工标记（Fairseq中的Encoder-Langtok tgt）。 编码器和解码器参数在所有语言对之间共享。 我们合并了来自所有四种语言的所有训练数据，以训练一个大小约为100,000的联合子词词汇表。 因此，我们可以在编码器和解码器中共享嵌入层。 由于每个方向的训练示例数是相同的，所以我们从六种语言对中对批次进行循环迭代。Given that multiple language pairs are involved for this task, we further experimented with multilingual NMT approaches after the submission period. We followed the approach in (Johnson et al., 2017), which adds an artiﬁcial token in each source sentence for indicating the target translation language (--encoder-langtok tgt in Fairseq). Encoder and decoder parameters are shared among all the language pairs. We merged all training data from all four languages for training a joint subword vocabulary of size 100,000 approximately. As a result, we can share the embedding layer in the encoder and decoder. Since the number of training examples for each direction is the same, we iterate round-robin over batches from the six language pairs.

​		如表2所示，我们的多语言结果并没有显示NMT系统的改进，在单次解码方面落后于单向模型不超过2个BLEU点。 然而，多语言模型中的参数共享将参数总数减少到与一个单向模型的参数总数大致相同。 在实践中，这可以潜在地简化多语言翻译的生产部署。 有效地，该模型能够对不包括在此任务中的语言对（例如汉语-韩语）执行零激发翻译，尽管我们将此问题留待将来的研究。As shown in Table 2, our multilingual result did not show improvement in the NMT systems, falling behind the unidirectional model by not more than 2 BLEU points for single decoding. Nonetheless, parameter sharing in multilingual model reduces the total number of parameters to approximately the same as that of one unidirectional model. In practice, this can potentially simplify production deployment for multiple language translation. Effectively, the model is able to perform a zero-shot translation for language pairs not included in this task (such as Chinese-Korean), although we left this for future investigation.

# 3 JPX Timely Disclosure Task

## 3.1 Task Description

​		及时披露任务评估了由日本交流集团(JPX)建设的及时披露文档语料库(TDDC)中的日文到英文翻译。 该语料库由140万句日英平行句子组成，这些句子取自2016年至2018年间过去的及时披露文件。 验证和测试集被进一步分成两个子数据集:1）名词和短语（“X项”）和2）完整文本（“X文本”）。我们使用组织者给出的数据分割，没有额外的外部资源。在这个任务中，我们对不同预处理过程对模型性能的影响做了一个简短的研究。

## 3.2 Data Processing

​		MT系统通常包括复杂的预处理/后处理流水线，这通常是语言特定的。 这通常在流水线中形成一个长链:标记化/分段truecasing翻译DtrueCasing去标记化。

​		虽然Moses（Koehn et al.，2007），实验管理系统6和SacreMoses 7等工具简化了数据处理管道，但处理各种语言在维护语言规范资源和规则方面产生了重大的技术债务。 虽然存在语言不可知论的标记化/真实化方法（如Evang et al.，2013；Susanto et al.，2016），但来自管道中各个组件的错误仍在传播。 相反，我们建议使用单步预处理，使用语句表子词标记器。
​		SentencePiece是一个无监督的标记器，可以直接对原始句子进行学习，预标记化是一个可选的步骤。 这大大简化了训练过程，因为我们可以将数据直接输入到SentencePiece中，从而生成基于BPE的子词标记。 我们合并了源句和目标句，以训练32,000个标记的共享词汇表，100%的字符覆盖率，没有进一步的筛选。 我们从训练集中移除空行和超过250个子词标记的句子。 项目和文本子数据集的处理方式相同。 我们将项目和文本开发数据集连接在一起进行模型验证。

## 3.3 Model

​		对于及时公开任务，我们使用了如表3所示的8头6层变压器。 总体模型类似于JPO模型，除了一些不同之处:1）更小的退出概率，2）更大的每批令牌数量，3）延迟更新。 特别地，累积每个GPU上的多个子批的梯度，这减少了处理时间的差异并减少了通信时间（Ott et al.，2019）。 使用--update-freq2，可以有效地将批处理大小增加一倍。 我们用不同的随机种子训练4个独立的模型，对项目和文本测试集进行集成解码。 每个模型经过40个历元的训练，在验证集中选择性能最好的检查点。

![image-20200516090915065](https://tva1.sinaimg.cn/large/007S8ZIlly1geu0q33omdj30b60ccjsy.jpg)

## 3.4 Results

​		表4显示了及时公开任务的模型性能。 人类评估将我们的最佳提交排在“项目”测试集的第一位，“文本”测试集的第二位。 在提交期结束后，我们对在数据预处理中加入日语分割的效果做了进一步的研究。 我们使用Juman标记了日语文本，并重新训练了我们的模型。 比较他们在单次解码上的BLEU得分，我们观察到标记化在项目数据上略有改善，而在文本数据上则显著改善了2.5个BLEU点，这可能会提高我们在排行榜上的得分。 然而，单步预处理大大简化了我们的训练和翻译流程。 这对于在生产环境中为多种语言部署MT系统特别有帮助，因为它允许我们构建端到端系统，而不依赖于语言（特别是前/后处理）。

![image-20200516090957297](https://tva1.sinaimg.cn/large/007S8ZIlly1geu0qtdpt6j30k006q400.jpg)

表4:JPX任务结果。 注意，我们没有提交包含日语分词（用*标记）的模型输出，它用作比较目的。

# 4 JIJI Newswire Task

## 4.1 Task Description

​		新闻通讯社的任务评估了时事集语料库上的日英翻译。 该语料库由时事出版社与国家信息和通信技术研究所(NICT)合作创建。 数据集包含20万个平行句子用于训练，2000个用于验证，2000个用于测试。 除了提供的语料库，我们没有使用任何外部资源。 在这个任务中，我们研究了根据我们的训练集的大小选择合适的变压器网络大小的重要性。

## 4.2 Data Processing

​		我们运行Juman7.01版进行日语分词，但英语句子没有标记化。 标记化后，将日语和英语句子组合并输入到句子表中，用于BPE子词单元的训练。 子词词汇量为32,000个，字符覆盖率为100%，无需进一步填充。 我们进一步从训练集中移除空行和超过250个子词标记的句子。 我们还尝试将句子直接输入到句首，而不预先标记日语，但我们观察到这样做在时事任务上的表现较弱。

## 4.3 Model

​		Sennrich和Zhang(2019)通过减少词汇量和仔细的超参数调优，在低资源设置下适应了基于RNN的NMT系统。 类似地，考虑到JIJI语料库是一个相对较小的数据集，我们将系统自适应技术应用于基于Transformer的NMT系统来完成这项任务。 如表5所示，我们选择缩小变压器模型，以防止过度调整。 我们做了如下调整:(i)嵌入和隐藏维数减半，(ii)减少注意头和编码器/解码器层数，(iii)通过dropout和权重衰减增加正则化(--weight-decay 0.0001）。 我们对两个方向都使用了相同的模型。 考虑到这个数据集的大小，我们能够为JIJI任务运行更长的历元:日语英语150历元，英语日语100历元。 我们将这个缩放模型(MINI)的性能与以前用于JPX任务的模型设置(BASE)进行比较。

![image-20200516091238625](/Users/suntian/Library/Application Support/typora-user-images/image-20200516091238625.png)

## 4.4 Results

​		表6显示了我们的模型在newswire任务上的性能。 我们可以观察到，MINI模型的单次解码性能明显优于基本模型，约5个BLEU点。 这些结果验证了我们的假设，即在低资源设置下，通过更仔细的超参数调整，而不需要过多依赖辅助资源，可以提高NMT性能。 总体而言，我们提交的两个翻译方向的BLEU评分和受限设置均在排行榜上名列前茅。 不幸的是，我们的系统输出是唯一被人评估的受限提交，因此我们无法进行比较评估。

![image-20200516091315900](https://tva1.sinaimg.cn/large/007S8ZIlly1geu0u9gyflj30f406mq45.jpg)

表6:JIJI任务结果。 注意，我们没有提交基本模型输出（用*标记），它用于比较目的。

# 5 Mixed-domain Task

## 5.1 Task Description

​		混合域任务评估了仰光计算机研究大学(UCSY)（Ding et al.，2018）和亚洲语言树库(ALT)语料库（Ding et al.，2019）的缅甸-英语翻译。 模型在UCSY语料库上进行训练，然后在ALT语料库上进行验证和测试。 UCSY语料库包含大约200,000个句子，ALT验证和测试集各有1,000个句子。 没有使用其他资源来训练我们的模型以进行任务参与。

## 5.2数据处理

​		对于混合域任务，没有采取特殊的预处理步骤来处理数据； 句子被直接输入到句子表以产生子词标记。 我们使用Marian 8工具包（Junczys-Dowmunt et al.，2018）对两个不同大小的变压器模型进行了实验。

## 5.3模型

​		使用类似的模型设置，如(i)表3中的JPX模型（具有100%字符覆盖率(BASE)下的32,000个子字令牌）和(ii)表5中的JIJI模型（具有100%字符覆盖率(MINI)下的10,000个子字令牌），我们分别训练一个模型来比较混合域任务中的(i)和(ii)。 我们只参加了英国人到缅甸的任务。

## 5.4结果

​		表7显示了我们的英语到缅甸模型的结果。 由于缅甸-英语语言对的低资源性质和领域适应的额外困难，在未来的工作中，我们将探索扩展通用领域的语言资源，以进一步提高该语言对的翻译质量。

​		我们编制了带有各种缅英数据集的神话语料库9，研究者可以利用这些数据集来改进缅英模型。 所创建的数据集范围从手动清理的字典到使用商业翻译API和无监督机器翻译算法综合翻译的数据。

![image-20200516091544961](https://tva1.sinaimg.cn/large/007S8ZIlly1geu0wulsxej30d205cjs3.jpg)

# 6 结论

​		在本文中，我们向WAT2019翻译共享任务提交了我们的文件。 我们在不同的任务中训练了类似的基于Transformer的NMT系统。 我们发现，具有少量磁头的较浅的变压器在较小的数据集上表现得更好。 我们还发现了简化数据处理管道和模型性能之间的权衡。 最后，我们尝试了一个简单的方法来训练一个多语言的NMT系统，我们将在未来的工作中沿着这个方向继续我们的研究。
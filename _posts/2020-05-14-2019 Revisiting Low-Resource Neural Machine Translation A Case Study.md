---
layout:     post           # 使用的布局（不需要改）
title:      2019 Revisiting Low-Resource Neural Machine Translation A Case Study           # 标题 
subtitle:   2019 Revisiting Low-Resource Neural Machine Translation A Case Study           #副标题
date:       2020-05-14             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/post-bg-coffee.jpeg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - nlp
    - paper
    - 机器翻译
    - 低资源机器翻译

---

# 2019 Revisiting Low-Resource Neural Machine Translation A Case Study

![image-20200514135717023](https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@master/assets/picgoimg/20200720164027.jpg)

# 摘要

​		研究表明,<u>在低资源条件下,神经机器翻译(NMT)的性能明显下降,表现不如基于短语的统计机器翻译( PBSMT,并且需要大量辅助数据才能获得有竞争力的结果</u>。在本文中,我们重新评估了这些结果的有效性,认为它们是由于缺乏系统适应低资源设置的结果。
​		我们讨论了在培训低资源NMT系统时需要注意的一些缺陷,以及最近在低资源环境中特别有用的技术,从而形成了一套用于低资源NMT的最佳实践。

​		在使用不同数量的IWSLT14训练数据的德语-英语实验中,我们表明,**在不使用任何单语或多语辅助数据的情况下,优化后的NMT系统可以在数据量远低于之前所述的情况下胜过 PBSMT**。我们还将这些技术应用于低资源的韩语-英语数据集,比以前报告的结果高出4倍。

# 1介绍

​		神经机器翻译(NMT)在高资源数据条件下取得了令人印象深刻的表现,成为该领域的主导( Sutskever et al.,2014; Bandana等人,2015年 Vaswani et al,2017),最近的研究认为,这些模型是高度数据效率低下的,并且在低数据条件下表现不佳的基于短语的统计机器翻译( PBSMT)或非监督方法(Koeh和 Knowles,2017; Lample等,2018b)。**在本文中,我们重新评估了这些结果的有效性,认为它们是由于缺乏系统适应低资源设置结果**。

​		我们的主要贡献如下:

-   我们探索低资源的最佳实践NMT,评估它们在消融ablation研究中的重要性我们重现了NMT和PB-SMT在不同数据条件下的比较
-   ,结果表明,在遵循我们的最佳实践时,NMT仅用10万字的并行训练数据就优于PBSMT。



# 2 相关工作

## 2.1 跨系统比较的低资源翻译质量

![image-20200514140114847](https://tva1.sinaimg.cn/large/007S8ZIlly1gerxx908d0j30b60bkta5.jpg)

图1:低资源条件下 PBSMT和NMT的质量( Koehn和Knowles,2017)。

​		图1再现了 Koehn和 Knowles(2017)的一个图,**图中显示他们的NMT系统平行训练数据仅在超过1亿个单词时才比 PBSMT系统表现更好**。 Lample等人(2018b)的结果与此类似,表明**在可用的并行资源很少的情况下,非监督NMT的性能优于监督系统**。在两篇论文都对NMT系统<u>超参数是典型的高资源设置，作者没有调整超参数,也没有改变网络架构,以优化NMT以适应低资源条件</u>。

## 2.2改进低资源神经网络机器翻译

​		**关于低资源NMT的大量研究集中在开发单语数据,或涉及其他语言对的并行数据**。

​		<u>改善单语数据的NMT的方法包括从单独训练的语言模型的整合</u>(G¨ ulc ehre et al,2015)<u>到带有附加目标的部分NMT模型的培训,包括一个语言建模目标</u>(G¨ uIc ehre et al.,2015; Sennrich等2016b; Ramachandran et al,2017),<u>一个自动编码目标</u>( Luong et al,2016; Currey et al.,2017)<u>或往返目标,其中模型被训练来预测单语(目标侧)训练数据已被反向翻译为源语言</u>( Sennrich et al,2016b; He et al.,2016; Cheng等,2016)。<u>作为一个极端的例子,完全依赖单语数据的模型已被证明是有效的</u>( Artetxe et al,2018b; Lample等,2018a; Artetxe等,2018a; Lample等,2018b)。

​		<u>同样,来自其他语言对的并行数据可以用来对网络进行预训练或共同学习表示</u>( Zoph et al.,2016;Chen等人,2017; Nguyen和 Chiang2017;Neu- big and Hu,2018;顾等,2018a,b;科米和博贾尔,2018)。
​		**虽然半监督和非监督方法已被证明对一些语言对非常有效,但它们的有效性取决于大量合适的辅助数据的可用性,以及其他条件的满足**。例如,无监督方法的有效性是受损时语言形态不同,或者当培训领域不匹配( Sogaard et al,2018)更广泛地说,这类研究仍然接受这样一个前提,即NMT模型是数据效率低下的,并且需要大量辅助数据来训练。<u>在这项工作中,我们希望重新审视这一点,并将重点放在更有效地利用少量并行训练数据的技术上</u>。没有辅助数据的低资源NMT受到的关注较少;这方面的工作包括Ostling¨和 Tiedemann,2017 Nguyen和 Chiang2018)

# 3 低资源神经机器翻译方法

## 3.1 主流提升

​		我们考虑使用Koehn and Knowles(2017 Six Challenges for Neural Machine Translation)的超参数作为我们的基线。这个基线没有利用各种先进的NMT架构和培训技巧。与基线相比,我们使用了 **BiDeep RNN架构**( Miceli Barone et al,2017Deep Architectures for Neural Machine Translation)、**标签平滑**( Szegedy et al.,2016Rethinking the Inception Architecture for Computer Vision)、 **dropout**(Srivastava et al., 2014 Dropout: A Simple Way to Prevent Neural Networks from Overﬁtting), **word dropout**(Sennrichetal.,2016aEdinburgh Neural Machine Translation Systems for WMT 16.)、**层标准化**( Ba et al.,2016 Layer Normalization)和**绑定嵌入**- dings( Press and wol,2017 Using the Output Embedding to Improve Language Models)

## 3.2 语言表示

​		像BPE( Sennrich et al,2016c)这样的子单词表示已经成为实现开放词汇翻译的流行选择。**BPE有一个超参数,即合并操作的数量,它决定了最终词汇表的大小。在高资源环境下,词汇量对翻译质量的影响较小;** <u>Haddow等人(2018)在比较30k和90k个子单词时报告了混合结果</u>。
​		在低资源环境中,大词汇表导致在训练时将低频(ub)单词表示为原子单位,学习这些单词的良好高维表示的能力令人怀疑。 <u>Sennrich等人(2017a)提出了子单词单位的最小频率阈值,并将任何较不频繁的子单词拆分为更小的单位或字符我们期望这样的阈值可以减少对数据集的词汇表大小进行仔细调整的需要,从而在更小的数据集上实现更积极的分割</u>。

## 3.3 超参数调优

​		**由于训练时间较长,超参数很难通过网格搜索进行优化,而且经常在实验中重复使用**。然而高资源和低资源设置之间的最佳实践是不同的。虽然高资源设置的趋势是使用更大和更深的模型, Nguyen和Chiang(2018)对更小的数据集使用更小和更少的层。**之前的工作已经证明NMT** (Mor- ishita et al., 2017: (Neishi et aL., 2017)但是我们发现在**低资源设置中使用较小的batches是有益的**。**More aggressive dropout,包括整个单词随机删除dropping whole words(Ga和Ghahramani,2016),也可能是更重要的**。根据以前的工作和我们自己的直觉,我们报告了窄超参数搜索的结果。

## 3.4 词汇模型

​		**最后,我们对 Nguyen和 Chiang**(2018 Improving Lexical Choice in Neural Machine Translation)**的词法模型进行了实现和测试,该模型已被证明在低数据条件下是有益的**。<u>核心思想是训练一个简单的前馈网络,即词汇模型,与原始的注意NMT模型相结合。词法模型在第t步的输入为源嵌入的加权平均值f注意权重a与主模型共享)。经过一个前馈层(带跳跃连接)后,词法模型的输出hlt与原始模型的隐藏状态相结合,然后进行 softmax计算。</u>

![image-20200514142810973](https://tva1.sinaimg.cn/large/007S8ZIlly1gerypa238qj30eu04kaaf.jpg)

​		我们的实现在词法模型(Implementation released in [Nematus](https://github.com/EdinburghNLP/nematus):)中增加了dropout和层规范化。

# 4 实验

## 4.1 数据和预处理

​		我们使用来自2014年WSLT德语→英语共享翻译任务的TED数据( Cettolo et al,2014)。我们使用与 Ranzato et al(2016)相同的数据清理和训练/开发分割,结果是159.00个训练数据的并行语句,7584个用于开发。
​		作为第二语言对,我们在[韩语-英语数据集](https://sites.google.com/site/koreanparalleldata/)中评估我们的系统,包含大约9000个训练数据的平行句子,1000个用于开发,2000个用于测试。

​		对于 PBSMT和NMT,我们使用 Moses脚本应用相同的标记和 truecasing。对于NMT,我们还学习了BPE分词与3万个合并操作,共享德语和英语,并独立为韩国→英语。
​		为了模拟不同数量的训练资源,我们对IWSLT训练语料库进行了5次随机抽样,每一步丢弃一半的数据。在完整的训练语料库上学习Truecaser和BPE分割;作为我们的实验之一,我们在每个子语料库中将子单词单位的频率阈值设置为10(见32)。表1显示了每个子语料库的统计数据,包括子单词词汇表。
![image-20200514143803334](https://tva1.sinaimg.cn/large/007S8ZIlly1geryzjk8efj30ea0beaba.jpg)

表1:针对IWSLT14 De en数据的不同子集和针对Ko en数据的训练语料库大小和子词词汇量。

​		使用 sacreBLEU对翻译输出进行了 detruecase、detox- enized,并将其与大小写BLEU进行比较( Papineni et al.,2002;邮报》2018)。4与Ranzato等人(2016)一样,我们报告了2014年WSLT的连接开发集BLEU(tst2010,tst2011tst2012,dev2010,dev2012)。

## 4.2 PBSMT基线

​		我们使用 Moses( Koehn et al,2007)来训练一个PBSMT系统。我们使用MGIZA(Ga0和ogel2008)训练单词对齐,使用mplz( heet a.,2013)训练5-gram LM。特征权重在开发集上进行优化,以批量MRA最大化BLEU( Cherry和 Foster,2012)—我们在指定的地方执行多次运行。与Koehn和 Knowle(2017)不同,我们没有为LM使用额外的数据。 PBSMT和NMT都可以受益于单语数据,因此单语数据的可用性不再是 PBSMT的唯一优势(见2.2)。

## 4.3 NMT系统

​		我们用 Nematus训练神经系统( Sennrich et al,2017b)。我们的基线主要遵循以下设置( Koehn和Knowles,2017);我们使用adam(Kingma and Ba2015)并基于 dev set bleu执行 early stop。我们以令牌的数量表示批处理大小,并在基线中将其设置为400(与之前工作中使用的80个句子的批处理大小相当)
​		我们随后添加了第3节中描述的方法,即bideep rnn、标签平滑、 dropout、绑定嵌入、层标准化、更改BPE词汇表大小、批处理大小,模型深度,正则化参数和学习率。详细的超参数见附录A。

# 5 结果

![image-20200514144132210](https://tva1.sinaimg.cn/large/007S8ZIlly1gerz366ydgj30ug0gi0wk.jpg)

​		图2:德语→英语学习曲线,表示为 PBSMT和NMT的BLEU作为并行训练数据量的函数。

​		表2显示了在基准NMT系统中添加不同方法对超低数据条件(训练数据100 words)和完整ⅣSLT14训练语料库(320万 words)的影响。我们的“主流改进”在两种数据条件下都增加了大约6-7个BLEU。
​		在超低数据条件下,减少BPE词汇量非常有效(+4.9BLEU)。将批处理大小减少到1000令牌将导致BLEU增益为0.3,词法模型将产生额外的+06BLEU。但是,主动word) dropout(+34BLE∪)和调优其他超参数(+0.7BLEU)的效果比词法模型更强,并且在优化配置(8)之上添加词法模型(⑨)并不能提高性能。6所有这些对超低数据设置的适应性产生了94个BLEU(72-16.6)。在全SLT数据上训练的模型对我们的变化不太敏感(31.9→32.8BLEU),最优超参数根据数据条件的不同而不同。随后,我们仍然将优化后的超参数应用于其它数据条件超低数据条件(8),韩语→英语,for simplicity。
​		要与 PBSMT进行比较,并跨越不同的数据设置,请考虑图2,其中显示了 PBSMT的结果我们的NMT基线和我们优化的NMT系统。对于320万字的训练数据,我们的NMT基线仍低于 PBSMT系统,这与 Koehn和 Knowles(2017)的结果一致。然而,我们优化的NMT系统显示了强大的改进,并优于所有数据设置的PBSMT系统。附录B中显示了一些翻译示例。

![image-20200514144716767](https://tva1.sinaimg.cn/large/007S8ZIlly1gerz95gpnhj30he0b00vb.jpg)

​       表3:使用Multi-Bleu.perl对标记化和小写测试集的完整IWSLT14德语英语数据的结果。	

![image-20200514145454221](https://tva1.sinaimg.cn/large/007S8ZIlly1gerzh2qyq2j30d20b2t9x.jpg)

​	图2:德语英语学习曲线，显示BLEU作为平行训练数据量的函数，用于PBSMT和NMT。

​		为了与以前的工作进行比较,我们在表3中报告了完整的WSLT14训练集的小写和标记化结果。我们的结果远远超过 Wiseman和Rush(2016)报告的基于 rnnbased的结果,并与该数据集上的最佳报告结果持平。

![image-20200514144515125](https://tva1.sinaimg.cn/large/007S8ZIlly1gerz719eewj30cs06umy0.jpg)

表4:韩语→英语成绩。报告了三次训练的平均和标准偏差。

​		表4显示了韩语→英语的结果,使用与德语-英语相同的配置(1、2和8)。我们的结果证实了我们所应用的技术在数据集上是成功的,并且产生了比以前在这个数据集上报道的更强大的系统,达到了10.37与Gu等(2018b)报道的597BLEU相比。

# 6 结论

​		我们的结果表明,在低数据环境下,NMT实际上是一种合适的选择,并且在并行训练数据比之前宣称的少得多的情况下,NMT可以胜过PBSMT。近年来,低资源大地电磁测深研究的主要趋势是更好地开发单语和多语资源。我们的结果表明,低资源NMT对超参数非常敏感,比如BPE词汇量、单词删除等,通过遵循一套最佳实践,我们可以在不依赖辅助资源的情况下训练具有竞争力的NMT系统。这对于那些无法获得大量单语数据或涉及相关语言的多语数据的语言具有实际的相关性。即使我们只专注于使用并行数据,我们的研究结果也与工作相关使用辅助数据来提高资源缺乏MT。监督系统作为一个重要的基准来判断semisupervised或无监督方法的有效性,和质量监督系统训练数据可以
直接影响 semi-supervised工作流,例如反向翻译语的数据。

# A Hyperparameters

​		表5列出了消融研究中不同实验使用的超参数(表2)。除了验证间隔和子单词词汇量之外，超参数在不同的数据设置下保持不变(见表1)。

![image-20200514150245750](https://tva1.sinaimg.cn/large/007S8ZIlly1gerzp97mcbj30r60pmq76.jpg)

表5:表2中报告的NMT系统配置。 空的数据表明，与以前的系统相比，超参数没有变化。

# B Sample Translations

![image-20200514150340849](https://tva1.sinaimg.cn/large/007S8ZIlly1gerzq7whqcj30te0ggn1o.jpg)

表6:基于短语的SMT和NMT系统在100K/3.2M字并行数据上训练的德语英语翻译实例。		

​		表6显示了一些示例翻译，它们代表了我们的PBSMT和NMT系统的典型错误，使用超低(10万字)和低(320万字)数据量进行训练。对于像blutbefleckten(“血迹斑斑”)或西班牙文(“西班牙人”、“西班牙人”)这样的生词，PBSMT系统默认为复制，而NMT系统在子单词级别上产生翻译，并取得不同的成功(blue-flect、bleed;spaniers,略ans)。NMT系统即使使用很少的数据也可以学习一些语法消歧，例如das 和die 作为关系代词(“that” 、“which”“who”)的翻译，而PBSMT生成的语法翻译较少。另一方面，极低资源的NMT系统ig-取消了一些不熟悉的单词，以支持更流畅，但语义不充分的翻译:erobert(“征服”)被翻译成do，而richtig aufgezeichnet( “注册的cor- rectly” ，“正确记录”)成为真正的一个的事情。
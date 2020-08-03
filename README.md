# 可控文本生成

@(2020年8月1号)

**可控文本生成**是十分困难的一件事情，目前大家的主要思路有亮点 
- **属性分离** ：把文本的style和content 分离开，论文主要研究的焦点是如何分离两者，也即是如何让两者正交。
- **调整隐向量** ：主要做法就是训练一个AutoEncoder，把句子压缩到一个隐向量，然后根据需要调整隐向量，把调整后的隐向量送入预先训练的AutoEncoder的解码器部分，使之生成符合要求的文本。




## 属性分离

> 把文本的style和content 分离开，论文主要研究的焦点是如何分离两者，也即是如何让两者正交。

[Style Transfer from Non-Parallel Text by Cross-Alignment](https://link.zhihu.com/?target=https%3A//papers.nips.cc/paper/7259-style-transfer-from-non-parallel-text-by-cross-alignment.pdf) 这篇文章是NIPS 2017的文章，主要是为了建立一个共享的隐变量空间 Z , 论文说，如果简单粗暴的使用最简单的VAE可能会建模进去文本全部的信息，也就是说把文本的风格也建模进去了。文章为了分离两者，用了对抗训练的方法。

[Style Transfer in Text: Exploration and Evaluation](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1711.06861) 这篇文章聚焦于如何获取一个不包含文本属性的content representation。文章采用了类似于多任务学习的方式，用了一种对抗训练的方式提取出content representation，再接着使用两个模型基于content representation进行decode。

[Style Transfer Through Back-Translation. Shrimai Prabhumoye](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1804.09000) 这是ACL 2018这篇文章，它用了一个很巧妙的方法得到了一个不包含style的content vector。作者先把带style的句子通过一个预训练好的英翻法的机器翻译，得到含有style的法语句子，再通过法翻英系统，把encoder端的隐藏层表示拿出来（命名为z）。作者说由于这两个翻译系统是在non-style的文本上训练的，所以z应该是non-style的content表示，从而再根据content的表示z过两个不同的deocder端，得到不同style的文本。最后，为了确保两个decoder端的确decode出了不同style的文本，还加了一个分类器一块儿训练。

[Delete, Retrieve, Generate: A Simple Approach to Sentiment and Style Transfer](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1804.06437) 这是NAACL 2018的文章，它的思路是先删去源句子中的情绪词，然后用其他情绪词代替，做法简单粗暴，只是在句子中换词。



## 调整隐向量

> 主要做法就是训练一个AutoEncoder，把句子压缩到一个隐向量，然后根据需要调整隐向量，把调整后的隐向量送入预先训练的AutoEncoder的解码器部分，使之生成符合要求的文本。

[A Dual Reinforcement Learning Framework for Unsupervised Text Style Transfer](https://link.zhihu.com/?target=https%3A//export.arxiv.org/pdf/1905.10060) 
这是IJCAI-2019的文章。这篇文章不做content和style的分离，直接学习两种风格之间的一步映射关系。


[Controllable Unsupervised Text Attribute Transfer via Editing Entangled Latent Representation](https://arxiv.org/abs/1905.12926v1) 这篇文章是我想follow的文章，下面的论述部分会详细说明原因。

风格和内容分离，是很难做好分离，现在大家更倾向于把隐向量微调来生成符合要求的文本。作者说他们不同于传统的方法，将属性和内容表示分开进行建模。作者直接使用内容和属性缠绕在一起的表示。
文中的模型主要分为两部分，一个是基于transformer的AutoEncoder，一个是Attribute 的Classifier。

![Alt text](./1596467965497.png)

作者首先将AutoEncoder和Attribute Classifier分开来训练
- 然后使用encoder部分去获得source sentence的隐层表示
- 再用FGIM算法去不断编译这个隐层表示，直到这个表示能够被分类器判定为target属性
- 最后在使用decoder从这个隐层表示获取target text.


FGIM算法的思想非常的简单，它使用预训练的隐向量属性分类器，来更新隐层表示z，作者固定了Attribute Classifier的参数值，只改变输入z的值，使z来适应分类器。然后把z送入decoder。这样就生成符合要求的句子了。


### 论述follow的原因
这篇文章很显然有两个不足的地方。
1. 隐向量空间是离散的，对于更细粒度的生成是无法完成的，我们可以想办法让z的空间连续。
2. 生成的文本类似文本的风格转换，不能自由的生成文本，也就是说，如果我们把z约束到高斯分布，采样一个z，然后用分类器微调下，就可以得到随机生成的符合要求的句子。
### note
虽然上面的想法很美好，但是不确定调参能否work，因为这篇论文的代码在github，提了很多issue，在它的基础上改进，是否会work。
![Alt text](./1596468693702.png)

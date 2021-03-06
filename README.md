# 摘要
    目前用于许多NLP（自然语言处理）任务的方法是递归神经网络，特别是LSTMs（长短期记忆网络），和卷积神经网络。然而，这些框架相比于深层卷积网络都很浅，深层卷积网络一直都在推进计算机视觉的技术水平发展。此论文提出一种新的架构：VDCNN，用于文本处理，其直接作用于字符级别上，只是用小卷积层和池化层，最多使用29层，这是第一次将vdcnn用于文本处理。

# 1、介绍
    自然语言处理的任务是处理文本来提取信息然后用不同的方式来表示信息。
    许多NLP方法都将词作为基本单元。一个很重要的步骤是词连续表示的引入（Bengio et al., 2003）。这些词嵌入技术是当前技术发展水平。然而，我们如何最好的表示一个词语序列，例如一整个句子，其中有复杂的句法和语义联系。大体上，对于相同的句子，我们可能面临局部和长距离相关性。目前的主流方法是把句子看成是标识（字符或词）的序列，然后用RNN（时间递归神经网络）来处理。标识通常以一定的顺序来处理，从左到右，RNN内部状态可以记忆整个序列。最流行也是最成功的RNN变体是LSTMs。LSTMs可以对长距离关联性很好的建模。LSTMs对序列处理有用，但是缺乏针对具体问题的架构。
    全连接层可以学习任何实函数，但使用一个深层的特定问题架构（分层表示）可以得到更好的结果。通过这些手段，搜索空间非常受约束，使用梯度下降可以学习有效的解决方案。卷积网络适用于计算机视觉是由于图像的组成结构。文本也拥有相似的特性，由字符组成n-grams、词干、短语、句子等。
    自然语言处理目前的一个挑战是开发处能够根据具体任务来学习整个句子的层级表示的深层框架。在这篇论文中，建议使用多达29层卷积层的深层框架来完成目标。此框架的设计受目前计算机视觉进展的启发。

# 相关工作
    目前有一大块领域关于研究情感分析，或者更一般的在句子分类上。最初的方法是遵循两个传统的阶段计划：人工提取特征，接着是分类。典型的特征包括词袋模型、n-grams和他们的tfidf值。最近，词语或字符，被映射到一个低维空间，这些词嵌入的结合得到一个输入句子的固定大小表示，然后将此表示作为分类器的输入。这通常表现不佳，因为所有标识的顺序都被忽略了。
    另一种方法是结构递归神经网络。其主要思想是使用一个外部工具，叫做分析器，它指定了词嵌入组合的顺序。每个结点的上下文使用权重而组合在一起，顶节点的状态用来训练分类器。时间递归神经网络（RNN）可以看作是结构递归神经网络的一个特例：组合序列化，通常从左至右。RNN的最后状态被用作句子的固定大小表示，或者作为所有隐藏状态的一个组合。
    2014年Kim提出一个相当浅的网络：一个卷积层和一个最大池化层，最后连接一个全连接层，并采用dropout。
    2014年Kalchbrenner提出一个5层网络。其主要不同点在于多时域k最大池化层的引入。这可以检测句子中前k个最重要的特征，不依赖与他们的具体位置，并保留相对顺序。k值取决于句子长度和这一层在网络中的位置。
    2015，Zhang第一次完全在字符级别进行情感分析，他们采用6层卷积层和3个全连接层。卷积层的大小为3或7，同时使用简单的最大池化层。
    2014，Dos使用字符级别，一个词的字符嵌入通过最大化操作组合在一起，然后它们和词嵌入一起使用与一个浅层架构。
    2016，Yang提出用于文档分类的基于分层注意的网络，第一次将注意作用在文档中的一个句子和句子中的一个词语。他们的框架在包含多个句子的样本数据集中表现很好。
    2016，Xiao将rnn和cnn结合成一个网络用于句子分类，一个5层卷积网络用来学习高级别的特征，并以此作为LSTM的输入。最初动机是想使用更少的参数。
    总结，没有发现使用超过6层卷积层的网络来用于句子分类，每人尝试更深的网络或没有更好的表现。这与目前计算机视觉的发展形成强烈反差。

# VDCNN 框架
    模型最开始部分是一个查阅表（look-up table），生成大小为（f0，s）的2维张量，包含s个字符的嵌入，s为固定大小1024，f0可以看作是输入文本的“RGB”维。
    首先第一层，用64个大小为3的卷积，随后接一系列时域卷积块，受VGG和ResNets启发，本设计遵从两个规则：
（1）：对于相同输出的时域分辨率，每层都有相同数量的特征映射
（2）：当时域分辨率减半时，特征映射的数量翻倍。
    这有助于减少存储空间。网络包含3个池化操作（每次使用两个池化层，来使时域分辨率减半），产生了三次不同级别的特征映射(128,256,512)。这些卷积块的输出是一个512×sd的张量。输出的张量可以看成是输入文本的高级表示。作为可变大小向量集的文本表示，对于神经机器翻译尤其是与注意力模型结合时是很有价值的。卷积在标识上提取ngram特征，不同的ngram长度需要建立短/长跨度关系的模型，构建一个使用很多层小卷积的框架。叠加这4层可以跨越9个标识，但网络可以自学如何在深度分层行为中最好的结合这些不同的3-gram特性。
    输出单元的数量取决与分类任务，在所有实验中，设置隐藏层单元数为2048，k为8。在全连接层中没有使用drop-out，只使用时域批量归一化（temporal batch norm）来正则化网络。
    每个卷积块包含两个卷积层，每个卷积层后面紧跟一个时域批量归一化和ReLU激活函数。所有时域卷积的大小为3，并使用填补（padding）来保留时域分辨率。
    
# 评估
    使用8个大规模数据集，样本从120k到3.6m，类别从2到14，即类别很少，涵盖情感分析、话题分类、新闻分类。由于类标签很少，导致每个样本都提供很少的梯度信息，这使得训练大架构变得困难。
    字典总共69个字符。输入的文本被填充到1014这一固定长度，更长的文本会被缩短（截断），字符嵌入的大小是16.
    使用SGD（随机梯度下降）来训练，mini-batch大小为128，初始学习率为0.01，momentum为0.9
    使用最大的数据集，对于9层网络，每次迭代在24分钟到2小时45分钟之间，对于29层网络，每次迭代在50分钟到7小时之间。10~15次迭代就可收敛。使用Torch7来实现。使用不带dropout的时域批量归一化。
    

# 论文标题：InterBiasing:通过偏置中间预测来提高看不见的单词识别

## 摘要

尽管端到端语音识别方法最近取得了进展，但它们的输出偏向于训练数据的词汇，导致对未知术语或专有名词的识别不准确。为了提高给定一组此类术语的识别精度，我们提出了一种基于自调节CTC的自适应无参数方法。我们的方法通过用校正的标签替换中间的CTC预测，然后将其传递给后续层，提高了误识别目标关键字的识别精度。首先，我们使用文本到语音和识别模型为关键字列表创建成对的正确标签和识别错误实例。我们使用这些对来用标签替换中间预测误差。通过调节标签上编码器的后续层，可以对目标关键字进行声学评估。用日语进行的实验表明，我们的方法成功地提高了未知单词的F1分数。

## 引言

近年来，深度神经网络的快速发展导致了端到端（E2E）语音识别模型的显著识别性能，包括连接主义时间分类（CTC）[1]、递归神经网络传感器[2]和基于注意力的编码器-解码器[3,4]。这些模型现在被许多用户广泛用于会议记录和人工智能对话系统等实际应用中。然而，这些机器学习模型的性能在很大程度上取决于可用的训练数据。因此，识别培训数据中不易获得的罕见词，如内部术语、人名和行业特定术语，仍然是一个具有挑战性的问题。在许多实际情况下，这些罕见的术语是可以提前获得的，例如，从用户名、会议聊天记录或文档中，甚至是用户注册的单词中。对允许用户通过利用此类信息进行定制的技术的需求日益增长。此外，由于每个用户都有独特的单词要求，并且新单词不断添加，因此希望模型不需要额外的训练。解决这些局限性的传统方法包括使用加权有限状态传感器[5,6,7]。然而，这些方法需要创建和调整解码图，这对于可以使用无图解码的E2E语音识别器（如使用CTC的模型）来说是不理想的。此外，已经探索了其他方法，如使用文本嵌入的深度偏置[8,9]和集成纠错机制[10]。然而，值得注意的是，在文本嵌入方法中，模型的表示取决于其训练数据，这限制了对训练数据中包含的单词的偏见。一种既不需要模型再训练也不需要解码图生成的方法是关键字增强波束搜索（KBBS）[11]。这是一种实用的技术，如果关键字列表中的单词出现在波束搜索假设中，则会给出额外的分数。然而，它的局限性在于，如果目标关键字不在搜索范围内，它就无法奖励奖金。这个问题在日语等拼写多样的语言中尤为明显；如果目标关键字的拼写与训练数据中的拼写不同，则目标关键字不太可能出现在搜索梁中。CTC的峰值后验分布[12]进一步加剧了这一问题。为了使关键字增强有效，目标关键字的后验概率必须是一个高值。在本文中，我们提出了一种新的偏置方法InterBiasing，用于自适应CTC[13]，旨在根据目标关键字调节声学模型中的中间层。自调节CTC在非自回归E2E模型中取得了最先进的性能[14]。在这种架构中，CTC预测在中间编码器层中计算，后续层以预测的CTC序列为条件。然后在多个层中重复此过程。它可以被解释为单个编码器内预测结果的迭代细化[15,16,17]。通过用正确的标签替换目标词汇的中间预测，并在其上调节后续层，我们可以利用迭代细化的框架来增强中间预测，同时考虑目标词汇。所提出的方法的过程可以描述如下。首先，我们使用文本到语音（TTS）合成生成与关键字列表相对应的语音。接下来，将该语音输入到识别模型中，该模型会产生成对的正确标签和识别错误。随后，我们利用这些对通过用精确的标签替换它们来纠正中间预测误差。通过确保编码器的后续层由这些正确的标签通知，我们可以对目标关键字进行声学评估。我们的方法对抗了CTC的峰值后验分布，并允许目标关键字出现在搜索波束中，在那里可以进一步增强。此外，由于只有从话语假设中搜索到的关键字会对中间输出产生偏差，因此无论关键字列表的大小如何，目标词都可能存在偏差。

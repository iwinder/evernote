[Introduction](https://hyperledger-fabric.readthedocs.io/en/latest/whatis.html)
[Hyperledger Fabric 1.3 官方文档翻译 目录](https://blog.csdn.net/hugowang/article/details/82863415)
[Hyperledger Fabric（目录）](https://segmentfault.com/a/1190000015952232)

一般来说，区块链是一个不可变的交易账本,维护在一组对等节点（peer nodes）组成的分布式网络（distributed network）中。每个节点都维护着一份交易账本的副本， 其交易记录都已通过共识协议验证并分组到块中，每个块都包含链接的前一个块的哈希值。

区块链的第一个也是最广为人知的应用是比特币加密货币，其他应用跟随了它的步伐。以太坊是采用了不同的方法另一种加密货币，集成了许多与比特币相同的特性，但增加了智能合约，以创建一个分布式应用平台。基本上，这些是公共网络，对所有人开放，参与者匿名互动。

随着比特币，以太坊和其他一些衍生技术的普及，对将区块链，分布式账本和分布式应用平台的基础技术应用于更具创新性的企业用例的兴趣也在增长。然而，许多企业用例需要的性能特征是非许可链（permissionless blockchain ）技术无法给予的。此为，在许多用例中，参与者的身份是一项硬性需求，例如在必须准守“了解客户”（Know-Your-Customer,简称：KYC）和“反洗钱”（Anti-Money Laundering,简称：AML）法规的金融交易中。

针对企业用途，我们需要考虑以下要求：

- 参与者必须是被识别的或可识别的
- 网络需要获得许可
- 高性能的交易吞吐量
- 低延迟的交易确认
- 与商业交易相关的交易和数据的隐私和机密性

虽然许多早期的区块链平台正在适应企业用途，Hyperledger Fabirc从一开始就被设计适应于企业用途。以下部分描述了Hyperledger Fabirc(Fabirc)如何将自己与其他区块链平台区分开来，并描述了其架决策的一些动机。






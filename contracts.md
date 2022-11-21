![image](https://user-images.githubusercontent.com/118715807/203031108-78a4e6c0-d9e5-484e-8b89-e3951a11d224.png)
1、部署
contract必须部署指定初始(“genesis”)状态根哈希，指定网络管理器(参见“Governance”部分)，并链接退出队列(参见“Cenosorship resistance审查阻力”部分)。

2、Governance
网络的治理将从一个单独的合约中摘录，在ZKSync合约中注册为networkGovernor。它有以下能力:
更改验证器集。
添加新的令牌(添加后不能删除令牌)。
启动迁移到新合约(请参阅“Migration”一节)。

3、Cenosorship resistance
ZKSync采用了优先队列(软执行)和Exodus mode(硬执行)机制，以加强抗审查和保证资金的可回收性。

4、Deposits
要进行存款，用户可以:
发送ETH到智能合约(将由默认功能处理)，
或调用depositERC20()函数对已注册的ERC20令牌执行transferFrom。注意:用户必须在ERC20令牌合约上调用approve()才能授权ZKSync合约执行此操作。
该存款创建存款优先级请求，该请求放在相应的优先级请求映射中，并发出NewPriorityRequest(opType, pubData, expirationBlock)事件，以通知验证器必须将此请求包含到即将到来的块中。处理优先级请求的完整PriorityQueue逻辑在优先级请求一节中描述。

当验证器提交一个包含电路操作的存储块时，为该存储创建存储链操作，以验证是否符合优先队列请求。如果成功，那么它们的计数将被添加到该块的优先级请求计数中。如果区块未被验证，则直接丢弃链上存款操作和存款优先级请求。

如果区块被还原，则直接丢弃链上存款操作。

如果ZKSync合约已经进入Exodus mode模式，且区块未被验证，则该区块的存款优先请求所持有的资金将累积到所有者的根链余额中，以使其可以提取。这种撤销链上操作和完全退出优先级请求被简单地丢弃。

5、Withdrawals
5.1 Partial withdrawal
这是一个标准的取款操作。当一个带有partial_exit电路操作的块被提交时，将创建用于该退出的链上退出操作。如果区块被验证，链上取款操作的资金将计入用户的根链余额。

如果区块被还原，那么这个链上取款操作就会被直接丢弃。

用户可以通过调用withwithth()或withdrawERC20()函数随时从根链余额中提取资金。

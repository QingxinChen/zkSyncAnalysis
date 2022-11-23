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

用户可以通过调用withdrawETH()或withdrawERC20()函数随时从根链余额中提取资金。  

5.2 Full exit  
如果用户认为他的交易被验证者审查，他可以请求这种昂贵的操作来提取资金。  

用户必须向ZKSync合约函数registerFullExit()发送一个交易。此函数创建完全退出优先级请求，并将其置于相应的优先级请求映射中，并发出NewPriorityRequest(serialId, opType, pubData, expirationBlock)事件，以通知验证器必须将此请求包含到即将到来的块中。处理优先级请求的完整PriorityQueue逻辑在优先级请求一节中描述。  

当验证器提交一个包含电路操作full_exit的块时，将为该提取创建相应的链上提取操作，以验证是否符合优先队列请求。如果成功，那么它们的计数将被添加到该块的优先级请求计数中。如果区块被验证，链上取款操作的资金将累积到用户的根链余额中，链上取款操作和全部退出优先级请求将被丢弃。  

如果区块被还原，那么这个链上取款操作就会被直接丢弃。  

如果ZKSync合约已经进入Exodus mode，并且区块未被验证，那么这个退出链上操作和全部退出优先级请求就会被丢弃。  

6、Block committment  
只有验证器集合的发送方才能提交块。  

已提交但未验证的块的数量由MAX_UNVERIFIED_BLOCKS常量限制，以确保gas总是足够在验证未及时发生时恢复所有已提交的块。  

7、Block verification  
任何人都可以对提交的块执行验证。  

8、Reverting expired blocks  
如果第一个提交的区块在EXPECT_VERIFICATION_IN ETH区块中没有被验证，所有未验证的区块将被还原，链上操作和优先级请求持有的资金将被释放并存储在根链余额中。  

9、Priority queue  
这个队列将在单独的合约中实现，以确保优先级操作，如存款和full_exit将被及时处理，并将被包含在ZKSync的块中(导致需要反向块的情况将不会发生)，而且，在full_exit交易的情况下，用户总是可以提取资金(抵制审查)。它的功能分为两部分:请求队列和Exodus Mode模式。  

当用户向ZKSync合约发送交易时触发NewPriorityRequest事件。另外，关于它的一些信息将严格按照到达的顺序存储在映射(操作类型和过期块)中。NewPriorityRequest事件结构:  

serialId—优先级请求的序列号  
opType—操作类型  
pubData—请求数据  
expirationBlock -当请求过期时，以太坊块的数量计算如下:  
expirationBlock = block.number + 250 - about 1 hour for the transaction to expire, block.number - current Ethereum block number.  
当在提交的块中发现相应的交易时，必须记录它们的计数。如果块得到验证，则从映射中删除满足的优先级请求的此计数。  

如果通过Exodus Mode将区块还原，则该区块的存款优先级请求所持有的资金将累积到所有者的根链余额中，以使其可以提取。这个存款优先级请求将从映射中删除。  

10、Fees for Priority Requests  
为了发送优先请求，用户必须支付一些额外的费用。该费用将从用户发送到存款或完全退出功能的以太币数量中扣除。该费用将是验证器在块中包含这些交易的工作的报酬。一笔交易费用计算如下:fee = FEE_COEFF * (BASE_GAS + gasleft) * gasprice，其中  

FEE_COEFF优先请求交易的费用系数  
BASE_GAS—交易的基本燃气成本(通常为21000)  
Gasleft——交易代码执行的剩余气体  
gasprice-燃气交易价格  
如果用户发送的以太数超过了所需，差额将返回给用户。  

11、Validators' responsibility  
验证器必须按RIGHT顺序订阅NewPriorityRequest事件，以便在一些即将到来的块中包含优先级交易。验证器需要尽快将这些交易包含在块中，这是由于进入Exodus Mode的可能性越来越大(如下所述)。  

12、Exodus mode  
如果请求队列处理太慢，它将在ZKSync合约中触发出埃及模式。这个时刻由第一个(最老的)优先级请求和最老的expirationBlock值决定。如果当前以太坊区块号>=最老的到期区块号，将进入Exodus mode模式。  

在Exodus mode中，合约冻结所有块处理，所有用户必须退出。所有现有的区块承诺都将被恢复。  

每个用户都可以通过调用exit()函数提交SNARK证明，证明她拥有ZKSync最新验证状态下的资金。如果证明验证成功，整个用户金额将累积到她的链上余额，并设置一个标志，以防止同一令牌余额的两次退出。  

13、Migration  
(待后期实施)  

ZKSync将始终有一个严格的选择政策:我们保证用户的资金是可回收的条件下，用户选择在存款时的资金，无论什么。迁移到新版本的合同应该是容易和便宜的，但必须要求单独的选择加入或允许用户退出。  

更新机制应遵循以下工作流程:  

网络调控器可以计划更新，指定目标合约和ETH块截止日期。  
不能取消预定的更新(以便继续迁移，即使在等待迁移时迁移模式被激活;否则我们将需要用一个单独的程序来恢复计划迁移的资金)。  
用户可以通过单独的ZKSync操作选择加入:将特定的令牌余额移动到一个特殊迁移帐户的子树中。这个子树还必须维护和更新每个令牌的总余额计数器。  
迁移帐户必须有一个专用的硬编码account_id(用于指定)。  
当到达预定的ETH块时，任何人都必须能够密封迁移。  
在迁移被密封之后，任何人都必须能够通过提供来自迁移帐户子树的金额的SNARK证明来转移每个令牌的总余额。  
当迁移被密封后，合约进入出迁移模式:没有选择加入的人现在可以退出。因此，根状态将保持冻结状态。  
新合约将直接从旧的合约中读取最新的状态根(这是安全的，因为根状态是冻结的，不能更改)。  

14、todo / Questions / Unsorted  
引入键哈希?  
单独部署密钥?  
单位转换:不同的令牌?  
描述ERC20 transferFrom的授权  
执行电路中每个区块的最大存款/出口

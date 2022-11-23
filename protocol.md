1、术语表  
L1: layer-1区块链(以太坊)  
Rollup: layer-2区块链(zkSync)  
Owner:控制L2中某些资产的用户。  
Operator:操作Rollup的实体。  
Eventually:在有限的时间内发生。  
Assets in rollup:由所有者控制的L2智能合约中的资产。  
Rollup key:所有者的私钥，用于控制存放的资产。  
Rescue signature:使用所有者的私钥对所有者的消息进行签名的结果，该结果用于Rollup内部交易。  

2、设计  
2.1 概述  
zkSync为ETH和ERC20可替换令牌传输实现了ZK rollup协议(以下简称“rollup”)。  

总体上rollup工作流程如下:  

  用户可以通过从L1存储资产或从其他所有者接收转移来成为rollup中的所有者。  
  所有者可以互相转让资产。  
  所有者可以将其控制下的资产提取到L1地址。  
Rollup操作需要操作员的帮助，操作员将交易滚到一起，计算正确状态转换的零知识证明，并通过与Rollup合约交互影响状态转换。  

2.2 假设  
密码学的假设:  
DLP是坚不可摧的。  
Rescue hash and sha256是抗碰撞的。  
架构中使用的ZKP方案是安全的(需单独提供正式证明)。  

L1区块链假设:  
L1协议安全。  
L1最终可以抵抗审查:在有限的时间内，一个足够高价格的L1 tx将在一个区块中被挖掘出来。  
所有者可以访问完整的L1存档(可以在任何时候检索L1链的所有块体)。  

操作的假设:  
rollup key由所有者控制，在任何时候都不会被泄露。  

2.3 协议不变声明  
1. 持续所有权:在rollup中存入的资产立即处于指定的、惟一的所有者的控制之下。  
2. 控制:在rollup中的资产不能被转移(改变所有者)、改变价值、消失或移出rollup，除非所有者发起相应的操作。  
3.最终可回收性:所有者控制的资产最终可被提取到所有者选择的L1地址，而无需除L1矿工以外的任何其他方的合作。  

这特别包括下列声明:  
3.1. Rollup中的双重花费是不可能的。  
3.2. rollup总是在完全准备金的情况下运行:rollup中每种资产类型的总供应量等于其所有存款金额减去其所有提取金额的总和。  
3.3. 根哈希总是正确的，L2状态不能损坏  
3.4. 状态总是可以从发布到L1的calldata中恢复  
3.5. 智能合约不能被锁定  

3、数据格式  
3.1 数据类型  

3.2 Amount packing  
金额和费用在zkSync中使用简单的浮点运算基础进行压缩。  

浮点数有以下几个部分:尾数、基数和指数。尾数(在本例中总是非负的)保存浮点数的有效位数。指数表示尾数和符号应乘以的基数的幂。这些组件按如下方式组合得到浮点值:  
sign * mantissa * (radix ^ exponent)  

3.3 State Merkle Tree  
账户和余额树表示:  
![image](https://user-images.githubusercontent.com/118715807/203212317-a2cb1e9c-bbf9-4aa9-8246-a7612ebab85a.png)  

Legend:  
Ha是指account树高。(24）  
Hb为balance树高。(8)  
我们直接有一个主Accounts树和它的叶子，Accounts也有子树余额树和它们自己的余额叶。  

3.3.1 叶哈希  
叶哈希是下面描述的它的位串表示的 rescue hash。为了得到位串表示，每个字段被编码为从最低有效(位的LE顺序)开始的位，并按照它们在结构中出现的顺序进行连接。  

3.3.2  Account leaf  
每个帐户被插入到Accounts树中，作为具有最小空闲id (AccountId)的叶节点。  

state_tree_root使用balance_tree_root根哈希的Rescue hash(作为字段元素)和为未来子树根哈希保留的零字段元素进行组合，然后填充到256位(通过在LE位表示中添加0位)  

空叶子包含:state_tree_root使用空余额子树计算。帐户的空余额子树在每个叶中都为零。  

3.3.3 Balance leaf  
空叶包含值等于零。  

3.4 zkSync block pub data format  
Rollup块pub data由Rollup操作pub data序列组成。最大块大小是一个恒定值。如果块中包含的操作的大小不足以完全填充它，则其余的操作将由空的Noop操作填充。  

4、zkSync operations  
zkSync操作分为Rollup交易(由Rollup账户在Rollup内部发起)和Priority操作(由以太坊账户在主链上发起)。  

Rollup transactions:  
Noop  
Transfer  
Transfer to new  
Withdraw (Partial exit)  
Change pubkey  
Forced Exit  
MintNFT  
WithdrawNFT  
Swap  

Priority operations:  
Deposit  
Full exit  

Full list: https://docs.google.com/spreadsheets/d/1ejK1MJfVehcwjgjVDFD3E2k1EZ7auqbG_y0DKidS9nA/edit#gid=0  

Legend:  
Transaction:用户可以直接提交给操作者的内容。  
Priority operation优先级操作:用户可以向zkSync智能合约提交什么。  
Rollup operation:部分Rollup块表示交易或优先级操作。  
Onchain operation链上操作:操作员可以将什么放入到Rollup块pubdata(操作pubdata)中。  
Node implementation节点实现:描述操作的节点模型。  
Circuit implementation电路实现:描述操作及其witness的电路模型。  
Chunk块:操作的维度。每个数据块都有自己的部分公共数据(10字节)，由目击者提供。  
Significant bytes有效字节:操作占用的所有字节中有多少字节是有效的(包括操作号)。  
Hash:SHA-256函数以操作的pubdata为输入的结果。用于操作标识。  

4.0 Rollup operation生命周期  
用户创建交易或优先级操作。  
处理此请求后，operator创建一个Rollup操作并将其添加到块中。  
一旦区块完成，操作员将其作为a block commitment区块承诺提交给zkSync智能合约。智能合约检查了一些Rollup操作的部分逻辑。  
块的证明被提交给zkSync智能合约作为块验证。如果验证成功，则认为新状态已完成。  

circuit描述中使用的定义  
为了便于理解，下面用类似python的伪代码描述关键circuit逻辑。它描述了在执行汇总操作后应该保留的不变量，以及如果该不变量为真，则应该执行的帐户树更新。  
有两个主要的不变函数:tree_invariants和pubdata_invariants。这些函数中的每个谓词都应该求值为true。  
伪代码语言中使用的函数和数据结构:  
#返回指定id的帐户树中的帐户;它总是返回一些东西，因为默认情况下帐户树是由空帐户填充的。  
get_account_tree (account_id)→account  

#Unpacks packed金额或费用;这些函数隐式地执行了金额和费用是packable的不变条件  
unpack_amount(packed_amount) -> StateAmount  
unpack_fee(packed_fee) -> StateAmount  

#检查帐户是否为空:address, pubkey_hash, nonce和所有余额为0。  
is_account_empty(Account) -> bool  

#返回给定交易的签名  
recover_signer_pubkey_hash(tx) -> RollupPubkeyHash  

#账户数据结构  
Account.balance[token_id] -> StateAmount # returns account balance of the given token  
Account.nonce -> Nonce # Nonce of the account  
Account.address -> EthAddress # Address of the account  
Account.pubkey_hash -> RollupPubkeyHash # Hash of the public key set for the account  

# Constants  
MAX_TOKENS = 2**32 # maximum number of tokens in the Rollup(including "ETH" token)  
MAX_FUNGIBLE_TOKENS = 2**16 # maximum number of fungible tokens in the Rollup(including "ETH" token)  
MAX_NONCE = 2**32 # max possible nonce  

4.1 Noop operation  
此操作用于将零字节(在calldata中很便宜)的块填充到完整的块容量。  

4.2 Transfer  
在Rollup帐户之间转移资金。  

todo：User transaction干嘛用的？"amount": "12340000000000"怎么转成amount: 0x5bf0aea003  

4.3 Transfer to new  
将资金从Rollup帐户转移到新的Rollup帐户(动态地为Rollup地址分配一个空闲的帐户号码)。因此，“帐户创建”将首先执行，即分配对应的RollupAddress - AccountId。然后通常的资金在Rollup账户之间的转移就会发生。  

4.4 Withdraw (Partial Exit)  
从Rollup账户中提取资金到指定以太坊地址的适当余额。  

4.7 Deposit  
将以太坊账户的资金存入指定的Rollup账户。存款作为优先操作开始——用户调用合约方法depositERC20来存放以太坊，或depositERC20来存放ERC-20代币。之后，操作符在块中包含此操作。如果需要，将在帐户树中创建新帐户。  
operator审查  
operator可能出于某种原因没有在块中包含此操作。然后，通过智能合约上设置的以太坊区块数量，启动exodus mode。它将允许收件人帐户(即与msg。sender == Deposit.to)从zkSync合同中提取资金。  

4.8 Full exit  
如果用户认为他的交易被验证者审查，他可以请求该操作来提取资金。  
它作为优先级操作开始——用户调用合约方法fullExit。之后，操作者在块中包含此操作。  

如果服务器端出现了错误，那么完整退出操作将包含在pubdata中数量为0(0)的块中。  

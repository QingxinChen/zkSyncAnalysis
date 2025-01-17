1、Rollup 是一大类 Layer-2 扩容方案的统称，特指先在链下进行复杂的计算和状态维护，再将与状态更改相关的数据通过合约调用的方式，利用更便宜的 CALLDATA 在链上保存。任何人都能根据链上保存的数据复原出全局的状态，从而消除因数据可用性问题带来的安全风险。Rollup 将大量交易卷起 / 汇总成为一个交易，在保证数据可用性的前提下提高 TPS。

2、ZK Rollup 和 Optimistic Rollup 是目前较热的两种不同方案，它们的核心区别如下：  
2.1 ZK Rollup 方案的关键在于 ZK，它的每一次的状态转变都需要提供零知识证明，并由主链上的合约进行验证，只有验证通过才能更改状态。即，ZK Rollup 的状态转变严格依赖于密码学证明。  
2.2 Optimistic Rollup 方案中，每次状态转变无需严格验证，它是先乐观地假设每次转变都是正确的（这就是 Optimistic 一词的由来），然后在一定时限内可以对某次转变进行挑战，如果挑战成功就证明之前的提交有问题，会惩罚提交者并将状态回滚。即，Optimistic Rollup 的状态转变依赖于经济激励和博弈。  

两种方案的差别还可以从证明模型角度来看：前者为 Validity Proof（有效性证明），只有提供了「有效性证明」的状态才会被写入主链合约；后者为 Fraud Proof （错误性证明），用户需要在挑战期内对异常提供「错误性证明」，举报不正确的状态。  
在安全性上，ZK Rollup 更具优势。因为 Optimistic Rollup 或基于 Fraud Proof 的二层扩容协议必须在挑战期内投诉举报，因此可以构造一个场景让矿工配合做恶，在挑战期内拒绝掉所有提交 Fraud Proof 的投诉交易，导致不正确的状态转变会被确认，攻击者可从合约中盗取资金。但这种攻击方式对 ZK Rollup 无效，因为它的合约中始终有正确性校验。  
在 TPS 上，ZK Rollup 利用 zk-SNARK 技术在保障数据正确性的同时压缩了链上计算量。所有交易的计算过程不用在合约中执行， Operator （运营者）只需把存储账户状态的 Merkle Tree 的 Merkle Root、交易数据和 zk-SNARK 证明提交至合约，合约验证通过后将新的状态写入。  
因为 zk-SNARK 证明大小（很小）与验证时间（很快）是常数，不会随交易数量增长，因此 ZK Rollup 可以极大地提高交易 TPS。ZK Rollup 的链上性能限制仅取决于 CALLDATA 存储数据的成本，随着以太坊伊斯坦布尔升级的完成，CALLDATA 使用成本降为原来的 1/4，ZK Rollup 的性能则获得 4 倍提升，TPS 可达到近 2000。  

3、ZK Rollup 就是在Layer 2 维护Merkle tree（默克尔树）。它最开始的时候非常简单，里边什么都没有，然后当你交易或支付时，就会改这棵树里边的数据，这个改动本身是要放到以太坊上去的，作为一个数据存到以太坊上。（注：并不是把这棵树放在以太坊上，这棵树在Layer 2，但要把对这棵树的改动的数据放在以太坊上）  
因此，你可以通过以太坊把这棵树在任何一个时间点的历史状态全部恢复出来，恢复出来之后，你可以通过树根来验证你恢复的数据是不是对的。  
最极端的状况，如果这个交易所在某一个状态停止了，破产了或者团队跑路了，这个时候可以通过以太坊的数据把这棵树恢复出来，任何一个用户都可以把从叶子节点到根的一串数据拿过来，这一串的数据叫默克尔证明。  
把这个证明扔到以太坊的智能合约里，合约就会算这个默克尔证明能不能证明你确实是在这棵树里面，如果能证明你在这棵树里，就会把这棵树里标记的你有多少钱从以太坊的智能合约里解锁出来，直接转到你的以太坊账号里。  
总的来讲，关键的一点是把包括Merkle tree 的根在内的数据上链，第二个点是这个默克尔证明本身是能够通过智能合约去验证的，而且一旦验证成功就把钱打给对方。这就是在最不理想的情况下提现的方式。  
比如说做1000笔交易，改动可能是大约4000个叶子节点，可能要算几万次哈希算出一个根，但这些数据都在Layer 2，最后扔到以太坊上的数据其实就是三种：第一个是对各个叶子怎么改的；第二个是对根怎么改的；第三个是一个证明来证明前两者的一致性。  
零知识证明做什么呢？零知识证明就是验证改这棵树里边这么多数据的时候，这个根的计算跟各种改动是能够匹配得上的。零知识证明只是做了这么一件事，就是证明数据的一致性。  
它对应的也不是每一笔交易怎么去证明，而是说一大堆交易打成一个包/块，如何去证明这个包，它是一个批处理的过程。  

4、Layer 2 是有一个智能合约在Layer 1 上面的，出入金的时候要经过这个智能合约，Layer 1 和Layer 2 之间价值转换的相关部分也是通过智能合约来做的操作。这个智能合约可以跟Layer 1 的任何的项目做交互，在出入金的那个交易里面跟Layer 1 做可组合。

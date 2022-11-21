1、zkSync smart contract:部署在以太坊区块链上的Solidity智能合约，用于管理用户的余额，并验证zkSync网络中执行的操作的正确性。
2、Prover application证明应用程序:为已执行块创建证明的工作应用程序。证明应用程序轮询可用作业的服务器应用程序，一旦出现新的块，服务器就提供一个见证(输入数据以生成证明)，然后验证程序开始工作。一旦生成了证明，就将其报告给Server应用程序，然后Server将证明发布到智能合约。证明程序应用程序被认为是按需工作程序，因此可以有许多证明程序(如果服务器负载很高)或根本没有证明程序(如果没有传入事务)。生成证明是一项非常消耗资源的工作，因此运行证明程序应用程序的机器必须具有现代化的CPU和大量RAM。
3、Server application:运行zkSync网络的节点。它能做以下事情:
监视链上操作(如存款)的智能合约。
接受交易。
生成zkSync链块。
请求已执行块的证明。
将数据发布到智能合约。

服务器应用程序有两种可用形式:
a 单片应用程序，它从一个二进制文件中提供所有所需的功能。这种形式便于开发需要。对应的板条箱为core/bin/server。
b 微服务应用程序，它们能够彼此独立地工作:
核心服务(Core /bin/zksync_core)维护事务内存池并提交新块。
API服务(core/bin/zksync_api)提供了一个服务器“前端”:REST API & JSON RPC HTTP/WS实现。
Ethereum Sender service(core/bin/zksync_eth_sender)通过将相应的以太坊交易发送到L1智能合约来finalizes the blocks。
Witness Generator service (core/bin/zksync_witness_generator) 创建证明者证明块所需的输入数据，并实现一个私有API服务器供证明者交互。

4、本节提供了此存储库中存在的文件夹/子项目的概述。
/bin:帮助使用zkSync应用程序的基础架构脚本。
/contracts:所有与zkSync智能合约相关的东西。
  /contracts:智能合约代码
  /scripts && /src。ts:用于智能合约管理的TypeScript脚本。
/core:实现zkSync网络的子项目代码。
  /bin: zkSync网络运行所必须的应用程序。
    /server: zkSync服务器应用程序。
    /prover: zkSync验证程序。
    /data_restore: 从智能合约恢复zkSync网络状态的实用程序。
    /key_generator:为网络生成验证密钥的实用程序。
    /parse_pub_data:解析zkSync操作pubdata的实用程序。
    /zksync_core: zkSync服务器核心微服务。
    /zksync_api: zkSync服务端API微服务。
    /zksync_eth_sender: zkSync服务器以太坊发送端微服务。
    /zksync_witness - Generator: zkSync服务器见证生成器和证明服务器微服务。
  /lib:上面二进制文件的依赖关系。
    /basic_types:带有基本zkSync原语声明的板条箱，例如地址。
    /circuit:强制zkSync网络中执行事务的正确性的加密环境。
    /config:加载zkSync应用程序的配置选项的实用程序。
    /contracts:zkSync契约接口和ABI的加载器。
    /crypto:在zkSync板条箱之间使用的加密原语。
    /eth_client:提供与以太坊节点交互的接口的模块。
    /prometheus_exports: Prometheus数据导出器。
    /prover_utils:与证明生成相关的实用程序。
    /state:一个用于服务器级生成块的zkSync事务的快速预电路执行器。
    /storage:封装的数据库接口。
    /types: zkSync网络操作、事务和常见类型。
    /utils: zkSync板条箱的其他助手。
    /vlog:用于详细日志记录的实用程序库。
  /tests: zkSync网络的测试基础设施。
    /loadnext:用于zkSync服务器高负载测试的应用程序。
    /test_account:可用于测试的zkSync帐户的表示。
    /testkit:一个相对低级的zkSync测试库和测试套件。
    /ts-tests: TypeScript中实现的集成测试集。需要运行服务器和验证程序应用程序才能操作。
/docker:用于开发zkSync和将zkSync打包到生产环境的Dockerfiles。
/etc:配置文件。
  /env:.env文件包含zkSync Server / Prover的不同配置的环境变量。
  /js: JavaScript应用程序(如Explorer)的配置文件。
  /token:支持的以太坊ERC-20 token的配置。
/infrastructure:应用程序不是zkSync核心的一部分，但与之相关。
/keys:电路模块的验证键。
/sdk:用不同的编程语言实现zkSync网络的客户端库。
  /zkSync -crypto: zkSync网络加密原语，可编译为WASM。
  /zkSync. js: zkSync的JavaScript / TypeScript客户端库。
  /zkSync -rs: zkSync的Rust客户端库。

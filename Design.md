# Carnot

carnot的构建目标是实现代币在TTC和Cosmos两种不同的区块链间的跨链转移，包括其基于Cosmos SDK开发的侧链和需要部署在TTC上的智能合约。通过Cosmos SDK这个框架，我们构建一个简单的具有业务功能的状态机，以其作为媒介和智能合约产生交互。

Cosmos（Tendermint）和TTC均采用DPOS共识机制，所以在这两种区块链中，均存在最终确定性。

## 系统组成

### 状态机

状态机是基于Cosmos SDK开发的部分，其具有特定的业务功能，能够作为侧链跟Cosmos Hub相连接，并按照cosmos本身的功能，支持在此侧链上的代币向挂接于Hub的其他侧链转移。

同时，其可以根据指定的TTC主网RPC地址及所部署的智能合约地址，可以用向TTC主网发送交易（ttc交易）的形式来读写合约，完成跨链交互。每个节点读取的外部信息也会封装成为特定消息（cosmos交易），保存在本状态机之上，使得这个侧链的所有节点能够对于具体的代币转移操作达成共识。

### 智能合约

智能合约是部署于TTC主网，用于记录次carnot的跨链交易行为及状态。被动接受carnot所发出的调用请求。


## 跨链转移步骤

如代币由TTC向Cosmos转移

1. TTC主网地址A，调用TTC主网的智能合约，传入的参数包括Cosmos上的提币目标地址B，同时将对应的TTC发送到此合约。（ERC20类似） 
2. carnot通过RPC接口访问TTC主网，获取当前代充值的交易及其状态，并将其转变为Cosmos上的一条交易进行广播。
3. carnot判断2中所述交易数量满足终确定条件的交易状态更改，并开始处理跨链转移。
4. carnot对于Cosmos侧链中的目标地址添加某种特定种类的代币。
5. carnot记录Cosmos的出块数量，当其达到最终确定条件时，更改TTC主链上合约中此交易的状态，交易完成。（可选步骤）

反向转移的操作步骤与之类似，但过程更为简单，carnot只需要分别发送ttc交易即可。

需要注意的是在转移过程中，每一步都需要对当前的状态机中全部数据进行核对（并非仅仅本条交易），且需要注意两方账户中的代币余额，以保证交易的进行。

当任何一步没有在指定的块数之内没有完成时，都应该有相应的回退操作，避免用户的资金损失。

在跨链操作是，需要设计不同操作权限的账号以控制风险，并对carnot及智能合约的操作进行双向的确认/核验。

跨网转移的合约需要在某种设定的权限制约下，可以进行代币的提取及冲入。（是否要考虑非唯一通道）

[English](./vbft-intro.md) / 中文

<h1 align="center">VBFT算法介绍</h1> 

<p align="center" class="version">Version 1.0.0 </p>

2018年3月30日，本体完成了第一批项目在GitHub上开源。作为新一代基础性公有链平台，本体也将推出全新基于可验证随机函数（VRF）的共识算法VBFT。以下我们将为您介绍本体最新的共识网络架构及其VBFT共识算法。VBFT算法也将在GitHub上开源。

VBFT是一个结合PoS、VRF(Verifiable Random Function)和BFT的全新共识算法，是OCE （Ontology Consensus Engine）的核心共识算法。VBFT可以支持共识群体的规模性扩展，通过VRF保障了共识群体生成的随机性和公平性，同时保证快速地达到状态终局性。

Ontology的核心网络主要由两部分组成:

* 共识网络

>
共识网络由所有共识节点组成，负责对Ontology网络中事务请求进行共识，生成区块，维护一致性账本，并将共识后区块分发到同步节点网络中 。

* 共识候选网络

>
候选网络中的节点不参与共识，但保持与共识网络的同步状态，实时将最新的共识区块更新到自己维护的账本中。候选网络也对共识网络进行监控，监控共识网络状态，对共识区块进行验证，并协助管理Ontology网络。

![VBFT Network](./images/vbft-network.jpeg)

共识网络的规模通过共识管理合约进行管理。共识网络中的每个节点都由其节点管理人锁定对应的Stake。

## Ontology共识网络的构建

Ontology共识网络是由Ontology共识管理合约构建的，共识管理合约永久性在Ontology网络中运行，且定期更新共识网络中节点列表，更新共识网络中VBFT算法的配置参数。

在VBFT算法参数中，一个重要的参数为共识网络节点的PoS表。VBFT运行过程中，所有节点根据当前的共识PoS表，随机选择每一轮参与共识的节点，由随机选择的节点完成对应轮的共识工作。

## VBFT算法概述

VBFT算法可以认为是传统BFT算法在可验证随机方向的一个改进。在VBFT算法中，首先基于VRF在共识网络中依次选择出一轮共识的备选区块提案节点集，区块验证节点集和区块确认节点集，然后由选出的节点集完成共识。

由于VRF引入的随机性，每轮区块的备选提案节点／验证节点／确认节点都不相同，而且难以预测，从而极大提高共识算法的抗攻击性。

VBFT算法可以概述如下：

VBFT的每轮共识中，

1. 根据VRF从共识网络中选择备选提案节点，各个备选节点将独立提出备选区块；
2. 根据VRF从共识网络中选择多个验证节点，每个验证节点将从网络中收集备选的区块，进行验证，然后对最高优先级的备选区块进行投票；
3. 根据VRF从共识网络中选择多个确认节点，对上述验证节点的投票结果进行统计验证，并确定出最终的共识结果。
4. 所有节点都将接收确认节点的共识结果，并在一轮共识确认后开启新的共识。

## VBFT的VRF 

当前VBFT算法中的每一轮区块的VRF值都是由前一轮共识区块所确定的。具体算法是从上一个区块中的提取易变信息，然后计算哈希，生成的1024位的 哈希值，将此哈希值作为下一个区块的VRF值 。

## VBFT的Peer Choice

VBFT算法以上一轮共识后的可验证随机值为索引，在PoS表中选择节点参与新一轮的共识，由于PoS表的生成兼顾了节点所属人的PoS信息和整个共识网络的整体治理策略，虽然VRF值本身可以假设为均匀分布的随机值，VBFT的随机节点选择依然是服从Ontology的共识网络管理策略。

由于一个区块生成的VRF值是可验证的，在不发生区块分叉的情况下所有节点对于同一高度区块的VRF也将是一致的。VBFT算法中基于VRF在PoS表中选择节点是顺序进行，因此每个VRF值都确定了一个备选提案节点的顺序，此随机选择的节点顺序也是共识一致的。

## VBFT的fork choice

Ontology作为一个公有链，运行在公有网络之中，必然面临着公有网络中的故障和恶意攻击。虽然VBFT共识算法通过随机方法选择节点参与共识，已经很大提高网络攻击的难度，但在发生网络隔离时依然面临着分叉的风险。

在前面一节介绍过每一个区块的VRF将可以确定一种节点排序顺序，在VBFT进行fork choice时，VBFT将节点的排序顺序定义为节点的优先级顺序，然后基于此优先级顺序可以计算每个fork分叉的优先级权重，每个节点根据各个分叉的优先级权重选择合适的分叉。

由于每个区块都是由VRF确定节点的优先级顺序，对于恶意产生的分叉，很难或者说不可能持续维持自己的高优先级，因此恶意产生的分叉将很快消亡。也因此，VBFT算法也提供了快速的状态终局性 。

## VBFT的auto configuration

为维护Ontology共识网络的网络质量，Ontology共识管理合约将定期自动更新共识网络中的节点列表。在发生网络风险时，共识管理合约也支持通过基于Stake的投票，强制更新共识网络中的节点列表。

一个新的节点在获得更多Stake，并且确认满足共识网络的节点性能需求后，将在下一次共识网络更新时被加入共识网络。

共识网络自动更新的时间是以区块为单位。每一次更新的共识网络在完成给定数目的区块共识后，下一个区块的备选提案节点必须构建一个共识管理合约执行事务，并将其作为区块中第一个事务打包到提案区块中；对应的共识验证节点和确认节点也将以此验证提案区块的有效性。

在包含共识管理合约事务的区块完成共识后，每个节点将自动执行共识管理合约，更新共识节点列表，至此完成共识节点列表的更新。


## 性能对比


| 共识机制 | 适用场景 | 性能效率 | 共识确认时间 | 共识确认时间举例 | 共识节点数量 | 防恶意节点数        | 资源消耗 | 安全可控 |
| ------------------ | -------------------- | ------------ | ---------------------------- | ------------------------------------ | ------------------------ | --------------------------------------- | ---------------- | ------------ |
| POW                | 公有链               | <20tps       | 高    | 比特币：60分钟  <br> 以太坊：1分钟    | -                        | 50%                                     | 高               | 低           | POS                | 公有链               | <20tps       | 高                           |                                      | -      | 50%    | 中               | 中           |
| DPOS               | 公有链               | >500tps      | 中                           | 比特股：10秒                         | 小于30                   | 有              | 低               | 高           |
| PBFT               | 联盟链/专有链        | >1000tps     | 低                           | FISCO-BCOS：1秒   <br> Fabric：1秒   | 小于30                   | 不超过1/3共识节点                       | 低               | 高           |
| VBFT               | 公有链/专有链        | >3000tps     | 中                           | ontology testnet：5-10秒             | 小于1000                 | 可配置拜占庭容错数目，不超过1/3共识节点 | 低               | 高           |
| Paxos/RAFT         | 联盟链/专有链        | >5000tps     | 低                           | FISCO-BCOS：1秒                      | 小于30                   | 无                                      | 低               | 高           |


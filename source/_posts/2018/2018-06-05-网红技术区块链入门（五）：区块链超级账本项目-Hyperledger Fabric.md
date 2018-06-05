---
title: 网红技术区块链入门（五）：区块链超级账本项目-Hyperledger Fabric
comments: true
categories:
  - Popular Technology - Blockchain
tags:
  - 区块链
abbrlink: 22032ed1
date: 2018-06-05 14:33:36
---
【引言】Hyperledger Fabric是The Linux Foundation® 主办的 Hyperledger® 项目之一；旨在作为开发模块化体系结构的区块链应用程序的基础，以便诸如共识和会员服务等组件可以即插即用。（建议参考版本：Hyperledger Fabric v0.6）
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-05-14.png" width="500">
<!-- more -->

# Fabric的架构

## 架构概述
&emsp;&emsp;Hyperledger Fabric 构建于一种模块化架构之上，该架构将交易处理分为 3 个阶段：分布式逻辑处理和协商（“链代码”）、交易订购，以及交易验证和提交。这种分离提供了一些优势：不同节点类型之间需要的信任和验证水平更低，网络可伸缩性和性能得到了优化。
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-05-15.jpg" width="75%">

## 三大组件
+ 区块链服务（Blockchain）：区块链服务提供一个分布式账本平台，也就是为交易链的状态机变更的历史记录提供服务。
+ 链码服务（Chaincode）：链码包含所有的处理逻辑，并对外提供接口，外部通过调用链码接口来改变世界观。链码强制读取或更改键值对或其他状态数据库信息的规则。链码功能针对分类帐当前状态数据库执行，并通过事务提议启动。链码执行导致一组可以提交到网络并应用于所有对等体上的分类帐的键值写入（写入集）。
+ 成员权限管理（Membership）：通过基于 PKI 的成员权限管理，平台可以对接入的节点和客户端的能力进行限制。

## 概念术语
+ Auditability（审计性）：在一定权限和许可下，可以对链上的交易进行审计和检查。
+ Block（区块）：代表一批得到确认的交易信息的整体，准备被共识加入到区块链中。
+ Blockchain（区块链）：由多个区块链接而成的链表结构，除了首个区块，每个区块都包括前继区块内容的 hash 值。
+ Certificate Authority（CA）：负责身份权限管理，又叫 Member Service 或 Identity Service。
+ Chaincode（链上代码或链码）：区块链上的应用代码，扩展自“智能合约”概念，支持golang、nodejs 等，运行在隔离的容器环境中。
+ Committer（提交节点）：1.0 架构中一种 peer 节点角色，负责对 orderer 排序后的交易进行检查，选择合法的交易执行并写入存储。
+ Confidentiality（保密）：只有交易相关方可以看到交易内容，其它人未经授权则无法看到。
+ Endorser（背书节点）：1.0 架构中一种 peer 节点角色，负责检验某个交易是否合法，是否愿意为之背书、签名。
+ Enrollment Certificate Authority（ECA，注册 CA）：负责成员身份相关证书管理的CA。
+ Ledger（账本）：包括区块链结构（带有所有的可验证交易信息，但只有最终成功的交易会改变世界观）和当前的世界观（world state）。Ledger 仅存在于 Peer 节点。
+ MSP（Member Service Provider，成员服务提供者）：成员服务的抽象访问接口，实现对不同成员服务的可拔插支持。
+ Non-validating Peer（非验证节点）：不参与账本维护，仅作为交易代理响应客户端的REST 请求，并对交易进行一些基本的有效性检查，之后转发给验证节点。
+ Orderer（排序节点）：1.0 架构中的共识服务角色，负责排序看到的交易，提供全局确认的顺序。
+ Permissioned Ledger（带权限的账本）：网络中所有节点必须是经过许可的，非许可过的节点则无法加入网络。
+ Privacy（隐私保护）：交易员可以隐藏交易的身份，其它成员在无特殊权限的情况下，只能对交易进行验证，而无法获知身份信息。
+ Transaction（交易）：执行账本上的某个函数调用。具体函数在 chaincode 中实现。
+ Transactor（交易者）：发起交易调用的客户端。
+ Transaction Certificate Authority（TCA，交易 CA）：负责维护交易相关证书管理的CA。
+ Validating Peer（验证节点）：维护账本的核心节点，参与一致性维护、对交易的验证和执行。
+ World State（世界观）：是一个键值数据库，chaincode 用它来存储交易相关的状态。

# 数据结构

## 交易的数据结构
```
message Transaction {
    enum Type {
        UNDEFINED = 0;
        // deploy a chaincode to the network and call Ìnit` function
        CHAINCODE_DEPLOY = 1;
        // call a chaincode Ìnvoke` function as a transaction
        CHAINCODE_INVOKE = 2;
        // call a chaincode `query` function
        CHAINCODE_QUERY = 3;
        // terminate a chaincode; not implemented yet
        CHAINCODE_TERMINATE = 4;
    }
    
    # 交易类型：目前包括 Deploy、Invoke、Query、Terminate 四种；
    Type type = 1;
    
    //store ChaincodeID as bytes so its encrypted value can be stored
    # 链码编号 chaincodeID：交易针对的链码；
    bytes chaincodeID = 2;
    
    # 负载内容的 hash 值：Deploy 或 Invoke 时候可以指定负载内容；
    bytes payload = 3;
    
    # 交易相关的 metadata 信息；
    bytes metadata = 4;
    
    # uuid：代表交易的唯一编号；
    string uuid = 5;
    
    # 链码编号 chaincodeID：交易针对的链码；
    google.protobuf.Timestamp timestamp = 6;
    
    # 交易的保密等级 ConfidentialityLevel；
    ConfidentialityLevel confidentialityLevel = 7;
    
    # 交易的保密协议版本
    string confidentialityProtocolVersion = 8;
    
    # 临时生成值 nonce：跟安全机制相关；
    bytes nonce = 9;
    bytes toValidators = 10;
    
    # 交易者的证书信息 cert；
    bytes cert = 11;
    
    # 签名信息 signature；
    bytes signature = 12;
}
```

## 区块的数据结构
```
# 注意具体的交易信息并不存放在区块中；交易和区块是分开的
message Block {
    # 版本号 version：协议的版本信息；
    uint32 version = 1;
    
    # 时间戳 timestamp：由区块提议者设定；
    google.protobuf.Timestamp timestamp = 2;
    
    # 交易信息的默克尔树的根 hash 值：由区块所包括的交易构成；
    repeated Transaction transactions = 3;
    
    # 世界观的默克尔树的根 hash 值：由交易发生后整个世界的状态值构成；
    bytes stateHash = 4;
    
    # 前一个区块的 hash 值：构成链所必须；
    bytes previousBlockHash = 5;
    
    # 共识相关的元数据：可选值；
    bytes consensusMetadata = 6;
    
    # 非 hash 数据：不参与 hash 过程，各个 peer 上的值可能不同，例如本地提交时间、交易处理的返回值等；
    NonHashData nonHashData = 7;
}
```

## 补充概念

### 世界观
&emsp;&emsp;世界观用于存放链码执行过程中涉及到的状态变量，是一个键值数据库。典型的元素为[chaincodeID, ckey]: value结构。为了方便计算变更后的 hash 值，一般采用默克尔树数据结构进行存储。

### 接口和操作
&emsp;&emsp;链码需要实现 Chaincode 接口，以被 VP 节点调用。链码目前支持的交易类型包括：部署（Deploy）、调用（Invoke）和查询（Query）。

### 容器
&emsp;&emsp;在实现上，链码需要运行在隔离的容器中，超级账本采用了Docker作为默认容器。关于Docker，会另起一个专题讲解。对容器的操作支持三种方法：build、start、stop，对应的接口为 VM。

### 成员权限管理
&emsp;&emsp;证书有三种，Enrollment，Transaction，以及确保安全通信的 TLS 证书。
+ 注册证书 ECert：颁发给提供了注册凭证的用户或节点，一般长期有效；
+ 交易证书 TCert：颁发给用户，控制每个交易的权限，一般针对某个交易，短期有效。
+ 通信证书 TLSCert：控制对网络的访问，并且防止窃听。

# Fabric reference

## 交易的流程
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-05-18.png" width="85%">

## 交易的生命周期
&emsp;&emsp;1) 应用程序将交易提案提交给背书对等节点。2) 背书策略规定需要多少个背书者和/或何种背书者组合来签署提案。背书者执行链代码，以便在网络对等节点中模拟该提案，并创建一个读/写集。3) 然后背书对等节点将经过签署的提案回复（背书）发回给应用程序。4) 应用程序将交易和签名提交给订购服务，后者 5) 创建一批或一组交易，并将它们传送给提交对等节点。6) 提交对等节点收到一批交易后，对于每个交易，它会 7) 确认满足背书策略，并检查读/写集以检测冲突的交易。如果两项检查都通过，则将该组交易提交到账本，并在状态数据库中反映出每个交易的状态更新。
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-05-19.png" width="85%">

## Model
&emsp;&emsp;As the graphic shows, the Fabric is a permissioned system (in which all the players know each other as opposed to the anonymous world of Bitcoin), with strong identity management. In a permissioned system, there are thus distinct roles for the users and the validators. Users invoke and deploy their transactions, which are then validated to create a new version of the blockchain (i.e., ledger). The key cryptographic element is an enhanced version of Practical Byzantine Fault Tolerance (PBFT) known as Sieve.
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-05-16.jpg" width="85%">

## What's next?
&emsp;&emsp;The current Fabric implementation is v0.6. Elli concluded her session with revealing the major upcoming features of the v1.0 release:
+ Separate the functions of validation into endorsers and consensus nodes
 + Every chaincode may have different endorsers
 + Endorsers have state, run tx, and validate tx for their chaincode
 + Consensus nodes order already-validated tx
+ Scales better, computation effort can be distributed
+ Permits confidential state on blockchain (seen only by endorsers)
<img style="clear: both;display: block;margin:auto;" src="/img/2018-06-05-17.png" width="85%">



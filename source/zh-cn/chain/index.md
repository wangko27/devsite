title: 跨链方案总体设计
---

## 一、行业背景

区块链是去中心化应用的操作系统。需要跨链技术把一个个运行的操作系统连接起来，才能大力发展生态，形成区块链的互联网。

## 二、设计目标

* 定义跨链标准，实现不同区块链间的通信。
* 搭建“卫星链”，实现不同区块链间的资产流转。

## 三、总体方案

### 3.1总体架构

![design](./image/3.1.png)

说明：

​	一条独立的卫星链，负责和所有链对接。以开放的方式实现链间通信。

​	基于NULS模块仓库实现的区块链（生态内的区块链），可以通过模块选择的方式添加跨链模块，使其在底层上可以和卫星链互通。

​	以太坊和比特币等不受NULS影响的公有链，需要通过特殊的机制实现协议的转换，将公有链的协议和NULS跨链协议做适配，达到统一协议通讯的目的。

​	所有区块链都只和卫星链通信，交易的验证由卫星链负责，各平行链信任卫星链的验证结果。

* 链间连接方式

  区块链上的每个节点都运行跨链模块，每个节点都连接卫星链上的部分节点。通过随机算法决定连接哪些节点，尽可能地确保节点连接的分散，保证网络的安全性。

* 跨链交易是如何实现的

  假设A链的账户a1要转移其持有的资产a到b链的账户b1中，则处理流程如下：

  * 先在A链上发起跨链交易，由A链先行确认；
  * 当达到一定数量的区块确认后，交易被跨链模块推送到卫星链的节点中；
  * 卫星链接收到交易后进行确认，确认的方式分为两步：
    * 1、通过询问A链上的节点，确认该交易是否已被确认，并且以跨链协议发送到卫星链中的交易是正确、真实的；
    * 2、在卫星链中以拜占庭容错算法对交易进行确认，若不能获得大部分节点的认同，则该交易视为无效。
  * 交易打包到卫星链的区块中；
  * 节点将该跨链交易推送到B链中；
  * B链节点通过连接的所有卫星链的节点，对该交易进行确认，若确认不通过则丢弃该交易；
  * 若通过确认，则创建对应的资产到目标地址中；
  * 在B链共识中确认该交易。
  * 完成，该资产可以在B链使用。

* 多算法适配

  卫星链支持市面上大部分数学算法，包括摘要算法、对称加密、非对称加密等，可以通过算法库提供的统一的接口进行使用。

* 社区化治理

  卫星链会内置社区治理机制，包括系统运行参数修改、协议升级、恶意链处理、社区资金使用等功能。



## 四、 卫星链设计

###  4.1 卫星链架构

 ![layer](./image/4.1.jpg)

* 卫星链使用POC共识机制，结合拜占庭容错机制实现跨链交易的确认和打包，做到去中心化与性能、安全性的兼顾。
* 卫星链上的协议是统一定义的NULS跨链协议，每个节点都会连接多个区块链的多个节点。
* 卫星链提供链管理机制，用来管理所有在卫星链上登记的对等区块链。登记的内容包括链信息、资产信息、跨链抵押金等。
* 当一条区块链上收到其他链的资产时，需要在本链产生对应的资产。不同区块链上的token，都以资产的方式在其他链上存储。
* 一条区块链中转入其他链资产的明细会在卫星链中存储。该资产转出这条区块链时会进行验证，不允许非法的资产从该区块链中产生。对有恶意的区块链，会通过社区机制进行处理，如：暂停跨链、中止跨链、没收保证金等。
* 卫星链将提供api使用手册，任何开发者都可以根据手册开发自己的钱包、浏览器、轻钱包等工具。
* 为了降低卫星链的业务复杂度，将不在卫星链中运行智能合约。
* 卫星链中提供协议供应用扩展，可以使用该协议进行DApp的开发和跨链协议的优化。

### 4.2 卫星链的运行

![](./image/4.2.png)

* 卫星链以模块化的方式架构。
* 每个模块都是一个可以独立运行的微服务。
* 微服务之间直接通过http协议通信。
* 模块不限制开发语言。
* 提供微内核模块负责服务管理、配置管理和数据共享功能。
* 卫星链的模块在一定程度上将可以与NULS主网共用，所以卫星链的模块也会和NULS的模块一样加入到NULS模块仓库中，供“链工厂”等应用直接使用。
* 每个模块在使用的同时都支持扩展。即如果模块仓库中的模块只能满足部分业务需求时，可以对该模块进行扩展以避免重新开发。

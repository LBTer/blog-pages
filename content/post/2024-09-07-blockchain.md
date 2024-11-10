---
title:       "区块链学习"
subtitle:    ""
description: ""
date:        2024-09-07
author: "LBT"
image:       ""
tags:        ["blockchain"]
categories:  ["Tech" ]
---

[北京大学肖臻老师《区块链技术与应用》公开课](https://www.bilibili.com/video/BV1Vt411X7JF?p=1)
{{<bilibili BV1Vt411X7JF>}}

**笔记**
## 比特币
### 原理
加密货币crypto -currency
密码学的的哈希函数cryptographic hash function
    性质1: collision resistance（碰撞抗性）
        求digest
        性质无法证明，只是有些函数经实践说明很难碰撞
        MD5可以人为制造哈希碰撞
    性质2: hiding（无法逆推）
        前提是输入空间足够大，且均匀，就不能被遍历算出
        digital commitment/digital equivalent of a sealed envelope
    性质3: puzzle frienddly（hash值不可预测）

挖矿原理：
H（block, header）<= target
工作量证明
difficult to solve， but easy to verify

比特币用的哈希函数：SHA-256（secure hash algorithm）
比特币的账户：公私钥对（private key + public key）
asymmetric encryption algorithm


### 数据结构
- hash pointers（指向结构体的指针，存**结构体+hash值**）
block chain is a linked list using hash pointers

block0<-block1<-block2<-...... \
其中block2的hash指针的hash值是block1整个块的hash值（块+block1的hash指针值）

实现temper-evident log，

- merkle tree
```
            【H() H()】--H() root hash
            /       \
    【H() H()】     【H() H()】
    /       \       /       \
   tx       tx     tx       tx
```
只要记住root hash ，就知道是否做了修改

区块分为block header（只有根hash）和block body（交易列表），作用是提供merkle proof（tx交易节点到根的路径为merkle proof）
全节点，轻节点（只有block header）如何证明交易？
!()[]
proof of membership/proof of inclusion（merkle tree中证明了存在某个交易，复杂度Ologn）
proof of non-membership（整棵树H()传递，复杂度On）（如果是有序的hash值，可以Ologn）



### 挖矿原理
tx的codebase+header noise

memory less(无记忆性，挖到下一个区块的预期时间不变)

### 攻击方法
- 分叉攻击（重复消费），矿场的出现让51%攻击更有可能了（他只需要动员）
- Boycott，封锁攻击，一出现某个账户的交易，立刻分叉不承认
- selfish mining（藏着很多block不发布）
- - 风险是可能挖不过别人，好处是减少了竞争

一般交易都是信任


### bitcoin network
所有节点都是平等的
simple、robust、but not efficient
下一个选取的节点可能比较远，所以世界各地的交易时间都差不多

### 挖矿难度
difficulty=difficulty_1_target/target

target=target*actual time/expected time
一开始挖矿像是买彩票，后面出现了矿池

### 比特币的保证
前提：系统中大多数节点都是合法的
密码学
合作挖矿

### 比特币芯片
ASIC，定制的芯片，只能用于挖比特币；GPU也不能发挥很好（主要是以太坊在用）

## 以太坊
###
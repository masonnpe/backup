---
title: BASE理论
categories: 分布式
tags: [BASE]
---

BASE理论是对CAP理论中一致性和可用性权衡的结果，如果无法做到强一致性，那就要采取合适的方法使系统达到最终一致性。传统的数据库系统要求强一致性(ACID)，BASE理论强调通过牺牲强一致性来达到可用性。在实际业务场景中，要结合业务对一致性的要求，将ACID和BASE结合起来使用。

## 基本可用(BasicallyAvailable)

分布式系统在出现故障的时候，保证核心功能可用，允许损失部分可用性。

## 软状态(SoftState)

允许系统中的数据存在中间状态，即系统不同节点的数据副本之间进行同步的过程存在时间延迟

## 最终一致性(EventuallyConsistent)

系统中所有的数据副本，在经过一段时间的同步后，最终能达到一致的状态。

<!--more-->
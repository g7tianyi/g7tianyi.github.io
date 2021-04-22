---
layout: post
title: 存储系统相关论文
categories: tech
tags:
  - paper
  - cloud
  - virtualization
comments: true
featured: false
published: true
---

> **持续添加中..**

- [The Google File System](https://pdos.csail.mit.edu/6.824/papers/gfs.pdf)，GFS最重要的设计理念是任务服务器故障是正常现象，然后通过软件的方式自动容错，在保证系统可靠性和可用性的同时，大大降低系统成本，现在再看GFS的很多设计，很多概念似乎已经烂大街，而这也正是GFS的伟大之处。
- [Finding a needle in Haystack: Facebook's Photo Storage](https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf)，Facebook的相片存储系统，一个重要的理念是多个逻辑文件共享一个物理文件
- [Dynamo: Amazon's Highly Available Key-value Store](http://courses.cse.tamu.edu/caverlee/csce438/readings/dynamo-paper.pdf)，Dynamo将众多分布式技术融合到一个系统内，学习Dynamo的设计对理解分布式系统的理论很有帮助。
- [Bigtable: A Distributed Storage System for Structured Data](https://github.com/g7tianyi/cs-papers/blob/master/distributed-system/Bigtable-%20A%20Distributed%20Storage%20System%20for%20Structured%20Data.pdf)，分布式表格系统的始祖

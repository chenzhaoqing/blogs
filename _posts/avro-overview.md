title: Apache Avro™ 1.8.2 文档 (1)
categories: translation
comments: true
tags:
  - avro
  - serialization
  - docs
  - translation
date: 2018-08-01 20:00:00
updated: 2018-08-08 17:09:00
---
Avro 官方文档翻译（http://avro.apache.org/docs/current/index.html）

## 引言

Apache Avro™ 是一个数据序列化系统。
<!--more-->
Avro提供：

> * 丰富的数据结构
> * 一个简洁，快速，二进制数据格式
> * 一个用于持久化数据容器（container）文件
> * RPC
> * 易与动态语言集成。读写数据文件和使用或实现RPC协议时，不需要“代码生成”过程。实现代码生成作为一个可选的优化，只对于静态类型语言是有意义的。



## 模式（Schemas）

Avro依赖于模式。当读取Avro数据时，写入该数据时使用的schema总是已知的。这允许在写入数据的每个值时没有开销，使得序列化的速度快，数据小。数据和模式在一起是自完备的，所以这更利于动态语言和脚本语言的使用。

当Avro数据存储于文件时，他的schema与数据存储在一起，所以该文件以后可以被其他任何程序处理。如果读取数据的程序预期的是不同的schema，这也是可以解决的，因为这两个schema都是已知的。

当Avro在RPC中使用时，客户端和服务端会在连接握手时交换schema.(这个过程可以优化，使得对于大多数的调用，schema并不需要真的传输。) 由于客户端和服务端都有双方完整的schema，所以同名字段， 缺失字段，额外字段等字段对应问题，都可以容易的解决。

Avro schemas以JSON来定义。这使得已有JSON库的各种语言很容易实现。

## 与其他系统的对比

Avro schemas提供的功能与Thrift, Protocol Buffers等系统类似。Avro与它们根本的不同点在于：

> * *动态类型*：Avro不要求代码生成。数据总是与一个schema相伴，允许在没有代码生成，没有静态数据类型等情况下全面的处理数据。这更利于构建一个通用的数据处理系统和语言。
> * *无标记的数据*：由于读数据时schema是已知的，大量地减少了需要与数据编码在一起的类型信息，所以序列化后的数据更小。
> * *无手动分配的字段ID*: 当schema改变时，对于数据处理过程新的和旧的schema都是已知的，因此差异带来的问题可以通过字段名字象征性的解决。

*Apache Avro, Avro, Apache, and the Avro and Apache logos are trademarks of The Apache Software Foundation.*
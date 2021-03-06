---
title: "DHT网络中获取MetaInfo"
date: 2019-11-02 10:05:33 +0800
categories: [Network]
tags: [DHT]
---
## DHT 网络加入

### 协议

协议原文链接  
[http://www.bittorrent.org/beps/bep_0000.html](http://www.bittorrent.org/beps/bep_0000.html)  
数据交换通过**Bencode**进行包装发送,这里只讲实现方式.

主要几个请求

1. ping
2. find_node
3. get_peers
4. announce_peer

## ping

用于判断DHT节点是否存活

## find_node

用于加入DHT网络节点

## get_peers

查询Peers

## announce_peer

**被动**获取MetaInfo的主要方法  
为什么说是被动呢,因为这种方式获取MetaInfo需要别人主动请求你,你才可以知道.

```
arguments:  {"id" : "<querying nodes id>",
  "implied_port": <0 or 1>,
  "info_hash" : "<20-byte infohash of target torrent>",
  "port" : <port number>,
  "token" : "<opaque token>"}
```

参数说明

* id 对方nodeID
* implied_port 值 只有 0和1
* info_hash hash字节
* port 端口
* token token 之前对方向你请求get_peers方法时你返回给对方的token(用来校验)

implied_port 参数

> There is an optional argument called implied_port which value is either 0 or 1. If it is present and non-zero, the port argument should be ignored and the source port of the UDP packet should be used as the peer's port instead. This is useful for peers behind a NAT that may not know their external port, and supporting uTP, they accept incoming connections on the same port as the DHT port

解释  
如果存在且非零，则应忽略port参数，而应将UDP数据包的源端口用作对等方的端口。 这对于NAT后的对等点（可能不知道其外部端口）并且支持uTP很有用，它们支持与DHT端口在同一端口上的传入连接

例子

UDP 请求 IP 10.1.1.1 端口 6881

```json
{
    "id": "12345678901234567890",
    "implied_port":1,
    "info_hash":"12345678901234567890",
    "port":6582,
    "token":"abcdefg"
}
```

Peer 之间的μTP通讯地址 IP 10.1.1.1 端口 6881

```json
{
    "id": "12345678901234567890",
    "implied_port":0,
    "info_hash":"12345678901234567890",
    "port":6582,
    "token":"abcdefg"
}
```

或

```json
{
    "id": "12345678901234567890",
    "info_hash":"12345678901234567890",
    "port":6582,
    "token":"abcdefg"
}
```

Peer 之间的μTP通讯地址 IP 10.1.1.1 端口 6582

## 如何获得更多的announce_peer请求

主要通过**find_node**方法
初始化时请求节点(创世节点)
router.bittorrent.com:6881
router.utorrent.com:6881
dht.transmissionbt.com:6881
请求完后根据返回的nodes进行解析,分别对nodes里的节点发送**find_node**请求,这样你就加入到别人的route table 里了,就容易收到**announce_peer**,依次类推用返回回来的nodes再找nodes.这样收到的消息会越来越多

## PeerToPeer

Peer之间通讯协议是基于**TCP协议的μTP**

### 握手

| 字段 | 描述 |
| --- | --- |
| protocolLength | 协议**BitTorrent protocol**的长度.值: **19** (1 byte)|
| protocolName | 固定协议名称:**BitTorrent protocol** (19 byte)|
| extension | 扩展握手协议 (8 byte)|
| infohash | infohash (20 byte)|
| peer_id | peerId (20 byte) [http://www.bittorrent.org/beps/bep_0020.html](http://www.bittorrent.org/beps/bep_0020.html) 例: **-WW0001-123456789012**|

对端返回的也是相同格式的数据需要校验协议的**协议名称**和**infohash**


TCP 在数据传输时是以流的方式传输你如何确定流是否完整和结束呢?
在传输时用4个字节来表示发送的数据的长度.
例:
需要传输abcd字符. 实际传输的数据是4abcd 其中的4就代表了后面数据的长度,所以每次在读取数据时 先要读取数据长度 然后再根据数据长度 读取实际数据 4 这个长度值 需要占据4个byte.


### 扩展握手


扩展握手是发送一个**Bencode**内容:
```json
{
    "m": {
        "ut_metadata":1
    }
}
```
接受到的消息格式也是**Bencode**内容 [http://www.bittorrent.org/beps/bep_0010.html](http://www.bittorrent.org/beps/bep_0010.html)
```json
{
    "m": {

    },
    "p":"",
    "v":"",
    "yourip":"",
    "ipv6":"",
    "ipv4":"",
    "reqq":"",
}
```

我们需要获取metainfo 内容
就需要检查它是否有以下内容
[http://www.bittorrent.org/beps/bep_0009.html](http://www.bittorrent.org/beps/bep_0009.html)
```json
{
    "m":{
        "ut_metadata":3
    },
    "metadata_size":31235
}
```
**ut_metadata**代表这个扩展ID是3 需要记录方便后面获取
**metadata_size**代表总metadata_size 大小

> The metadata is handled in blocks of 16KiB (16384 Bytes). The metadata blocks are indexed starting at 0. All blocks are 16KiB except the last block which may be smaller.

metadata是分块进行传输的 每一块的大小是16384 所以需要用总的metadata_size值去除于每一块16384 余数进一算出一共多少block
block值从0开始
metadata_size = 31235
blockNumber = 2 {0,1}
根据协议进行数据块请求
```json
{"msg_type": 0, "piece": 0}
{"msg_type": 0, "piece": 1}
```
回复内容
```json
{"msg_type": 1, "piece": 0, "total_size": 3425}
```
```
d8:msg_typei1e5:piecei0e10:total_sizei34256eexxxxxxxx...
```
后面的xxxxxxxx...就是每一块block信息,所有块顺序连接起来后的内容也是**Bencode**编码
只有正确的获取到到每一个block信息最后才能被解析
最后解析的内容属于种子信息里的info字段内容
[http://www.bittorrent.org/beps/bep_0003.html](http://www.bittorrent.org/beps/bep_0003.html)
格式:
**单文件**
```json
{
    "name":"",
    "piece length": 12,
    "pieces":"",
    "length":1234
}
```
**多文件**
```json
{
    "name":"",
    "piece length": 12,
    "pieces":"",
    "files":[
        {
            "length":1234,
            "path": [
                "a dir",
                "b dir",
                "file"
            ]
        }
    ]
}
```
> 代码实现示例 [GitHub](https://github/zpqsunny/dht)

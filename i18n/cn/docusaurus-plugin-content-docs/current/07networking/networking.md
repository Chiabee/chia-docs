---
sidebar_position: 1
---

# 7.1 联网

> Networking

奇亚协议是异步和点对点的。它运行在端口 8444（或农民和时间领主的其他端口）上的网络套接字之上。所有节点既充当客户端又充当服务器，并且可以与其他对等点保持长期连接。

奇亚协议中的每条消息都由字节组成，使用可流式传输格式，并作为网络套接字消息发送。每条消息由三部分组成。
1. 第一，跨越 1 个字节的字段，代表正在传输的消息类型以及如何对数据进行解码。
2. 其次，一个可选的 2 字节 ID，每个连接使用它来跟踪请求和响应。
3. 第三，数据，它是协议消息之一的可流式传输编码表示。

```python
class Message(Streamable):
    # one of ProtocolMessageTypes
    type: uint8
    # message id
    id: Optional[uint16]
    # Message data for that type
    data: bytes

```

奇亚协议消息的最大 `(4 + 2^32 - 1) = 4294967299` 字节长度约为 4GB。

<details>
<summary>原文参考</summary>

The Chia protocol is asynchronous and peer-to-peer. It runs on top of WebSockets on port 8444 (or other ports for farmers and timelords). All nodes act as both clients and servers, and can maintain long-term connections with other peers.

Every message in the Chia protocol is composed of bytes, using the Streamable format, and sent as a WebSocket message. Each message is composed of three parts.
1. A field spanning 1 byte, representing the type of message being transmitted, and how to decode the data.
2. Second, an optional 2-byte ID, which is used per connection to keep track of requests and responses.
3. Third, the data, which is a Streamable encoded representation of one of the protocol messages.


```python
class Message(Streamable):
    # one of ProtocolMessageTypes
    type: uint8
    # message id
    id: Optional[uint16]
    # Message data for that type
    data: bytes

```

Chia protocol messages have a max length of `(4 + 2^32 - 1) = 4294967299` bytes, or around 4GB.

</details>

## 握手

奇亚协议中的所有对等节点（无论是农民、完整节点、时间领主等）都充当服务器和客户端（对等节点）。一旦在两个对等方之间发起连接，双方都会发送 Handshake 消息和 HandshakeAck 消息以完成握手。对等方的 节点 ID 是其 [x.509](https://en.wikipedia.org/wiki/X.509) [DER](https://wiki.openssl.org/index.php/DER) 证书的 SHA-256 哈希值 。

```python
class Handshake(Streamable):
    network_id: str                         # Network id, usually the genesis challenge of the blockchain
    protocol_version: str                   # Protocol version to determine which messages the peer supports
    software_version: str                   # Version of the software, to debug and determine feature support
    server_port: uint16                     # Which port the server is listening on
    node_type: uint8                        # NodeType (full node, wallet, farmer, etc.)
    capabilities: List[Tuple[uint16, str]]  # Key value dict to signal support for additional capabilities/features

```

握手完成后，双方都可以发送奇亚协议消息，并随时通过关闭网络套接字断开连接。

<details>
<summary>原文参考</summary>

- ## Handshake

All peers in the Chia protocol (whether they are farmers, full nodes, timelords, etc.) act as both servers and clients (peers). As soon as a connection is initiated between two peers, both peers send a Handshake message, and a HandshakeAck message to complete the handshake. A peer's node_id is the SHA-256 hash of their [x.509](https://en.wikipedia.org/wiki/X.509) [DER](https://wiki.openssl.org/index.php/DER) certificate. 

```python
class Handshake(Streamable):
    network_id: str                         # Network id, usually the genesis challenge of the blockchain
    protocol_version: str                   # Protocol version to determine which messages the peer supports
    software_version: str                   # Version of the software, to debug and determine feature support
    server_port: uint16                     # Which port the server is listening on
    node_type: uint8                        # NodeType (full node, wallet, farmer, etc.)
    capabilities: List[Tuple[uint16, str]]  # Key value dict to signal support for additional capabilities/features

```

After the handshake is completed, both peers can send Chia protocol messages, and disconnect at any time by closing the WebSocket.

</details>

## 心跳

心跳消息由网络套接字库定期发送。因此，无响应的对等点将断开连接。

如果某个节点在一段时间内没有收到来自对等方的任何消息，即使正在收到心跳，该节点也会断开连接并将该对等方从活动对等方列表中删除。

<details>
<summary>原文参考</summary>

- ## Heartbeat

Heartbeat messages are sent periodically by the WebSocket libraries. Peers that are unresponsive will therefore be disconnected.

If a node does not receive any message from a peer for a certain period of time, even if heartbeats are being received, then the node will disconnect and remove the peer from the active peer list.

</details>

## 介绍人

对于加入去中心化网络的新节点，他们必须选择所有在线节点的子集来连接。

为促进这一过程，奇亚和其他用户将临时运行一些介绍人节点，它们将抓取网络并支持一个协议消息：get\_peers\_introducer。 然后，介绍人将返回调用节点将尝试连接的已知最近对等节点的随机子集。

DNS 介绍器也有不同的名称，它们返回随机可靠的对等点进行连接。

例如：`dig dns-introducer.chia.net`。

未来将招募更多DNS介绍人；检查[奇亚区块链 GitHub 存储库](https://github.com/Chia-Network/chia-blockchain "chia-blockchain on GitHub") 以获取更新。仅在应用程序初始启动时或在对等数据库没有好的对等时才联系介绍人。

<details>
<summary>原文参考</summary>

- ## Introducer

For a new peer to join the decentralized network, they must choose a subset of all online nodes to connect to.

To facilitate this process, a number of introducer nodes will temporarily be run by Chia and other users, which will crawl the network and support one protocol message: get_peers_introducer. The introducer will then return a random subset of known recent peers that the calling node will attempt to connect to.

DNS introducers are also available at different names, which return random reliable peers to connect to.

For example: `dig dns-introducer.chia.net`.

More DNS introducers will be recruited in the future; check the [chia-blockchain GitHub repository](https://github.com/Chia-Network/chia-blockchain "chia-blockchain on GitHub") for updates. The introducer is only contacted at initial launch of the application, or if the peer database has no good peers.

</details>

## RPC

除了下一页描述的奇亚协议之外，还有一个本地 RPC 协议允许通过 HTTP 对节点或钱包进行简单的控制。RPC 协议的所有请求和响应都采用 JSON 格式，以简化接口。这允许执行诸如获取链的提示、获取特定块、添加连接、停止节点等操作。完整节点 UI 使用 RPC 连接到完整节点。

RPC API 以 WebSocket 和 HTTP 格式提供。

<details>
<summary>原文参考</summary>

- ## RPC

Aside from the Chia protocols described in the next page, there is also a local RPC protocol to allow simple control over a node or wallet through HTTP. All requests and responses for the RPC protocol are in JSON, to simplify the interface. This allows doing things like getting the tips of the chain, getting a specific block, adding connections, stopping the node, etc. The full node UI connects to the full node using the RPC. 

The RPC APIs are provided in both WebSocket and HTTP format.

</details>

## 传入和传出连接

Chia 软件有多个规则和检查，以确保一个节点连接到几个好的对等点。

例如，传出连接（我们的节点与外部节点建立的连接）排名高于传入连接。这是因为我们无法验证传入的对等点是否是攻击的一部分。

每个节点将尝试连接到 8 个（依赖于实现的）外部对等点。只要一个节点连接到至少一个快速且无恶意的对等点，该节点就应该能够跟上并保持与最重的区块链的共识。

<details>
<summary>原文参考</summary>

- ## Incoming and Outgoing Connections

The Chia software has multiple rules and checks to make sure a node is connected to several good peers.

For example, outgoing connections (connections which our node makes to external nodes) are ranked higher than incoming ones. This is because we cannot verify whether incoming peers are part of an attack or not.

Each node will try to connect to 8 (implementation-dependent) external peers. As long as a node is connected to at least one fast and non-malicious peer, the node should be able to keep up with, and maintain, consensus with the heaviest blockchain.

</details>

## 禁令

如果对等方似乎行为不诚实，则可以将其断开连接并暂时禁止重新连接。禁止的原因包括（但不限于）超出对每种类型的协议消息提供的限制，发送无效信息，以及使节点在处理消息时抛出异常。

禁令的持续时间取决于问题的严重程度。应注意不要意外禁止诚实的同行。不同的实现也可能有更大或不同的速率限制。

<details>
<summary>原文参考</summary>

- ## Bans

If a peer appears to be acting dishonestly, it can be disconnected and temporarily banned from reconnecting. Reasons for banning include (but are not limited to) exceeding the limits provided for each type of protocol message, sending invalid information, and making the node throw an exception when handling a message.

The duration of the ban depends on the severity of the issue. Care should be taken to not ban honest peers by accident. Different implementations might have larger or different rate limits as well.

</details>

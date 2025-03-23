# 1.WebSocket是什么？

websocket 协议 HTML5 带来的新协议, WebSocket 是 **基于TCP** 的一种新的应用层网络协议。它提供了一个全双工的通道，允许服务器和客户端之间实时双向通信。因此，在 WebSocket 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输，客户端和服务器之间的数据交换变得更加简单。

HTTP 的生命周期通过 Request 来界定，也就是 Request 一个 Response，那么在 Http1.0 协议中，这次 Http 请求就结束了。在 Http1.1 中进行了改进，是的有一个 connection: Keep-alive，也就是说，在一个 Http 连接中，可以发送多个 Request，接收多个 Response。 但是必须记住，在 Http 中一个 Request 只能对应有一个 Response，而且这个 Response 是被动的，不能主动发起。

WebSocket 是基于 Http 协议的，或者说借用了 Http 协议来完成一部分握手，在握手阶段 与 Http 是相同的。我们来看一个 websocket 握手协议的实现，基本是 2 个属性，`upgrade， connection`。

```javascript
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== Sec-WebSocket-Protocol: chat, superchat Sec-WebSocket-Version: 13
Origin: http://example.com

```

```javascript
// 创建ws连接
const ws = new WebSocket('ws://localhost:8080/test');
ws.onopen = function() {
    console.log('WebSocket 连接已经建立。');
    ws.send('Hello, server!');
};
ws.onmessage = function(event) {
    console.log('收到服务器消息：', event.data);
};
ws.onerror = function(event) {
    console.error('WebSocket 连接出现错误：', event);
};
ws.onclose = function() {
    console.log('WebSocket 连接已经关闭。');
}
```



# 2.与 HTTP 协议的区别

WebSocket 与 HTTP 协议相比，WebSocket 具有以下优点：

1. 更高的实时性能：WebSocket 允许服务器和客户端之间实时双向通信，从而提高了实时通信场景中的性能。

2. 更少的网络开销：HTTP 请求和响应之间需要额外的数据传输，而 WebSocket 通过在同一个连接上双向通信，减少了网络开销。

3. 更灵活的通信方式：HTTP 请求和响应通常是一一对应的，而 WebSocket 允许服务器和客户端之间以多种方式进行通信，例如消息 Push、事件推送等。

4. 更简洁的 API：WebSocket 提供了简洁的 API，使得客户端开发人员可以更轻松地进行实时通信。

5. 数据格式：WebSocket 支持文本和`二进制数据`的传输，而 HTTP 主要是传输文本数据。

6. 数据传输方式：WebSocket 实现了全双工通信，客户端和服务器可以同时发送和接收数据，而 HTTP 是单向的，客户端发起请求，服务器响应数据。

7. 协议标识：WebSocket 使用 ws:// 或 wss:// 前缀标识，而 HTTP 使用 http:// 或 https://

当然肯定有缺点的：

1. 不支持无连接: WebSocket 是一种持久化的协议，这意味着连接不会在一次请求之后立即断开。这是有利的，因为它消除了建立连接的开销，但是也可能导致一些资源泄漏的问题。

2. 不支持广泛: WebSocket 是 HTML5 中的一种标准协议，虽然现代浏览器都支持，但是一些旧的浏览器可能不支持 WebSocket。

3. 需要特殊的服务器支持: WebSocket 需要服务端支持，只有特定的服务器才能够实现 WebSocket 协议。这可能会增加系统的复杂性和部署的难度。

4. 数据流不兼容: WebSocket 的数据流格式与 HTTP 不同，这意味着在不同的网络环境下，WebSocket 的表现可能会有所不同。

# 短轮询、长轮询和 WebSocket 间的区别？

## 短轮询

短轮询的基本思路:
浏览器每隔一段时间向浏览器发送 http 请求，服务器端在收到请求后，不论是否有数据更新，都直接进行 响应。
这种方式实现的即时通信，本质上还是浏览器发送请求，服务器接受请求的一个过程，通过让客户端不断的进行请求，使得客户端能够模拟实时地收到服务器端的数据的变化。

### 优缺点👇

优点是比较简单，易于理解。
缺点是这种方式由于需要不断的建立 http 连接，严重浪费了服务器端和客户端的资源。当用户增加时，服务器端的压力就会变大，这是很不合理的。 

## 长轮询

长轮询的基本思路:
首先由客户端向服务器发起请求，当服务器收到客户端发来的请求后，服务器端不会直接进行响应，而是先将 这个请求挂起，然后判断服务器端数据是否有更新。
如果有更新，则进行响应，如果一直没有数据，则到达一定的时间限制才返回。客户端 JavaScript 响应处理函数会在处理完服务器返回的信息后，再次发出请求，重新建立连接。

### 优缺点👇

长轮询和短轮询比起来，它的优点是「明显减少了很多不必要的 http 请求次数」，相比之下节约了资源。
长轮询的缺点在于，连接挂起也会导致资源的浪费。


# 3. 工作原理(重要)

## 3.1  握手阶段

WebSocket在建立连接时需要进行握手阶段。握手阶段包括以下几个步骤：

客户端向服务端发送请求，请求建立WebSocket连接。

请求中包含一个`Connection: Upgrade`参数，用于标识希望连接升级

请求中包含一个`Upgrade: WebSocke`t参数，用于标识该请求是WebSocket连接。

请求中包含一个`Sec-WebSocket-Key`参数，用于生成WebSocket的随机密钥。

服务端接收到请求后，生成一个随机密钥，并使用随机密钥生成一个新的`Sec-WebSocket-Accept`参数。

客户端接收到服务端发送的新的`Sec-WebSocket-Accept`参数后，使用原来的随机密钥和新的`Sec-WebSocket-Accept`参数共同生成一个新的`Sec-WebSocket-Key`参数，用于加密数据传输。

客户端将新的`Sec-WebSocket-Key`参数发送给服务端，服务端接收到后，使用该参数加密数据传输。

## 3.2  数据传输阶段

建立连接后，客户端和服务端就可以通过WebSocket进行实时双向通信。数据传输阶段包括以下几个步骤：

客户端向服务端发送数据，服务端收到数据后将其转发给其他客户端。
服务端向客户端发送数据，客户端收到数据后进行处理。
双方如何进行相互传输数据的 具体的数据格式是怎么样的呢？WebSocket 的每条消息可能会被切分成多个数据帧（最小单位）。发送端会将消息切割成多个帧发送给接收端，接收端接收消息帧，并将关联的帧重新组装成完整的消息。

发送方 -> 接收方：ping。

接收方 -> 发送方：pong。

ping 、pong 的操作，对应的是 WebSocket 的两个控制帧

## 3.2  关闭阶段

当不再需要WebSocket连接时，需要进行关闭阶段。关闭阶段包括以下几个步骤：

客户端向服务端发送关闭请求，请求中包含一个WebSocket的随机密钥。
服务端接收到关闭请求后，向客户端发送关闭响应，关闭响应中包含服务端生成的随机密钥。
客户端收到关闭响应后，关闭WebSocket连接。

总的来说，WebSocket通过握手阶段、数据传输阶段和关闭阶段实现了服务器和客户端之间的实时双向通信。

# WebSocket 数据帧结构和控制帧结构

## 1.  数据帧结构
WebSocket 数据帧主要包括两个部分：帧头和有效载荷。以下是 WebSocket 数据帧结构的简要介绍：

帧头：帧头包括四个部分：fin、rsv1、rsv2、rsv3、opcode、masked 和 payload_length。

其中，fin 表示数据帧的结束标志，rsv1、rsv2、rsv3 表示保留字段，opcode 表示数据帧的类型，masked 表示是否进行掩码处理，payload_length 表示有效载荷的长度。

有效载荷：有效载荷是数据帧中实际的数据部分，它由客户端和服务端进行数据传输。

## 2.  控制帧结构

除了数据帧之外，WebSocket 协议还包括一些控制帧，主要包括 Ping、Pong 和 Close 帧。以下是 WebSocket 控制帧结构的简要介绍：

Ping 帧：Ping 帧用于测试客户端和服务端之间的连接状态，客户端向服务端发送 Ping 帧，服务端收到后需要向客户端发送 Pong 帧进行响应。

Pong 帧：Pong 帧用于响应客户端的 Ping 帧，它用于测试客户端和服务端之间的连接状态。

Close 帧：Close 帧用于关闭客户端和服务端之间的连接，它包括四个部分：fin、rsv1、rsv2、rsv3、opcode、masked 和 payload_length。其中，opcode 的值为 8，表示 Close 帧。

# WebSocket 的安全性和跨域问题如何处理？

WebSocket 支持通过` wss://` 前缀建立加密的安全连接，使用 TLS/SSL 加密通信，确保数据的安全性。在使用加密连接时，服务器需要配置相应的证书。
对于跨域问题，WebSocket 遵循同源策略，只能与同源的服务器建立连接。如果需要与不同域的服务器通信，可以使用 CORS（跨域资源共享）来进行跨域访问控制。

# WebSocket 的性能如何优化？有哪些注意事项和最佳实践？

为了优化 WebSocket 的性能，可以考虑以下几个方面：

- 减少数据量：合理控制发送的数据量大小，避免不必要的数据传输。
- 心跳机制：通过定时发送心跳消息，保持连接的活跃状态，防止连接被关闭。
- 数据压缩：可以使用压缩算法对数据进行压缩，减少网络传输的数据量。
- 服务器端优化：合理配置服务器端的连接数和资源管理，以支持更多的并发连接

## WebSocket 的缺点：

兼容性：部分老旧的浏览器可能不支持 WebSocket，需要进行兼容处理。
服务器支持：服务器需要支持 WebSocket 协议和相关处理逻辑。




# 谈谈websocket的心跳机制和重连机制

## 心跳机制（Ping/Pong）
心跳机制用来确保WebSocket连接的活跃状态，并且帮助监测连接是否断开。这通常通过发送轻量级的消息，即“Ping”和“Pong”消息来实现。

- Ping消息：由客户端或服务器发送，主要用来检测对方是否还在连接中。

- Pong消息：作为对Ping的响应，由接收方回复，确认连接仍然有效。
这种机制不仅可以帮助维持连接不被网络设备（如防火墙和负载均衡器）在无数据传输时关闭，还可以确保双方都能及时知晓连接的状态。

## 重连机制
当WebSocket连接由于网络问题或其他原因意外断开时，重连机制会尝试重新建立连接，以恢复通信。

实现重连通常涉及以下几个步骤：

- 监测连接状态：客户端需要有机制能够侦测到WebSocket连接的关闭或错误。
- 触发重连：一旦检测到连接断开，客户端将自动或手动触发重连过程。
- 延迟和重试策略：通常在重连尝试之间设定延迟时间，并可能采用逐步增加的延迟（指数退避策略）来避免频繁重试带来的网络压力。
- 限制重连次数：为避免无限重连，设定最大重连次数或总重连时间。
- 这两个机制的实现需要在客户端和服务器端的支持，具体实现细节可能根据使用的编程语言或框架有所不同。在实际应用中，合理配置心跳间隔和重连策略对于维护一个高效、稳定的WebSocket服务非常关键。


```javascript

const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', function connection(ws) {
    console.log('Client connected');

    ws.on('message', function incoming(message) {
        console.log('received: %s', message);
    });

    ws.on('pong', () => {
        ws.isAlive = true;
    });

    // 心跳机制
    const interval = setInterval(() => {
        wss.clients.forEach((client) => {
            if (!client.isAlive) return client.terminate();

            client.isAlive = false;
            client.ping();
        });
    }, 30000); // 每30秒检测一次

    ws.on('close', () => {
        clearInterval(interval);
    });
});

```

```javascript
const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', function connection(ws) {
    console.log('Client connected');

    ws.on('message', function incoming(message) {
        console.log('received: %s', message);
    });

    ws.on('pong', () => {
        ws.isAlive = true;
    });

    // 心跳机制
    const interval = setInterval(() => {
        wss.clients.forEach((client) => {
            if (!client.isAlive) return client.terminate();

            client.isAlive = false;
            client.ping();
        });
    }, 30000); // 每30秒检测一次

    ws.on('close', () => {
        clearInterval(interval);
    });
});


```

# 当某一次没了心跳，可能是什么原因


- 网络延迟或不稳定：网络延迟或临时的网络中断可能导致心跳消息在发送或接收过程中延迟，从而被错误地认为连接已断开。

- 服务器负载过重：如果服务器在处理大量连接或数据时过载，可能无法及时处理或响应心跳消息。

- 客户端处理延迟：客户端可能因为处理大量的数据或由于客户端自身性能问题而未能及时发送或响应心跳消息。

- 防火墙或安全设备策略：有些网络安全设备可能会阻止或过滤掉持续的心跳信号，尤其是在它们看起来像是非必要的网络流量时。

- WebSocket连接已被关闭：如果WebSocket连接因为其他原因（如服务器关闭连接）已被关闭，但是客户端或服务器尚未完全意识到这一点，心跳消息也可能无法成功发送或接收。

# 如何定义已经断开连接 断开了以后如何处理

- 关闭事件（Close Event）：WebSocket API提供了一个close事件，当连接关闭时，无论是客户端还是服务器主动关闭，或因为任何错误导致关闭，都会触发此事件。

- 错误事件（Error Event）：当连接中发生错误，导致连接不可用时，会触发error事件。这通常意味着连接已经不可恢复，将很快关闭或已经关闭。

- 心跳超时：如果在预设的时间内没有收到对方的心跳响应（无论是Ping还是Pong），可以认为连接已断开。设置超时机制，如果在给定的时间内没有收到响应，则主动关闭连接。

# 重连还是失败如何处理

1. 用户通知
通知用户连接失败，并提供一些可能的解决方案，例如检查网络设置或稍后重试。这可以通过更新界面上的状态消息或显示一个警告框来完成。

2. 手动重连选项
提供一个手动重连的按钮或选项，让用户在检查并确认他们的网络设置无误后，可以尝试再次连接。这种用户驱动的重连策略可以减少服务器压力并给予用户更多控制。

3. 增加重连间隔
如果自动重连持续失败，可以考虑增加重连尝试之间的间隔时间。例如，从每1秒一次逐渐增加到每几分钟尝试一次。这种方法称为指数退避策略，可以有效减少对服务器的冲击。

4. 错误日志和监控
确保所有重连尝试和失败都被适当记录，包括错误消息和重连尝试的时间点。这可以帮助开发团队追踪问题的根源，并改进未来的错误处理和系统设计。

1. 考虑备用方案
如果WebSocket服务不可用，可以考虑暂时使用其他通信方式，例如轮询（Polling）或长轮询，直到WebSocket服务恢复正常。这确保了应用仍能获取必要的更新，虽然可能不如WebSocket实时
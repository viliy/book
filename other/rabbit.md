# Install

> 单节点
1. docker search rabbitmq:management
2. docker pull rabbitmq:management
3. docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management

> 集群

## 1. 创建Rabbitmq网络
```
docker network create rabbitmqnet
```
## 2. 创建节点

 创建三个节点

* RABBITMQ_ERLANG_COOKIE 使用相同的cookie值

```
# docker run -d \
--name=rabbitmq1 \
-p 5672:5672 \
-p 15672:15672 \
-e RABBITMQ_NODENAME=rabbitmq1 \
-e RABBITMQ_ERLANG_COOKIE='YZSDHWMFSMKEMBDHSGGZ' \
-h rabbitmq1 \
--net=rabbitmqnet \
rabbitmq:management

# docker run -d \
--name=rabbitmq3 \
-p 5673:5672 \
-p 15673:15672 \
-e RABBITMQ_NODENAME=rabbitmq2 \
-e RABBITMQ_ERLANG_COOKIE='YZSDHWMFSMKEMBDHSGGZ' \
-h rabbitmq1 \
--net=rabbitmqnet \
rabbitmq:management

# docker run -d \
--name=rabbitmq3 \
-p 5674:5672 \
-p 15674:15672 \
-e RABBITMQ_NODENAME=rabbitmq3 \
-e RABBITMQ_ERLANG_COOKIE='YZSDHWMFSMKEMBDHSGGZ' \
-h rabbitmq1 \
--net=rabbitmqnet \
rabbitmq:management

```

## 3. 加入节点1

```
### Disk Node
# docker exec rabbitmq2 bash -c \
"rabbitmqctl stop_app && \
rabbitmqctl reset && \
rabbitmqctl join_cluster rabbitmq1@rabbitmq1 && \
rabbitmqctl start_app"

### Ram Node
# docker exec rabbitmq3 bash -c \
"rabbitmqctl stop_app && \
rabbitmqctl reset && \
rabbitmqctl join_cluster --ram rabbitmq1@rabbitmq1 && \
rabbitmqctl start_app"

```

## 4. 退出集群

```
# docker exec rabbitmq3 bash -c \
"rabbitmqctl stop_app && \
rabbitmqctl reset && \
rabbitmqctl start_app"

```

# web管理

1. 浏览器输入： *127.0.0.1:5672*
2. 输入默认账号密码：guest   guest

# docker-compose & haproxy

> see: https://github.com/viliy/docker-compose/tree/master/RabbitMQ

# FAQ

## 1. 什么叫做消息队列

消息（Message）是指在应用间传送的数据。消息可以非常简单，比如只包含文本字符串，也可以更复杂，可能包含嵌入对象。
消息队列（Message Queue）是一种应用间的通信方式，消息发送后可以立即返回，由消息系统来确保消息的可靠传递。消息发布者只管把消息发布到 MQ 中而不用管谁来取，消息使用者只管从 MQ 中取消息而不管是谁发布的。这样发布者和使用者都不用知道对方的存在。

## 2. 使用场景

以常见的订单系统为例，用户点击【下单】按钮之后的业务逻辑可能包括：扣减库存、生成相应单据、发红包、发短信通知。在业务发展初期这些逻辑可能放在一起同步执行，随着业务的发展订单量增长，需要提升系统服务的性能，这时可以将一些不需要立即生效的操作拆分出来异步执行，比如发放红包、发短信通知等。这种场景下就可以用 MQ ，在下单的主流程（比如扣减库存、生成相应单据）完成之后发送一条消息到 MQ 让主流程快速完结，而由另外的单独线程拉取MQ的消息（或者由 MQ 推送消息），当发现 MQ 中有发红包或发短信之类的消息时，执行相应的业务逻辑。
以上是用于业务解耦的情况，其它常见场景包括最终一致性、广播、错峰流控等等

## 3. RabbitMQ 特点

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。
AMQP ：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

> RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：
1. 可靠性（Reliability）
RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。
2. 灵活的路由（Flexible Routing）
在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。
3. 消息集群（Clustering）
多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。
高可用（Highly Available Queues）
队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
4. 多种协议（Multi-protocol）
RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。
5. 多语言客户端（Many Clients）
RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。
6. 管理界面（Management UI）
RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。
7. 跟踪机制（Tracing）
如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
8. 插件机制（Plugin System）
RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

## 4. RabbitMQ 概念模型

1. 简单理解
消费者（consumer）订阅某个队列。生产者（producer）创建消息，然后发布到队列（queue）中，最后将消息发送到监听的消费者。
![IMAGE](https://www.zhaqq.top/uploads/image/editormd/2019/04/03/ZSTCfZ90XRCQ6C8D0v0Y8k0MDcosjM.jpg)

2. 深入理解
![IMAGE](https://www.zhaqq.top/uploads/image/editormd/2019/04/03/4wM81Sjo38MigYNrV71r7ZXfEM3GsY.jpg)

> 模块

1.  Message
消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

2. Publisher
消息的生产者，也是一个向交换器发布消息的客户端应用程序。

3. Exchange
交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

4. Binding
绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

5. Queue
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

6. Connection
网络连接，比如一个TCP连接。

7. Channel
信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

8. Consumer
消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

9. Virtual Host
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

10. Broker
表示消息队列服务器实体。